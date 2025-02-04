# FOKS-TROT

## minifilter透明加解密过滤驱动

## 引言：  
本项目是个实验性项目，且作者对于文件系统等的理解难免会存在偏差，因此可能会产生误导，望读者辩证的学习，并且请读者遵循相关的开源协议。  
因为之前写过一个minifilter的透明加密解密驱动，但当时水平确实有限，有很多的问题，没有找到原因，只是进行了规避，导致在错误的基础上又产生了错误，所以在之前项目开发经验的基础上，写了这个项目。  
这个项目也打算作为毕设，如有雷同，纯属雷同s(-__-)b  

## 简介：  
本项目是一个使用minifilter框架的透明加密解密过滤驱动，当进程有写入特定的文件扩展名（比如txt，docx）文件的倾向时自动加密。授权进程想要读取密文文件时自动解密，非授权进程不解密，显示密文，且不允许修改密文，这里的加密或解密只针对NonCachedIo。桌面端也可以发送特权加密和特权解密命令，实现单独加密或解密。  
1.本项目使用双缓冲，授权进程和非授权进程分别使用明文缓冲和密文缓冲；  
2.使用StreamContext存放驱动运行时的文件信息，使用文件标识尾的方式，在文件的尾部4KB储存文件所需的解密信息；  
3.使用AES 128-ECB模式，并且使用密文挪用（Ciphertext stealing）的方法，避免明文必须分块对齐(padding)的问题；  
4.Write和Read使用SwapBuffers的方式进行透明加密解密；  
5.特权加密和特权解密使用重入（Reentry）的方式，使驱动加密解密文件；  
6.解决FileRenameInformationEx和FileRenameInformation问题，因此可以自动加密解密docx，doc，pptx，ppt，xlsx，xls等使用tmp文件重命名方式读写的文件；  

## 编译及使用方法：  
1.安装CNG库：  
https://www.microsoft.com/en-us/download/details.aspx?id=30688  
需要在微软官网下载Cryptographic Provider Development Kit,  
项目->属性的VC++目录的包含目录，库目录设置相应的位置  
链接器的常规->附加库目录C:\Windows Kits\10\Cryptographic Provider Development Kit\Lib\x64  
输入->附加依赖项一定要设置为ksecdd.lib  
2.在Utils.c-> PocBypassIrrelevantFileExtension设置要过滤的文件扩展名，->PocIsUnauthorizedProcess设置非授权进程  
3.使用Visual Studio 2019编译Debug x64驱动，编译User和UserDll  
4.建议在Windows 10 x64，NTFS环境运行（这里主要是FltFlushBuffers2只支持NTFS），

# 其他实现细节在开发文档.pdf中