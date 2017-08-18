![](/assets/QRONZ-logo.png)

# IDF-API文档

![Release version](https://img.shields.io/badge/release-v1.0.0-blue.svg)

## RFID 系列设备驱动封装，提供网络和串口操作指令API

* [版本说明](/ban-ben-shuo-ming.md)
* [教程](/jiao-cheng.md)
* [IDF-U4](/idf-u4.md)
* * [控制器](/idf-u4/kong-zhi-qi-chuang-jian.md)
  * [系统设置操作](/idf-u4/can-shu-she-zhi.md)
  * [ISO 18000-6C标签操作](/idf-u4/iso-18000-6cbiao-qian-cao-zuo.md)
  * [错误码](/idf-u4/cuo-wu-ma.md)

## Build状态

| Windows | Linux | Android |
| :---: | :---: | :---: |
| ![](https://img.shields.io/badge/build-passing-brightgreen.svg) | ![](https://img.shields.io/badge/build-never built-lightgrey.svg) | ![](https://img.shields.io/badge/build-never built-lightgrey.svg) |

## 简介

IDF-API是基于C++封装的类C接口设备驱动库。

* IDF-API封装成类C接口Dll，可以给C，C++，C\#，Java，Delphi，Python等不同编程语言调用。
* IDF-API独立，不依赖与其他formwork和支持库。
* IDF-API兼容Window xp及以上系统。（后续将支持Linux等其他操作系统）
* IDF-API在同一进程内可以实现255个设备控制器。

## 安装

IDF-API需要将hofonidfdriver.dll复制到生成目录下。

* 对于c和c++项目，需要将hofonidfdriver.h复制到项目include目录中。如果是静态导入，可以包含hofonidfdriver.lib。
* 对于其他语言，可以参考相应demo实现API的调用。

## 用法一览

此简单例子实现一个控制器的创建，打开，以及EPC等信息的实时读取。

```c
#include <stdio.h>
#include "hofonidfdriver.h"

void printHex(uint8_t * buffer, int len)
{
	for (int i = 0; i < len; i++)
	{
		printf("%02X ", buffer[i]);
	}
}

void __cdecl real_time_inventory_callback(int id, InventoryData_t data, void* arg)
{
	printf("Real read ");
	printf("PC: ");
	printHex(data.Pc, 2);
	printf("EPC: ");
	printHex(data.Epc, data.EpcLen);
	printf("AntId:%d Freq:%f RSSI:%ddbm", data.AntId, data.Freq, data.Rssi);
	printf("\r\n");
}

void __cdecl real_time_complete_callback(int id, BOOL result, uint8_t antenna_number, uint16_t read_rate, int total_read, void* arg)
{
	if (result)
	{
		printf("Read complete\r\n");
		printf("AntNo:%d,ReadRate:%d,TotalRead:%d\r\n", antenna_number, read_rate, total_read);
		printf("Press enter continue,other exit\r\n");
	}
	else
	{
		printf("Read failed\r\n");
		enum ReaderErrorCode code = IDFU4_GetLastErrorCode(id);
		printf("Last error code:%d\r\n", code);
		printf("Press enter continue,other exit\r\n");
	}
}

int main()
{
	printf("This is a idf-u4 c demo\r\n");
	int id = IDFU4_CreateController(TcpClient, "192.168.0.10", 23443);
	if (IDFU4_Open(id))
	{
		printf("Open controller success\r\n");
	}
	else
	{
		printf("Open controller failed\r\n");
	}
	do 
	{
		IDFU4_RealTimeInventory(id, 1, real_time_inventory_callback, real_time_complete_callback, NULL);
	} while (getchar() == '\n');
	IDFU4_Close(id);
	IDFU4_DestroyController(id);
	return 0;
}
```



