# bash

.bashrc、.bash_profile区别: 

| 文件 | 描述 | 执行时间 | 生效时间 | 是否验证 |
| :-- | :-: | :--: | :--: | --: |
| /etc/profile | 为系统的每个用户设置环境信息 | 用户第一次登录时执行 | 重启后生效 | 否 |
| /ect/bashrc | 为每个运行bash shell的用户设置环境信息 | bash shell被打开时执行 | 重新打开bash | 否 |
| ~/.bash_profile | 当前用户专用的bash shell配置信息 | 用户登录shell时执行 | 重新登录生效 | 是 |
| ~/.bashrc | 当前用户专用的bash shell配置信息 | 每次打开bash shell时执行 | 重新打开shell生效 | 否 |

> 注：~/.bash_profile 是交互式的，login方式进入bash运行的，而~/.bashrc是交互式non-login方式进入bash运行的。

