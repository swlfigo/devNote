# pod install 报错: can't find gem cocoapods (>= 0.a) with executable pod (Gem::GemNotFoundException)



因使用 brew 安装工具导致 ruby 环境错乱， 执行 `pod install` 时报错提示找不到 gem 可执行文件



```bash
Traceback (most recent call last):
    2: from /usr/local/bin/pod:23:in `<main>'
    1: from /Library/Ruby/Site/2.6.0/rubygems.rb:294:in `activate_bin_path'
/Library/Ruby/Site/2.6.0/rubygems.rb:275:in `find_spec_for_exe': can't find gem cocoapods (>= 0.a) with executable pod (Gem::GemNotFoundException)
```



##### 解决办法:

1. 重新安装 ruby 环境（默认安装最新版本）



```bash
> rvm reinstall ruby --disable-binary
```

- 运行结果



```bash
mruby-1.3.0 - #removing src/mruby-1.3.0 - please wait
mruby-1.3.0 - #removing rubies/mruby-1.3.0 - please wait
RVM does not have prediction for required space for mruby-1.3.0, assuming 150MB should be enough, let us know if it was not.
Checking requirements for osx.
Certificates bundle '/usr/local/etc/openssl@1.1/cert.pem' is already up to date.
Requirements installation successful.
Installing Ruby from source to: /Users/jack/.rvm/rubies/mruby-1.3.0, this may take a while depending on your cpu(s)...
mruby-1.3.0 - #downloading 1.3.0, this may take a while depending on your connection...
mruby-1.3.0 - #extracting 1.3.0 to /Users/jack/.rvm/src/mruby-1.3.0 - please wait
mruby-1.3.0 - #compiling - please wait
mruby-1.3.0 - #installing - please wait
Install of mruby-1.3.0 - #complete
Required ruby-2.7.0 is not installed.
To install do: 'rvm install "ruby-2.7.0"'
```

1. 重新安装 cocoapods

```undefined
> gem install cocoapods
```

- 运行结果：



```css
Successfully installed cocoapods-1.9.3
Parsing documentation for cocoapods-1.9.3
Done installing documentation for cocoapods after 1 seconds
1 gem installed
```

再重新执行 `pod install` 就 ***OK\*** 了





### Ruby 的安装与切换

列出已知的 Ruby 版本

```
rvm list known
```

安装一个 Ruby 版本

```
rvm install 2.2.0 --disable-binary
```

这里安装了最新的 2.2.0, `rvm list known` 列表里面的都可以拿来安装。

切换 Ruby 版本

```
rvm use 2.2.0
```

如果想设置为默认版本，这样一来以后新打开的控制台默认的 Ruby 就是这个版本

```
rvm use 2.2.0 --default 
```

查询已经安装的 ruby

```
rvm list
```

卸载一个已安装版本

```
rvm remove 1.8.7
```

