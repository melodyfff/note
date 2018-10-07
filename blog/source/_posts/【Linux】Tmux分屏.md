---
title: 【Linux】Tmux分屏
tags: [Linux]
date: 2018-10-07
---

## 1.Tmux
> Arch维基: https://wiki.archlinux.org/index.php/Tmux_(简体中文)  
> 官方WIKI: https://github.com/tmux/tmux/wiki

## 2.常用命令

```bash
tmux new -s ok　　    # 创建名为ok的会话
tmux ls　　           # 显示会话列表
tmux a　　            # 连接上一个会话

tmux a -t ok　　      # 连接指定会话

tmux rename -t s1 s2　# 重命名会话s1为s2

tmux kill-session　　 # 关闭上次打开的会话

tmux kill-session -t s1　　# 关闭会话s1

tmux kill-session -a -t s1　#　关闭除s1外的所有会话

tmux kill-server　　  # 关闭所有会话
```

### 快捷键
> `PREFIX` 指 `C-b`(默认)  
> `C-b` 指 `Ctrl+b`  
> `PREFIX` 可在配置`.tmux.conf`文件中替换,建议替换为`C-a`

#### Panels操作
- `PREFIX + %` 左右分割窗格
- `PREFIX + "` 上下分割窗格
- `PREFIX + x` 关闭当前窗格
- `PREFIX + {` 当前窗格前移
- `PREFIX + }` 当前窗格后移
- `PREFIX + o` 顺时针切换窗格
- `PREFIX + C-o` 逆时针旋转当前窗口的窗格
- `PREFIX + space` 重新排列当前窗口下的所有窗格
- `PREFIX + ;` 上次使用窗格
- `PREFIX + !` 将当前窗格置于新窗口
- `PREFIX + z` 最大化当前窗格，再次按下恢复
- `PREFIX + Up|Down|Left|Right` 根据箭头方向切换窗格

#### Windos操作
- `PREFIX + c` 新建窗口
- `PREFIX + w` 窗口列表
- `PREFIX + p` 切换至上一窗口
- `PREFIX + n` 切换至下一窗口
- `PREFIX + ,` 重命名窗口
- `PREFIX + .` 修改当前窗口索引编号
- `PREFIX + 0-9` 根据id索引编号切换窗口
- `PREFIX + f` 根据窗口名查找窗口,模糊匹配

#### Session操作
- `PREFIX + s` session列表
- `PREFIX + $` 重命名session
- `PREFIX + d` 分离当前session
- `PREFIX + D` 分离指定session



## 3.自用配置

#### .tmux.conf
```bash
# open mouse
set -g mouse on

# switch prefix
set -g prefix C-a

# key-bind
bind | split-window -h
bind - split-window -v

bind -n S-Left previous-window
bind -n S-Right previous-window

# switch panels
bind k selectp -U # switch to panel Up
bind j selectp -D # switch to panel Down 
bind h selectp -L # switch to panel Left
bind l selectp -R # switch to panel Right

bind q killp # kill panel

# status justify center
set-option -g status-justify centre

# left bottom
#set-option -g status-left '#[bg=black,fg=green][#[fg=cyan]#S#[fg=green]]'
#set-option -g status-left-length 20

# window list
setw -g automatic-rename on
set-window-option -g window-status-format '#[dim]#I:#[default]#W#[fg=grey,dim]'
set-window-option -g window-status-current-format '#[fg=cyan,bold]#I#[fg=blue]:#[fg=cyan]#W#[fg=dim]'

# right bottom
set -g status-right '[%Y-%m-%d %H:%M]'

# Easy config reload
bind-key r source-file ~/.tmux.conf \; display-message "Config Reloaded"
```