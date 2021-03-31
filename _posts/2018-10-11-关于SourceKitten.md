---
layout: mypost
title: 关于SourceKitten的一些知识
---

### SourceKitten是一个基于SourceKit的命令行工具，可以匹配Swift AST(抽象语法数)，可以迅速到处项目文档，代码格式化等等。

+ 这文章先大概讲解一下几个用SourceKitten实现的工具，例如SwiftLint，Jazzy，Sourceery，SwiftMocky，SourceKittenDaemon。但是用SourceKitten其实可以做很多事情，比如检测方法是否有被全局调用，删除废弃代码等。这些都是思考到的东西，后续可能会向这方面实践一下

### [Jazzy](https://github.com/realm/jazzy)的使用

+ 前几天有用Jazzy更新项目文档，这里将实际操作和遇到的问题提一提，以作记录，方便查询

+ iOS 生成文档的框架挺多的，例如这里将要提到的Jazzy，SourceDocs，AppleDoc，但是AppleDoc不支持Swift，SourceDocs生成的是Markdown,所以项目中优先采用了Jazzy。Jazzy是可以为Swift、OC产生具有Apple风格的代码文档工具

+ 安装Jazzy

```
    sudo gem install jazzy
```

+ 如果没有安装Ruby环境，或者需要升级请自行百度

+ 查看相关用户  

```
    jazzy -h
```

+ 项目工程根目录下
```
    cd ProjectPath【项目工程目录下】
```

+ 运行Jazzy生成项目文档

    - jazzy 生成文档可以配置对应参数，如默认生成的Swift文档只包含```public```和```open```声明的内容，我们可以通过```-min-acl```标志设置生成文档可以包含的权限，如```jazzy --min-acl private```，这样基本上可以生成最高权限的文档。

```
    <!-- 生成项目文档 可以操作README增加参数 -->
    jazzy 
```

+ 运行完之后可以发现 会提示有跳过多少private和fileprivate的文件
```
    skipped 718 private or fileprivate symbols (use `--min-acl` to specify a different minimum ACL)
```

+ 同时生成的文档目录保存在项目根目录下的```docs/publish```目录下，打开index就可以查询到对应的文档



### SourceKitten是一个基于SourceKit的命令行工具，可以匹配Swift AST(抽象语法数)，可以迅速到处项目文档，代码格式化等等。

+ 这文章先大概讲解一下几个用SourceKitten实现的工具，例如SwiftLint，Jazzy，Sourceery，SwiftMocky，SourceKittenDaemon。但是用SourceKitten其实可以做很多事情，比如检测方法是否有被全局调用，删除废弃代码等。这些都是思考到的东西，后续可能会向这方面实践一下

### [Jazzy](https://github.com/realm/jazzy)的使用

+ 前几天有用Jazzy更新项目文档，这里将实际操作和遇到的问题提一提，以作记录，方便查询

+ iOS 生成文档的框架挺多的，例如这里将要提到的Jazzy，SourceDocs，AppleDoc，但是AppleDoc不支持Swift，SourceDocs生成的是Markdown,所以项目中优先采用了Jazzy。Jazzy是可以为Swift、OC产生具有Apple风格的代码文档工具

+ 安装Jazzy

```
    sudo gem install jazzy
```

+ 如果没有安装Ruby环境，或者需要升级请自行百度

+ 查看相关用户  

```
    jazzy -h
```

+ 项目工程根目录下
```
    cd ProjectPath【项目工程目录下】
```

+ 运行Jazzy生成项目文档

    - jazzy 生成文档可以配置对应参数，如默认生成的Swift文档只包含```public```和```open```声明的内容，我们可以通过```-min-acl```标志设置生成文档可以包含的权限，如```jazzy --min-acl private```，这样基本上可以生成最高权限的文档。

```
    <!-- 生成项目文档 可以操作README增加参数 -->
    jazzy 
```

+ 运行完之后可以发现 会提示有跳过多少private和fileprivate的文件
```
    skipped 718 private or fileprivate symbols (use `--min-acl` to specify a different minimum ACL)
```

+ 同时生成的文档目录保存在项目根目录下的```docs/publish```目录下，打开index就可以查询到对应的文档


* * *


### [SwiftLint](https://github.com/realm/SwiftLint),加强Swift编码风格，统一编码规范

+ 程序猿基本上都要与其他同事进行协作开发，所以制定项目规范至关重要。代码规范包括在项目规范中，所以代码统一规范也是至关主要，之前就和同事讨论过项目文件结构，以及统一使用商定好的代码块。在编码规范中每个人都有自己的特色，例如常说的`Tab` 和 `Space`, `if`后需不需要换行，`enum`元素可不可以大写，有差异所以在项目维护上来其实并不是一件好事，所以有了`SwiftLint`，主要用于Swift代码规范格式，在协作中也可以更好的去Codereview。

+ 当然独行侠也同样是需要符合Swift编码规范，毕竟一不小心就有可能写出傻逼代码。

+ 如何使用`SwiftLint`:
    
   - 全局安装
    
        1. 全局安装`SwiftLint`
        
        ```
            brew install swiftlint
        ```
        
        2. 项目工程中添加编译脚本，在`target->Build Phase-> New Run Script Phase`中的黑框添加以下代码
    
        ```
            if which swiftlint >/dev/null; then
                swiftlint
            else
                echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
            fi
        ```
        3. 编译项目即可


    - Pod导入
         
         1. 在Podfile中配置SwiftLint版本，之后`pod install`即可
         
        ```
            pod 'SwiftLint', '~> 0.27.0'
        ```
        2. 项目工程中添加编译脚本，在`target->Build Phase-> New Run Script Phase`中的黑框添加以下代码
        ```
           "${PODS_ROOT}/SwiftLint/swiftlint"
        ```
        3. `Build`项目之后Swift就导入到项目中了。
        
+ Build之后项目工程中或多或少会有一些警告和错误，基本上都是因为代码不规范引起的，在全局安装了`SwiftLint`之后，可以通过以下命令自动修复一些格式不匹配的地方。
    
    ```
        swift autocorrect
    ```
     
+ 可以根据个人或者团队商定选取一些强制执行的规则，配置好`.swiftlint.yml`文件

* * *


