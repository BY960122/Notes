## windows 激活码

- http://fastlinkvpn.vip/

## 禁止U盘复制电脑文件
> 打开运行 -> 输入gpedit.msc -> 本地计算机策略 -> 管理模板 -> 系统 -> 可移动储存访问 -> 在右边找到可移动磁盘拒绝写入访问 -> 双击选择已启用确定,然后再测试一下能否复制电脑文件到U盘

## 电脑关机快捷方式
```sh
# 右击鼠标惦记添加快捷方式,输入
shutdown -s -t 0
# 这里的0代表时间,比如100,那么我们点击之后100秒才会启动关机程序
```

## 完全删除OneDrive
```sh
# 编辑文本文档,拷贝以下内容
@ECHO OFF
%SystemRoot%\SysWOW64\OneDriveSetup.exe /uninstall
RD "%UserProfile%\OneDrive" /Q /S
RD "%LocalAppData%\Microsoft\OneDrive" /Q /S
RD "%ProgramData%\Microsoft OneDrive" /Q /S
RD "C:\OneDriveTemp" /Q /S
REG Delete "HKEY_CLASSES_ROOT\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}" /f
REG Delete "HKEY_CLASSES_ROOT\Wow6432Node\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}" /f
END
# 另存为.cmd所有文件,以管理员方式运行
```

## 家庭版添加组策略 gpedit.msc
```sh
@echo off
pushd "%~dp0"
dir /b C:\Windows\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt
dir /b C:\Windows\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt
for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"C:\Windows\servicing\Packages\%%i"
pause
# 另存为.bat文件,以管理员方式运行
```

## 删除桌面图标小箭头
```sh
# 删除箭头
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 29 /d "%systemroot%\system32\imageres.dll,197" /t reg_sz /f
taskkill /f /im explorer.exe
attrib -s -r -h "%userprofile%\AppData\Local\iconcache.db"
del "%userprofile%\AppData\Local\iconcache.db" /f /q
start explorer

# 复制以上代码到本文,另存为.bat文件,以管理员方式运行

# 恢复箭头
reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 29 /f
taskkill /f /im explorer.exe
attrib -s -r -h "%userprofile%\AppData\Local\iconcache.db"
del "%userprofile%\AppData\Local\iconcache.db" /f /q
start explorer
Pause
```

## 删除此电脑下的文件夹
```sh
# 新建文本文档
Windows Registry Editor Version 5.00
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{f86fa3ab-70d2-4fc7-9c99-fcbf05467f3a}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{d3162b92-9365-467a-956b-92703aca08af}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{B4BFCC3A-DB2C-424C-B029-7FE99A87C641}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{3dfdf296-dbec-4fb4-81d1-6a3438bcf4de}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{088e3905-0323-4b02-9826-5d99428e115f}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{24ad3ad4-a569-4530-98e1-ab02f9417aa8}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{0DB7E03F-FC29-4DC6-9020-FF41B59E513A}]

# 另存为.reg文件,运行
# 同理,恢复方法
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{f86fa3ab-70d2-4fc7-9c99-fcbf05467f3a}]
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{d3162b92-9365-467a-956b-92703aca08af}]
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{B4BFCC3A-DB2C-424C-B029-7FE99A87C641}]
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{3dfdf296-dbec-4fb4-81d1-6a3438bcf4de}]
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{088e3905-0323-4b02-9826-5d99428e115f}]
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{24ad3ad4-a569-4530-98e1-ab02f9417aa8}]
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{0DB7E03F-FC29-4DC6-9020-FF41B59E513A}]
```

## 安装msi时2503,2502错误及其解决
```sh
# 管理员方式运行cmd,然后执行
msiexec /package "D:\Mysoft\Navicat_12.1\sqlncli_x64.msi"
```

## 右下角图标无法点开
```sh
# 可能是误删了一个目录
C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy
```

## 解决端口占用
```sh
# 1.查看端口占用
netstat -ano | findstr "1080"
## TCP    192.168.1.5:1080       27.148.207.206:443     CLOSE_WAIT      7896
# 2.继续查询是哪个程序
tasklist | findstr "7896"
# SearchApp.exe                 7896 Console                    1    146,040 K
# 3.关掉进程
taskkill /T /F /PID 7896
# 或者关掉程序
taskkill /F /IM SearchApp.exe
```

## 软件卸载后,右键菜单仍然存在
```
windows + R 输入: regedit
定位到 HKEY_CLASSES_ROOT\Directory\shell\Open as OneNote Notebook
还可能存在于
HKEY_CLASSES_ROOT\*\shell
```

## cmd 刷新
```sh
ipconfig /flushdns
```

## 完全删除病毒库
```
windows + R 输入: regedit
1.定位到 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SecurityHealthService,右侧找到 start 并双击将 数值数据 由2更改为4
2.定位到 HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender , 右侧添加 DisableAntiSpyware (CWD32) 设置值为 1
```
