# shell 

### .bashrc、.bash_profile区别: 

| 文件 | 描述 | 执行时间 | 生效时间 | 是否验证 |
| :-- | :-: | :--: | :--: | --: |
| /etc/profile | 为系统的每个用户设置环境信息 | 用户第一次登录时执行 | 重启后生效 | 否 |
| /ect/bashrc | 为每个运行bash shell的用户设置环境信息 | bash shell被打开时执行 | 重新打开bash | 否 |
| ~/.bash_profile | 当前用户专用的bash shell配置信息 | 用户登录shell时执行 | 重新登录生效 | 是 |
| ~/.bashrc | 当前用户专用的bash shell配置信息 | 每次打开bash shell时执行 | 重新打开shell生效 | 否 |

> ~/.bash_profile 是交互式的，login方式进入bash运行的	
> ~/.bashrc是交互式non-login方式进入bash运行的。


##### 问题：如何避免脚本之前加上冗长的文件路径，实现在任何路径下都可以执行脚本？
将脚本修改为全局调用。

* 修改.bash_profile，把脚本路径添加到$PATH后面，对当前用户生效`export PATH=$PATH:MY_PATH`。

> $PATH脚本执行会从PATH路径下查找脚本命令。
> “/bin”、“/sbin”、“/usr/bin”、“/usr/sbin”、“/usr/local/bin”等路径已经在系统环境变量中，如果脚本在这些路径下，可直接执行。如果不在就需要在$PATH之后拼接，用冒号分隔。

# cmd

### sed 文件内容操作
内容替换/删除：
`sed -i “.tmp” ’s,old str,’”new str”’,’ file`
`sed -i “.tmp” ‘5d’ file // 删除第5行`
其中“.tmp”为备份扩展名，为空则不备份，如果恰好磁盘耗尽，文件可能损坏。

### cp 拷贝
cp -d 当拷贝的文件有符号链接时，保存链接(当前文件为**替身文件**)。
cp -p 保留源文件或目录的属性
cp -R 递归。
cp -a 同“-dpR”

### mv 移动文件
mv -f 如果目录非空，报错`Directory not empty`，非空不允许覆盖。

### 计算文件夹大小
du -sh *


## shell 参数格式

内置命令getopts，帮助处理选项和参数，只能处理短选项，无法处理长选项

getopt可以兼顾长选项和短选项的处理，getopt命令不是标准unix命令，一般linux会自带，如果没有需要去下载安装。用法：
```
ARGS=`getopt -o ab:c:: --long along,blong:,clong:: -n 'example.sh' -- "$@"`
```
* -o或--options选项后面接可接受的短选项，如ab:c::，表示可接受的短选项为-a -b -c，其中-a选项不接参数，-b选项后必须接参数，-c选项的参数为可选的。
* -l或--long选项后面接可接受的长选项，用逗号分开，冒号的意义同短选项。
* -n选项后接选项解析错误时提示的脚本名字 