# GTK4新手教程

本教程是 [Gtk4-tutorial](https://github.com/ToshioCP/Gtk4-tutorial) 的中文版本。

#### 仓库内容

本教程将教你如何使用C语言和Gtk4库开发程序。本教程主要面向初学者，因此只会设计Gtk4中比较基础的部分。本教程的内容组织如下：

- 第3节至第21节介绍一些基础控件的使用，会编写一个简单的编辑器 `tfe` (Text File Editor)。
- 第22节至第25节介绍与绘图相关的 GtkDrawingArea。
- 第26节至第29节介绍列表模型（list model）和列表视图（list view），包括 GtkListView，GtkGridView 和 GtkColumnView，另外还会介绍 GtkExpression。

#### Gtk4文档

你可以从 [Gtk API 文档](https://docs.gtk.org/gtk4/index.html) 和 [Gnome 开发者文档](https://developer.gnome.org/) 获得更多相关资料.

这两个网站是最近才上线的（2021年8月）。

旧文档可以访问 [Gtk Reference Manual](https://developer-old.gnome.org/gtk4/stable/) 和 [Gnome Developer Center](https://developer-old.gnome.org/)。新网站还在开发中，所以你可以访问旧网站。

如果你想了解 GObject 和类型系统, 可以参考 [GObject tutorial](https://github.com/ToshioCP/Gobject-tutorial)。GObject 相关的细节非常易懂，而且对于我们编写 Gtk4 程序很有帮助。

#### 参与贡献

本教程还未完成，虽然所有的代码都在 Gtk4 的基础上经过测试，可能还是会出现一些 Bug。如果你发现了任何 Bug、错误、文字等问题，可以去英文版仓库提交 [issue](https://github.com/ToshioCP/Gtk4-tutorial/issues)。中文版会跟进更新。你也可以在修改相关文件之后发起 [pull request](https://github.com/ToshioCP/Gtk4-tutorial/pulls)。在更正时请只修改 src 目录下的文件，然后运行 `rake` 重新生成输出文件。gfm 目录下的 GFM 文件会自动更新。

如果有任何问题都可以在 issue 中发布。任何问题都会帮助提升本教程的质量。

#### 如何获取HTML和PDF版本

目前中文版暂不提供HTML和PDF版本，英文版HTML和PDF版本的获取方法请参考[英文版仓库](https://github.com/ToshioCP/Gtk4-tutorial)。

## 目录

没有链接说明还未翻译。

1. [要求和许可](src/sec01.md)
2. [在Linux上安装Gtk4](src/sec02.md)
3. [GtkApplication 和 GtkApplicationWindow](src/sec03.md)
4. [控件介绍 (1)](src/sec04.md)
5. [控件介绍 (2)](src/sec05.md)
6. [字符串和内存管理](src/sec06.md)
7. [控件介绍 (3)](src/sec07.md)
8. [定义子对象](src/sec08.md)
9. UI 文件和 GtkBuilder
10. 构建系统
11. 初始化和销毁实例
12. 信号
13. TfeTextView 中的函数
14. GtkNotebook 中的函数
15. tfeapplication.c
16. tfe5 源文件
17. 菜单和行为
18. 状态行为
19. 菜单和行为的 UI 文件
20. GtkMenuButton、加速器、字体、pango 和 gsettings
21. XML模板和组合控件
22. GtkDrawingArea 和 Cairo
23. 周期性事件
24. 结合 GtkDrawingArea 和 TfeTextView
25. Tiny turtle graphics interpreter
26. GtkListView
27. GtkGridView 和激活信号
28. GtkExpression
29. GtkColumnView
