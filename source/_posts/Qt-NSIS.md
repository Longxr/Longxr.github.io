---
title: Qt-NSIS
categories:
  - Qt
tags:
  - Qt
  - NSIS
date: 2018-07-02 17:19:02
---

<Excerpt in index | 摘要> 
用NSIS打包Qt程序<!-- more -->
<The rest of contents | 余下全文>
	
> 写好了程序之后要关心的就是怎么将程序打包成一个安装包发给用户了，这里就推荐一个功能比较全面的打包脚本NSIS。在一般情况下，只要不是特别傻逼的需求 (纠结安装包字符串显示的大爷惹不起惹不起....) 都是能实现的。

## Qt程序处理
### 编译程序的release版
1.IDE版
构建方式换成Release，点击Qt Creator的运行按钮

2.命令行版(方便自动化构建)
```
set PATH=c:\Qt\Qt5.7.0\5.7\mingw53_32\bin;%PATH%
set PATH=c:\Qt\Qt5.7.0\Tools\mingw530_32\bin;%PATH%
cd %~dp0
mingw32-make.exe clean
qmake.exe Demo.pro -spec win32-g++ "CONFIG+=release" && mingw32-make.exe qmake_all
mingw32-make.exe -j4
```

### 查找Qt程序需要的动态库文件
直接将Qt的release目录拷贝到一个没装Qt的电脑里，会发现无法运行，这是因为有的Qt核心的动态库并不会在构建的时候放到该目录下，需要额外添加，Qt给我们提供了一个方便的工具来查找程序需要的相关dll，windeployqt.exe。
1. widget版：
```
set PATH=c:\Qt\Qt5.7.0\5.7\mingw53_32\bin;%PATH%
set PATH=c:\Qt\Qt5.7.0\Tools\mingw530_32\bin;%PATH%
windeployqt.exe Demo.exe --dir %PRODUCT_DIR%
```
2.含有qml界面版：
```
set PATH=c:\Qt\Qt5.7.0\5.7\mingw53_32\bin;%PATH%
set PATH=c:\Qt\Qt5.7.0\Tools\mingw530_32\bin;%PATH%
windeployqt.exe --qmldir   %QML_FILE_DIR% Demo.exe  --dir %PRODUCT_DIR%
```
3.莫名其妙缺少dll版：
上面通过windeployqt.exe查找后还是缺失的话，直接去Qt放dll的目录，把所有dll拷贝到release目录下，把你的程序运行起来，然后删除所有dll，提示占用不能删除留下来的就是你需要的。

### NSIS打包
NSIS有界面化的工具HM VNISEdit，不过一般都得改，就直接用文本编辑器手敲好了。可以先用工具生成一个默认的模板看看，下面主要讲下特殊需求怎么通过脚本实现。

1.安装/卸载程序互斥：同时只能开启一个安装/卸载程序：
```
;安装程序互斥
!define Mutex_Install     "Demo_Mutex_Install"
!define Mutex_UnInstall   "Demo_Mutex_UnInstall"


Function CreateMutex
;检查安装互斥
ReCheck:
System::Call 'kernel32::CreateMutexA(i 0, i 0, t "${Mutex_Install}") i .R1 ?e'
Pop $R0
System::Call 'kernel32::CloseHandle(i R1) i.s'
;检查卸载互斥
System::Call 'kernel32::CreateMutexA(i 0, i 0, t "${Mutex_UnInstall}") i .R3 ?e'
Pop $R2
System::Call 'kernel32::CloseHandle(i R3) i.s'
;判断安装/卸载互斥的存在
${If} $R0 != 0
MessageBox MB_RetryCancel|MB_ICONEXCLAMATION "$(MutexInstallMessage)" IdRetry ReCheck
Quit
${ElseIf} $R2 != 0
MessageBox MB_RetryCancel|MB_ICONEXCLAMATION "$(MutexUninstallMessage)" IdRetry ReCheck
Quit
${Else}
;创建安装互斥
System::Call 'kernel32::CreateMutexA(i 0, i 0, t "${Mutex_Install}") i .R1 ?e'
Pop $R0
StrCmp $R0 0 +2
Quit
${EndIf}
FunctionEnd

Function Un.CreateMutex
;检查安装互斥
ReCheck:
System::Call 'kernel32::CreateMutexA(i 0, i 0, t "${Mutex_Install}") i .R1 ?e'
Pop $R0
System::Call 'kernel32::CloseHandle(i R1) i.s'
;检查卸载互斥
CheckUnInstall:
System::Call 'kernel32::CreateMutexA(i 0, i 0, t "${Mutex_UnInstall}") i .R3 ?e'
Pop $R2
System::Call 'kernel32::CloseHandle(i R3) i.s'
;判断安装/卸载互斥的存在
${If} $R0 != 0
MessageBox MB_RetryCancel|MB_ICONEXCLAMATION "$(MutexInstallMessage)" IdRetry ReCheck
Quit
${ElseIf} $R2 != 0
MessageBox MB_RetryCancel|MB_ICONEXCLAMATION "$(MutexUninstallMessage)" IdRetry ReCheck
Quit
${Else}
;创建卸载互斥
System::Call 'kernel32::CreateMutexA(i 0, i 0, t "${Mutex_UnInstall}") i .R1 ?e'
Pop $R0
StrCmp $R0 0 +2
Quit
${EndIf}
FunctionEnd

Function .onInit
Call CreateMutex
...
FunctionEnd
```

2. 检测所安装的程序是否有在运行，在的话弹个框提示杀进程：
```
...
 Push $R0
 CheckProc:
	Push "${PRODUCT_MAIN_APP_EXE}"
	ProcessWork::existsprocess
	Pop $R0
	IntCmp $R0 0 Done
	; MessageBox MB_OKCANCEL|MB_ICONSTOP $(InstallWarningMessage) IDCANCEL Exit
	MessageBox MB_OKCANCEL|MB_ICONSTOP "$(InstallWarningMessage)" IDCANCEL Exit
	Push "${PRODUCT_MAIN_APP_EXE}"
	Processwork::KillProcess
	Sleep 1000
	Goto CheckProc
 Exit:
	 Abort
 Done:
	Pop $R0
...
```

3. 检测是否安装过旧版本，有的话提示卸载：
```
...
var /GLOBAL UNINSTALL_PROG
var /GLOBAL OLD_VER
var /GLOBAL OLD_PATH
	
ClearErrors
  ReadRegStr $UNINSTALL_PROG ${PRODUCT_UNINST_ROOT_KEY} ${PRODUCT_UNINST_KEY} "UninstallString"
  IfErrors  FinishInit
  
  ReadRegStr $OLD_VER ${PRODUCT_UNINST_ROOT_KEY} ${PRODUCT_UNINST_KEY} "DisplayVersion"
  MessageBox MB_YESNOCANCEL|MB_ICONQUESTION "$(Uninstall_Old_Message)" /SD IDYES IDYES UninstallOld IDNO FinishInit
  Abort
  
UninstallOld:
  StrCpy $OLD_PATH $UNINSTALL_PROG -10

  ExecWait '"$UNINSTALL_PROG" /S _?=$OLD_PATH' $0
  DetailPrint "uninst.exe returned $0"
  Delete "$UNINSTALL_PROG"
  RMDir $OLD_PATH

FinishInit:
...
```

4. 安装包多语言：
```
; 语言选择窗口常量设置
!define MUI_LANGDLL_REGISTRY_ROOT "${PRODUCT_UNINST_ROOT_KEY}"
!define MUI_LANGDLL_REGISTRY_KEY "${PRODUCT_UNINST_KEY}"
!define MUI_LANGDLL_REGISTRY_VALUENAME "NSIS:Language"

!define MUI_LANGDLL_WINDOWTITLE "$(LangDialog_Title)"
!define MUI_LANGDLL_INFO "$(LangDialog_Text)"
!define MUI_LANGDLL_ALLLANGUAGES

; 安装界面包含的语言设置
!insertmacro MUI_LANGUAGE "English" ;first language is the default language
!insertmacro MUI_LANGUAGE "SimpChinese"


; 安装预释放文件
!insertmacro MUI_RESERVEFILE_LANGDLL
;!insertmacro MUI_RESERVEFILE_INSTALLOPTIONS

Function .onInit
 !insertmacro MUI_LANGDLL_DISPLAY 

FunctionEnd
```

5. 界面美化
如果制作的安装包不需要多语言的设置的话，可以通过一些别人写好的插件进行界面美化，如：[nsNiuniuSkin](http://www.ggniu.cn/download.htm#)
下载下来就带了2个demo，改改就能自己用了，比较方便。Demo图示：
![Demo1.png](https://upload-images.jianshu.io/upload_images/2756183-820580357046cb8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Demo2.png](https://upload-images.jianshu.io/upload_images/2756183-0492847b8884378a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



