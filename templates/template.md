# obsidian
## 1 样式
### 1.1 callout
> [!note]
> this is a note callout
> > [!todo]
> > this is a inner todo callout
> > ```
> > def hello()
> > 	print("hello world")
> > ```

## 2 code block

+  首行可添加语言种类, 如python, c

```python
def hello()
	print("hello world")
```

## 3 link
+ [[]]前加!表示预览状态
+ 图片
	 ![[demo.png|528]]
 + 标题
	[[#1.1 callout|callout]]
+ 其他笔记
	[[bbr_v2|bbr_v2_note]]
### 3.1 checkbox
- [ ] working
- [y] finish
- [x] kill
## 4 插件
### 4.1 number headings
+ 命令面板输入number heading，选择添加number
### 4.2 list callouts
+ ? hello
	+ hello
+ ! world
	+ wrold
### 4.3 hightlightr
+ this is a <mark style="background: #BBFABBA6;">highlight</mark> <mark style="background: #FFF3A3A6;">text</mark>
### 4.4 pdf++
+ pdf注释
+ quote
> ([[BBR_v2.pdf#page=1&selection=1,10,3,8|BBR_v2, p.1]])
> ongestion Control 
+ link
[[BBR_v2.pdf#page=1&selection=0,10,36,6&color=red|BBR_v2, p.1]]
+ embed
![[BBR_v2.pdf#page=3&selection=179,0,189,2&color=red]]
+ callout
> [!PDF|yellow] [[BBR_v2.pdf#page=1&selection=186,1,219,0&color=yellow|BBR_v2, p.1]]
> hat supports packet-delivery acknowledgment. Thus far, open source implementations are available for TCP [RFC793] and QUIC [RFC9000]. This document specifies version 2 of the BBR algorithm, also sometimes referred to as 

+ quote in callout
> [!PDF|yellow] [[BBR_v2.pdf#page=1&selection=54,5,133,2&color=yellow|BBR_v2, p.1]]
> > idth and Round-trip propagation time") uses recent measurements of a transport connection's delivery rate, round-trip time, and packet loss rate to build an explicit model of the network path. BBR then uses this model to control both how fast it sends data and the maximum volume of data it allows in flight in the network at any time. Relative to loss-based co
> 
> 

+ rectangler selection
![[BBR_v2.pdf#page=1&rect=58,307,514,552|BBR_v2, p.1|658]]
### 4.5 vim-im-control
+ vim中输入法切换, 需要将get current im改为fcitx5-remote -n, 将on insert enter改为fcitx5-remote -s {{im}}
### 4.6 pdf-better-export
+ 将标题导出为pdf的书签
### 4.7 excalidraw
+ 极度方便的画图软件
![[demo.excalidraw|309]]
### 4.8 canvas
+ 官方自带白板工具，比excalidraw更轻量化和简洁
![[canvas.canvas|canvas]]
### 4.9 keyboard analysis
+ 快捷键查看
### 4.10 settings search
+ 更好的搜索设置内容
### 4.11 anuppuccin
+ 主题美化，可选择主题，checkbox/callout等样式
- [ ] Unchecked
- [x] Checked
- [>] Rescheduled
- [<] Scheduled
- [!] Important
- [-] Cancelled
- [/] In Progress
- [?] Question
- [*] Star
- [n] Note
- [l] Location
- [i] Information
- [I] Idea
- [S] Amount
- [p] Pro
- [c] Con
- [b] Bookmark
- ["] Quote
- [u] Up
- [d] Down
- [w] Win
- [k] Key
- [f] Fire
- [0] Speech bubble 0
> [!Note]
 
> [!Abstract]
 
> [!Todo]
 
> [!info]
 
> [!tip]
 
> [!success]
 
> [!question]
 
> [!warning]

> [!failure]

> [!danger]

> [!bug]

> [!example]

> [!quote]
### 4.12 style settings
+ 便于设置各插件的样式
### 4.13 paste image rename
+ 粘贴图片时允许自命名
### 4.14 clear unused image
+ 自动清理未引用附件/图片
### 4.15 attachflow
+ 附件重命名
+ 链接删除(类似linux硬链接的删除)
+ ! 图片缩放
### 4.16 easy typing
+ 中文输入增强
+ 中英文，行内代码，latex 公式，链接，数字之间自动添加空格：中英 en $x+y=z$ `hello`
+ 中文符号自动配对且光标移到中间：【】，《》
+ 行内代码使用 tab 可跳到\`后,
+ 连续两次，；！。可转为英文符号，连续 3 次$可以转换为公式块
+ 自定义正则：对某些正则定义的 pattern 不进行格式化

### 4.17 tasks
- [ ] tbd
### 4.18 open in new tab
+ 每次从新窗口打开而不是覆盖旧窗口
### 4.19 hover editors
+ 鼠标在链接上，点 ctrl 可预览
### 4.20 kanban
+ 使用看板列出不同笔记
### 4.21 projects
+ 管理一个项目/文件夹中的笔记，支持日历，看板，表格等多种视图
### 4.22 calendar
+ 方便管理日历，周历

| name | age |
| ---- | --- |
| lee  | 2   |
