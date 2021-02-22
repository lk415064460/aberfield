---
layout: mypost
title:  iOS开发小技巧
---

### Pod更新缓慢

1. 使用CocoaPods进行update或者install 操作时，本地需要同步更新远程的Pod Specs库，所以速度会稍慢。但是我们可以通过下面两个方法去规避解决他:
<br />
 + 不检查Pod Specs库更新，可以在update或者install时增加如下参数，可以跳过Pod Specs库的检查更新

 ```
$   pod install --verbose --no-repo-update
$   pod update --verbose --no-repo-update
 ```

2. 通过设置终端走代理 
+ 即使跳过了Specs库的更新检查，并且update、install也用了梯子，但是下载更新速度还是慢的令人发指，绝大多数是20~30k/s。这个时候我们可以设置一下，让终端也走代理。我这边用的是```shadowSocksX-NG ```，我们可以在通过一下命令:

```
// 打开 .bash_profile 文件
$   open -t ~/.bash_profile

// 如果提示没有此文件，那需要自己创建一个,然后再打开
$   touch ~/.bash_profile

// 将下面代码复制到文件中
export ALL_PROXY=socks5://127.0.0.1:1086
export http_proxy="http://127.0.0.1:1087"
export https_proxy="http://127.0.0.1:1087"

// 终端命令启动source
$   source ~/.bash_profile
```

+ 处理完之后如果没有快起来那自行Google一下了　```Mac 如何让终端走代理```


3. 使用CocoaPods specs国内镜像
+ 使用国内对Github上的Specs仓库镜像，现在每十分钟就会进行一次同步，基本上和主仓库一直的。设置方法:

```
$   pod repo remove master

$   pod add master https://git.oschina.net/6david9/Specs.git

$   pod repo update
```

+ 然后再Podfile头部制定source
<br />

```
$   source 'https://git.oschina.net/6david9/Specs.git'  
```

+ 整个过程会把Specs仓库Clone到本地，可能会需要点时间。

### pod install 的 pod update的区别、以及pod outdated作用

+ ```pod install```的作用是在下载、安装新的库的同时，也会把你安装的每个库版本写入到```Podfile.lock```文件中。这个文件记录你每个安装库的版本号，并且锁定了这些版本。

+ 当你使用```pod install```安装更新pods库时，pod会下载在```podfile.lock```中库的明确版本，并不会去检查是否该库有新的版本。

+ ```pod update``` 的作用是将库更新到最新的版本，它并不会管```podfile.lock```中锁定的版本号。当你运行```pod update```时，项目所有引用的第三方库将会全部被更新到最新版本，并且写入到```podfile.lock```文件中。当然我们可以指定某个库更新到最新版本，命令是 ```【pod update 库名称】```。

+ 正确用法：使用```Pod update``` 可以使用```【pod update 库名称】``` 更新指定库，尽量避免直接使用```pod update```全量更新，特别是没办法搬走Pod库升级对你项目有多大影响的时候。 然后添加删除或者升级到指定版本库时可以先修改```Podfile```文件，然后使用```pod install```。

+ 同时在协同开发是，需要保证每个人的Pod库版本一直，那么我们需要提交一下```Podfile.lock```文件。


### 去除Pod所有库警告

+ 想着可以通过Pod的钩子```Installer```配置相应的```build_setting```参数来避免警告内容，如下

```
post_install do |installer|
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['ENABLE_BITCODE'] = true
            # 设置Build中Other_Warning_Flags 参数
            config.build_settings['Other_Warning_Flags'] = '-Wno-receiver-expr -Wno-unguarded-availability -Wno-unknown-warning-option -Wno-strict-prototypes -Wno-implicit-retain-self -Wno-documentation -Wno-strict-prototypes -Wno-deprecated-implementations'
#            if target.name == 'Charts' or target.name == 'MonkeyKing' or target.name == 'McPicker' or target.name == 'SnapKit' or target.name == 'KeychainAccess' or target.name == 'Kingfisher' or target.name == 'ImageSlideshow' or target.name == 'SwiftTimer' then
#                config.build_settings['SWIFT_VERSION'] = '4.0'
#            else
#                config.build_settings['SWIFT_VERSION'] = '3.2'
#            end
        end
    end
end
```

然后发现他🐱的居然没用。

然后发现可以直接在```Podfile```配置如下参数，可以完美的避免警告，如下

```
platform :ios, '8.0'
# 避免警告的命令
inhibit_all_warnings!
use_frameworks!
```

