---
title: swift自建pod引用.a库
---

项目中遇到要封装推送的时候，遇到一个问题，大概是，JPUSH是一个.a库，不能在swift中直接`import JPUSHService`, 也不能在swift的framework中新建桥接文件去引用`import "JPUSHService.h"`。
  
解决方法是：在自建库的目录中新建一个Modula的目录，在Module的目录中创建`modulemap`文件，在这个`modulemap`中去引用这个`JPUSHService.h`, 然后在指定`modulemap`文件为这个新建的`modulemap`文件

```ruby
s.preserve_paths = ['Module/module.modulemap']
s.pod_target_xcconfig = {
        'SWIFT_INCLUDE_PATHS' => ['$(PODS_ROOT)//Module', '$(PODS_TARGET_SRCROOT)/xxx/Module'],
}
```
新建的modulemap文件内容如下
```module
module JPUSHPackageBridge {
    header "JPUSHService.h"
    export *
}
```

然后我们就可以在我们的自建库里面去`import JPUSHPackageBridge`就可以访问·JPUSHService.h·中的内容了。

在用本地pod集成的时候遇到一个问题，因为我们的`SWIFT_INCLUDE_PATHS`指定的`$(PODS_ROOT)/Module`和` $(PODS_TARGET_SRCROOT)/xxx/Module`，它们是pod install过后的的路径，但是本地pod的路径没有办法去指定，所以在集成的时候会出现找不到`JPUSHPackageBridge`的错误，这个时候可以直接发布到测试的仓库中，pod下来然后去完善代码。还有一种解决办法是再创建一个`Test`的pod，把刚刚的`module.modulemap`文件放到它的目录中去，在`Test.podspec`中去指定`s.preserve_paths = ['Module/module.modulemap']`, 发布这个`Test.podspec`到测试仓库中, 在xxx库的`xxx.podspec`中去重新指定`'SWIFT_INCLUDE_PATHS' => ['$(PODS_ROOT)/Module', '$(PODS_TARGET_SRCROOT)/Test/Module']`，这样xxx的pod库就可以在本地进行开发。

使用中还遇到一个问题，因为在项目中使用到了一些JPUSH的API，而封装的pod中也是依赖到了JPUSH，在编译中会报`abort:trap`的错误，这个目前的解决方案是封装项目中JPUSH的API到封装的pod中去，让项目只通过封装的pod去访问。

还有一个问题，在使用cocoapods-binary 插件时，需要把引用的```JPush```和```JCore```单独放在podfile里面，设置```:binary => false```, 同时自己建的库也要设置此选项。

关于iOS`module`的内容：https://hechen.xyz/post/swift-and-modules/
