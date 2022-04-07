---
title: 记一次 nvm-windows 错误排查
date: 2022-04-07 07:01:53
tags: Debug
excerpt: 因为 nvm 安装后需要对命令行重启以便重新加载环境变量，而 vs code 因为架构的特殊，如果有多个窗口同时运行，仅关闭当前窗户的话 vs code 的命令行环境变量是不会重新加载的。所以在一些需要重启命令行的操作时请确定命令行相关进程彻底结束后再重新启动。
---

## 结论

原因在于没有确认命令行环境是否彻底重启。  
因为 `nvm` 安装后需要对命令行重启以便重新加载环境变量，而 vs code 因为架构的特殊，如果有多个窗口同时运行，仅关闭当前窗户的话 vs code 的命令行环境变量是不会重新加载的。所以在一些需要重启命令行的操作时请确定命令行相关进程彻底结束后再重新启动。  

## 错误表现

在 Windows 命令行下（CMD/PowerShell）运行 `nvm -v` 抛出错误：

```powershell
ERROR open \settings.txt: The system cannot find the file specified
```

## 错误发生的环境

1. 操作系统为 Windows10 20H2，使用本地管理员账户。
2. 通过 **scoop** 安装了 **nvm** v1.1.9。
3. 此时在 vs code 自带的终端抛出了上述错误，简单地重启了当前 vs code 的窗口，依旧发生了同样的错误。

## 排查错误的思路

1. 分析报错信息，大概是是 nvm 的程序在运行时访问某个文件，结果无法找到该文件。先大胆猜测是命令执行位置的问题。
2. 值得一提的是在刚安装完的终端是可以正常运行 `nvm` 命令的。那么我们来对比一下两个终端里执行 `nvm` 命令的位置是否有差别：

   ```powershell
   Get-Command nvm
   ```

   经过对比两边终端的 `nvm` 命令的位置没有区别。

3. 那么接下来推测可能是环境变量的差异，在两边的终端分别执行：

   ```powershell
   Get-ChildItem env:
   ```

   经过对比发现 vs code 的终端少了两个环境变量：

   ```powershell
   NVM_HOME                       C:\Users\PC-21\scoop\apps\nvm\current
   NVM_SYMLINK                    C:\Users\PC-21\scoop\persist\nvm\nodejs\nodejs
   ```

   去到 `C:\Users\PC-21\scoop\apps\nvm\current` 可以找到我们需要的 `\settings.txt` 文件

4. 然后可以分析，之前的重启 vs code 为什么没有让 vs code 的终端重新加载变更后的环境变量。回想后发现我只重启了 vs code 当前工作区窗口，而另一个 vs code 窗口并没有重启。然后我就把所有的 vs code 窗口关闭，保证 vs code 进程已经退出后重新打开。此时进入终端发现环境变量已经更新，`nvm` 命令也能正常执行。

