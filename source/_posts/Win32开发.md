---
title: Win32 API 开发
date: 2023-09-12
tags: 
	- win32
	- Visual Studio
categories:
	- 学习笔记
	- 10-Wechat Hook相关笔记
---

# 调试方法

## DOS窗口调试（推荐）

```cpp
//1.声明全局变量，用来接收标准输出句柄
HANDLE g_hOutput = 0; //接收标准输出句柄

//2.入口添加下面两行代码

AllocConsole(); //为调用进程分配一个控制台
g_hOutput = GetStdHandle(STD_OUTPUT_HANDLE); //获得输出句柄


//3.在需要调试输出地方添加下面三行

char szText[256] = { 0 }; //设置一个数据存储区
sprintf_s(szText,"坐标：x = %d,y = %d\n",xPos,yPos); //把存储数据赋值给szText 
WriteConsole(g_hOutput,szText,strlen(szText),NULL,NULL); //打印到控制台

```

# 消息相关 

## 消息处理优化

```cpp
//消息处理
MSG msg;
while (1) {
    if (PeekMessage(&msg, NULL, 0, 0, PM_NOREMOVE)) {
        if(GetMessage(&msg, NULL, 0, 0)) {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
        else {
            return 0;
        }
        return msg.wParam;
    }
    else {
        //空闲处理
    }
}
```

# 乱码处理

```cpp
string utf8_to_ansi(string strUTF8) {
	UINT nLen = MultiByteToWideChar(CP_UTF8, NULL, strUTF8.c_str(), -1, NULL, NULL);
	WCHAR *wszBuffer = new WCHAR[nLen + 1];
	nLen = MultiByteToWideChar(CP_UTF8, NULL, strUTF8.c_str(), -1, wszBuffer, nLen);
	wszBuffer[nLen] = 0;
	nLen = WideCharToMultiByte(936, NULL, wszBuffer, -1, NULL, NULL, NULL, NULL);
	CHAR *szBuffer = new CHAR[nLen + 1];
	nLen = WideCharToMultiByte(936, NULL, wszBuffer, -1, szBuffer, nLen, NULL, NULL);
	szBuffer[nLen] = 0;
	strUTF8 = szBuffer;
	delete[]szBuffer;
	delete[]wszBuffer;
	return strUTF8;
}

//调用
nickname = "灏戝叺";
char temp[100];
	strcpy_s(temp,strlen(utf8_to_ansi(nickname).c_str())+1, utf8_to_ansi(nickname).c_str());

```

