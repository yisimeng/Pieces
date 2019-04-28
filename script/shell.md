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

Linux下的 sed 是GPL的，Mac的是BSD的，所以使用起来有些不同，Mac 的`-i`后需要添加备份的扩展名。

**内容替换**

```
# 将old替换为new
$ sed -i “.tmp” ’s,old,’”new”’,’ file
```

**按行删除**

```
# 删除第5行
$ sed -i “.tmp” ‘5d’ file 
```

**整行替换**： 通过正则将整行内容作为匹配的字符串

```
# 将第 2 行内容替换为 sss
$ sed -i "" '2s/^.*/sss/' test.txt
```

**注意：**“.tmp”为备份扩展名，为空则不备份，如果恰好磁盘耗尽，文件可能损坏。

### cp 拷贝
cp -d 当拷贝的文件有符号链接时，保存链接(当前文件为**替身文件**)。
cp -p 保留源文件或目录的属性
cp -R 递归。
cp -a 同“-dpR”

### mv 移动文件
mv -f 如果目录非空，报错`Directory not empty`，非空不允许覆盖。

### 计算文件夹大小
du -sh *

### expr

暂时理解为执行操作。

expr命令可以实现数值运算、数值或字符串比较、字符串匹配、字符串提取、字符串长度计算等功能。它还具有几个特殊功能，判断变量或参数是否为整数、是否为空、是否为0等。

### shell 获取日期和时间

current_dateTime="`date +%Y-%m-%d,%H:%m:%s`"
echo $current_dateTime

## shell 参数格式

内置命令getopts，帮助处理选项和参数，只能处理短选项，无法处理长选项

getopt可以兼顾长选项和短选项的处理，getopt命令不是标准unix命令，一般linux会自带，如果没有需要去下载安装。用法：
```
ARGS=`getopt -o ab:c:: --long along,blong:,clong:: -n 'example.sh' -- "$@"`
```
* -o或--options选项后面接可接受的短选项，如ab:c::，表示可接受的短选项为-a -b -c，其中-a选项不接参数，-b选项后必须接参数，-c选项的参数为可选的。
* -l或--long选项后面接可接受的长选项，用逗号分开，冒号的意义同短选项。
* -n选项后接选项解析错误时提示的脚本名字 

### export

**export 功能说明**：设置或显示环境变量。

**语　　法**：export [-fnp][变量名称]=[变量设置值]

**补充说明**：在shell中执行程序时，shell会提供一组环境变量。export可新增，修改或删除环境变量，供后续执行的程序使用。export的效力仅限于该次登陆操作。

**参　　数**：

* -f 　代表[变量名称]中为函数名称。
* -n 　删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中。
* -p 　列出所有的shell赋予程序的环境变量。


### 从指定字符（子字符串）截取

这种截取方式无法指定字符串长度，只能从指定字符（子字符串）截取到字符串末尾。Shell 可以截取指定字符（子字符串）右边的所有字符，也可以截取左边的所有字符。

#### 1. 使用 \# 号截取右边字符串

`${string#*chars}`

意思是忽略左边所有字符，截取chars之后的所有字符。string 表示要截取的字符串，chars 是指定的字符（或字符串），* 是通配符表示任意长度。

```
url="https://github.com/yisimeng/Pieces.git"
echo ${url#*:}

输出： //github.com/yisimeng/Pieces.git
```

**注意**：chars 可以是字符也可以是字符串，遇到即止。

* 上例中，`echo ${url#*s:}`，`echo ${url#*ps:}`，`echo ${url#*https:}`，最终答案都一样。
* 如果chars为“/”，那么遇到第一个“/”，截取为：`/github.com/yisimeng/Pieces.git`。

如果希望从最后一个指定字符开始截取右边的字符串，可以用`##`

`${string##*chars}`

```
url="https://github.com/yisimeng/Pieces.git"
echo ${url#*/}
echo ${url##*/}

输出： 
/github.com/yisimeng/Pieces.git
Pieces.git
```

#### 2. 使用 % 截取左边字符串

`${string%chars*}`

因为要截取左边字符串，所以使用`*`忽略右边的字符串。具体使用与上面类似。

```
url="https://github.com/yisimeng/Pieces.git"
echo ${url%/*}
echo ${url%%/*}

输出： 
https://github.com/yisimeng
https:
```

### 计算

整数计算: expr。 中间必须有空格:

```
a=`expr 3 - 1`
echo $a
```

浮点运算：不支持浮点运算，可以借助 bc, awk 处理：

```
a=$(echo "2.1-0.3"|bc)
echo $a

b=$(awk 'BEGIN{print 2.1-0.3 }')
echo $b
```

### Mac 剪贴板

自带剪贴板命令：pbcopy 和 pbpaste

```
# 写入剪贴板
$ echo "yisimengba" | pbcopy
# 或者
$ echo pbcopy < echo "yisimengba"

# 读取剪贴板
$ pbpaste
# 保存内容到文件
$ pbpaste > ~/yisimengba.txt
```

## 笔记

1. $- 相当于当前shell的运行参数，根据参数里是否有 i 来判断是否交互模式。