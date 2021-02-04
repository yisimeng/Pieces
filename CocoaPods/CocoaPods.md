# CocoaPods

先说下CocoaPods拉取开源库的原理：

CocoaPods 有一个开源的索引仓库[Specs](https://github.com/CocoaPods/Specs)，仓库存放着所有开源库的各个版本的`.podspec`文件，`.podspec`文件包含中记录着源码的地址。首次使用CocoaPods时，会将这个文件库克隆到本地`~/.cocoapods/repos/master`。

1. 在Podfile目录下执行 `pod install` 命令，会从本地的索引库查找该库的`.podsepc`，如果本地不存在会从远程拉取最新的索引库。
2. 根据索引库中查到的`.podspec`文件内容，获取源码地址。
3. 从源码地址拉取对应版本的代码。

![specs](../images/Cocoapods_Specs.png)

> 使用是可以发现，首次导入一个开源库时速度较慢，之后再导入时会很快。是因为CocoaPods在本地会有一个缓存目录，存放开源库的源码，首次下载后，再次导入该库时，会直接从本地复制过去。
>    
> 查看缓存列表使用`pod cache list`,缓存路径为`~/Library/Caches/CocoaPods/Pods/`。

## 私有仓库

### 私有库的创建

由于CocoaPods的索引仓库是开源的，所有人都可以访问。公司的项目如果也想使用CocoaPods管理源码，而不开放源码的话，我们可以通过创建私有仓库来模拟官方的Specs仓库。

步骤：

1. 在私有git上创建一个索引仓库，例：YSMSpecs，用于存放索引文件。
2. 将远程索引库添加到本地，`pod repo add YSMSpecs YSMSpecs_source_url`。使用`pod repo`可以查看本地的索引仓库列表。
   
    ```
    $ pod repo add YSMSpecs https://github.com/yisimeng/YSMSpecs.git
    $ pod repo
    
    master   // 公有索引仓库
    - Type: git (master)
    - URL:  https://github.com/CocoaPods/Specs.git
    - Path: /Users/duanzengguang/.cocoapods/repos/master
    YSMSpecs  //私有索引仓库
    - Type: git (master)
    - URL:  https://github.com/yisimeng/YSMSpecs.git
    - Path: /Users/duanzengguang/.cocoapods/repos/YSMSpecs
    ```

3. 本地创建我们的源码工程，可以使用`pod lib create YSMKit`，创建一个模板工程。
4. 在模板工程里进行开发并替换 ReplaceMe 文件，修改`.podspec`文件(版本号，源码地址)，推送到远程源码仓库，打tag，提交。源码仓库部署完成。
5. CocoaPods不允许有Podspecs lints错误，所以需要进行Podspecs lints（翻译不好，会检查语法错误）验证。这里可以使用`pod lib lint`或者`pod spec lint`,区别在于前者不会联网，而后者还会检查外部的仓库和相关的标签。

    ```
    $ pod lib lint
    -> YSMKit (0.1.0)
    YSMKit passed validation.
    ```
    
6. 检查没有错误之后，推送`.podspec`文件到本地的索引仓库，本地索引仓库会自动push到远程索引仓库。`pod repo push YSMSpecs YSMKit.podspec`，这一步会自动进行`pod spec lint`联网检查。索引库部分完成。

    ```
    $ pod repo push YSMSpecs YSMKit.podspec
    Validating spec
     -> YSMKit (0.1.0)
    Updating the `YSMSpecs' repo
    Already up to date.
    Adding the spec to the `YSMSpecs' repo
     - [Add] YSMKit (0.1.0)
    Pushing the `YSMSpecs' repo   // 会自动推送到远程仓库
    
    $ pod search YSMKit
    -> YSMKit (0.1.0)
       YSMKit is my kit
       pod 'YSMKit', '~> 0.1.0'
       - Versions: 0.1.0 [YSMSpecs repo]
    ```
    

到这里私有仓库是搞完了。
    
> 第6步如果使用`pod trunk push YSMKit.podspec`，会将索引库推送到官方的Specs仓库中。

### 私有库的使用

1. 在宿主工程的Podifle文件中引入私有库：`pod 'YSMKit'`
2. 在文件的最上方添加索引库地址

```
source 'https://github.com/yisimeng/YSMSpecs.git'
source 'https://github.com/CocoaPods/Specs.git'
```
然后执行 pod install，就可以使用了。

> 注意： 如果用到了其他开源库的话，一定要加上官方Specs地址，否则只会去查找私有索引库。

### 私有库的维护

后续需要更新维护仓库代码的流程：

1. 在源码工程修改代码之后，修改`.podspec`文件版本号，把源码提交，打上tag，推送到远程仓库。
2. 把`.podspec`文件提交到本地的私有索引库中（会自动帮我们提交到远程索引库）。
3. 回到宿主工程，修改Podfile中的版本，执行`pod install`（不行就`pod update`）。

## 注意事项

1. 在podfile文件中 增加`inhibit_all_warnings!`，这样pod的工程不会显示任何警告。
2. **切记：swift中需要设置好访问权限！！!**
3. 新引入一些库之后执行`pod install`，会报错**Build Settings**，例如：引入Swift库之后经常会报的“swift version”的错误，需要去修改Build Setting，其实可以在Podfile中的`post_install`中修改。

```
post_install do |installer|
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['SWIFT_VERSION'] = '3.2'
        end
    end
end
```

`pre_install`：编译之前可以添加修改（还没想到可以做哪些事情）。

4. 已经打包好上传完成的仓库，本地拉不下来，报错： fatal: Unable to create '/Users/duanzengguang/.cocoapods/repos/specs/.git/index.lock': File exists.

将index.lock 删除。