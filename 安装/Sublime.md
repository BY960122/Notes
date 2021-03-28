# 下载 Sublime
- https://www.sublimetext.com/3

## Sublime破解
- https://longhujing.github.io/2019/12/14/Sublime-Text3-%E7%A0%B4%E8%A7%A3%EF%BC%88%E6%8C%81%E7%BB%AD%E6%9B%B4%E6%96%B0%EF%BC%89/
- https://hexed.it/
```
## 1.工具栏上找到Open file,上传 sublime_text.exe
## 2.按住Ctrl + f,搜索97 94 0D,配置为00 00 00
## 3.点击工具栏上的Export导出到本机替换原来的 sublime_text.exe
## 4.打开sublime text3,输入激活码
```txt
----- BEGIN LICENSE -----
TwitterInc
200 User License
EA7E-890007
1D77F72E 390CDD93 4DCBA022 FAF60790
61AA12C0 A37081C5 D0316412 4584D136
94D7F7D4 95BC8C1C 527DA828 560BB037
D1EDDD8C AE7B379F 50C9D69D B35179EF
2FE898C4 8E4277A8 555CE714 E1FB0E43
D5D52613 C3D12E98 BC49967F 7652EED2
9D2D2E61 67610860 6D338B72 5CF95C69
E36B85CC 84991F19 7575D828 470A92AB
------ END LICENSE ------

或者
ZYNGA INC.
50 User License
EA7E-811825
927BA117 84C9300F 4A0CCBC4 34A56B44
985E4562 59F2B63B CCCFF92F 0E646B83
0FD6487D 1507AE29 9CC4F9F5 0A6F32E3
0343D868 C18E2CD5 27641A71 25475648
309705B3 E468DDC4 1B766A18 7952D28C
E627DDBA 960A2153 69A2D98A C87C0607
45DC6049 8C04EC29 D18DFA40 442C680B
1342224D 44D90641 33A3B9F2 46AADB8F

或者
----- BEGIN LICENSE -----
Member J2TeaM
Single User License
EA7E-1011316
D7DA350E 1B8B0760 972F8B60 F3E64036
B9B4E234 F356F38F 0AD1E3B7 0E9C5FAD
FA0A2ABE 25F65BD8 D51458E5 3923CE80
87428428 79079A01 AA69F319 A1AF29A4
A684C2DC 0B1583D4 19CBD290 217618CD
5653E0A0 BACE3948 BB2EE45E 422D2C87
DD9AF44B 99C49590 D2DBDEE1 75860FD2
8C8BB2AD B2ECE5A4 EFC08AF2 25A9B864
------ END LICENSE ------

或者
----- BEGIN LICENSE -----
冰忆Dylan
Single User License
EA7E-1237771-449146
8D868314 CF0BD70A C2234109 8352472B
D402DC48 9343C0D9 A8FE5B04 F46EAF67
37D5EC5B B9C40D48 37676A66 C2F10A94
BF312826 506D03B5 38618843 E1321390
1021FD9F 6187D594 7D1BDDE3 33596953
B0E9EE92 A3E521F8 35B01A50 8E1CC4A8
D10B8F67 D20C3AF7 DF34875C 98AD9301
AAFFE843 9C563CFC 77C63DC1 FD3DF1D3
------ END LICENSE ------
```

## Sublime 编译 Java
### 1.直接在安装路径下找到\Packages\Java.sublime-package文件
### 2.用解压缩软件打开,找到JavaC.sublime-build文件,将 shell_cmd 中的javac改成javaRun,保存
### 3. 打开后右键编译,再保存,或者
```json
{
	"shell_cmd": "javaRun \"$file\"",
	"file_regex": "^(...*?):([0-9]*):?([0-9]*)",
	"selector": "source.java",
    "encoding":"cp936"
}
```
### 3.在jdk安装路径下的bin目录中新建一个javaRun.bat批处理文件,内容如下：
```sh
	# Java8
@ECHO OFF
cd %~dp1
::ECHO Compiling %~nx1.......
IF EXIST %~n1.class (
DEL %~n1.class
)
javac -encoding UTF-8 %~nx1
IF EXIST %~n1.class (
::ECHO -----------Result-----------
java %~n1
)
IF EXIST %~n1.class (
DEL *.class
)
	# Java11
@ECHO OFF
cd %~dp1
IF EXIST %~n1.class (
DEL %~n1.class
)
java -Dfile.encoding=UTF-8 %~nx1 
if %ERRORLEVEL% GTR 0 (
echo OFF
)
```
### 4.测试
```txt
写一个 hello world程序
ctrl+B 运行
```

## Sublime 编译 Scala
### 1.选择tools -> build system -> new build system
```json
{
	"shell_cmd": "scala \"$file\"",
	"selector": "source.scala"
}
```
### 2.保存文件名为: Scala.sublime-build

## Sublime 个人配置
### 1.点击菜单栏“Preferences”=> "Settings-User" 进入个人参数设置页面；
### 2.在大括号"｛｝"里面插入下面代码：
```json
{
	"update_check": false,
	"remeber_open_files":false, 
	"word_wrap": "false"
}
```

## Sublime 快捷键
```sh
# 删除行
ctrl+shift+k
# 复制行
ctrl+shift+d
# 查找后全部替换
ctrl+h
ctrl+shift+center
# 批量编辑单词,按2次
ctrl+d
# 多行编辑
ctrl+alt+↑↓
# 跳转到某行
crtl+g
# 查找文件,查找当前文件
ctrl+p,输入@查找当前文件
# 选中括号中的内容
ctrl+shift+m
```

## Sumlime 添加到右键快捷菜单
### 注意文件编码设置为ANSI,内容里面有一个是文件,有一个是目录
```sh
# 把以下代码,复制到SublimeText3的安装目录,然后重命名为：sublime_addright.reg,然后双击就可以了

Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT*\shell\SublimeText]
@="Open with Sublime"
"Icon"="D:\Mysoft\SublimeText3\sublime_text.exe,0"
[HKEY_CLASSES_ROOT*\shell\SublimeText3\command]
@="D:\Mysoft\SublimeText3\sublime_text.exe %1"

[HKEY_CLASSES_ROOT\Directory\shell\SublimeText3]
@="Open with Sublime"
"Icon"="D:\Mysoft\SublimeText3\sublime_text.exe,0"
[HKEY_CLASSES_ROOT\Directory\shell\SublimeText3\command]
@="D:\Mysoft\SublimeText3\sublime_text.exe %1"
```

### 一些实用的Package
- https://github.com/wbond/package_control
- https://packagecontrol.io/
```txt
## 按住 ctrl+shift+P 输入 install 选择 install package
Chinese Localization
Bracket Highlighter
ConvertToUTF8
SFTP

```