---
layout:     post
title:      Mac
subtitle:   Mac实用技巧
date:       2019-06-01
author:     owl city
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - Mac
    - zsh
    - tmux
---

> - Create Date: 2019-12-03
> - Update Date: 2019-12-03

## 丰富终端的应用：
> **[Awesome Mac! Mac精品应用](https://wangchujiang.com/awesome-mac/index.zh.html)**

#### 1.brew
- 安装后可以使用 brew install … 安装一些软件
- `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

#### 2.iTerm2（更加好用的terminal）
- `command+enter`   全屏
- `command+d`   垂直分屏
- `command+shift+d`   水平分屏
- `command+u`   清楚当前行
- `command+l`  清屏
- `command+t` 新建标签
- `command+w` 关闭标签
- `command+方向键` 切换标签


#### 4.安装zsh
- `sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
- 更换主题: 编辑`.zshrc`, 更新`ZSH_THEME="agnoster"`
  - 如需针对主题进行自定义配置, 编辑 `~/.oh-my-zsh/themes/agnoster.zsh-theme`
  - 如去除多余的显示用户名和host显示, 将`prompt_context`方法`then`后面的语句直接注释掉

- 使用插件: 编辑`.zshrc`, 在`plugins`中添加, 如`plugins=(git autojump sublime tmux kubectl zsh-autosuggestions zsh-syntax-highlighting)`

#### 5.autojump
	brew install autojump
	修改~/.zshrc，找到plugins,添加git autojump
	另起一行添加   [[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew —prefix)/etc/profile.d/autojump.sh
	source ~/zshrc
	然后使用  j   kafka   可以找到最近使用的kafka文件夹，实现快速跳转


#### 3.Tmux（终端复用）
###### 常用命令
- `tmux new -s <name>` 在回话外创建一个新的回话  
- `tmux ls`  获取回话列表
- `tmux attach -t <name>`  在回话外进入回话
- `tmux attach 或者 tmux a` 进入列表第一个回话
- `快捷键+d`  临时退出不删除session  
- `快捷键 +  kill-session` 在回话内退出并删除session
- `tmux kill-session -t <name>`  在回话外删除指定session
- `option+鼠标选中` 选中复制, option对应windows键盘的control+alt,

快捷键常用操作：
- `d` 隐藏回话
- `c` 新开一个窗口
- `$` 退出当前窗口
- `,` 重命名窗口
- `数字` 切换到第几个窗口
- `n` 切换到上一个窗口
- `p` 切换到上一个窗口
- `l` 切换到最后一个窗口
- `w` 以菜单的方式显示和选择窗口
- `"`  横向分割窗口
- `%` 纵向分割窗口
- `o` 调到下一个分割窗口
- `上下左右` 调到指定方向的分割窗口
- `x` 关闭当前分割窗口
- `!` 关闭所有分割窗口
- `alt + 上下左右` 可以改变窗口的大小

###### 自定义配置实例(.tmux.conf)
```shell
unbind C-b
set -g prefix C-x
set -g base-index 1
set -g pane-base-index 1
set -g default-terminal "screen-256color"
bind r source ~/.tmux.conf \; display "Configuration reloaded!"
set-option -g mouse on
set -g message-style "bg=#00346e, fg=#ffffd7"
set -g status-style "bg=#00346e, fg=#ffffd7"
set -g status-left "#[bg=#0087ff] ❐ #S "
set -g status-right "#{?client_prefix, ~ , } #[bg=#0087ff] #h #[bg=red] %Y-%m-%d %H:%M "
set -g status-left-length 52
set -g status-right-length 451
set -g status-fg white
set -g status-bg colour234
set -g pane-border-fg colour245
set -g pane-active-border-fg colour39
set -g message-fg colour16
set -g message-bg colour221
set -g message-attr bold
setw -g mode-keys vi
set -g mouse on
bind ] run "reattach-to-user-namespace pbpaste | tmux load-buffer - && tmux paste-buffer"
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "reattach-to-user-namespace pbcopy"
set-window-option -g mode-keys vi
```

## mac安装并使用kafka：
- 环境要求：java环境，brew工具
    - 1.brew install kafka
    - 2.启动kafka(kafka依赖zookeepee,需先启动zookeeper)：
        ```shell
        zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties
        ```

    - 3.创建topic：kafka-topic-create -zookeeper localhost:2181 -replication-factor 1 -partitions 1 -topic test
    - 4.发送消息： kafka-console-producer -broker-list localhost:9092 -topic test
    - 5.消费消息：kafka-console-consumer -bootstrap-server localhost:9092 -topic test -from-beginning
