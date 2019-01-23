---
layout:     post
title:      Android Init Code Analysis
subtitle:   安卓init代码分析！
date:       2019-01-23
author:     Earl
header-img: img/post-bg-new-3.jpg
catalog: true
tags:
    - Android
---

> 安卓init代码分析，或有不全，后续再更。

# Android启动代码init.c分析

Android启动代码init.c入口代码位于system/core/init.c文件中，从main函数入手。

	int main(int argc, char **argv)
	{
	    int fd_count = 0;
	    struct pollfd ufds[4];
	    char *tmpdev;
	    char* debuggable;
	    char tmp[32];
	    int property_set_fd_init = 0;
	    int signal_fd_init = 0;
	    int keychord_fd_init = 0;
	    bool is_charger = false;

	    NOTICE("The android init process begin.\n");

	    if (!strcmp(basename(argv[0]), "ueventd"))
		return ueventd_main(argc, argv);

	    if (!strcmp(basename(argv[0]), "watchdogd"))
		return watchdogd_main(argc, argv);

	    NOTICE("Clear umask info.\n");

	    /* clear the umask */
	    umask(0);

		/* Get the basic filesystem setup we need put
		 * together in the initramdisk on / and then we'll
		 * let the rc file figure out the rest.
		 */
	    mkdir("/dev", 0755);
	    mkdir("/proc", 0755);
	    mkdir("/sys", 0755);

	    mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
	    mkdir("/dev/pts", 0755);
	    mkdir("/dev/socket", 0755);
	    mount("devpts", "/dev/pts", "devpts", 0, NULL);
	    mount("proc", "/proc", "proc", 0, NULL);
	    mount("sysfs", "/sys", "sysfs", 0, NULL);

		/* indicate that booting is in progress to background fw loaders, etc */
	    close(open("/dev/.booting", O_WRONLY | O_CREAT, 0000));

		/* We must have some place other than / to create the
		 * device nodes for kmsg and null, otherwise we won't
		 * be able to remount / read-only later on.
		 * Now that tmpfs is mounted on /dev, we can actually
		 * talk to the outside world.
		 */
	    open_devnull_stdio();
	    klog_init();
	    property_init();

	    NOTICE("Get hardware info.\n");

	    get_hardware_name(hardware, &revision);

	    process_kernel_cmdline();

	    NOTICE("SElinux...\n");

	    union selinux_callback cb;
	    cb.func_log = log_callback;
	    selinux_set_callback(SELINUX_CB_LOG, cb);

	    cb.func_audit = audit_callback;
	    selinux_set_callback(SELINUX_CB_AUDIT, cb);

	    selinux_initialize();
	    /* These directories were necessarily created before initial policy load
	     * and therefore need their security context restored to the proper value.
	     * This must happen before /dev is populated by ueventd.
	     */
	    restorecon("/dev");
	    restorecon("/dev/socket");
	    restorecon("/dev/__properties__");
	    restorecon_recursive("/sys");

	    is_charger = !strcmp(bootmode, "charger");

	//    INFO("property init\n");
	    NOTICE("property init\n");
	    property_load_boot_defaults();

	    NOTICE("reading config file\n");
	    NOTICE("Analysis init.rc file.\n");
	    init_parse_config_file("/init.rc");

	    action_for_each_trigger("early-init", action_add_queue_tail);

	    queue_builtin_action(wait_for_coldboot_done_action, "wait_for_coldboot_done");
	    queue_builtin_action(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");
	    queue_builtin_action(keychord_init_action, "keychord_init");
	    queue_builtin_action(console_init_action, "console_init");

	    /* execute all the boot actions to get us started */
	    action_for_each_trigger("init", action_add_queue_tail);

	    /* Repeat mix_hwrng_into_linux_rng in case /dev/hw_random or /dev/random
	     * wasn't ready immediately after wait_for_coldboot_done
	     */
	    queue_builtin_action(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");
	    queue_builtin_action(property_service_init_action, "property_service_init");
	    queue_builtin_action(signal_init_action, "signal_init");

	    /* Don't mount filesystems or start core system services if in charger mode. */
	    if (is_charger) {
		action_for_each_trigger("charger", action_add_queue_tail);
	    } else {
		action_for_each_trigger("late-init", action_add_queue_tail);
	    }

	    /* run all property triggers based on current state of the properties */
	    queue_builtin_action(queue_property_triggers_action, "queue_property_triggers");


	#if BOOTCHART
	    queue_builtin_action(bootchart_init_action, "bootchart_init");
	#endif

	    for(;;) {
		int nr, i, timeout = -1;

		execute_one_command();
		restart_processes();

		if (!property_set_fd_init && get_property_set_fd() > 0) {
		    ufds[fd_count].fd = get_property_set_fd();
		    ufds[fd_count].events = POLLIN;
		    ufds[fd_count].revents = 0;
		    fd_count++;
		    property_set_fd_init = 1;
		}
		if (!signal_fd_init && get_signal_fd() > 0) {
		    ufds[fd_count].fd = get_signal_fd();
		    ufds[fd_count].events = POLLIN;
		    ufds[fd_count].revents = 0;
		    fd_count++;
		    signal_fd_init = 1;
		}
		if (!keychord_fd_init && get_keychord_fd() > 0) {
		    ufds[fd_count].fd = get_keychord_fd();
		    ufds[fd_count].events = POLLIN;
		    ufds[fd_count].revents = 0;
		    fd_count++;
		    keychord_fd_init = 1;
		}

		if (process_needs_restart) {
		    timeout = (process_needs_restart - gettime()) * 1000;
		    if (timeout < 0)
			timeout = 0;
		}

		if (!action_queue_empty() || cur_action)
		    timeout = 0;

	#if BOOTCHART
		if (bootchart_count > 0) {
		    if (timeout < 0 || timeout > BOOTCHART_POLLING_MS)
			timeout = BOOTCHART_POLLING_MS;
		    if (bootchart_step() < 0 || --bootchart_count == 0) {
			bootchart_finish();
			bootchart_count = 0;
		    }
		}
	#endif

		nr = poll(ufds, fd_count, timeout);
		if (nr <= 0)
		    continue;

		for (i = 0; i < fd_count; i++) {
		    if (ufds[i].revents & POLLIN) {
			if (ufds[i].fd == get_property_set_fd())
			    handle_property_set_fd();
			else if (ufds[i].fd == get_keychord_fd())
			    handle_keychord();
			else if (ufds[i].fd == get_signal_fd())
			    handle_signal();
		    }
		}
	    }

	    return 0;
	    }

## ueventd和watchdogd
	   
main函数开始定义了一些变量，紧接着是两个if判断：

	    if (!strcmp(basename(argv[0]), "ueventd"))
		return ueventd_main(argc, argv);

	    if (!strcmp(basename(argv[0]), "watchdogd"))
		return watchdogd_main(argc, argv);
		
libc库函数basename()函数会将argv[0]中的字符串去除前面的斜杠和前缀，获得"ueventd"或者"watchdogd"，如果等于任何一个就会转去执行相应的main函数，为什么会有这两个main函数入口？从system/core/init/Android.mk可以看出端倪：

	# Make a symlink from /sbin/ueventd and /sbin/watchdogd to /init
	SYMLINKS := \
		$(TARGET_ROOT_OUT)/sbin/ueventd \
		$(TARGET_ROOT_OUT)/sbin/watchdogd

是的，ueventd和watchdogd是init的两个软连接，当执行这两个进程时，执行的还是init进程，init会根据命令行参数决定是否进入ueventd_main或者watchdogd_main。

## 清理umask

	    /* clear the umask */
	    umask(0);
	    
umask()函数会将系统umask值设成参数mask&0777后的值，然后将先前的umask值返回。

		#if defined(__BIONIC_FORTIFY)

		__BIONIC_FORTIFY_INLINE
		mode_t umask(mode_t mode) {
		#if !defined(__clang__)
		  if (__builtin_constant_p(mode)) {
		    if ((mode & 0777) != mode) {
		      __umask_invalid_mode();
		    }
		    return __umask_real(mode);
		  }
		#endif
		  return __umask_chk(mode);
		}
		#endif /* defined(__BIONIC_FORTIFY) */

## 创建目录和挂载文件系统

		/* Get the basic filesystem setup we need put
		 * together in the initramdisk on / and then we'll
		 * let the rc file figure out the rest.
		 */
	    mkdir("/dev", 0755);
	    mkdir("/proc", 0755);
	    mkdir("/sys", 0755);

	    mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
	    mkdir("/dev/pts", 0755);
	    mkdir("/dev/socket", 0755);
	    mount("devpts", "/dev/pts", "devpts", 0, NULL);
	    mount("proc", "/proc", "proc", 0, NULL);
	    mount("sysfs", "/sys", "sysfs", 0, NULL);

		/* indicate that booting is in progress to background fw loaders, etc */
	    close(open("/dev/.booting", O_WRONLY | O_CREAT, 0000));
	    
把虚拟文件系统tmpfs、devpts、proc、sysfs分别挂载/dev、/dev/pts、/proc、/sys目录下。通过创建文件"/dev/.booting"来表示目前正处于启动中的状态， /dev是内存文件系统，不会保存的，每次开机都要重新创建。这个是指示，目前在booting过程中，具体干什么用的，介绍ueventd的时候就清楚了。就是加载设备的fimware用的。

## 创建

除了根目录(/)，我们必须由一些地方来创建kmsg和null的设备节点，否则我们稍后无法重新挂载根目录为只读。由于tmpfs文件系统已经挂载到/dev分区，我们可以和外部交流。

		/* We must have some place other than / to create the
		 * device nodes for kmsg and null, otherwise we won't
		 * be able to remount / read-only later on.
		 * Now that tmpfs is mounted on /dev, we can actually
		 * talk to the outside world.
		 */
	    open_devnull_stdio();
	    klog_init();  //创建/dev/__kmsg__节点
	    property_init();
	      
接下来遇到函数open_devnull_stdio()，  位于system/core/init/util.c文件，这个函数做了什么呢？首先创建了一个设备节点"/dev/__null__"，然后打开这个“设备文件”，并将对应的文件描述符保存在fd中。然后使用unlink(name)删除这个文件，确实是把文件删除了，但是现在系统里借助这个fd还是能够找到文件的内容的。俗话说“皮之不存毛将焉附”，可是这里生生的就是皮不在了，但是毛仍然存在。好了，下面使用dup2函数将文件描述符为0,1,2的信息重定向到以这个fd为文件描述符的文件中。在linux中文件描述符为0,1,2的是什么东西？标准输入，标准输出，标准错误输出。在进行到这里之后就关闭掉原来的fd。好吧把这个函数重新看一遍，它完成了什么功能？将标准输入，标准输出，标准出错输出重定向到了一个设备文件中。

	void open_devnull_stdio(void)
	{
	    int fd;
	    static const char *name = "/dev/__null__";
	    if (mknod(name, S_IFCHR | 0600, (1 << 8) | 3) == 0) {
		fd = open(name, O_RDWR);
		unlink(name); 
		if (fd >= 0) {
		    dup2(fd, 0); //重定向标准输入到/dev/__null__
		    dup2(fd, 1); //重定向标准输出到/dev/__null__
		    dup2(fd, 2); //重定向错误输出到/dev/__null__
		    if (fd > 2) {
		        close(fd);
		    }
		    return;
		}
	    }

	    exit(1);
	}

这个文件看完了，回到main函数中。接下来是调用函数klog_init()，位于system/core/libcutils中，创建设备结点：/dev/__kmsg__。klog_init()函数是将init进程中的log，打印到内核printk的log buffer中的方法。这对调试init很有帮助，毕竟此时没有shell，通用的log输出方法，如printf等还不能工作，借助底层已有的内核调试功能当然是最好的了。fcntl(klog_fd, F_SETFD, FD_CLOEXEC)表示当在子进程中使用exec执行其他程序的时候会把这个文件描述符关闭。
	
	void klog_init(void)
	{ 
	    static const char *name = "/dev/__kmsg__";

	    if (klog_fd >= 0) return; /* Already initialized */
	    
	    if (mknod(name, S_IFCHR | 0600, (1 << 8) | 11) == 0) {
		klog_fd = open(name, O_WRONLY);
		if (klog_fd < 0)
		        return;
		fcntl(klog_fd, F_SETFD, FD_CLOEXEC);
		unlink(name);
	    }
	}
	
到main函数中，接下来的函数是property_init()。从名字上可以看出来这里是为了初始化属性相关的东西。属性？android中的属性是什么？andorid中的属性很多，如果看源代码的话你能发现andorid属性是一个相当有趣的东西。慢慢的会给你展开这幅画卷。property_init()函数位于system/core/init/property_service.c文件中。

	void property_init(void)
	{   
	    init_property_area();
	}

	static int init_property_area(void)
	{   
	    if (property_area_inited)
		return -1;

	    if(__system_property_area_init())
		return -1;
	    
	    if(init_workspace(&pa_workspace, 0))
		return -1;

	    fcntl(pa_workspace.fd, F_SETFD, FD_CLOEXEC);
	    
	    property_area_inited = 1;
	    return 0;
	}
	
	static int init_workspace(workspace *w, size_t size)
	{
	    void *data;
	    int fd = open(PROP_FILENAME, O_RDONLY | O_NOFOLLOW);
	    if (fd < 0)
		return -1; 

	    w->size = size;
	    w->fd = fd; 
	    return 0;
	}

	typedef struct {
	    size_t size;
	    int fd; 
	} workspace;

回到main函数中。继续执行函数get_hardware_name(hardware,&revision)。这个函数位于system/core/init/util.c文件中。

    get_hardware_name(hardware, &revision);
    
 这个函数的实参hardware是一32个元素的字符数组，revision为一个unsigned值。这个函数从"/proc/cpuinfo"文件读取相应字符串到data中，然后调用 strstr函数将data中"\nHardware"开始的字符保存到hw中，将"\nHardware"开始的字符保存到rev中。这里的strstr就是查找第二个参数在第一个参数第一次出现的地址，将地址赋值给一个字符指针后，这个字符指针就能找到从这个地址开始往后的字符。
    
	void get_hardware_name(char *hardware, unsigned int *revision)
	{
	    const char *cpuinfo = "/proc/cpuinfo";
	    char *data = NULL;
	    size_t len = 0, limit = 1024;
	    int fd, n;
	    char *x, *hw, *rev;

	    /* Hardware string was provided on kernel command line */
	    if (hardware[0])
		return;

	    fd = open(cpuinfo, O_RDONLY);
	    if (fd < 0) return;

	    for (;;) {
		x = realloc(data, limit);
		if (!x) {
		    ERROR("Failed to allocate memory to read %s\n", cpuinfo);
		    goto done;
		}
		data = x;

		n = read(fd, data + len, limit - len);
		if (n < 0) {
		    ERROR("Failed reading %s: %s (%d)\n", cpuinfo, strerror(errno), errno);
		    goto done;
		}
		len += n;

		if (len < limit)
		    break;

		/* We filled the buffer, so increase size and loop to read more */
		limit *= 2;
	    }

	    data[len] = 0;
	    hw = strstr(data, "\nHardware");
	    rev = strstr(data, "\nRevision");

	    if (hw) {
		x = strstr(hw, ": ");
		if (x) {
		    x += 2;
		    n = 0;
		    while (*x && *x != '\n') {
		        if (!isspace(*x))
		            hardware[n++] = tolower(*x);
		        x++;
		        if (n == 31) break;
		    }
		    hardware[n] = 0;
		}
	    }

	    if (rev) {
		x = strstr(rev, ": ");
		if (x) {
		    *revision = strtoul(x + 2, 0, 16);
		}
	    }

	done:
	    close(fd);
	    free(data);
	}
	
## process_kernel_cmdline()

    process_kernel_cmdline();

process_kernel_cmdline解析内核启动参数。内核允许bootloader启动自己时传递参数。启动参数首先在\device\loongson\loongson2k\BoardConfig.mk下定义，

	static void process_kernel_cmdline(void)
	{
	    /* don't expose the raw commandline to nonpriv processes */
	    chmod("/proc/cmdline", 0440);
	    
	    /* first pass does the common stuff, and finds if we are in qemu.
	     * second pass is only necessary for qemu to export all kernel params
	     * as props.
	     */
	    import_kernel_cmdline(0, import_kernel_nv);
	    if (qemu[0])
		import_kernel_cmdline(1, import_kernel_nv);

	    /* now propogate the info given on command line to internal variables
	     * used by init as well as the current required properties
	     */
	    export_kernel_boot_props();
	}
	
首先修改/proc/cmdline文件权限，0440即表明只有root用户或root组用户可以读写该文件，其他用户无法访问。

随后连续调用import_kernel_cmdline函数，第一个参数标识当前Android设备是否是模拟器(1为模拟器)，第二个参数一个函数指针。

import_kernel_cmdline函数将/proc/cmdline内容读入到内部缓冲区中，并将cmdline内容的以空格拆分成小段字符串，依次传递给import_kernel_nv函数处理。
	
	void import_kernel_cmdline(int in_qemu,
                           void (*import_kernel_nv)(char *name, int in_qemu))
	{   
	    char cmdline[2048];
	    char *ptr;
	    int fd;

	    fd = open("/proc/cmdline", O_RDONLY);
	    if (fd >= 0) {
		int n = read(fd, cmdline, sizeof(cmdline) - 1);
		if (n < 0) n = 0;
		
		/* get rid of trailing newline, it happens */
		if (n > 0 && cmdline[n-1] == '\n') n--;

		cmdline[n] = 0;
		close(fd);
	    } else {
		cmdline[0] = 0;
	    }

	    ptr = cmdline;
	    while (ptr && *ptr) {
		char *x = strchr(ptr, ' ');
		if (x != 0) *x++ = 0;
		import_kernel_nv(ptr, in_qemu);
		ptr = x;
	    }
	}

	static void import_kernel_nv(char *name, int for_emulator)
	{
	    char *value = strchr(name, '=');
	    int name_len = strlen(name);

	    if (value == 0) return;
	    *value++ = 0;
	    if (name_len == 0) return; 

	    if (for_emulator) {
		/* in the emulator, export any kernel option with the
		 * ro.kernel. prefix */
		char buff[PROP_NAME_MAX];
		int len = snprintf( buff, sizeof(buff), "ro.kernel.%s", name );

		if (len < (int)sizeof(buff))
		    property_set( buff, value );
		return;
	    }
	    
	    if (!strcmp(name,"qemu")) {
		strlcpy(qemu, value, sizeof(qemu));
	    } else if (!strncmp(name, "androidboot.", 12) && name_len > 12) {
		const char *boot_prop_name = name + 12;
		char prop[PROP_NAME_MAX];
		int cnt;

		cnt = snprintf(prop, sizeof(prop), "ro.boot.%s", boot_prop_name);
		if (cnt < PROP_NAME_MAX)
		    property_set(prop, value);
	    }
	}
	
	static void export_kernel_boot_props(void)
	{   
	    char tmp[PROP_VALUE_MAX];
	    int ret;
	    unsigned i;
	    struct {
		const char *src_prop;
		const char *dest_prop;
		const char *def_val;
	    } prop_map[] = {
		{ "ro.boot.serialno", "ro.serialno", "", },
		{ "ro.boot.mode", "ro.bootmode", "unknown", },
		{ "ro.boot.baseband", "ro.baseband", "unknown", },
		{ "ro.boot.bootloader", "ro.bootloader", "unknown", },
	    };

	    for (i = 0; i < ARRAY_SIZE(prop_map); i++) {
		ret = property_get(prop_map[i].src_prop, tmp);
		if (ret > 0)
		    property_set(prop_map[i].dest_prop, tmp);
		else
		    property_set(prop_map[i].dest_prop, prop_map[i].def_val);
	    }
	    
	    ret = property_get("ro.boot.console", tmp);
	    if (ret)
		strlcpy(console, tmp, sizeof(console));

	    /* save a copy for init's usage during boot */
	    property_get("ro.bootmode", tmp);
	    strlcpy(bootmode, tmp, sizeof(bootmode));

	    /* if this was given on kernel command line, override what we read
	     * before (e.g. from /proc/cpuinfo), if anything */
	    ret = property_get("ro.boot.hardware", tmp);
	    if (ret)
		strlcpy(hardware, tmp, sizeof(hardware));
	    property_set("ro.hardware", hardware);

	    snprintf(tmp, PROP_VALUE_MAX, "%d", revision);
	    property_set("ro.revision", tmp);

	    /* TODO: these are obsolete. We should delete them */
	    if (!strcmp(bootmode,"factory"))
		property_set("ro.factorytest", "1");
	    else if (!strcmp(bootmode,"factory2"))
		property_set("ro.factorytest", "2");
	    else
		property_set("ro.factorytest", "0");
	}

在内核启动完毕之后，启动参数可通过/proc/cmdline查看。

## SELinux

	    union selinux_callback cb;
	    cb.func_log = log_callback;
	    selinux_set_callback(SELINUX_CB_LOG, cb);

	    cb.func_audit = audit_callback;
	    selinux_set_callback(SELINUX_CB_AUDIT, cb);

	    selinux_initialize();
	    /* These directories were necessarily created before initial policy load
	     * and therefore need their security context restored to the proper value.
	     * This must happen before /dev is populated by ueventd.
	     */
	    restorecon("/dev");
	    restorecon("/dev/socket");
	    restorecon("/dev/__properties__");
	    restorecon_recursive("/sys");
	    
如果启用了SELinux机制，接下来将加载selinux策略，并初始化文件安全上下文以及属性安全上下文。

## 充电模式

	    is_charger = !strcmp(bootmode, "charger");
	    
这里首先判断机器是否是以字符模式启动，将结果保存到变量is_charger中。如果不是字符模式启动则执行函数property_load_boot_defaults()。这个函数是和属性相关的操作。函数位于system/core/init/property_service.c文件中。


## 加载系统属性

	    NOTICE("property init\n");
	    property_load_boot_defaults();
	    
	void property_load_boot_defaults(void)
	{   
	    load_properties_from_file(PROP_PATH_RAMDISK_DEFAULT, NULL);
	}

  在这个函数中，会读取PROP_PATH_RAMDISK_DEFAULT文件中的内容，这个文件中存储的是系统的一些默认的属性值。最后调用load_properties函数加载这些属性。注意到，我们这里传送过去的是data，也就是我们刚才读出来的数据。看一下load_properties这个函数。
	
	/*
	 * Filter is used to decide which properties to load: NULL loads all keys,
	 * "ro.foo.*" is a prefix match, and "ro.foo.bar" is an exact match.
	 */
	static void load_properties_from_file(const char *fn, const char *filter)
	{   
	    char *data;
	    unsigned sz;
	    
	    data = read_file(fn, &sz);

	    if(data != 0) {
		load_properties(data, filter);
		free(data);
	    }
	}

  这个函数主要完成对传入的字符串进行解析，然后调用property_set函数。

	/*
	 * Filter is used to decide which properties to load: NULL loads all keys,
	 * "ro.foo.*" is a prefix match, and "ro.foo.bar" is an exact match.
	 */
	static void load_properties(char *data, const char *filter)
	{
	    char *key, *value, *eol, *sol, *tmp, *fn;
	    size_t flen = 0;

	    if (filter) {
		flen = strlen(filter);
	    }

	    sol = data; 
	    while ((eol = strchr(sol, '\n'))) {
		key = sol;
		*eol++ = 0;
		sol = eol;
	
		while (isspace(*key)) key++;
		if (*key == '#') continue;
	
		tmp = eol - 2;
		while ((tmp > key) && isspace(*tmp)) *tmp-- = 0;
	    
		if (!strncmp(key, "import ", 7) && flen == 0) {
		    fn = key + 7;
		    while (isspace(*fn)) fn++;
		    
		    key = strchr(fn, ' ');
		    if (key) { 
			*key++ = 0;
			while (isspace(*key)) key++;
		    }

		    load_properties_from_file(fn, key);

		} else {
		    value = strchr(key, '=');
		    if (!value) continue;
		    *value++ = 0;

		    tmp = value - 2;
		    while ((tmp > key) && isspace(*tmp)) *tmp-- = 0;

		    while (isspace(*value)) value++;

		    if (flen > 0) {
			if (filter[flen - 1] == '*') {
			    if (strncmp(key, filter, flen - 1)) continue;
			} else {
			    if (strcmp(key, filter)) continue;
			}
		    }

		    property_set(key, value);
		}
	    }
	}


下面分别使用strlen函数获得传过来的那两个参数的长度并保存在namelen和valuelen中。再下面就是一些判断逻辑，namelen不能大于PROP_NAME_MAX，valuelen 不能大于PROP_VALUE_MAX 。

	int property_set(const char *name, const char *value)
	{
	    prop_info *pi;
	    int ret;

	    size_t namelen = strlen(name);
	    size_t valuelen = strlen(value);

	    if (!is_legal_property_name(name, namelen)) return -1;
	    if (valuelen >= PROP_VALUE_MAX) return -1;
	    
	    pi = (prop_info*) __system_property_find(name);

	    if(pi != 0) {
		/* ro.* properties may NEVER be modified once set */
		if(!strncmp(name, "ro.", 3)) return -1;
		
		__system_property_update(pi, value, valuelen);
	    } else {
		ret = __system_property_add(name, namelen, value, valuelen);
		if (ret < 0) {
		    ERROR("Failed to set '%s'='%s'\n", name, value);
		    return ret;
		}
	    }
	    /* If name starts with "net." treat as a DNS property. */
	    if (strncmp("net.", name, strlen("net.")) == 0)  {
		if (strcmp("net.change", name) == 0) {
		    return 0;
		}
	       /*
		* The 'net.change' property is a special property used track when any
		* 'net.*' property name is updated. It is _ONLY_ updated here. Its value
		* contains the last updated 'net.*' property.
		*/
		property_set("net.change", name);
	    } else if (persistent_properties_loaded &&
		    strncmp("persist.", name, strlen("persist.")) == 0) {
		/*
		 * Don't write properties to disk until after we have read all default properties
		 * to prevent them from being overwritten by default values.
		 */
		write_persistent_property(name, value);
	    } else if (strcmp("selinux.reload_policy", name) == 0 &&
		       strcmp("1", value) == 0) {
		selinux_reload_policy();
	    }
	    property_changed(name, value);
	    return 0;
	}

	void property_changed(const char *name, const char *value)
	{   
	    if (property_triggers_enabled)
		queue_property_triggers(name, value);
	} 
	
## 解析init.rc文件

	 init_parse_config_file("/init.rc");

首先使用read_file函数读出"/init.rc"中的内容，保存到data中。然后调用parse_config(fn,data)进行真正的解析。
	 
init_parse_config_file位于system/core/init/init_parser.c文件中。

	int init_parse_config_file(const char *fn)
	{
	    char *data;
	    data = read_file(fn, 0);
	    if (!data) return -1;
	    
	    parse_config(fn, data);
	    DUMP();
	    return 0;
	} 
	
对init.rc文件的解析另开专题讲解，这里知道函数的用途就行。

## 把指定的action端插入到action的尾部

	    action_for_each_trigger("early-init", action_add_queue_tail);

	    queue_builtin_action(wait_for_coldboot_done_action, "wait_for_coldboot_done");
	    queue_builtin_action(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");
	    queue_builtin_action(keychord_init_action, "keychord_init");
	    queue_builtin_action(console_init_action, "console_init");

	    /* execute all the boot actions to get us started */
	    action_for_each_trigger("init", action_add_queue_tail);

	    /* Repeat mix_hwrng_into_linux_rng in case /dev/hw_random or /dev/random
	     * wasn't ready immediately after wait_for_coldboot_done
	     */
	    queue_builtin_action(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");
	    queue_builtin_action(property_service_init_action, "property_service_init");
	    queue_builtin_action(signal_init_action, "signal_init");

	    /* Don't mount filesystems or start core system services if in charger mode. */
	    if (is_charger) {
		action_for_each_trigger("charger", action_add_queue_tail);
	    } else {
		action_for_each_trigger("late-init", action_add_queue_tail);
	    }

	    /* run all property triggers based on current state of the properties */
	    queue_builtin_action(queue_property_triggers_action, "queue_property_triggers");


	#if BOOTCHART
	    queue_builtin_action(bootchart_init_action, "bootchart_init");
	#endif
	
这部分代码其实主要就是两个函数action_for_each_trigger和queue_builtin_action函数。我们首先看一下action_for_each_trigger函数，这个函数位于system/core/init/init_parser.c文件中。我们以第一次使用它的那段代码为例分析一下。

	action_for_each_trigger("early-init", action_add_queue_tail);
	void action_for_each_trigger(const char *trigger,
		                     void (*func)(struct action *act))
	{   
	    struct listnode *node;
	    struct action *act;
	    list_for_each(node, &action_list) {
		act = node_to_item(node, struct action, alist);
		if (!strcmp(act->name, trigger)) {
		    func(act);
		}
	    }
	}
	
这个函数逻辑很简单，首先在action_list中寻找那个和early-init同名的那个action结构体，如果能够找到的话就执行传入的第二个参数名的函数。

	void action_add_queue_tail(struct action *act)
	{
	    list_add_tail(&action_queue, &act->qlist);
	}

现在我们看看另一个函数queue_builtin_action，我们仍然是以第一次使用它的地方为例。

	queue_builtin_action(wait_for_coldboot_done_action,"wait_for_coldboot_done");
	void queue_builtin_action(int (*func)(int nargs, char **args), char *name)
	{
	    struct action *act;
	    struct command *cmd;
	    
	    act = calloc(1, sizeof(*act));
	    act->name = name;
	    list_init(&act->commands);
	    list_init(&act->qlist);
		
	    cmd = calloc(1, sizeof(*cmd));
	    cmd->func = func;
	    cmd->args[0] = name;
	    cmd->nargs = 1;
	    list_add_tail(&act->commands, &cmd->clist);
	    
	    list_add_tail(&action_list, &act->alist);
	    action_add_queue_tail(act);
	} 
	
  在这个函数函数中构造了一个action结构体和一个command结构体，并且将command结构体链入action结构体，将action结构体链入action_list，最后调用action_add_queue_tail函数，即将这个action链入全局链表action_queue。

## for循环

	    for(;;) {
		int nr, i, timeout = -1;

		execute_one_command();
		restart_processes();

		if (!property_set_fd_init && get_property_set_fd() > 0) {
		    ufds[fd_count].fd = get_property_set_fd();
		    ufds[fd_count].events = POLLIN;
		    ufds[fd_count].revents = 0;
		    fd_count++;
		    property_set_fd_init = 1;
		}
		if (!signal_fd_init && get_signal_fd() > 0) {
		    ufds[fd_count].fd = get_signal_fd();
		    ufds[fd_count].events = POLLIN;
		    ufds[fd_count].revents = 0;
		    fd_count++;
		    signal_fd_init = 1;
		}
		if (!keychord_fd_init && get_keychord_fd() > 0) {
		    ufds[fd_count].fd = get_keychord_fd();
		    ufds[fd_count].events = POLLIN;
		    ufds[fd_count].revents = 0;
		    fd_count++;
		    keychord_fd_init = 1;
		}

		if (process_needs_restart) {
		    timeout = (process_needs_restart - gettime()) * 1000;
		    if (timeout < 0)
		        timeout = 0;
		}

		if (!action_queue_empty() || cur_action)
		    timeout = 0;

	#if BOOTCHART
		if (bootchart_count > 0) {
		    if (timeout < 0 || timeout > BOOTCHART_POLLING_MS)
		        timeout = BOOTCHART_POLLING_MS;
		    if (bootchart_step() < 0 || --bootchart_count == 0) {
		        bootchart_finish();
		        bootchart_count = 0;
		    }
		}
	#endif

		nr = poll(ufds, fd_count, timeout);
		if (nr <= 0)
		    continue;

		for (i = 0; i < fd_count; i++) {
		    if (ufds[i].revents & POLLIN) {
		        if (ufds[i].fd == get_property_set_fd())
		            handle_property_set_fd();
		        else if (ufds[i].fd == get_keychord_fd())
		            handle_keychord();
		        else if (ufds[i].fd == get_signal_fd())
		            handle_signal();
		    }
		}
	    }

	    return 0;
	}
	
首先调用execute_one_command函数。

	void execute_one_command(void)
	{   
	    int ret, i;
	    char cmd_str[256] = "";
	    
	    if (!cur_action || !cur_command || is_last_command(cur_action, cur_command)) {
		cur_action = action_remove_queue_head();
		cur_command = NULL;
		if (!cur_action)
		    return;
		INFO("processing action %p (%s)\n", cur_action, cur_action->name);
		cur_command = get_first_command(cur_action);
	    } else {
		cur_command = get_next_command(cur_action, cur_command);
	    }   

	    if (!cur_command)
		return;

	    ret = cur_command->func(cur_command->nargs, cur_command->args);
	    if (klog_get_level() >= KLOG_INFO_LEVEL) {
		for (i = 0; i < cur_command->nargs; i++) {
		    strlcat(cmd_str, cur_command->args[i], sizeof(cmd_str));
		    if (i < cur_command->nargs - 1) {
		        strlcat(cmd_str, " ", sizeof(cmd_str));
		    }
		}
		INFO("command '%s' action=%s status=%d (%s:%d)\n",
		     cmd_str, cur_action ? cur_action->name : "", ret, cur_command->filename,
		     cur_command->line);
	    }
	}
	
涉及两个变量：

	static struct action *cur_action = NULL;
	static struct command *cur_command = NULL;
	
 由于是第一次调用这个函数，这里的cur_action，cur_command现在仍未NULL，所以第一个if里面的代码可以执行。里面首先调用了action_remove_queue_head函数。
 
	struct action *action_remove_queue_head(void)
	{
	    if (list_empty(&action_queue)) {
		return 0;
	    } else {
		struct listnode *node = list_head(&action_queue);
		struct action *act = node_to_item(node, struct action, qlist);
		list_remove(node);
		list_init(node);
		return act;
	    }
	}
	
  在这个函数中首先获得第一个action结构体，然后将这个结构体对应的节点从action_queue里面删除，最后返回这个action结构体。
  回到execute_one_command函数中，调用get_first_command去获得一个command结构体指针。看一下他的实现函数。
  
  	static struct command *get_first_command(struct action *act)
	{
	    struct listnode *node;
	    node = list_head(&act->commands);
	    if (!node || list_empty(&act->commands))
		return NULL;

	    return node_to_item(node, struct command, clist);
	}
	
	 static struct command *get_next_command(struct action *act, struct command *cmd)
	{   
	    struct listnode *node;
	    node = cmd->clist.next;
	    if (!node)
		return NULL;
	    if (node == &act->commands)
		return NULL;
	    
	    return node_to_item(node, struct command, clist);
	} 
	
这个函数首先获得head节点，然后利用node_to_item将这个节点转化成command结构体指针。

再次回到execute_one_command函数中，如果能够正确的获得command结构体指针，便会执行cur_command->func函数。

这样我们知道main函数中调用execute_one_command函数的作用了。首先从action_queue链表中获得头部的action结构体，并将这个头部的action结构体从action_queue移除。如果我们是第一次使用这个action结构体的话，我们会调用get_first_command函数去获得链接在它上面的第一个command结构体，并执行这个结构体对应的函数；如果我们不是第一次使用这个action结构体的话，我们会调用get_next_command函数去获得链接在这个action结构体的下一个command结构体，并执行对应的函数。

在main函数中，我们现在是处于一个for循环中。所以，每当我们在for循环中调用execute_one_command的时候，便会获得一个command结构体，并执行其对应的函数。当我们获得action结构体的最后一个command结构体后，再次调用execute_one_command函数的时候，我们就会从action_queue中去获得新的action结构体，并重复获得command结构体，执行函数的步骤，直到最终我们把action_queue链表中的所有的action结构体上的所有的command都调用一遍。

 我们把思维拉回到main函数中，接下来有一个restart_processes函数。这个函数是做什么的？如果我们启动的service有死掉的话，这个函数便会检测到，并且调用相应的函数去重新打开service，即实现restart功能。

	static void restart_processes()
	{
	    process_needs_restart = 0;
	    service_for_each_flags(SVC_RESTARTING,
		                   restart_service_if_needed);
	}
	
	void service_for_each_flags(unsigned matchflags,
		                    void (*func)(struct service *svc))
	{
	    struct listnode *node;
	    struct service *svc;   
	    list_for_each(node, &service_list) {
		svc = node_to_item(node, struct service, slist);
		if (svc->flags & matchflags) {
		    func(svc);
		}
	    }
	}
	
  注意到我们传给service_for_each_flags函数的参数中unsigned matchflags是SVC_RESTARTING，在service_for_each_flags函数中，我们首先获得一个service结构体，然后让这个结构体的flags匹配我们传递过来的SVC_RESTARTING，如果我们这个结构体的flags恰好是SVC_RESTARTING的话，就执行我们作为参数传递过来的那个函数即restart_service_if_needed。
  
 我们回到main函数中。下面是关于poll函数的使用，poll在这里用于检测socket变化的函数。看代码，如果条件满足的话便会初始化struct pollfd ufds[4]中各个成员变量。其实这里我们仅仅操作了ufds数组中的三个成员。我们先看第一个成员，如果(!property_set_fd_init && get_property_set_fd() > 0)这个条件满足的话就会进行下面的初始化。

intproperty_set_fd_init = 0;

所以!property_set_fd_init这里是1，get_property_set_fd()函数是定义在system/core/init/property_service.c文件中的函数。

	int get_property_set_fd()
	{
	    return property_set_fd;
	}

他返回的是property_set_fd变量，这个变量是多少？

static int property_set_fd = -1;

这里的property_set_fd = -1是初始值，那么在什么时候这个值能够满足我们的判断条件呢。还记得前面前面执行过下面一行代码吗？

queue_builtin_action(property_service_init_action,"property_service_init");

这段代码的作用是创建一个action结构体和一个command结构体，将这个command结构体链入了action结构体，并将property_service_init_action赋值给了command结构体的func变量。这样在main函数的for循环下找到这个action结构体后，会依次找到action结构体下面的command结构体，并调用对应的func成员函数。最终会调用到我们的这个property_service_init_action函数。我们看看这个函数的实现代码。

	static int property_service_init_action(int nargs, char **args)
	{
	    /* read any property files on system or data and
	     * fire up the property service.  This must happen
	     * after the ro.foo properties are set above so
	     * that /data/local.prop cannot interfere with them.
	     */
	    start_property_service();
	    return 0;
	}
	void start_property_service(void)
	{
	    int fd;
	 
	    load_properties_from_file(PROP_PATH_SYSTEM_BUILD);
	    load_properties_from_file(PROP_PATH_SYSTEM_DEFAULT);
	    load_override_properties();
	    /* Read persistent properties after all default values have been loaded. */
	    load_persistent_properties();
	 
	    fd = create_socket(PROP_SERVICE_NAME, SOCK_STREAM, 0666, 0, 0);
	    if(fd < 0) return;
	    fcntl(fd, F_SETFD, FD_CLOEXEC);
	    fcntl(fd, F_SETFL, O_NONBLOCK);
	 
	    listen(fd, 8);
	    property_set_fd = fd;
	}
	
看到start_property_service函数的最后一行没？这里改变了property_set_fd的值！这样我们main函数下的if判断就能够通过了也就可以去初始化对应的成员变量。

 同理，当我们调用signal_init_action和keychord_init_action函数的时候，我们会让下面的那两个if判断通过，最终这三个都会设置完毕。我们仔细看一下这里对events成员赋值都是POLLIN，这样当后期有read events可操作时就会返回。

 在main函数中紧跟着这三个if后面仍然是两个if结构，这两个是关于timout的设置就不看了。最后调用poll函数。如果poll能检测到socket变化，就利用一个for循环去判断发生变化的是哪一个socket。并最终调用对应的函数去处理这些变化。
