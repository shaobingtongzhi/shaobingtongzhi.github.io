---
title: Inno Setup 教程
date: 2023-10-22
tags:	
  - Inno Setup
  - 流程制作
  - 桌面程序
categories:
  - 实战
  - EXE流程制作
---



# 认识Inno Setup

## 介绍

一款开源、免费的可以用来制作exe软件安装流程的打包软件！！！

官网地址：[https://jrsoftware.org](https://jrsoftware.org)

特点：

- 支持自 2006 年以来的每个 Windows 版本
- 支持创建**单个 EXE** 来安装程序，以便于在线分发
- 标准窗口向导界面
- 完整的**卸载**功能
- 在任何地方创建快捷方式，包括在“开始”菜单和桌面上
- 创建注册表和 .INI 条目
- 在安装之前、期间或之后运行其他程序
- 支持**多语言**安装
- 集成 Pascal 脚本引擎选项，用于高级运行时安装和卸载自定义

## 下载安装 Inno Setup

下载地址： 

[https://jrsoftware.org/download.php/is.exe](https://jrsoftware.org/download.php/is.exe)

中文语言包：

[https://raw.githubusercontent.com/jrsoftware/issrc/main/Files/Languages/Unofficial/ChineseSimplified.isl](https://raw.githubusercontent.com/jrsoftware/issrc/main/Files/Languages/Unofficial/ChineseSimplified.isl)

***注意*** ：这个中文语言包是生成的软件安装过程语言包，不是Inno Setup的汉化包

放置位置如下图所示：

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231010/放置语言包.y0wpym1lbww.webp)



# 简单使用

## 生成安装过程

图标网站：[https://iconbuddy.app/](https://iconbuddy.app/)

png转ico：[https://www.aconvert.com/cn/icon/png-to-ico](https://www.aconvert.com/cn/icon/png-to-ico/)

步骤：

1. 使用脚本向导创建一个新的脚本
2. 下一步。。。

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231010/选择语言包.rltatgkgllc.webp)

注意：这次是可以选择中文语言包了

***BUG 1***

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231010/乱码.1cgekw0bftnk.webp)

***解决方案***

用记事本查看语言包`ChineseSimplified.isl`的保存格式，将保存格式设置为 `ANSI` 即可

## 几个重要信息的设置

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231010/配置安装信息.12ky74ym1lg0.webp)

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231010/应用名称发布者.58qfd7buhf40.webp)

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231010/设置默认安装目录.6jdfntw2jx80.webp)

## 修改安装包图标

图标中至少包含以下尺寸：16x16、32x32、48x48、64x64和256x256。

如果未指定该指令或该指令为空，则将使用支持上述大小的内置图标

```isl
[Setup]
;安装包图标
SetupIconFile=MyProgSetup.ico
```



## 修改安装过程中的图片

```isl
[Setup]
;安装向导样式，有两个值：classic 、modern 分别表示 经典、现代
WizardStyle=classic
;安装向导左侧图片 大小要求：164 x 314
WizardImageFile=WizClassicImage.bmp 
;安装向导右侧小图片 大小要求：55 x 55
WizardSmallImageFile=WizClassicSmallImage-IS.bmp
```

# 进阶

>  场景一：我们开发的软件不只有exe文件，还包含了一些资源文件（图片、音频等）、dll动态库文件等等，那我们就需要把这些东西也一并打包

>  场景二：我们开发了一款软件，这个软件的运行依赖与另外一个指定版本的软件，所以就需要实现在安装我们这个软件之前，先安装好另外一个软件

Pascal脚本功能（类似Pascal的现代Delphi）为在运行时自定义安装或卸载添加了许多新的可能性。

- 支持在自定义条件下中止安装程序或卸载启动。

- 支持在运行时向安装程序添加自定义向导页面。

- 支持在安装之前、期间或之后从Pascal脚本中提取和调用DLL或其他文件。

- 支持可以执行任何操作的脚本常量-普通常量、从注册表读取、从ini读取和从命令行读取常量可以执行更多操作。

- 支持在自定义条件下运行时删除类型、组件和/或任务。

- 支持根据自定义条件有条件地安装[Files]、[Registry]、[Run]等条目。

- Pascal脚本有很多支持功能，几乎是Inno Setup本身所能做的一切。

## Pascal语法基础

```pascal
;注释使用英文封号
;全局变量
var
  变量名称: 变量类型;
;常量
const 
  常量名称 =  '常量内容';
function 函数名称（形参:形参类型,...）:返回值类型
begin
end;

;if结构
if xxx then
begin
内容
end else begin
;为假时
end;

if xxx then
begin
内容
end;
```

## [Files]项

[Files]项内定义安装程序要在用户系统上安装的任何文件

```pascal
[Files]
Source: "D:\Program Files (x86)\Inno Setup 6\Examples\{#MyAppExeName}"; DestDir: "{app}"; Flags: ignoreversion
```

- Source：必填项，源文件名称，可以是一个通配符，用于在单个条目中指定一组文件

- DestDir：必填项，文件要安装在用户系统上的目录，几乎总是以其中一个目录常量开头。如果指定的路径在用户的系统中不存在，则会自动创建该路径，如果为空，则会在卸载过程中自动删除该路径。

- Flags：扩展参数，多个项可以通过空格分隔

  - deleteafterinstall：指示安装程序照常安装文件，但在安装完成（或中止）后将其删除。这对于提取在脚本的[运行]部分执行的程序所需的临时数据非常有用，**注意：不能与isreadme、regserver、regtypelib、restartreplace、sharedfile或uninsoveruninstall标志组合使用**

  - ignoreversion：不比较版本信息直接替换现有文件，无论其版本号如何，**注意：此标志只能用于应用程序专用的文件，而不能用于共享系统文件**

  - isreadme：标明是“自述”文件。安装中只有一个文件可以具有此标志。当文件具有此标志时，用户将询问他们是否希望在安装完成后查看自述文件。如果选择“是”，安装程序将使用文件类型的默认程序打开该文件。因此，README文件应该始终以.txt、.wri或.doc等扩展名结尾。

    **请注意，如果安装程序必须重新启动用户的计算机（由于安装了带有restartreplace标志的文件，或者如果AlwaysStart[Setup]部分指令为yes），则用户将无法查看自述文件**

## [code]项

[Code]部分是一个可选部分，用于指定Pascal脚本。Pascal脚本可以通过多种方式用于自定义安装或卸载。请注意，创建Pascal脚本并不容易，需要Inno Setup的经验和Pascal或至少类似编程语言的编程知识。

### Event Functions

#### InitializeSetup()

```pascal
function InitializeSetup(): Boolean;
Called during Setup's initialization. Return False to abort Setup, True otherwise.
;在安装程序初始化期间调用。返回False可中止安装程序，否则返回True。

```

### **Registry functions**

#### RegKeyExists()

```pascal
function RegKeyExists(const RootKey: Integer; const SubKeyName: String): Boolean;
;Returns True if the specified registry key exists.
;如果指定的注册表项存在，则返回True。

Example:
begin
  if RegKeyExists(HKEY_AUTO, 'Software\Jordan Russell\Inno Setup') then
  begin
    // The key exists
  end;
end;
```

#### RegValueExists()

```pascal
function RegValueExists(const RootKey: Integer; const SubKeyName, ValueName: String): Boolean;
;Returns True if the specified registry key and value exist.
;如果指定的注册表项和值存在，则返回True。

Example:
begin
  if RegValueExists(HKEY_CURRENT_USER, 'Console', 'WindowSize') then
  begin
    // The value exists
  end;
end;
```

#### RegGetSubkeyNames()

```pascal
function RegGetSubkeyNames(const RootKey: Integer; const SubKeyName: String; var Names: TArrayOfString): Boolean;

;Opens the specified registry key and reads the names of its subkeys into the specified string array Names. Returns True if successful, False otherwise.
;打开指定的注册表项，并将其子项的名称读取到指定的字符串数组names中。如果成功，则返回True，否则返回False。

Example:
var
  Names: TArrayOfString;
  I: Integer;
  S: String;
begin
  if RegGetSubkeyNames(HKEY_CURRENT_USER, 'Control Panel', Names) then
  begin
    S := '';
    for I := 0 to GetArrayLength(Names)-1 do
      S := S + Names[I] + #13#10;
    MsgBox('List of subkeys:'#13#10#13#10 + S, mbInformation, MB_OK);
  end else
  begin
    // add any code to handle failure here
  end;
end;
```

#### RegGetValueNames()

```pascal
function RegGetValueNames(const RootKey: Integer; const SubKeyName: String; var Names: TArrayOfString): Boolean;

;Opens the specified registry key and reads the names of its values into the specified string array Names. Returns True if successful, False otherwise.

;打开指定的注册表项，并将其值的名称读取到指定的字符串数组names中。如果成功，则返回True，否则返回False。
Example:
var
  Names: TArrayOfString;
  I: Integer;
  S: String;
begin
  if RegGetValueNames(HKEY_CURRENT_USER, 'Control Panel\Mouse', Names) then
  begin
    S := '';
    for I := 0 to GetArrayLength(Names)-1 do
      S := S + Names[I] + #13#10;
    MsgBox('List of values:'#13#10#13#10 + S, mbInformation, MB_OK);
  end else
  begin
    // add any code to handle failure here
  end;
end;
```

#### RegQueryStringValue()

```pascal
function RegQueryStringValue(const RootKey: Integer; const SubKeyName, ValueName: String; var ResultStr: String): Boolean;
;Queries the specified REG_SZ- or REG_EXPAND_SZ-type value, and returns the data in ResultStr. Returns True if successful. When False is returned, ResultStr is unmodified.
;查询指定的REG_SZ或REG_EXPAND_SZ类型的值，并在ResultStr中返回数据。如果成功，则返回True。当返回False时，ResultStr是未修改的。
Example:
var
  Country: String;
begin
  if RegQueryStringValue(HKEY_CURRENT_USER, 'Control Panel\International',
     'sCountry', Country) then
  begin
    // Successfully read the value
    MsgBox('Your country: ' + Country, mbInformation, MB_OK);
  end;
end;
```

### File funcions

#### Exec()

```pascal
function Exec(const Filename, Params, WorkingDir: String; const ShowCmd: Integer; const Wait: TExecWait; var ResultCode: Integer): Boolean;

;执行可执行文件
Example:
var
  ResultCode: Integer;
begin
  // Launch Notepad and wait for it to terminate
  if Exec(ExpandConstant('{win}\notepad.exe'), '', '', SW_SHOW,
     ewWaitUntilTerminated, ResultCode) then
  begin
    // handle success if necessary; ResultCode contains the exit code
  end
  else begin
    // handle failure if necessary; ResultCode contains the error code
  end;
end;
```



#### ExtractTemporaryFile()

```pascal
procedure ExtractTemporaryFile(const FileName: String);

;将指定的文件从[Files]部分提取到临时目录中。要查找临时目录的位置，请使用ExpandConstant（'｛tmp｝'）。
;当安装程序退出时，提取的文件将自动删除。

Example:
[Files]
Source: "Readme.txt"; Flags: dontcopy noencryption

[Code]
function InitializeSetup: Boolean;
var
  S: AnsiString;
begin
  // Show the contents of Readme.txt (non Unicode) in a message box
  ExtractTemporaryFile('Readme.txt');
  if LoadStringFromFile(ExpandConstant('{tmp}\Readme.txt'), S) then
  begin
    MsgBox(S, mbInformation, MB_OK);
  end;

  Result := True;
end;
```

### Dialog functions

#### MsgBox()

```pascal
function MsgBox(const Text: String; const Typ: TMsgBoxType; const Buttons: Integer): Integer;
;消息框
;TMsgBoxType is defined as:
;TMsgBoxType = (mbInformation, mbConfirmation, mbError, mbCriticalError);
;Supported flags for Buttons are:
;MB_OK, MB_OKCANCEL, MB_ABORTRETRYIGNORE, MB_YESNOCANCEL, MB_YESNO, MB_RETRYCANCEL, ;MB_DEFBUTTON1, MB_DEFBUTTON2, MB_DEFBUTTON3, MB_SETFOREGROUND


```

### Setup Logging functions

#### Log()

```
procedure Log(const S: String);
;日志
;将指定的字符串记录在安装程序的日志文件中，需要 [Setup]项中的 SetupLogging 属性设置为yes
[Setup]
SetupLogging=yes;
```

## 关键代码

```pascal
[Code]
//1. 检测有没有安装微信,没有就安装，否则 进行下一步
//2. 检测安装的微信版本 是不是 2.6.8.52, 是的话就 直接安装软件 ，否则 提示用户安装指定版本
var
  Names: TArrayOfString;
  S: String;
  I: Integer;
  ResultCode: Integer;

const 
  NoInstallMsg =  '检测到当前系统未安装微信'#13#10'安装选择 是，不安装选择 否';
  NoSameMsg = '检测到当前系统安装版本与软件支持版本不一致'#13#10'安装选择 是，不安装选择 否';
  NeedVersion = '2.6.8.52';
  //NeedVersion = '3.2.1.154';
//自定义执行程序

//检测安装结果

function CheckInstallResult():boolean;
var 
  Version: String;
begin
  Result := true;
  if not RegKeyExists(HKEY_LOCAL_MACHINE,'SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\WeChat') then
  begin
    Result := false;
    Exit;
  end;
  if not RegQueryStringValue(HKLM,'SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\WeChat','DisplayVersion',Version) then 
  begin
    Result := false;
    Exit;
  end;
  MsgBox('检测到的版本：' + Version + #13#10 +  '需要安装的版本：' + NeedVersion ,mbInformation,MB_OK);
  if Version <> NeedVersion then 
  begin
    //MsgBox('不相等，终止',mbInformation,MB_OK);
    Result := false;
    Exit;
  end;
end;

//安装第三方软件
function InstallWeChat(Msg: String):boolean;
begin
  if MsgBox(Msg,mbConfirmation,MB_YESNO) = IDYES then
  begin
     //MsgBox('安装、、、',mbInformation,MB_OK)
     //1. 提取执行程序到临时目录
      ExtractTemporaryFile('{#InstallExeName}');
     //2. 执行可执行程序
      if Exec(ExpandConstant('{tmp}\{#InstallExeName}'), '', '', SW_SHOW,
         ewWaitUntilTerminated, ResultCode) then
      begin
        // handle success if necessary; ResultCode contains the exit code
        //MsgBox('ResultCode:' + IntToStr(ResultCode) + '  ' + SysErrorMessage(ResultCode) ,mbInformation,MB_OK);
        //MsgBox('ResultCode:' + SysErrorMessage(ResultCode) ,mbInformation,MB_OK);
        //todo 通过注册表检测 安装的版本，符合要求就 返回 true ,否则返回 false
        Result := CheckInstallResult();
      end
      else begin
        // handle failure if necessary; ResultCode contains the error code
        MsgBox('安装失败，ResultCode:' + SysErrorMessage(ResultCode) ,mbInformation,MB_OK);
        Result := false;
      end;
  end else
  begin
    //选择了否
    Result := false;
    //MsgBox('安装失败，安装程序将退出，谢谢使用',mbInformation,MB_OK);
  end;
end;

//初始化安装程序
function InitializeSetup(): Boolean;
begin
  Result := true;
  if not RegKeyExists(HKEY_LOCAL_MACHINE,'SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\WeChat') then
  begin
    //未安装微信，执行安装操作
    Result := InstallWeChat(NoInstallMsg);
  end;
  if not RegQueryStringValue(HKLM,'SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\WeChat','DisplayVersion',S) then 
  begin
    MsgBox('程序异常，退出安装',mbInformation,MB_OK);
    Result := false;  
    Exit; 
  end;
  if S <> NeedVersion then 
  begin
    //版本不一致，执行安装程序
    Result := InstallWeChat(NoSameMsg);
  end;
  if Result = false then
    MsgBox('安装失败，安装程序将退出，谢谢使用',mbInformation,MB_OK);
  Log('222');
end;
```

