---
title: "Tmux"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

将会话（还有内部的进程）与窗口解绑的工具

几个名词：

session 会话，理解成任务
window 窗口，当前呈现在面前的终端界面就是窗口
pane 窗格，把窗口切割成一小块一小块
一个会话可以有多个窗口，一个窗口可以又可以分隔成多个窗格
还允许多人实时共享对话

会话session操作

新建会话：tmux new -s <session-name>
分离会话：tmux detach / ctrl+b d
查看所有会话：tmux ls
接入会话：tmux attach -t <session-name>
tmux attach -t 0 会话编号
杀死会话： tmux kill-session -t 会话编号/<session-name>
切换会话：tmux switch -t 会话编号/<session-name>
重命名会话：tmux rename-session -t 会话编号/session-name <new name>
快捷键
ctrl+b d 分离会话
ctrl+b s 列出所有会话
ctrl+b $ 重命名当前会话
查看session历史输出信息: 进入指定会话后，按ctrl+b，松开后按[，就可以进入历史输出信息查看模式，按q退出
窗格pane操作

划分窗格(pane)  tmux split-window [-h]
移动光标 tmux select-pane -U/D/L/R
快捷键

窗口操作

新建窗口 tmux new-window -n <name>
切换窗口 tmux select -window -t <name/number>
重命名窗口 tmux rename-window <new name>
快捷键