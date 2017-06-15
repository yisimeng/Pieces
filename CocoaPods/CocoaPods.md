# CocoaPods

## 私有仓库

cocoapods管理代码的原理是：
![cocoapods 原理](https://cl.ly/0Q1k2D083I1O/download/719C8CC4-41F9-4268-9A7C-D8EB6EC1BC6A.png)
当我们使用cocoapods时，会将cocoapods的远程索引文件仓库clone到本地路径```/Users/yisimeng/.cocoapods/repos/master```，需要引入开源库时，会从本地的索引文件库中寻找到对应的'.podspec',读取到开源库的源码地址，然后从远程clone下来。

使用发现第一次导入一个开源库的时候比较慢，之后会很快安装完。这是因为第一次下载下来源码后会缓存到本地一份，使用```pod cache list```查看已经缓存的开源库的路径，路径为```/Users/yisimeng/Library/Caches/CocoaPods/Pods```，如果本地有对应的版本的话，不会从远程安装。

给我们自己创建一个私有的仓库步骤：

1、使用```pod repo```查看本地仓库，如果用过cocoapods的话会有一个cocoapods的官方仓库
![](https://cl.ly/1N0A421V0Y2m/download/5A723073-B419-414B-9A8A-B29B4058176E.png)

2、在github上创建一个远程索引仓库（），然后使用命令```pod repo add repoName repoRemotePath```添加到本地，用于存放私有库的索引文件，和cocoapods官方的master在同一路径下。```pod repo```查看本地索引仓库。
![pod repo](https://cl.ly/2u1i0a2Z190K/download/79B39DAE-DF33-424F-91EC-EA5D276EADD8.png)

3、在本地创建我们的源码工程，可以使用```pod lib create YSMCategory```，来创建一个pod模板工程。![pod lib create projectName](https://cl.ly/2N112I3g2Z0w/download/29670AC1-A83D-4A98-B93E-232E5892D14E.png)
我们可以看下自动生成的模板工程文件目录
![model 目录](https://cl.ly/072N0B2T3X2s/download/7A1D73FC-DC4A-4F5C-B3F8-7E72EFE33C20.png)
在Pods中的'Development Pods/YSMCategory/YSMCategory/Classes'文件下有个ReplaceMe.swift，这个就是需要替换成我们自己的代码文件。而工程中的'Podspec Metadata'文件中则是存放我们的'.podspec'索引文件和协议文件，还有README.md。

4、创建我们的远程源码仓库（私有的），然后和本地仓库关联起来。看下Podfile文件内容：

``` use_frameworks!
target 'YSMCategory_Example' do
  pod 'YSMCategory', :path => '../'
  target 'YSMCategory_Tests' do
    inherit! :search_paths
  end
end
```

path路径是'../'表示本地上一层目录文件夹
![local path](https://cl.ly/0V2s3y2o4410/download/7ECC2AF9-51CC-44EC-BB5B-F61B10EF8CC1.png)
'YSMCategory'文件在Podfile的上层文件目录下。所以这个pod会从本地文件中下载代码。
所以我们需要在文件中将'ReplaceMe'文件替换为源码文件，然后执行```pod install```，会直接在工程中替我们替换为源码文件（直接在工程中替换会有问题）。修改'.podspec'文件信息。```git remote add origin https://xxxx.git```，然后将本地代码提交到远程。

5、打上tag，并提交tag。

6、进行本地验证```pod lib lint```，通过后可以在进行一步远程验证```pod spec lint```，这一步不是必须的，push时会有一次验证。通过后向私有索引文件库提交，```pod repo push SpecName XXX.podspec```，这时我们只是向本地的索引文件库push，但是pod会自动帮我们push到远程的索引文件库。
![push Spec](https://cl.ly/2t352e0j1n2F/download/7D30DF1B-2FD2-4B3A-8FF3-5D76971999AD.png)
6.1、补充一点，如果最后push时，使用的代码是```pod trunk push```，则会将代码提交到官方的Specs文件中。

齐活。

下面测试一下私有库能不能用
先```pod search```查看一下
![search YSMCategory](https://cl.ly/0x3R1N1r2B2c/download/3412A63E-4050-40C5-A9A3-D401864F23A0.png)
是存在的。
然后创建工程引入试试。这里需要注意一下Podfile文件，默认是从cocoapods官方repo搜索的，但是索引文件在我们自己创建的索引库中，所以需要修改下'source'，将

```
source 'https://coding.net/u/yisimeng/p/YSMPrivateSpecs/git'
source 'https://github.com/CocoaPods/Specs.git'
```
写入文件头，私有库和共有库，然后执行install。
![install remote](https://cl.ly/3K2E3r0V2z1B/download/442F4971-0DA6-4E48-A4CE-DF0D68713EF2.png)

完事。

如果后续需要更新仓库代码的话，流程就是：

1、在Example工程中添加代码或文件，然后修改‘.podspec’文件版本号，把源代码提交到远程，打上tag并提交。
2、把‘.podspec’文件push到本地的私有索引库中（会自动帮我们push到远程索引库）。
3、回到我们的项目工程，修改Podfile中导入库文件的版本为最新，执行pod install（如果不行的话就pod update）。

** 切记：swift中需要设置好访问权限！！！ **

## 注意事项

1. 在podfile文件中 增加"inhibit_all_warnings!"，这样pod的工程不会显示任何警告。
