主页：[教程介绍](../README.md)，上一节：[第06节](sec06.md)，下一节：[第08节](sec08.md)


# 控件介绍 (3)

## Open 信号

### G\_APPLICATION\_HANDLES\_OPEN 标志

上一节我们使用 GtkTextView、GtkTextBuffer 和 GtkScrolledWindow 控件创建了一个简单的编辑器，我们现在来添加一个函数以读取文件，然后显示到编辑器中。实现此功能的方式有很多，因为这是给初学者的教程，所以我们选择最简单的一种方式来实现。

当程序启动时，我们将要打开的文件名作为参数：

    $ ./a.out filename

它将打开文件并将其内容插入到 GtkTextBuffer，为此，我们需要知道 GtkApplication（或 GApplication）是如何识别参数的，这在 [GIO API Reference, Application](https://docs.gtk.org/gio/class.Application.html) 中有相应的说明。

创建 GtkApplication 时，会提供一个标志（类型为 GApplicationFlags）作为参数：

~~~C
GtkApplication *
gtk_application_new (const gchar *application_id, GApplicationFlags flags);
~~~

本教程只解释两个标志，`G_APPLICATION_FLAGS_NONE` 和 `G_APPLICATION_HANDLES_OPEN`，如果要处理命令行参数，则需要 `G_APPLICATION_HANDLES_COMMAND_LINE` 标志。[GIO API Reference, g\_application\_run](https://docs.gtk.org/gio/method.Application.run.html) 中介绍了使用方法，并且该标志在 [GIO API Reference, ApplicationFlags](https://docs.gtk.org/gio/flags.ApplicationFlags.html) 中也有说明：

~~~
GApplicationFlags' Members

G_APPLICATION_FLAGS_NONE  Default. (No argument allowed)
  ... ... ...
G_APPLICATION_HANDLES_OPEN  This application handles opening files (in the primary instance).
  ... ... ...
~~~

总共有十个标志，但到目前为止我们只需要其中两个。我们已经使用了`G_APPLICATION_FLAGS_NONE`，这是最简单的标志，表示不允许任何参数，如果在运行应用程序时提供参数，则会发生错误。

标志 `G_APPLICATION_HANDLES_OPEN` 是第二简单的选项，它允许参数，但只允许文件名作为参数。应用程序假定所有参数都是文件名，我们将在创建 GtkApplication 时使用此标志：

~~~C
app = gtk_application_new ("com.github.ToshioCP.tfv3", G_APPLICATION_HANDLES_OPEN);
~~~

### open 信号

现在，当应用程序启动时，可以发出两个信号。

- activate 信号 --- 当没有参数时发出这个信号。
- open 信号 --- 当至少有一个参数时发出这个信号。

“open” 信号的处理程序定义如下：

~~~C
void user_function (GApplication *application,
                   gpointer      files,
                   gint          n_files,
                   gchar        *hint,
                   gpointer      user_data)
~~~

参数是：

- `application` --- 应用程序（通常是 GtkApplication）
- `files` --- GFiles 数组 [数组长度=n\_files] [元素类型是 GFile]
- `n_files` --- `files` 的元素个数
- `hint` --- 调用实例提供的提示字符串（通常可以忽略）
- `user_data` --- 连接 handler 时提供的数据

接下来将描述如何读取指定文件（GFile）。

## 创建一个文件查看器

### 什么是文件查看器？

我们要创建的文件查看器是显示文本文件内容的程序，我们的文件查看器将按如下方式工作：

- 当给定参数时，它将第一个参数视为文件名并打开它。
- 如果打开文件成功，它会读取文件的内容并将其插入到 GtkTextBuffer 中，然后显示窗口。
- 如果打开文件失败，会显示错误信息并退出。
- 如果没有参数，它将显示错误消息并退出。
- 如果有两个或多个参数，则忽略第二个和后面的其他参数。

执行此操作的程序如下所示：

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app, gpointer user_data) {
 5   g_print ("You need a filename argument.\n");
 6 }
 7 
 8 static void
 9 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint, gpointer user_data) {
10   GtkWidget *win;
11   GtkWidget *scr;
12   GtkWidget *tv;
13   GtkTextBuffer *tb;
14   char *contents;
15   gsize length;
16   char *filename;
17 
18   win = gtk_application_window_new (GTK_APPLICATION (app));
19   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
20 
21   scr = gtk_scrolled_window_new ();
22   gtk_window_set_child (GTK_WINDOW (win), scr);
23 
24   tv = gtk_text_view_new ();
25   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
26   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
27   gtk_text_view_set_editable (GTK_TEXT_VIEW (tv), FALSE);
28   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
29 
30   if (g_file_load_contents (files[0], NULL, &contents, &length, NULL, NULL)) {
31     gtk_text_buffer_set_text (tb, contents, length);
32     g_free (contents);
33     if ((filename = g_file_get_basename (files[0])) != NULL) {
34       gtk_window_set_title (GTK_WINDOW (win), filename);
35       g_free (filename);
36     }
37     gtk_widget_show (win);
38   } else {
39     if ((filename = g_file_get_path (files[0])) != NULL) {
40       g_print ("No such file: %s.\n", filename);
41       g_free (filename);
42     }
43     gtk_window_destroy (GTK_WINDOW (win));
44   }
45 }
46 
47 int
48 main (int argc, char **argv) {
49   GtkApplication *app;
50   int stat;
51 
52   app = gtk_application_new ("com.github.ToshioCP.tfv3", G_APPLICATION_HANDLES_OPEN);
53   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
54   g_signal_connect (app, "open", G_CALLBACK (app_open), NULL);
55   stat = g_application_run (G_APPLICATION (app), argc, argv);
56   g_object_unref (app);
57   return stat;
58 }
~~~

保存为 `tfv3.c`，编译运行：

    $ comp tfv3
    $ ./a.out tfv3.c

![File viewer](image/screenshot_tfv3.png)

现在我们来解释程序 `tfv3.c` 是如何工作的，首先，函数 `main` 与之前的版本相比只有两个变化。

- `G_APPLICATION_FLAGS_NONE` 被`G_APPLICATION_HANDLES_OPEN` 替换
- 添加了`g_signal_connect (app, "open", G_CALLBACK (on_open), NULL)`

接下来，添加处理程序 `app_activate`，它只负责输出错误消息然后退出程序（没有创建窗口程序会自动退出）。

主要功能在处理程序 `app_open` 中：

- 创建 GtkApplicationWindow、GtkScrolledWindow、GtkTextView 和 GtkTextBuffer 并将它们连接在一起
- 在 GtktextView 中将换行模式设置为 `GTK_WRAP_WORD_CHAR`
- 将 GtkTextView 设置为不可编辑，因为我们创建的程序不是编辑器，而只是查看器
- 读取文件并将文本插入GtkTextBuffer（将在后面详细解释）
- 如果文件未打开，则输出错误消息并销毁窗口，使应用程序退出

以下是程序中最重要的文件读取部分：

~~~C
if (g_file_load_contents (files[0], NULL, &contents, &length, NULL, NULL)) {
  gtk_text_buffer_set_text (tb, contents, length);
  g_free (contents);
  if ((filename = g_file_get_basename (files[0])) != NULL) {
    gtk_window_set_title (GTK_WINDOW (win), filename);
    g_free (filename);
  }
  gtk_widget_show (win);
} else {
  if ((filename = g_file_get_path (files[0])) != NULL) {
    g_print ("No such file: %s.\n", filename);
    g_free (filename);
  }
  gtk_window_destroy (GTK_WINDOW (win));
}
~~~

函数 `g_file_load_contents` 将文件内容加载到缓冲区中，它会自动分配缓冲区内存并让 “contents” 指向该缓冲区，缓冲区的长度设置为 `length`。如果文件的内容成功加载，则返回 TRUE，如果发生错误，则返回 FALSE。

如果此函数成功，则将内容插入 GtkTextBuffer，释放 `contents` 指向的缓冲区，设置窗口的标题，释放 `filename` 指向的内存，然后显示窗口。如果失败，它会输出错误消息并销毁窗口，导致程序退出。

## GtkNotebook

GtkNotebook 是一个有多个选项卡并包含多个子项的容器控件，显示的子项取决于选择了哪个选项卡。

![GtkNotebook](image/screenshot_gtk_notebook.png)

看上面的截图，左边是启动时的窗口。它显示文件 “pr1.c”，文件名在左侧选项卡中。单击右侧选项卡后，将显示文件 `tfv1.c` 的内容。

GtkNotebook 控件作为 GtkApplicationWindow 的子控件插入，每个显示的文件都包含一个 GtkScrolledWindow，代码如 `tfv4.c` 所示：

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app, gpointer user_data) {
 5   g_print ("You need a filename argument.\n");
 6 }
 7 
 8 static void
 9 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint, gpointer user_data) {
10   GtkWidget *win;
11   GtkWidget *nb;
12   GtkWidget *lab;
13   GtkNotebookPage *nbp;
14   GtkWidget *scr;
15   GtkWidget *tv;
16   GtkTextBuffer *tb;
17   char *contents;
18   gsize length;
19   char *filename;
20   int i;
21 
22   win = gtk_application_window_new (GTK_APPLICATION (app));
23   gtk_window_set_title (GTK_WINDOW (win), "file viewer");
24   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
25   gtk_window_maximize (GTK_WINDOW (win));
26 
27   nb = gtk_notebook_new ();
28   gtk_window_set_child (GTK_WINDOW (win), nb);
29 
30   for (i = 0; i < n_files; i++) {
31     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, NULL)) {
32       scr = gtk_scrolled_window_new ();
33       tv = gtk_text_view_new ();
34       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
35       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
36       gtk_text_view_set_editable (GTK_TEXT_VIEW (tv), FALSE);
37       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
38 
39       gtk_text_buffer_set_text (tb, contents, length);
40       g_free (contents);
41       if ((filename = g_file_get_basename (files[i])) != NULL) {
42         lab = gtk_label_new (filename);
43         g_free (filename);
44       } else
45         lab = gtk_label_new ("");
46       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
47       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
48       g_object_set (nbp, "tab-expand", TRUE, NULL);
49     } else if ((filename = g_file_get_path (files[i])) != NULL) {
50         g_print ("No such file: %s.\n", filename);
51         g_free (filename);
52     } else
53         g_print ("No valid file is given\n");
54   }
55   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0)
56     gtk_widget_show (win);
57   else
58     gtk_window_destroy (GTK_WINDOW (win));
59 }
60 
61 int
62 main (int argc, char **argv) {
63   GtkApplication *app;
64   int stat;
65 
66   app = gtk_application_new ("com.github.ToshioCP.tfv4", G_APPLICATION_HANDLES_OPEN);
67   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
68   g_signal_connect (app, "open", G_CALLBACK (app_open), NULL);
69   stat = g_application_run (G_APPLICATION (app), argc, argv);
70   g_object_unref (app);
71   return stat;
72 }
~~~

大多数更改都在函数 “app_open” 中，以下各项左边的数字是源代码中的行号。

- 11-13：定义了变量`nb`、`lab`和`nbp`，它们将分别指向一个新的 GtkNotebook、GtkLabel 和GtkNotebookPage 控件。
- 23：窗口的标题设置为 “file viewer”
- 25：窗口的大小设置为最大，因为大窗口适合文件查看器
- 27-28：GtkNotebook 被创建并插入到 GtkApplicationWindow
- 30-59：For 循环。每个循环对应一个文件名参数，`files[i]` 是第 i 个 GFile 对象
- 32-37：创建 GtkScrollledWindow, GtkTextView，并 GtkTextView 中获取对应的 GtkTextBuffer，GtkTextView 作为一个子控件连接到 GtkScrolledWindow。注意每个文件都有自己所属的控件，它们是在 for 循环中创建的
- 39-40：将文件的内容插入 GtkTextBuffer 并释放 `contents` 指向的内存
- 41-43：获取文件名并使用文件名创建 GtkLabel，然后释放 `filename`
- 44-45：如果 `filename` 为 NULL，则使用空字符串创建 GtkLabel
- 46: 将 GtkScrolledWindow 作为一个 Page 附加到 GtkNotebook，并传入 GtkLabel 做为选项卡名称，此时会自动创建一个 GtkNoteBookPage 控件，并将 GtkScrolledWindow 控件附加到 GtkNotebookPage，因此，结构是这样的：

    ~~~
        GtkNotebook -- GtkNotebookPage -- GtkScrolledWindow
    ~~~

- 47: 获取 GtkNotebookPage 控件并将 `nbp` 设置为指向此 GtkNotebookPage
- 48: GtkNotebookPage 有一个名为 “tab-expand” 的属性，如果设置为 TRUE，则选项卡会尽可能长地水平扩展，如果为 FALSE，则选项卡的宽度由标签的大小决定。`g_object_set` 是设置对象属性的通用函数。请参阅 [GObject API Reference，g\_object\_set](https://docs.gtk.org/gobject/method.Object.set.html)。
- 49-51：如果无法读取文件，则会显示 “No such file” 的消息，并释放 `filename` 缓冲区
- 52-53: 如果 `filename` 为 NULL，则输出 “No valid file is given” 消息
- 55-58：如果至少读取了一个文件，则 GtkNotebookPage 的数量大于零，if 结果为真则显示窗口，否则销毁窗口，从而导致程序退出

主页：[教程介绍](../README.md)，上一节：[第06节](sec06.md)，下一节：[第08节](sec08.md)
