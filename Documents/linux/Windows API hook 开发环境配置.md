# Windows API Hook 开发环境配置
<span id="top"></span>
<!-- MarkdownTOC -->

- [Detours安装和配置](#detours安装和配置)
	- [Detours介绍](#detours介绍)
	- [Detours安装](#detours安装)
	- [Detours编译](#detours编译)
- [Windows API Hook例子](#windows-api-hook例子)
	- [加入detours头文件和动态库文件](#加入detours头文件和动态库文件)
- [利用withdll进行远程dll注入](#利用withdll进行远程dll注入)

<!-- /MarkdownTOC -->

<a name="detours安装和配置"></a>
## Detours安装和配置

<a name="detours介绍"></a>
### Detours介绍

Detours是Windows下一个API Hook 开源框架。
[Detours官网][]
[Detours 3.0下载地址][]

<a name="detours安装"></a>
### Detours安装

msi文件一路next。记下安装目录。

<a name="detours编译"></a>
### Detours编译

安装完成后，进入目录`C:\Program Files\Microsoft Research\Detours Express 3.0`会看到下面的目录结构
```
	samples\
		...
		simple\
		...
		withdll\
		...
	src\
	...
```
其中src是需要编译的detours源码。simple和withdll是实现windows API Hook需要用到的2个示例。

编译步骤如下:
+ 以管理员权限启动cmd
+ 设置环境变量，不然nmake会出错
```
# "C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\vcvars32.bat"
```
+ 进入detours的src文件夹并进行编译
```
# cd C:\Program Files (x86)\Microsoft Research\Detours Express 3.0\src
# nmake
```
编译在src和samples同一目录会生成lib.X86文件夹，里面的detours.lib就是我们需要的。

<a name="windows-api-hook例子"></a>
## Windows API Hook例子

<a name="加入detours头文件和动态库文件"></a>
### 加入detours头文件和动态库文件

在vs中新建windows控制台程序，设置解决方案的属性。
+ 打开**项目**--**XXX 属性**
+ **配置属性**--**C/C++**--**常规** 在**附加目录项**中加入
```
C:\Program Files (x86)\Microsoft Research\Detours Express 3.0\include
```
+ **配置属性**--**链接器**--**常规** 在**附加库目录**中加入
```
C:\Program Files (x86)\Microsoft Research\Detours Express 3.0\lib.X86
```
+ **配置属性**--**链接器**--**输入** 在**附加依赖项**中加入`detours.lib`
+ **配置属性**--**C/C++**--**代码生成** 在**运行库**选择`/MT`
+ **配置属性**--**常规** 在**字符集**选择`未设置`
+ 在命令栏中选择`release`版本

#### withdll的编译

在源文件中新建cpp文件，拷贝detours安装目录下的`samples\withdll\withdll.cpp`内容到cpp中。编译。

#### simple的编译

在vs新建项目时，选择windows控制台程序，并选择dll。设置解决方案属性。拷贝`samples\simple\simple.cpp`到新建的cpp文件中，编译。

<a name="利用withdll进行远程dll注入"></a>
## 利用withdll进行远程dll注入

`withdll /d:simple.dll a.exe`会将simple.dll注入到a.exe中。注意`/d:simple.dll`分号后不要有空格。exe和dll用绝对地址和相对地址都可以。

[回到顶部](#top)

[Detours官网]:http://research.microsoft.com/en-us/projects/detours/
[Detours 3.0下载地址]:http://research.microsoft.com/en-us/downloads/d36340fb-4d3c-4ddd-bf5b-1db25d03713d/default.aspx