# Cocoapod Lib Lint & Push Problem

Cocoapod Lint 与 Push时候遇到问题如下

```shell
Encountered an unknown error (Could not find a `ios` simulator (valid values: ). Ensure that Xcode -> Window -> Devices has at least one `ios` simulator listed or otherwise add one.) during validation.
```



Cocoapod 版本 （1.10.1）

卸载 Cocoapod 降级 

首先，通过在终端中运行以下命令来确定已安装的Cocoapods版本：

```shell
gem list --local | grep cocoapods
```

您将看到类似以下的输出：

```
cocoapods (0.27.1, 0.20.2)
cocoapods-core (0.27.1, 0.20.2)
cocoapods-downloader (0.2.0, 0.1.2)
```

在这里，我安装了两个版本的Cocoapods。

要完全删除，请发出以下命令：

```shell
gem uninstall cocoapods
gem uninstall cocoapods-core
gem uninstall cocoapods-downloader
```



安装特定版本

```bash
sudo gem install -n /usr/local/bin  cocoapods -v 1.9.1
```