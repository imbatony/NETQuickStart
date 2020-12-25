## 下载和安装

本文介绍如何在 Windows 上安装 .NET。 .NET 由运行时和 SDK 组成。 运行时用于运行 .NET 应用，应用可能包含也可能不包含它。 SDK 用于创建 .NET 应用和库。 .NET 运行时始终随 SDK 一起安装。
最新版本的 .NET 是 5.0。

[下载](https://download.visualstudio.microsoft.com/download/pr/acff3e6a-d8d6-4c2a-a0cb-1853b58055cc/7910b2a414caa17d30b0cb82583cb542/dotnet-sdk-5.0.101-win-x64.exe)

### ## 检查下载是否正确

当下载完成后，创建一个命令行，运行以下命令

```bash
dotnet
```

如果安装成功则会出现以下提示

```	bash
Usage: dotnet [options]
Usage: dotnet [path-to-application]

Options:
-h|--help         Display help.
--info            Display .NET information.
--list-sdks       Display the installed SDKs.
--list-runtimes   Display the installed runtimes.

path-to-application:
The path to an application .dll file to execute.
```

