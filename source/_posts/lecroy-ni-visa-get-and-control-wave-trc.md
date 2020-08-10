---
title: 基于NI-VISA获取示波器波形数据
date: 2016/07/21
categories:
- 项目
tags:
- 项目
- 示波器
- NI-VISA
---

基于NI-VISA获取示波器波形数据
==================

实验室在做一个故障检测的项目，需要读取485串口发来的数据，为此使用了鼎阳的示波器SDS3054来获取和记录波形。示波器可以在触发信号下记录一次波形，但是项目要求波形能够远程实时获取和保存，并且能持续的记录。示波器支持NI-VISA编程，这就可以编程来实现示波器的控制。而NI-VISA的资料在网上也很少，费劲很大周折才搞出了这个一个小DEMO。在这过程中，也认识到查阅官方英文文档和直接使用搜索引擎之间的不同。如果有人需要其他帮助的，可以联系我。


## 项目背景
以下摘自百度：
NI-VISA(Virtual Instrument Software Architecture，以下简称为"VISA")是美国国家仪器  NI(National Instruments VISA)公司开发的一种用来与各种仪器总线进行通信的高级应用编程接口。VISA总线I/O软件是一个综合软件包，不受平台、总线和环境的限制，可用来对USB、GPIB、串口、VXI、PXI和以太网系统进行配置、编程和调试。VISA是虚拟仪器系统I/O接口软件。基于自底向上结构模型的VISA创造了一个统一形式的I/O控制函数集。一方面，对初学者或是简单任务的设计者来说， VISA提供了简单易用的控制函数集，在应用形式上相当简单; 另一方面，对复杂系统的组建者来说，VISA提供了非常强大的仪器控制功能与资源管理。

简单将，NI-VISA屏蔽了底层的硬件细节，程序员只需要调用NI-VISA提供的函数和API，以及相关的SCPI指令集，就可以完成对示波器等设备的控制。这也不得不佩服做底层的工程师。



## 项目需求

项目需要控制示波器完成：

在外源触发信号的控制下，示波器工作在single模式下，记录触发信号通道，485串口数据通道的波形，并且在触发信号（一个小方波，上升沿触发）到来时，保存当前的波形。并且可以被实时的远程获取，且在获取到这一帧波形时，能自动的继续记录下一次波形。

## 项目源码

项目中需要研究厂家的示波器保存文件TRC的文件格式，这里借鉴了一个外国网站上的Matlab代码 
下载：[main.m](/uploads/lecroy-ni-visa-get-and-control-wave-trc/main.m) [ReadLeCroyBinaryWaveform.m](/uploads/lecroy-ni-visa-get-and-control-wave-trc/ReadLeCroyBinaryWaveform.m)

这个项目开发IDE为VS2013，开发语言为C（C++），需要安装好NI-VISA的相关软件，可以在官网[Teledyne Lecroy](http://teledynelecroy.com/support/softwaredownload/)下载示波器Oscilloscope软件。同时采用的是网络方式进行远程控制，需要配置安装好LeCroyVICPPassport.dll（官网也能下载到），当然也可以使用USB控制方式，不过只要改动连接命令即可，这个官方手册可以查询（需要手册可以联系我）。

提供main.cpp，需要完整源码包的联系我[下载](/uploads/lecroy-ni-visa-get-and-control-wave-trc/161012_GetWaveTrcFromOscilloscope.rar)。
```C
#include "main.h"

int main(int argc, char *argv[]){
	char addr[100] = { 0 };
	char source[10][5] = { 0 };
	int src_num = 0;

	if (argc < 3){
		plog("Usage:  Get waveforms from an oscilloscope.\n");
		plog("Tips:   Should be used in cmd mode.\n");
		plog("Format: [ADDR][SOURCE1][SOURCE2][SOURCE3]...\n");
		plog("Example:\n\tVICP::192.168.1.133 C1 C2 Z1 Z2\n\tUSB0::0x05FF::0x1023::SDS300B21443023::INSTR C1 C2 Z1 Z2\n");
		plog("\nPress any key to exit...\n");
		getchar();
		return -1;
	}
	else{

		//此处需要添加addr地址合法性、通道名有效性判断

		char tmp[100] = { 0 };

		src_num = argc-2;
		strcpy(addr, argv[1]);
		
		int m = 0;
		for (; m < argc-2; m++){
			memset(tmp, 0, sizeof(tmp));
			strcpy(tmp, argv[m+2]);
			strcpy(source[m], _strupr(tmp));
		}

	}

	plog("ADDR:%s SOURCES(%d):",addr,src_num);
	for (int t = 0; t < src_num;t++){
		plog("%s ",source[t]);
	}
	printf("\n");

	//开始工作
	ViSession default_RM = -1;								    //default_resource_manager
	ViSession ins = -1;										    //instrument
	ViStatus  stat = -1;									    //status
	ViUInt32  rt_cnt = -1;								        //read_count(or return_count)
	ViUInt32  wt_cnt = -1;								        //write_count

	static unsigned char str_rt_buf[RT_BUF_SIZE] = { 0 };		//read_buf_size
	static unsigned char str_wt_buf[WT_BUF_SIZE] = { 0 };		//write_buf_size

	fflush(stdin);
	fflush(stdout);

	//Open a session and find resources
	if (fun_visa_viConnect(&default_RM, addr, str_rt_buf, &ins) == -1){
		goto ERR;
	}

	//设置一些属性，如超时时间等
	//属性设置前需要询问当前设置，设置完后也需要再次询问设置，以确认设置成功
	//stat=viSetAttribute();
	if (fun_visa_viPrintf(&ins, "CHDR OFF;CORD HI;CFMT DEF9,WORD,BIN\n","E001") == -1){
		goto ERR;
	}

	if (fun_visa_viPrintf(&ins, "WFSU SP,0,NP,0,FP,0,SN,0\n", "E002") == -1){
		goto ERR;
	}

	//获取波形数据并保存
	//建议用多线程实现
	//示波器触发频率不宜太频繁(即假设错误发生是偶然的)
	int rts = -1;
	int m = 0;
	char s_vbs_rt[100] = { 0 };
	char s_vbs_wt_ask[100] = { 0 };
	char s_vbs_wt_set[100] = { 0 };
	char filename[30] = { 0 };
	char time_tmp[20] = { 0 };
	char err_str[100] = { 0 };
	int err_times = 0;

	strcat(s_vbs_wt_ask, "VBS? 'Return=app.Acquisition.TriggerMode'\n");
	strcat(s_vbs_wt_set, "VBS 'app.Acquisition.TriggerMode=\"Single\"'\n");
	
	while (1){
		err_times = 0;

		//循环询问示波器是否进入‘stopped’状态
		memset(time_tmp, 0, sizeof(time_tmp));
		get_timeNow(time_tmp, sizeof(time_tmp));		
		plog("\n[%s] Waiting for trigger 'Stopped'...\n", time_tmp);
		while (1){
			memset(time_tmp, 0, sizeof(time_tmp));
			get_timeNow(time_tmp, sizeof(time_tmp));

			memset(err_str, 0, sizeof(err_str));
			strcat(err_str, " [");
			strcat(err_str, time_tmp);
			strcat(err_str, "]ask for trigger 'Stopped'");

			memset(s_vbs_rt, 0, sizeof(s_vbs_rt));
			if(fun_visa_viQuerf(&ins, (unsigned char *)s_vbs_wt_ask, (unsigned char *)s_vbs_rt, err_str) == -1){
				err_times++;
			}
			if (err_times == 5){
				goto ERR;
			}
			if (findstr(s_vbs_rt,"Stopped") >= 0){
				plog("[%s] Get trigger 'Stopped'!\n",time_tmp);
				break;
			}

			Sleep(1000);
		}

		//休息一秒，让示波器显示波形，从而各个缓冲区均有数据
		Sleep(1000);

		//根据Source数量依次获取波形数据并保存
		for (m = 0; m < src_num; m++){
			rts = -1;

			memset(filename, 0, sizeof(filename));
			strcat(filename,time_tmp);
			strcat(filename, "_");
			strcat(filename, source[m]);
			strcat(filename, ".trc");

			rts = fun_main_viGetWaveFromScopeAndSave(&ins, source[m], str_wt_buf, str_rt_buf, filename);
			if (rts == 0) {
				plog("Waveform in source %s has saved into file '%s'!\n", source[m], filename);
			}
			else{
				plog("Waveform data in source %s failed\n",source[m]);
			}

			//需要休息会，让示波器上一个通道缓冲区重置，顺便让计算机写入文件
			Sleep(200);
		}

		err_times = 0;
		
		//重新设置示波器为‘single’状态，注意示波器不宜触发太过频繁
		memset(time_tmp, 0, sizeof(time_tmp));
		get_timeNow(time_tmp, sizeof(time_tmp));
		plog("[%s] Switch trigger into 'Single'...\n", time_tmp);
		while (1){
			memset(time_tmp, 0, sizeof(time_tmp));
			get_timeNow(time_tmp, sizeof(time_tmp));

			memset(err_str, 0, sizeof(err_str));
			strcat(err_str, " [");
			strcat(err_str, time_tmp);
			strcat(err_str, "]switch trigger into 'Single'");

			memset(s_vbs_rt, 0, sizeof(s_vbs_rt));

			fun_visa_viPrintf(&ins,s_vbs_wt_set, err_str);
			Sleep(2000);
			if(fun_visa_viQuerf(&ins, (unsigned char *)s_vbs_wt_ask, (unsigned char *)s_vbs_rt, err_str)==-1){
				err_times++;
			}
			if (err_times == 5){
				goto ERR;
			}
			if (findstr(s_vbs_rt, "Single") >= 0){
				plog("[%s] Get trigger 'Single'!\n", time_tmp);
				break;
			}

			Sleep(1000);
		}

	}

ERR:
	stat = viClose(ins);
	stat = viClose(default_RM);
	fflush(stdin);
	fflush(stdout);
	plog("\nProgram exited.\n");
	return 0;
}
```


## 项目测试

设置示波器的IP，并使得远程计算机和示波器最好在同一网络区域中。打开示波器设置面板，选择VICP方式，可以获得设备IP地址。

在CMD中运行编译好的程序，比如scope.exe。

485的数据通过差分探头接在示波器的C1和C2通道上，触发信号接在C3通道上。
> &gt; scope VICP::192.168.1.133 C1 C2 C3


之后程序就能自动的获取持续的波形了。

（以上过程截图均省略）

> 完

