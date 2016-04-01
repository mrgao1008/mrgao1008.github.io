# exe文件分解

## 分解算法

## PE文件载入步骤

+ 文件Load进入内存		（ReadFile）
+ 申请内存存放不同的节	（placeSection）
+ 载入dll和设置IAT		（loadDll and setIAT）
+ 重定位				（funcReloc）
+ 运行EXE或者DLL		（doEntryPoint）

~~aaa~~