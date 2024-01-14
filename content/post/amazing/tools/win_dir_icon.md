---
title: "Windows 文件夹显示自定义"
date: 2023-10-26T11:20:58+08:00
draft: false

description: "Win 系统中的文件夹自定义高级操作"
---

只为了美化，Win 中的文件夹 ICON 可以自由设定，在 desktop.ini 文件中修改，如

## 自定义文件夹图片

可以读取 Resource DLL 资源文件中的图片

```ini
[.ShellClassInfo]
IconResource=C:\WINDOWS\System32\SHELL32.dll,6
```

`IconResource` 可以直接赋值为文件地址，如果为图片文件，资源 ID 必须为 0

```ini
[.ShellClassInfo]
IconResource=D:\icon.png,0
```

## 自定义文件夹名称

```ini
[.ShellClassInfo]
LocalizedResourceName=My Code
# 不支持中文
```

也可以使用 DLL 资源中的 string 资源

```ini
[.ShellClassInfo]
LocalizedResourceName=@%SystemRoot%\system32\shell32.dll,-21769
```

注意前面的 @ 符号，没有 @ 符号表示单纯字符串

## 自定义 Resource DLL

```rc
1 ICON "resource/favicon.ico"

STRINGTABLE
{
    2, "My Dir Name"
}
```

使用 RC 文件定义各种[资源](https://learn.microsoft.com/en-us/windows/win32/menurc/resource-definition-statements)，

ICONID 1 为自定义资源 id，ICON 为关键字，后跟资源文件位置。注意 ico 文件不能为简单的 图片文件，需要有 ico 元数据，它是一个包含 64x64, 32x32, 16x16 的规格的 ico 包, 可以使用[在线工具](https://www.xiconeditor.com/) 生成 ico 文件

STRINGTABLE 是字符资源

可以使用 MingGW 工具 `windres` 编译资源

```shell
windres -o myres.o myres.rc
ld --dll -o myres.dll myres.o
```

### 使用自定义 DLL

```ini
[.ShellClassInfo]
IconResource=D:\myres.dll,-1
LocalizedResourceName=@D:\myres.dll,-2
```
