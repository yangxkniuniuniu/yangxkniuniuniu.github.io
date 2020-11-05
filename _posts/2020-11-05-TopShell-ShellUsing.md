---
layout:     post
title:      Shell
subtitle:   Shell学习笔记
date:       2020-11-05
author:     owl city
header-img: img/post-bg-dog1.jpg
catalog: true
tags:
    - Shell
    - Bash
    - Linux
---

> - Create Date: 2020-11-05
> - Update Date: 2020-11-05

## Bash脚本set命令
set命令可以用来修改Shell环境的运行参数。常用的有如下四个
- `set -u`: 执行脚本的时候，在脚本头部加上这个时，后续如果遇到不存在的变量就会报错，并停止运行
- `set -x`: 用来在运行结果之前，先输出执行的那一行命令
- `set -e`: 脚本只要发生错误，就终止执行（但是不适用于管道命令）
- `set -o pipefail`: 只要一个子命令失败，这个管道命令就会失败，脚本就会终止运行

> 上面四个命令一般放在一起使用： `set -euxo pipefail` 或者 `set -eux;set -o pipefail`

## EOF
Shell中通常将EOF与 << 结合使用，表示后续的输入作为子命令或子Shell的输入，直到遇到EOF为止，再返回到主调Shell。
如与cat一起使用，多行追加输入到aa.txt文件中：
```shell
cat << EOF >> aa.txt
>a1
>b1
>EOF
```

## expect 自动交互脚本
#### 例：自动登录ssh执行命令
```shell
#!/usr/bin/expect -f
set timeout -1
spawn ssh user@ip CMD
expect {
	"*yes/no*" {
		send "yes\r"
		expect "*password*"
		send "$password\r"
	}
	"*password*" {
		send "$password\r"
	}
}
```
#### 在shell脚本中插入expect脚本片段
```shell
#!/bin/sh

# ...一系列的shell脚本

/usr/bin/expect <<EOF
set timeout -1
spawn ssh user@ip
expect {
	"*yes/no*" {
		send "yes\r"
		expect "*password*"
		send "$password\r"
	}
	"*password*" {
		send "$password\r"
	}
}
expect "*\$*"
send "exit\r"
expect eof
EOF
```

