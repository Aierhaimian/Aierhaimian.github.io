---
layout:     post
title:      Chisel Getting Started Guide
subtitle:   Chisel-A hardware construction language.
date:       2018-03-20
author:     Earl Du
header-img: img/post-bg-3.jpg
catalog: true
tags:
    - Language
---

> Chisel学习教程整理。

# What is Chisel? #

根据Chisel[官网](https://chisel.eecs.berkeley.edu/index.html)的介绍，Chisel是加州大学伯克利分校开发的开源硬件构建语言，它支持使用高度参数化的生成器和分层的特定领域硬件语言的高级硬件设计。列举其特性如下：

- Hardware construction language (not C to Gates)
- Embedded in the Scala programming language
- Algebraic construction and wiring
- Abstract data types and interfaces
- Bulk connections
- Hierarchical + object oriented + functional construction
- Highly parameterizable using metaprogramming in Scala
- Supports layering of domain specific languages
- Sizeable standard library including floating-point units
- Multiple clock domains
- Generates low-level Verilog designed to pass on to standard ASIC or FPGA tools
- Open source on github with modified BSD license
- Complete set of docs
- Growing community of adopters

关于Chisel的详细介绍还请参见Chisel[官方网站](https://chisel.eecs.berkeley.edu/index.html)。

# Getting Started #

**注：**



- 安装环境：Ubuntu16.04
- 参考github：[freechipsproject/chisel3](https://github.com/freechipsproject/chisel3/tree/f3a39aff35879f1959786b1178ec2ffe96164a35#arch-linux)
- 参考github：[ucb-bar/chisel-tutorial](https://github.com/ucb-bar/chisel-tutorial)
- 参考：[chisel.eecs.berkeley.edu/chisel-bootcamp.pdf](https://chisel.eecs.berkeley.edu/latest/chisel-bootcamp.pdf)

1.Installation

(1). Install Java

	sudo apt-get install default-jdk

(2). [Install sbt](https://www.scala-sbt.org/release/docs/Installing-sbt-on-Linux.html),  which isn't available by default in the system package manager:

	echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
	sudo apt-get update
	sudo apt-get install sbt

(3). Install Verilator. We currently recommend Verilator version 3.904. Follow these instructions to compile it from source.

i. Install prerequisites (if not installed already):

	sudo apt-get install git make autoconf g++ flex bison

ii. Clone the Verilator repository:

	git clone http://git.veripool.org/git/verilator

iii. In the Verilator repository directory, check out a known good version:

	git pull
	git checkout verilator_3_904

iv. In the Verilator repository directory, build and install:

	unset VERILATOR_ROOT # For bash, unsetenv for csh
	autoconf # Create ./configure script
	./configure
	make
	sudo make install

2.Chisel Tutorials

(1). Getting the repo

	$ git clone https://github.com/ucb-bar/chisel-tutorial.git
	$ cd chisel-tutorial
	$ git fetch origin
	$ git checkout release

(2). Executing Chisel

Testing Your System First make sure that you have sbt (the scala build tool) installed. See details in sbt.

	$ sbt run

This will generate and test a simple block (Hello) that always outputs the number 42 (aka 0x2a). You should see [success] on the last line of output (from sbt) and PASSED on the line before indicating the block passed the testcase. If you are doing this for the first time, sbt will automatically download the appropriate versions of Chisel3, the Chisel Testers harness and Scala and cache them (usually in ~/.ivy2).

(3). Completing the Tutorials

To learn Chisel, we recommend learning by example and just trying things out. To help with this, we have produced exercises with circuits (src/main/scala/problems) and their associated test harnesses (src/test/scala/problems) which have clearly marked places to complete their functionality and simple test cases. You can compare your work with our sample solutions in (src/main/scala/solutions) and (src/test/scala/solutions). This hierarchical organization and separation of circuits and tests is a good practice and we encourage you to understand it and use it in the future. Typically when you work on a problem you will have two open editor windows (vi, emacs, IDE, etc) one to edit the circuit and the other to edit the tests.

# First Met Chisel #

1.The Scala Programming Language

- Objected Oriented
	- Factory Objects, Classes
	- Traits, overloading etc
	- Strongly typed with type inference
- Functional
	- Higher order functions
	- Anonymous functions
	- Curring etc
- Extensible
	- Domain Specific Language(DSLs)
- Compiled to JVM
	- Good performance
	- Great Java interoperability
	- Mature debugging, execution environments
- Growing Popularity
	- Twitter
	- many Universities

推荐书籍：《Programming Scala》、《Programming in Scala》

2.Scala Bindings

	//constant
	val x = 1
	val (x, y) = (1, 2)
	
	//variable
	var y = 2
	y = 3

3.Scala Collections

	//Array's
	val tbl = new Array[Int](256)
	tbl(0) = 32
	val y = tbl(0)
	val n = tbl.length
	
	//ArrayBuffer's
	import scala.collection.mutable.ArrayBuffer
	val buf = new ArrayBuffer[Int]()
	buf += 12
	val z = buf(0)
	val l = buf.length
	
	//List's
	val els = List(1, 2, 3)
	val els2 = x :: y :: y :: Nil
	val a :: b :: c :: Nil = els
	val m = els.length
	
	//Tuple's
	val (x, y ,z) = (1, 2, 3)

4.Scala Maps and Sets

	import scala.collection.mutable.HashMap
	
	val vars = new HashMap[String, Int]()
	vars("a") = 1
	vars("b") = 2
	vars.size
	vars.contains("c")
	vars.getOrElse("c", -1)
	vars.keys
	vars.values

----------

	import scala.collection.mutable.HashSet
	
	val keys = new HashSet[Int]()
	keys += 1
	keys += 5
	keys.size -> 2
	keys.contains(2) -> false

5.Scala Iteration

	val tbl = new Array[Int](256)
	
	//loop over all indices
	for (i <- 0 until tbl.length)
		tbl(i) = i
	
	//loop of each sequence element
	val tbl2 = new ArrayBuffer[Int]
	for (e <- tbl)
		tbl2 += 2*e
	
	//loop over hashmap key / values
	for ((x, y) <- vars)
		println("K " + x + " V " + y)

6.Scala Functions

	//simple scaling function, e.g., x2(3) => 6
	def x2 (x: Int) = 2 * x
	
	//more complicated function with statements
	def f (x: Int, y: Int)= {
		val xy = x + y;
		if (x < y) xy eles -xy
	}

7.Scala Functional

	//simple scaling function, e.g., x2(3) => 6
	def x2 (x: Int) = 2 * x
	
	//produce list of 2 * elements, e.g., x2list(List(1, 2, 3)) => List(2, 4, 6)
	def x2list (xs: List[Int]) = xs.map(x2)
	
	//simple addition functions, e.g., add(1, 2) => 3
	def add (x: Int, y: Int) = x + y
	
	//sum all elements using pairwise reduction, e.g., sum(List(1, 2, 3)) => 6
	def sum (xs: List[Int]) = xs.foldLeft(0)(add)

8.Scala Object Oriented

	class Blimp(r: Double) {
		val rad = r
		println("Another Blimp")
	}
	
	new Blimp(10.0)

----------

	class Zep(h: Boolean, r: Double) extend Blimp(r) {
		val isHydrogen = h
	}
	
	new Zep(true, 100.0)

9.Scala singleton Objects

- like Java class methods
- for top level methods

	object Blimp {
		var numBlimps = 0
		def apply(r: Double) = {
			numBlimp += 1
			new Blimp(r)
		}
	}
	
	Blimp.numBlimp
	Blimp(10.0)

10.Algebraic Grapgh Construction

![](https://i.imgur.com/LLtf9kq.jpg)

11.Creating Module

![](https://i.imgur.com/gdItwmC.jpg)

12.Connecting Modules

![](https://i.imgur.com/HCKd6jT.jpg)

13.Defining Construction Functions

![](https://i.imgur.com/TkUlui6.jpg)

14.Functional Construction

![](https://i.imgur.com/wpDCy55.jpg)

