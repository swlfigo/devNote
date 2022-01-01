# MonkeyDev 开发问题

#### 1. MonkeyDev 报错An empty identity ...处理方式

Xcode10.3用MonkeyDev 创建LogosTweak项目提示错误解决办法
错误：**An empty identity is not valid when signing a binary for the product type ‘Dynamic Library’**

在该项目的`TARGETS`的`Build settings`中添加一个参数, 点击`Add User-Defined Setting`, 添加`CODE_SIGNING_ALLOWED=NO`, 重新编译, 问题解决.

![img](http://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/webp-20220101100344824.jpg)

![img](http://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/webp-20220101100353604.jpg)



#### 2. ld: building for iOS, but linking in .tbd file (/opt/theos/vendor/lib/CydiaSubstrate.framework/CydiaSubstrate.tbd) built for iOS Simulator, file ‘/opt/theos/vendor/lib/CydiaSubstrate.framework/CydiaSubstrate.tbd’ for architecture arm64 clang: error: linker command failed with exit code 1 (use -v to see invocation)



将  /opt/theos/vendor/lib/CydiaSubstrate.framework 下 CydiaSubstrate.tbd 文本打开，删除archs后面的两项，就可以编译成功了。