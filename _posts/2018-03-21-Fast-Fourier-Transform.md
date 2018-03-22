---
layout:     post
title:      Fast Fourier Transform(FFT)
subtitle:   Software and Hardware Implementation of Fast Fourier Transform Algorithm
date:       2018-03-21
author:     Earl Du
header-img: img/post-bg-3.jpg
catalog: true
tags:
    - Algorithm
---

> 详解FFT原理及算法，并使用C、Verilog实现。

# 傅里叶变换到底是什么？ #

关于这个问题，许多大牛对此早已有非常精彩的解答，故鄙人不再班门弄斧，直接列举链接如下：

- [傅里叶分析之掐死教程（完整版）](https://zhuanlan.zhihu.com/wille/19763358) ：此文对傅里叶分析的解释非常浅显易懂，非常容易理解，建议先看这个；
- [An Interactive Guide To The Fourier Transform](https://betterexplained.com/articles/an-interactive-guide-to-the-fourier-transform/) ：这是一篇墙外的关于傅里叶变换的介绍，文笔也是生动有趣，希望对大家有所帮助；
- [斯坦福大学公开课：傅里叶变换及其应用](http://open.163.com/special/opencourse/fouriertransforms.html) ：最后，贴上斯坦福大学的公开课，如果有时间，你可以听一下这门课程，通过这门课程的学习，估计你会对傅里叶变换会有一个非常深入的认识。

好啦，有了这些基础垫底，我想大家对于快速傅里叶变换的原理及其算法就不会那么陌生和害怕了。接下来，我们就对此展开讨论。

#  FFT算法   #

对于快速傅里叶变换的介绍通常是以对于信号的时频域分析为背景介绍，但是对于计算机专业的同学来讲，也许最容易理解的介绍方式是《算法导论》这本书中的第30章（第三版），多项式与快速傅里叶变换，这里面对基础知识介绍的非常详细。

# FFT C++语言实现 #

	#include<iostream>
	#include<fstream>
	#include<sstream>
	#include<string>
	#include<math.h>
	#include<sys/time.h>
	
	using namespace std;
	
	const float PI = 3.1415926;
	
	inline void swap(float &a, float &b)
	{
	    int tmp;
	    tmp = a;
	    a = b;
	    b = tmp;
	}
	    
	void showfile(float *xreal, float *ximag, int n)
	{
	    cout<<"The plural's number is: "<<n<<endl;
	    cout<<"The plural is:"<<endl;
	    for(int i=0;i<n;i++)
	    {
	        cout<<xreal[i]<<" "<<ximag[i]<<endl;
	    }
	    cout<<endl;
	}
	
	void bitReverse(float *xreal, float *ximag, int n)
	{
	    //位反置换Bit-Reverse-Permutaion
	    int i,j,a,b,p;
	    for(i=1,p=0;i<n;i*=2)
	    {
	        p++;
	    }
	    for(i=0;i<n;i++)
	    {
	        a = i;
	        b = 0;
	        for(j=0;j<p;j++)
	        {
	            b = (b << 1) + (a & 1); //b = b*2 + a%2;
	            a >>= 1; // a /= 2;
	        }
	        if(b > i)
	        {
	            swap(xreal[i],xreal[b]);
	            swap(ximag[i],ximag[b]);
	        }
	    }
	}
	
	void FFT(float *xreal, float *ximag, int n)
	{
	    //快速傅里叶变换，将复数x变换后仍保存在x中
	    //xreal,ximag分别是x的实部和虚部
	    float* wreal = new float[n/2];
	    float* wimag = new float[n/2];
	    float treal, timag, ureal, uimag, arg;
	    int m,k,j,t,index1,index2;
	
	    bitReverse(xreal, ximag, n);
	
	    //计算1的前n/2个n次方根的共轭复数
	    //W'j = wreal[j] + i*wimag[j], j=0,1,2,...n/2-1
	    arg = -2*PI/n;
	    treal = cos(arg);
	    timag = sin(arg);
	    wreal[0] = 1.0;
	    wimag[0] = 0.0;
	    for(j=1;j<n/2;j++)
	    {
	        wreal[j] = wreal[j-1]*treal - wimag[j-1]*timag;
	        wimag[j] = wreal[j-1]*timag + wimag[j-1]*treal;
	    }
	
	    for(m=2;m<=n;m*=2)
	    {
	        for(k=0;k<n;k+=m)
	        {
	            for(j=0;j<m/2;j++)
	            {
	                index1 = k+j;
	                index2 = index1 + m/2;
	                t = n*j/m; //旋转因子w的实部在wreal[]中的下标为t
	                treal = wreal[t]*xreal[index2] - wimag[t]*ximag[index2];
	                timag = wreal[t]*ximag[index2] + wimag[t]*xreal[index2];
	                ureal = xreal[index1];
	                uimag = ximag[index1];
	                xreal[index1] = ureal + treal;
	                ximag[index1] = uimag + timag;
	                xreal[index2] = ureal - treal;
	                ximag[index2] = uimag - timag;
	            }
	        }
	    }
	}
	
	int main(void)
	{
	    int N;
	    
	    ifstream fp("input.txt",ios::in);
	    if(!fp)
	    {
	        cout<<"Error opening input.txt"<<endl;
	    }
	
	    fp>>N;
	    float* xreal = new float[N];
	    float* ximag = new float[N];
	    
	    for(int i=0;i<N;i++)
	    {
	        fp>>xreal[i]>>ximag[i];
	    }
	    
	    cout<<"Data Input: "<<endl;
	    showfile(xreal, ximag, N);
	
	    FFT(xreal, ximag, N);
	
	    cout<<"The FFT output: "<<endl;
	    showfile(xreal, ximag, N);
	    
	    return 0;
	}

# FFT Verilog实现 #

mult32_24

	`timescale 1ns/100ps
	module mult32_24 (clk,rst,mult_a,mult_b,mult_out);
	
	parameter RST_LVL = 1'b0;
	
	input clk;
	input rst;
	input [31:0] mult_a;
	input [23:0] mult_b;  
	output [54:0] mult_out;
	
	wire msb0;
	wire [30:0] mult_a_unsigned;
	wire [22:0] mult_b_unsigned;
	reg msb_next0;
	reg [30:0] mult_a_next;
	reg [22:0] mult_b_next;
	wire msb1;
	wire [52:0] mult_store0;
	wire [52:0] mult_store1;
	wire [52:0] mult_store2;
	wire [52:0] mult_store3;
	wire [52:0] mult_store4;
	wire [52:0] mult_store5;
	wire [52:0] mult_store6;
	wire [52:0] mult_store7;
	wire [52:0] mult_store8;
	wire [52:0] mult_store9;
	wire [52:0] mult_store10;
	wire [52:0] mult_store11;
	wire [52:0] mult_store12;
	wire [52:0] mult_store13;
	wire [52:0] mult_store14;
	wire [52:0] mult_store15;
	wire [52:0] mult_store16;
	wire [52:0] mult_store17;
	wire [52:0] mult_store18;
	wire [52:0] mult_store19;
	wire [52:0] mult_store20;
	wire [52:0] mult_store21;
	wire [52:0] mult_store22;
	reg msb_next1;
	reg [52:0] adder_next0 [22:0];
	wire msb2;
	wire [53:0] sum_fir0;
	wire [53:0] sum_fir1;
	wire [53:0] sum_fir2;
	wire [53:0] sum_fir3;
	wire [53:0] sum_fir4;
	wire [53:0] sum_fir5;
	wire [53:0] sum_fir6;
	wire [53:0] sum_fir7; 
	wire [53:0] sum_fir8;
	wire [53:0] sum_fir9;
	wire [53:0] sum_fir10;
	wire [53:0] sum_fir11;
	reg msb_next2;
	reg [53:0] adder_next1 [11:0];
	wire msb3;
	wire [53:0] sum_sec0;
	wire [53:0] sum_sec1;
	wire [53:0] sum_sec2;
	wire [53:0] sum_sec3;
	wire [53:0] sum_sec4;
	wire [53:0] sum_sec5;
	reg msb_next3;
	reg [53:0] adder_next2 [5:0];
	wire msb4;
	wire [53:0] sum_tri0;
	wire [53:0] sum_tri1;
	wire [53:0] sum_tri2;
	reg msb_next4;
	reg [53:0] adder_next3 [2:0];
	wire msb5;
	wire [53:0] sum_for0;
	wire [53:0] sum_for1;
	reg msb_next5;
	reg [53:0] adder_next4 [1:0];
	wire msb6;
	wire [53:0] sum_fiv;
	reg msb_next6;
	reg [53:0] adder_next5;
	wire msb7;
	wire [53:0] result_unsigned;
	reg msb_next7;
	reg [53:0] result_next;
	wire [23:0] result_out;
	reg [23:0] result_next0;
	
	//由符号位异或得出乘积结果符号
	assign msb0 = mult_a[31] ^ mult_b[23];
	//两个输入数据求补码
	assign mult_a_unsigned = (mult_a[31])? (~mult_a[30:0] + 1'b1) : mult_a[30:0];  
	assign mult_b_unsigned = (mult_b[23])? (~mult_b[22:0] + 1'b1) : mult_b[22:0];
	
	//初始化数据到一级流水
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    msb_next0 <= 1'b0;
	    mult_a_next <= 31'b0;
	    mult_b_next <= 23'b0;
	  end
	  else begin
	    msb_next0 <= msb0;
	    mult_a_next <= mult_a_unsigned;
	    mult_b_next <= mult_b_unsigned; 
	  end
	end
	
	assign msb1 = msb_next0;
	assign mult_store0  = (mult_b_next[0 ])? {22'b0,mult_a_next} : 53'b0;
	assign mult_store1  = (mult_b_next[1 ])? {21'b0,mult_a_next, 1'b0} : 53'b0;
	assign mult_store2  = (mult_b_next[2 ])? {20'b0,mult_a_next, 2'b0} : 53'b0;
	assign mult_store3  = (mult_b_next[3 ])? {19'b0,mult_a_next, 3'b0} : 53'b0;
	assign mult_store4  = (mult_b_next[4 ])? {18'b0,mult_a_next, 4'b0} : 53'b0;
	assign mult_store5  = (mult_b_next[5 ])? {17'b0,mult_a_next, 5'b0} : 53'b0;
	assign mult_store6  = (mult_b_next[6 ])? {16'b0,mult_a_next, 6'b0} : 53'b0;
	assign mult_store7  = (mult_b_next[7 ])? {15'b0,mult_a_next, 7'b0} : 53'b0;
	assign mult_store8  = (mult_b_next[8 ])? {14'b0,mult_a_next, 8'b0} : 53'b0;
	assign mult_store9  = (mult_b_next[9 ])? {13'b0,mult_a_next, 9'b0} : 53'b0;
	assign mult_store10 = (mult_b_next[10])? {12'b0,mult_a_next,10'b0} : 53'b0;
	assign mult_store11 = (mult_b_next[11])? {11'b0,mult_a_next,11'b0} : 53'b0;
	assign mult_store12 = (mult_b_next[12])? {10'b0,mult_a_next,12'b0} : 53'b0;
	assign mult_store13 = (mult_b_next[13])? { 9'b0,mult_a_next,13'b0} : 53'b0;
	assign mult_store14 = (mult_b_next[14])? { 8'b0,mult_a_next,14'b0} : 53'b0;
	assign mult_store15 = (mult_b_next[15])? { 7'b0,mult_a_next,15'b0} : 53'b0;
	assign mult_store16 = (mult_b_next[16])? { 6'b0,mult_a_next,16'b0} : 53'b0;
	assign mult_store17 = (mult_b_next[17])? { 5'b0,mult_a_next,17'b0} : 53'b0;
	assign mult_store18 = (mult_b_next[18])? { 4'b0,mult_a_next,18'b0} : 53'b0;
	assign mult_store19 = (mult_b_next[19])? { 3'b0,mult_a_next,19'b0} : 53'b0;
	assign mult_store20 = (mult_b_next[20])? { 2'b0,mult_a_next,20'b0} : 53'b0;
	assign mult_store21 = (mult_b_next[21])? { 1'b0,mult_a_next,21'b0} : 53'b0;
	assign mult_store22 = (mult_b_next[22])? {mult_a_next,22'b0} : 53'b0;
	
	//传送数据到二级流水
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    msb_next1 <= 1'b0;
	    adder_next0[ 0] <= 53'b0;
	    adder_next0[ 1] <= 53'b0;  
	    adder_next0[ 2] <= 53'b0;
	    adder_next0[ 3] <= 53'b0;  
	    adder_next0[ 4] <= 53'b0;
	    adder_next0[ 5] <= 53'b0;  
	    adder_next0[ 6] <= 53'b0;
	    adder_next0[ 7] <= 53'b0;   
	    adder_next0[ 8] <= 53'b0;
	    adder_next0[ 9] <= 53'b0;  
	    adder_next0[10] <= 53'b0;
	    adder_next0[11] <= 53'b0;  
	    adder_next0[12] <= 53'b0;
	    adder_next0[13] <= 53'b0;  
	    adder_next0[14] <= 53'b0;
	    adder_next0[15] <= 53'b0; 
	    adder_next0[16] <= 53'b0;
	    adder_next0[17] <= 53'b0;  
	    adder_next0[18] <= 53'b0;
	    adder_next0[19] <= 53'b0;  
	    adder_next0[20] <= 53'b0;
	    adder_next0[21] <= 53'b0;  
	    adder_next0[22] <= 53'b0;              
	  end
	  else begin
	    msb_next1 <= msb1;
	    adder_next0[0] <= mult_store0;
	    adder_next0[ 1] <= mult_store1;  
	    adder_next0[ 2] <= mult_store2;
	    adder_next0[ 3] <= mult_store3;  
	    adder_next0[ 4] <= mult_store4;
	    adder_next0[ 5] <= mult_store5;  
	    adder_next0[ 6] <= mult_store6;
	    adder_next0[ 7] <= mult_store7;   
	    adder_next0[ 8] <= mult_store8;
	    adder_next0[ 9] <= mult_store9;  
	    adder_next0[10] <= mult_store10;
	    adder_next0[11] <= mult_store11;  
	    adder_next0[12] <= mult_store12;
	    adder_next0[13] <= mult_store13;  
	    adder_next0[14] <= mult_store14;
	    adder_next0[15] <= mult_store15; 
	    adder_next0[16] <= mult_store16;
	    adder_next0[17] <= mult_store17;  
	    adder_next0[18] <= mult_store18;
	    adder_next0[19] <= mult_store19;  
	    adder_next0[20] <= mult_store20;
	    adder_next0[21] <= mult_store21;  
	    adder_next0[22] <= mult_store22;                
	  end
	end
	
	assign msb2 = msb_next1;
	assign sum_fir0  = adder_next0[ 0] + adder_next0[ 1];
	assign sum_fir1  = adder_next0[ 2] + adder_next0[ 3]; 
	assign sum_fir2  = adder_next0[ 4] + adder_next0[ 5];
	assign sum_fir3  = adder_next0[ 6] + adder_next0[ 7];
	assign sum_fir4  = adder_next0[ 8] + adder_next0[ 9];
	assign sum_fir5  = adder_next0[10] + adder_next0[11];
	assign sum_fir6  = adder_next0[12] + adder_next0[13];
	assign sum_fir7  = adder_next0[14] + adder_next0[15];
	assign sum_fir8  = adder_next0[16] + adder_next0[17];
	assign sum_fir9  = adder_next0[18] + adder_next0[19];
	assign sum_fir10 = adder_next0[20] + adder_next0[21];
	assign sum_fir11 = adder_next0[22];
	
	//传送数据到三级流水
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    msb_next2 <= 1'b0;
	    adder_next1[ 0] <= 54'b0;
	    adder_next1[ 1] <= 54'b0;
	    adder_next1[ 2] <= 54'b0;   
	    adder_next1[ 3] <= 54'b0;
	    adder_next1[ 4] <= 54'b0;
	    adder_next1[ 5] <= 54'b0; 
	    adder_next1[ 6] <= 54'b0;
	    adder_next1[ 7] <= 54'b0;
	    adder_next1[ 8] <= 54'b0;
	    adder_next1[ 9] <= 54'b0;
	    adder_next1[10] <= 54'b0;
	    adder_next1[11] <= 54'b0;                
	  end
	  else begin
	    msb_next2 <= msb2;
	    adder_next1[ 0] <= sum_fir0;
	    adder_next1[ 1] <= sum_fir1;
	    adder_next1[ 2] <= sum_fir2;
	    adder_next1[ 3] <= sum_fir3;
	    adder_next1[ 4] <= sum_fir4;  
	    adder_next1[ 5] <= sum_fir5;
	    adder_next1[ 6] <= sum_fir6;
	    adder_next1[ 7] <= sum_fir7;
	    adder_next1[ 8] <= sum_fir8;
	    adder_next1[ 9] <= sum_fir9; 
	    adder_next1[10] <= sum_fir10;
	    adder_next1[11] <= sum_fir11;                 
	  end
	end
	
	assign msb3 = msb_next2;
	assign sum_sec0 = adder_next1[0] + adder_next1[1];
	assign sum_sec1 = adder_next1[2] + adder_next1[3];
	assign sum_sec2 = adder_next1[4] + adder_next1[5];
	assign sum_sec3 = adder_next1[6] + adder_next1[7];
	assign sum_sec4 = adder_next1[8] + adder_next1[9];
	assign sum_sec5 = adder_next1[10] + adder_next1[11];
	
	//传送数据到五级流水
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    msb_next3 <= 1'b0;
	    adder_next2[0] <= 54'b0;
	    adder_next2[1] <= 54'b0;
	    adder_next2[2] <= 54'b0;
	    adder_next2[3] <= 54'b0;
	    adder_next2[4] <= 54'b0;
	    adder_next2[5] <= 54'b0;    
	  end
	  else begin
	    msb_next3 <= msb3;
	    adder_next2[0] <= sum_sec0;
	    adder_next2[1] <= sum_sec1;
	    adder_next2[2] <= sum_sec2;
	    adder_next2[3] <= sum_sec3;
	    adder_next2[4] <= sum_sec4;
	    adder_next2[5] <= sum_sec5;   
	  end 
	end
	
	assign msb4 = msb_next3;
	assign sum_tri0 = adder_next2[0] + adder_next2[1];
	assign sum_tri1 = adder_next2[2] + adder_next2[3];
	assign sum_tri2 = adder_next2[4] + adder_next2[5];
	
	//传送数据到六级流水
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    msb_next4 <= 1'b0;
	    adder_next3[0] <= 54'b0;
	    adder_next3[1] <= 54'b0;
	    adder_next3[2] <= 54'b0;  
	  end
	  else begin
	    msb_next4 <= msb4;
	    adder_next3[0] <= sum_tri0;
	    adder_next3[1] <= sum_tri1;
	    adder_next3[2] <= sum_tri2;
	  end
	end
	
	assign msb5 = msb_next4;
	assign sum_for0 = adder_next3[0] + adder_next3[1];
	assign sum_for1 = adder_next3[2];
	
	//传送数据到七级流水
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    msb_next5 <= 1'b0;
	    adder_next4[0] <= 54'b0;
	    adder_next4[1] <= 54'b0;  
	  end
	  else begin
	    msb_next5 <= msb5;
	    adder_next4[0] <= sum_for0;
	    adder_next4[1] <= sum_for1;
	  end
	end
	
	assign msb6 = msb_next5;
	assign sum_fiv = adder_next4[0] + adder_next4[1];
	
	//传送数据到八级流水
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    msb_next6 <= 1'b0;
	    adder_next5 <= 54'b0;  
	  end
	  else begin
	    msb_next6 <= msb6;
	    adder_next5 <= sum_fiv;  
	  end
	end
	
	assign msb7 = msb_next6;
	assign result_unsigned = (adder_next5 == 54'b0)? 54'b0 : 
	                    (msb7)? (~adder_next5 + 1'b1) : adder_next5;
	
	//传送数据到九级流水                    
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    msb_next7 <= 1'b0;
	    result_next <= 54'b0;
	  end
	  else begin
	    msb_next7 <= msb7;
	    result_next <= result_unsigned;
	  end
	end                     
	
	assign result_out = (result_next == 54'b0)? 54'b0 : {msb_next7,result_next};
	
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) result_next0 <= 24'b0;
	  else result_next0 <= result_out;
	end
	
	assign mult_out = result_next0;
	endmodule

butter_unit

	`timescale 1ns/100ps
	module butter_2  ( 
	                 clk,
	                 rst,
	                 butt2_real0,
	                 butt2_imag0,
	                 butt2_real1,
	                 butt2_imag1,	//蝶形运算单元两个输入数据的实部和虚部
	                 factor_real,
	                 factor_imag,	//蝶形运算单元旋转因子数据的实部和虚部
	                 y0_real,
	                 y0_imag,
	                 y1_real,
	                 y1_imag		//蝶形运算两个输出因子数据的实部和虚部
	                 );
	
	parameter RST_LVL = 1'b0;
	  
	input clk;
	input rst;
	input [23:0] butt2_real0;
	input [23:0] butt2_imag0;
	input [23:0] butt2_real1;
	input [23:0] butt2_imag1;
	input [31:0] factor_real;
	input [31:0] factor_imag;
	output [23:0] y0_real;
	output [23:0] y0_imag;
	output [23:0] y1_real;
	output [23:0] y1_imag;
	
	wire [54:0] mult_real0;
	wire [54:0] mult_real1;
	wire [54:0] mult_imag0;
	wire [54:0] mult_imag1;
	wire [54:0] result_real0;
	wire [54:0] result_real1;
	wire [54:0] result_imag0;
	wire [54:0] result_imag1;
	reg [54:0] result_real0_next;
	reg [54:0] result_imag0_next;
	reg [54:0] result_real1_next;
	reg [54:0] result_imag1_next;
	wire [54:0] adder_real0;
	wire [54:0] adder_imag0;
	reg [54:0] adder_real0_next [10:0];
	reg [54:0] adder_imag0_next [10:0];
	wire [54:0] adder_real1;
	wire [54:0] adder_imag1;
	reg [54:0] adder_real1_next;
	reg [54:0] adder_imag1_next;
	wire [54:0] y0_result_real;
	wire [54:0] y0_result_imag;
	wire [54:0] y1_result_real;
	wire [54:0] y1_result_imag;
	reg [54:0] y0_result_real_next;
	reg [54:0] y0_result_imag_next;
	reg [54:0] y1_result_real_next;
	reg [54:0] y1_result_imag_next;
	wire [23:0] output0_real;
	wire [23:0] output0_imag;
	wire [23:0] output1_real;
	wire [23:0] output1_imag;
	reg [23:0] output0_real_next;
	reg [23:0] output0_imag_next;
	reg [23:0] output1_real_next;
	reg [23:0] output1_imag_next;
	
	mult32_24 U_mult_real0 (
	           .clk     (clk        ),
	           .rst     (rst        ),
	           .mult_a  (factor_real),
	           .mult_b  (butt2_real1),
	           .mult_out(mult_real0 )
	                      );
						  
	mult32_24 U_mult_real1 ( 
	           .clk     (clk        ),
	           .rst     (rst        ),
	           .mult_a  (factor_imag),
	           .mult_b  (butt2_imag1),
	           .mult_out(mult_real1 )
	                      );
						  
	mult32_24 U_mult_imag0 (
	           .clk     (clk        ),
	           .rst     (rst        ),
	           .mult_a  (factor_imag),
	           .mult_b  (butt2_real1),
	           .mult_out(mult_imag0 )
	                      );
						  
	mult32_24 U_mult_imag1 (
	           .clk     (clk        ),
	           .rst     (rst        ),
	           .mult_a  (factor_real),
	           .mult_b  (butt2_imag1),
	           .mult_out(mult_imag1 )
	                      );
	                      
	assign result_real0 = (mult_real0 == 55'b0)? 55'b0 : 
	             (mult_real0[54])? {13'b1111111111111,mult_real0[54:13]} : {13'b0,mult_real0[54:13]};  
	assign result_real1 = (mult_real1 == 55'b0)? 55'b0 : 
	             (mult_real1[54])? {13'b1111111111111,mult_real1[54:13]} : {13'b0,mult_real1[54:13]}; 
	assign result_imag0 = (mult_imag0 == 55'b0)? 55'b0 : 
	             (mult_imag0[54])? {13'b1111111111111,mult_imag0[54:13]} : {13'b0,mult_imag0[54:13]};
	assign result_imag1 = (mult_imag1 == 55'b0)? 55'b0 : 
	             (mult_imag1[54])? {13'b1111111111111,mult_imag1[54:13]} : {13'b0,mult_imag1[54:13]};  
	
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    result_real0_next <= 55'b0;
	    result_imag0_next <= 55'b0;
	    result_real1_next <= 55'b0;
	    result_imag1_next <= 55'b0;  
	  end
	  else begin
	    result_real0_next <= result_real0;
	    result_imag0_next <= result_imag0;
	    result_real1_next <= result_real1;
	    result_imag1_next <= result_imag1;    
	  end
	end
	                 
	assign adder_real1 = result_real0_next - result_real1_next;
	assign adder_imag1 = result_imag0_next + result_imag1_next; 
	
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    adder_real1_next <= 55'b0;
	    adder_imag1_next <= 55'b0;  
	  end
	  else begin
	    adder_real1_next <= adder_real1;
	    adder_imag1_next <= adder_imag1;
	  end
	end
	
	assign adder_real0 = (butt2_real0 == 24'b0)? 55'b0 : 
	            (butt2_real0[23])? {31'b1111111111111111111111111111111,butt2_real0} : {31'b0,butt2_real0};
	assign adder_imag0 = (butt2_imag0 == 24'b0)? 55'b0 : 
	            (butt2_imag0[23])? {31'b1111111111111111111111111111111,butt2_imag0} : {31'b0,butt2_imag0};
	
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    adder_real0_next[0] <= 55'b0;
	    adder_imag0_next[0] <= 55'b0;
	    adder_real0_next[1] <= 55'b0;
	    adder_imag0_next[1] <= 55'b0;
	    adder_real0_next[2] <= 55'b0;
	    adder_imag0_next[2] <= 55'b0;
	    adder_real0_next[3] <= 55'b0;
	    adder_imag0_next[3] <= 55'b0;
	    adder_real0_next[4] <= 55'b0;
	    adder_imag0_next[4] <= 55'b0;
	    adder_real0_next[5] <= 55'b0;
	    adder_imag0_next[5] <= 55'b0;
	    adder_real0_next[6] <= 55'b0;
	    adder_imag0_next[6] <= 55'b0;
	    adder_real0_next[7] <= 55'b0;
	    adder_imag0_next[7] <= 55'b0;
	    adder_real0_next[8] <= 55'b0;
	    adder_imag0_next[8] <= 55'b0;
	    adder_real0_next[9] <= 55'b0;
	    adder_imag0_next[9] <= 55'b0;
	    adder_real0_next[10] <= 55'b0;
	    adder_imag0_next[10] <= 55'b0;      
	  end
	  else begin
	    adder_real0_next[0] <= adder_real0;
	    adder_imag0_next[0] <= adder_imag0;
	    adder_real0_next[1] <= adder_real0_next[0];
	    adder_imag0_next[1] <= adder_imag0_next[0];
	    adder_real0_next[2] <= adder_real0_next[1];
	    adder_imag0_next[2] <= adder_imag0_next[1];
	    adder_real0_next[3] <= adder_real0_next[2];
	    adder_imag0_next[3] <= adder_imag0_next[2];
	    adder_real0_next[4] <= adder_real0_next[3];
	    adder_imag0_next[4] <= adder_imag0_next[3];
	    adder_real0_next[5] <= adder_real0_next[4];
	    adder_imag0_next[5] <= adder_imag0_next[4];
	    adder_real0_next[6] <= adder_real0_next[5];
	    adder_imag0_next[6] <= adder_imag0_next[5];
	    adder_real0_next[7] <= adder_real0_next[6];
	    adder_imag0_next[7] <= adder_imag0_next[6];
	    adder_real0_next[8] <= adder_real0_next[7];
	    adder_imag0_next[8] <= adder_imag0_next[7];
	    adder_real0_next[9] <= adder_real0_next[8];
	    adder_imag0_next[9] <= adder_imag0_next[8];
	    adder_real0_next[10] <= adder_real0_next[9];
	    adder_imag0_next[10] <= adder_imag0_next[9];    
	  end
	end
	
	assign y0_result_real = adder_real0_next[10] + adder_real1_next;
	assign y0_result_imag = adder_imag0_next[10] + adder_imag1_next;
	assign y1_result_real = adder_real0_next[10] - adder_real1_next;
	assign y1_result_imag = adder_imag0_next[10] - adder_imag1_next;
	
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    y0_result_real_next <= 55'b0;
	    y0_result_imag_next <= 55'b0;
	    y1_result_real_next <= 55'b0;
	    y1_result_imag_next <= 55'b0;  
	  end
	  else begin
	    y0_result_real_next <= y0_result_real;
	    y0_result_imag_next <= y0_result_imag;
	    y1_result_real_next <= y1_result_real;
	    y1_result_imag_next <= y1_result_imag;
	  end 
	end
	
	assign output0_real = (y0_result_real == 54'b0)? 24'b0 : 
	                   {y0_result_real_next[54],y0_result_real_next[22:0]};
	assign output0_imag = (y0_result_imag == 54'b0)? 24'b0 : 
	                   {y0_result_imag_next[54],y0_result_imag_next[22:0]};
	assign output1_real = (y1_result_real == 54'b0)? 24'b0 : 
	                   {y1_result_real_next[54],y1_result_real_next[22:0]};
	assign output1_imag = (y1_result_imag == 54'b0)? 24'b0 : 
	                   {y1_result_imag_next[54],y1_result_imag_next[22:0]};
	
	always @(posedge clk or negedge rst) begin
	  if (rst == RST_LVL) begin
	    output0_real_next <= 24'b0;
	    output0_imag_next <= 24'b0;
	    output1_real_next <= 24'b0;
	    output1_imag_next <= 24'b0;  
	  end
	  else begin
	    output0_real_next <= output0_real;
	    output0_imag_next <= output0_imag;
	    output1_real_next <= output1_real;
	    output1_imag_next <= output1_imag;    
	  end
	end
	
	assign y0_real = output0_real_next;
	assign y0_imag = output0_imag_next;
	assign y1_real = output1_real_next;
	assign y1_imag = output1_imag_next;
	 
	endmodule

fft_8

	`timescale 1ns/100ps
	module fft_8 (
	              clk,
	              rst,
	              butt8_real0,
	              butt8_imag0,
	              butt8_real1,
	              butt8_imag1,
	              butt8_real2,
	              butt8_imag2,
	              butt8_real3,
	              butt8_imag3,
	              butt8_real4,
	              butt8_imag4,
	              butt8_real5,
	              butt8_imag5,
	              butt8_real6,
	              butt8_imag6,
	              butt8_real7,
	              butt8_imag7,
	              y0_real,
	              y0_imag,
	              y1_real,
	              y1_imag,
	              y2_real,
	              y2_imag,
	              y3_real,
	              y3_imag,
	              y4_real,
	              y4_imag,
	              y5_real,
	              y5_imag,
	              y6_real,
	              y6_imag,
	              y7_real,
	              y7_imag              
	              );
	
	parameter RST_LVL = 1'b0;
	
	input clk;
	input rst;
	input [23:0] butt8_real0;
	input [23:0] butt8_imag0;
	input [23:0] butt8_real1;
	input [23:0] butt8_imag1;
	input [23:0] butt8_real2;
	input [23:0] butt8_imag2;
	input [23:0] butt8_real3;
	input [23:0] butt8_imag3;
	input [23:0] butt8_real4;
	input [23:0] butt8_imag4;
	input [23:0] butt8_real5;
	input [23:0] butt8_imag5;
	input [23:0] butt8_real6;
	input [23:0] butt8_imag6;
	input [23:0] butt8_real7;
	input [23:0] butt8_imag7;
	output [23:0] y0_real;
	output [23:0] y0_imag;
	output [23:0] y1_real;
	output [23:0] y1_imag;
	output [23:0] y2_real;
	output [23:0] y2_imag;
	output [23:0] y3_real;
	output [23:0] y3_imag;
	output [23:0] y4_real;
	output [23:0] y4_imag;
	output [23:0] y5_real;
	output [23:0] y5_imag;
	output [23:0] y6_real;
	output [23:0] y6_imag;
	output [23:0] y7_real;
	output [23:0] y7_imag;
	
	wire [23:0] gd_real0;
	wire [23:0] gd_imag0;
	wire [23:0] gd_real1;
	wire [23:0] gd_imag1;
	wire [23:0] hd_real0;
	wire [23:0] hd_imag0;
	wire [23:0] hd_real1;
	wire [23:0] hd_imag1;
	wire [23:0] gs_real0;
	wire [23:0] gs_imag0;
	wire [23:0] gs_real1;
	wire [23:0] gs_imag1;
	wire [23:0] hs_real0;
	wire [23:0] hs_imag0;
	wire [23:0] hs_real1;
	wire [23:0] hs_imag1;
	
	wire [23:0] g_real0;
	wire [23:0] g_imag0;
	wire [23:0] g_real1;
	wire [23:0] g_imag1;
	wire [23:0] g_real2;
	wire [23:0] g_imag2;
	wire [23:0] g_real3;
	wire [23:0] g_imag3;
	wire [23:0] h_real0;
	wire [23:0] h_imag0;
	wire [23:0] h_real1;
	wire [23:0] h_imag1;
	wire [23:0] h_real2;
	wire [23:0] h_imag2;
	wire [23:0] h_real3;
	wire [23:0] h_imag3;
	
	wire [31:0] factor_real0;
	wire [31:0] factor_imag0;
	wire [31:0] factor_real1;
	wire [31:0] factor_imag1;
	wire [31:0] factor_real2;
	wire [31:0] factor_imag2;
	wire [31:0] factor_real3;
	wire [31:0] factor_imag3;
	
	assign factor_real0 = 32'h00002000;
	assign factor_imag0 = 32'h00000000;
	assign factor_real1 = 32'h000016a0;
	assign factor_imag1 = 32'hffffe95f;
	assign factor_real2 = 32'h00000000;
	assign factor_imag2 = 32'hffffe000;
	assign factor_real3 = 32'hffffe95f;
	assign factor_imag3 = 32'hffffe95f;
	
	butter_2  U0_unit0( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(butt8_real0 ),
	          .butt2_imag0(butt8_imag0 ),
	          .butt2_real1(butt8_real4 ),
	          .butt2_imag1(butt8_imag4 ),
	          .factor_real(factor_real0),
	          .factor_imag(factor_imag0),
	          .y0_real    (gd_real0    ),
	          .y0_imag    (gd_imag0    ),
	          .y1_real    (gd_real1    ),
	          .y1_imag    (gd_imag1    )
	                 );
	butter_2  U0_unit1( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(butt8_real2 ),
	          .butt2_imag0(butt8_imag2 ),
	          .butt2_real1(butt8_real6 ),
	          .butt2_imag1(butt8_imag6 ),
	          .factor_real(factor_real0),
	          .factor_imag(factor_imag0),
	          .y0_real    (hd_real0    ),
	          .y0_imag    (hd_imag0    ),
	          .y1_real    (hd_real1    ),
	          .y1_imag    (hd_imag1    )
	                 );  
	butter_2  U0_unit2( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(butt8_real1 ),
	          .butt2_imag0(butt8_imag1 ),
	          .butt2_real1(butt8_real5 ),
	          .butt2_imag1(butt8_imag5 ),
	          .factor_real(factor_real0),
	          .factor_imag(factor_imag0),
	          .y0_real    (gs_real0    ),
	          .y0_imag    (gs_imag0    ),
	          .y1_real    (gs_real1    ),
	          .y1_imag    (gs_imag1    )
	                 );
	butter_2  U0_unit3( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(butt8_real3 ),
	          .butt2_imag0(butt8_imag3 ),
	          .butt2_real1(butt8_real7 ),
	          .butt2_imag1(butt8_imag7 ),
	          .factor_real(factor_real0),
	          .factor_imag(factor_imag0),
	          .y0_real    (hs_real0    ),
	          .y0_imag    (hs_imag0    ),
	          .y1_real    (hs_real1    ),
	          .y1_imag    (hs_imag1    )
	                 ); 
	                 
	
	butter_2  U1_unit0( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(gd_real0    ),
	          .butt2_imag0(gd_imag0    ),
	          .butt2_real1(hd_real0    ),
	          .butt2_imag1(hd_imag0    ),
	          .factor_real(factor_real0),
	          .factor_imag(factor_imag0),
	          .y0_real    (g_real0     ),
	          .y0_imag    (g_imag0     ),
	          .y1_real    (g_real2     ),
	          .y1_imag    (g_imag2     )
	                 ); 
	butter_2  U1_unit1( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(gd_real1    ),
	          .butt2_imag0(gd_imag1    ),
	          .butt2_real1(hd_real1    ),
	          .butt2_imag1(hd_imag1    ),
	          .factor_real(factor_real2),
	          .factor_imag(factor_imag2),
	          .y0_real    (g_real1     ),
	          .y0_imag    (g_imag1     ),
	          .y1_real    (g_real3     ),
	          .y1_imag    (g_imag3     )
	                 );                                  
	butter_2  U1_unit2( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(gs_real0    ),
	          .butt2_imag0(gs_imag0    ),
	          .butt2_real1(hs_real0    ),
	          .butt2_imag1(hs_imag0    ),
	          .factor_real(factor_real0),
	          .factor_imag(factor_imag0),
	          .y0_real    (h_real0     ),
	          .y0_imag    (h_imag0     ),
	          .y1_real    (h_real2     ),
	          .y1_imag    (h_imag2     )
	                 );                 
	butter_2  U1_unit3( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(gs_real1    ),
	          .butt2_imag0(gs_imag1    ),
	          .butt2_real1(hs_real1    ),
	          .butt2_imag1(hs_imag1    ),
	          .factor_real(factor_real2),
	          .factor_imag(factor_imag2),
	          .y0_real    (h_real1     ),
	          .y0_imag    (h_imag1     ),
	          .y1_real    (h_real3     ),
	          .y1_imag    (h_imag3     )
	                 );
	
	butter_2  U2_unit0( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(g_real0     ),
	          .butt2_imag0(g_imag0     ),
	          .butt2_real1(h_real0     ),
	          .butt2_imag1(h_imag0     ),
	          .factor_real(factor_real0),
	          .factor_imag(factor_imag0),
	          .y0_real    (y0_real     ),
	          .y0_imag    (y0_imag     ),
	          .y1_real    (y4_real     ),
	          .y1_imag    (y4_imag     )
	                 ); 
	
	butter_2  U2_unit1( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(g_real1     ),
	          .butt2_imag0(g_imag1     ),
	          .butt2_real1(h_real1     ),
	          .butt2_imag1(h_imag1     ),
	          .factor_real(factor_real1),
	          .factor_imag(factor_imag1),
	          .y0_real    (y1_real     ),
	          .y0_imag    (y1_imag     ),
	          .y1_real    (y5_real     ),
	          .y1_imag    (y5_imag     )
	                 ); 
	butter_2  U2_unit2( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(g_real2     ),
	          .butt2_imag0(g_imag2     ),
	          .butt2_real1(h_real2     ),
	          .butt2_imag1(h_imag2     ),
	          .factor_real(factor_real2),
	          .factor_imag(factor_imag2),
	          .y0_real    (y2_real     ),
	          .y0_imag    (y2_imag     ),
	          .y1_real    (y6_real     ),
	          .y1_imag    (y6_imag     )
	                 );                                                 
	butter_2  U2_unit3( 
	          .clk        (clk         ),
	          .rst        (rst         ),
	          .butt2_real0(g_real3     ),
	          .butt2_imag0(g_imag3     ),
	          .butt2_real1(h_real3     ),
	          .butt2_imag1(h_imag3     ),
	          .factor_real(factor_real3),
	          .factor_imag(factor_imag3),
	          .y0_real    (y3_real     ),
	          .y0_imag    (y3_imag     ),
	          .y1_real    (y7_real     ),
	          .y1_imag    (y7_imag     )
	                 );
	endmodule

fft_tb

	`timescale 1ns/100ps
	module fft_tb ;
	
	reg clk;
	reg rst;
	reg [23:0] x0_real;
	reg [23:0] x0_imag;
	reg [23:0] x1_real;
	reg [23:0] x1_imag;
	reg [23:0] x2_real;
	reg [23:0] x2_imag;
	reg [23:0] x3_real;
	reg [23:0] x3_imag;
	reg [23:0] x4_real;
	reg [23:0] x4_imag;
	reg [23:0] x5_real;
	reg [23:0] x5_imag;
	reg [23:0] x6_real;
	reg [23:0] x6_imag;
	reg [23:0] x7_real;
	reg [23:0] x7_imag;
	
	wire [23:0] y0_real;
	wire [23:0] y0_imag;
	wire [23:0] y1_real;
	wire [23:0] y1_imag;
	wire [23:0] y2_real;
	wire [23:0] y2_imag;
	wire [23:0] y3_real;
	wire [23:0] y3_imag;
	wire [23:0] y4_real;
	wire [23:0] y4_imag;
	wire [23:0] y5_real;
	wire [23:0] y5_imag;
	wire [23:0] y6_real;
	wire [23:0] y6_imag;
	wire [23:0] y7_real;
	wire [23:0] y7_imag;
	
	always #1 clk = ~clk;
	
	fft_8 U_fft8 (
	              .clk        (clk    ),
	              .rst        (rst    ),
	              .butt8_real0(x0_real),
	              .butt8_imag0(x0_imag),
	              .butt8_real1(x1_real),
	              .butt8_imag1(x1_imag),
	              .butt8_real2(x2_real),
	              .butt8_imag2(x2_imag),
	              .butt8_real3(x3_real),
	              .butt8_imag3(x3_imag),
	              .butt8_real4(x4_real),
	              .butt8_imag4(x4_imag),
	              .butt8_real5(x5_real),
	              .butt8_imag5(x5_imag),
	              .butt8_real6(x6_real),
	              .butt8_imag6(x6_imag),
	              .butt8_real7(x7_real),
	              .butt8_imag7(x7_imag),
	              .y0_real    (y0_real),
	              .y0_imag    (y0_imag),
	              .y1_real    (y1_real),
	              .y1_imag    (y1_imag),
	              .y2_real    (y2_real),
	              .y2_imag    (y2_imag),
	              .y3_real    (y3_real),
	              .y3_imag    (y3_imag),
	              .y4_real    (y4_real),
	              .y4_imag    (y4_imag),
	              .y5_real    (y5_real),
	              .y5_imag    (y5_imag),
	              .y6_real    (y6_real),
	              .y6_imag    (y6_imag),
	              .y7_real    (y7_real),
	              .y7_imag    (y7_imag)          
	              );
	
	initial
	begin
	clk = 0;
	rst = 0;
	x0_real = 24'd10;
	x1_real = 24'd20;
	x2_real = 24'd30;
	x3_real = 24'd40;
	x4_real = 24'd10;
	x5_real = 24'd20;
	x6_real = 24'd30;
	x7_real = 24'd40;
	
	x0_imag = 24'd0;
	x1_imag = 24'd0;
	x2_imag = 24'd0;
	x3_imag = 24'd0;
	x4_imag = 24'd0;
	x5_imag = 24'd0;
	x6_imag = 24'd0;
	x7_imag = 24'd0;
	
	#10 rst = 1;
	end  
	endmodule

注：

butt8_real0—7、butt8_imag0—7为输入的8位数据的实部和虚部；

y0_real~y7_real、y0_imag~y7_imag为8点fft的输出;

factor_real0~factor_real3、factor_imag0~factor_imag3为旋转因子的实部和虚部，旋转因子定义在fft_8内部,
对所有旋转因子都预先做了乘2^13的处理；

在本设计中采用了32*24树形九级流水线乘法器，由于是有符号数的乘法，所以流水线比较长，
蝶形单元的内部数据为55位，对蝶形单元中的所有的输出都做了截位处理，输出为24位，因为旋转因子并不会大于1，所以不会影响下一级的运算；

参考文献：[Implementation of Fast Fourier Transform(FFT) on FPGA using Verilog HDL](https://pdfs.semanticscholar.org/45d0/1d504aa107a087e14c4ba23c5eeab0e01766.pdf) 




