# 卸载 Cocoapods

```shell
#!/bin/zsh
cd `dirname $0`

echo "先卸载目前所有cocoapods相关"

#由于上次安装，可能是用不同的权限，所以多卸载一下，fml
gem list  cocoapods xcodeproj | grep -E 'cocoapods' | awk '{print $1}' | xargs -I {} sudo gem uninstall {} -q -a --force
gem list  cocoapods xcodeproj | grep -E 'cocoapods' | awk '{print $1}' | xargs -I {} gem uninstall {} -q -a --force


sudo gem uninstall xcodeproj -q -a --force
gem uninstall xcodeproj -q -a --force
```

