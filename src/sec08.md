主页：[教程介绍](../README.md)，上一节：[第07节](sec07.md)，下一节：[第09节](sec09.md)


# 定义子对象

## 一个简单的编辑器

在上一节中，我们制作了一个非常简单的文件查看器，现在我们继续重写它，把它变成非常简单的编辑器，它的源文件在 tfe1.c（text file editor 1）中。

GtkTextView 具有多行编辑功能。因此，我们不需要从头开始编写程序，我们只需在文件查看器中添加两个东西：

- 指向 GFile 实例的指针
- 用于写入文件的函数

有几种方法可以存储 GFile 的详细信息，例如：

- 使用全局变量
- 创建一个子对象，并在子对象中保存 GFile 实例的指针

使用全局变量很容易实现，例如定义一个足够大的指向 GFile 的指针数组：

~~~C
GFile *f[20];
~~~

变量 `f[i]` 对应于与第 i 个 GtkNotebookPage 关联的文件，然而这有两个问题。第一个是数组的大小问题，如果用户提供了太多参数（上例中超过 20 个），则无法存储额外的指向 GFile 实例的指针。二是程序维护难度加大。到目前为止，我们的程序很简单，但是如果继续开发它，程序的大小将会增长。一般来说，程序越大，跟踪和维护全局变量就越困难，全局变量可以在整个程序的任何地方使用和更改。

就维护而言，创建子对象是一个不错的方法（如果你了解面向对象编程，应该很容易理解本节的内容）。您需要注意的一件事是 “子对象” 和 “子控件”之间的区别，子对象包含并扩展其父对象，子控件只涉及UI上的包含关系。

![Child object of GtkTextView](image/child.png)

我们将 TfeTextView 定义为 GtkTextView 的子对象，它拥有 GtkTextView 所拥有的一切。具体来说，TfeTextView 有一个 GtkTextbuffer，它对应于 TfeTextView 内部的 GtkTextView。另一个重要的事情是 TfeTextView 还有一个额外的指向 GFile 的指针。

这就是 GObjects 的工作方式，理解 Gobject 的理论比较困难的，特别是对于初学者。因此，我将向您展示如何编写代码并避免理论方面的问题。如果您想了解 GObject 系统，请参阅[单独的教程](https://github.com/ToshioCP/Gobject-tutorial)。

## 如何定义 GtkTextView 的子对象

让我们定义 TfeTextView 对象，它是 GtkTextView 的子对象。首先看下面的程序：

~~~C
#define TFE_TYPE_TEXT_VIEW tfe_text_view_get_type ()
G_DECLARE_FINAL_TYPE (TfeTextView, tfe_text_view, TFE, TEXT_VIEW, GtkTextView)

struct _TfeTextView
{
  GtkTextView parent;
  GFile *file;
};

G_DEFINE_TYPE (TfeTextView, tfe_text_view, GTK_TYPE_TEXT_VIEW);

static void
tfe_text_view_init (TfeTextView *tv) {
}

static void
tfe_text_view_class_init (TfeTextViewClass *class) {
}

void
tfe_text_view_set_file (TfeTextView *tv, GFile *f) {
  tv -> file = f;
}

GFile *
tfe_text_view_get_file (TfeTextView *tv) {
  return tv -> file;
}

GtkWidget *
tfe_text_view_new (void) {
  return GTK_WIDGET (g_object_new (TFE_TYPE_TEXT_VIEW, NULL));
}
~~~

如果你对这个程序的背景理论感到好奇，可以查看 [GObject API Reference](https://docs.gtk.org/gobject/) 或参考 [GObject教程](https://github.com/ToshioCP/Gobject-tutorial)。对于初学者来说，不需要知道太多关于 GObject 的知识，只需记住以下说明就足够了：

- TfeTextView 分为两部分：Tfe 和 TextView。Tfe 被称为前缀、命名空间或模块，TextView 被称为对象。
- 有三种不同的标识符模式。TfeTextView（驼峰式）、tfe\_text\_view（这个用来写函数）和TFE\_TEXT\_VIEW（这个用来转换指向TfeTextView类型的指针）
- 首先，将 TFE\_TYPE\_TEXT\_VIEW 宏定义为 tfe\_text\_view\_get\_type()，名称始终为 (prefix)\_TYPE\_(object) 且字母为大写。替换文本始终是 (prefix)\_(object)\_get\_type () 并且字母是小写的
- 接下来，使用 G\_DECLARE\_FINAL\_TYPE 宏，参数是驼峰式的子对象名称、带下划线的小写、前缀（大写）、对象（带下划线的大写）和父对象名称（驼峰式）
- 声明结构体 \_TfeTextView，下划线是必须的，第一个成员是父对象，请注意，这不是指针，而是对象本身，第二个成员和之后的成员是子对象的成员，TfeTextView 结构有一个指向 GFile 实例的指针作为成员
- 使用 G\_DEFINE\_TYPE 宏。参数是驼峰大小写的子对象名称，带下划线的小写字母和父对象类型（前缀）\_TYPE\_（模块）
- 定义实例初始化函数 (tfe\_text\_view\_init)，通常你不需要做任何事情
- 定义类初始化函数（tfe\_text\_view\_class\_init），您无需在此对象中执行任何操作
- 编写要添加的功能代码（tfe\_text\_view\_set\_file 和 tfe\_text\_view\_get\_file），`tv` 是一个指向 TfeTextView 对象实例的指针，它是一个用标签 \_TfeTextView 声明的 C 结构，因此，该结构有一个成员 `file` 作为指向 GFile 实例的指针，`tv->file = f` 是将`f` 赋值给`tv` 指向的结构的成员`file`，这个例子显示了如何在子对象中扩展父对象的功能
- 编写一个函数来创建一个实例，它的名字是（前缀）\_（对象）\_new，如果父对象函数需要参数，这个函数也需要它们，你也可以添加一些自己的参数，使用 g\_object\_new 函数创建实例，参数是 (prefix)\_TYPE\_(object)、初始化属性的列表和 NULL，在此代码中不需要初始化任何属性，并且返回值被强制转换为 GtkWidget。

这个程序并不完美，它有一些问题，稍后会进行修改。

## close-request 信号

想象一下你正在使用这个编辑器，首先，您使用参数运行编辑器，参数是文件名。编辑器读取文件并显示包含文件文本的窗口，然后编辑文本。完成编辑后，退出编辑器，编辑器在窗口关闭之前更新文件。

GtkWindow 在关闭之前会发出 “close-request” 信号，我们可以将此信号和处理程序 “before_close” 进行连接，处理程序是一个 C 函数。当一个函数连接到某个信号时，我们称其为处理程序（handler）。发出信号 “close-request” 时调用函数 “before_close”。

~~~C
g_signal_connect (win, "close-request", G_CALLBACK (before_close), NULL);
~~~

参数 `win` 是一个 GtkApplicationWindow，其中定义了信号 “close-request”，`before_close` 是处理程序。`G_CALLBACK` 强制转换处理程序的类型为回调函数，`before_close` 的程序如下：

~~~c
 1 static gboolean
 2 before_close (GtkWindow *win, gpointer user_data) {
 3   GtkWidget *nb = GTK_WIDGET (user_data);
 4   GtkWidget *scr;
 5   GtkWidget *tv;
 6   GFile *file;
 7   char *pathname;
 8   GtkTextBuffer *tb;
 9   GtkTextIter start_iter;
10   GtkTextIter end_iter;
11   char *contents;
12   unsigned int n;
13   unsigned int i;
14 
15   n = gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb));
16   for (i = 0; i < n; ++i) {
17     scr = gtk_notebook_get_nth_page (GTK_NOTEBOOK (nb), i);
18     tv = gtk_scrolled_window_get_child (GTK_SCROLLED_WINDOW (scr));
19     file = tfe_text_view_get_file (TFE_TEXT_VIEW (tv));
20     tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
21     gtk_text_buffer_get_bounds (tb, &start_iter, &end_iter);
22     contents = gtk_text_buffer_get_text (tb, &start_iter, &end_iter, FALSE);
23     if (! g_file_replace_contents (file, contents, strlen (contents), NULL, TRUE, G_FILE_CREATE_NONE, NULL, NULL, NULL)) {
24       pathname = g_file_get_path (file);
25       g_print ("ERROR : Can't save %s.", pathname);
26       g_free (pathname);
27     }
28     g_free (contents);
29   }
30   return FALSE;
31 }
~~~

项目左侧的数字是源代码中的行号。

- 15: 获取 `nb` 中页面的页数。
- 16-29：关于每个页面的索引的 For 循环。
- 17-19：获取 GtkScrolledWindow、TfeTextView 和指向 GFile 的指针，指针在 `app_open` 处理程序运行时存储，稍后会展示相关代码
- 20-22: 获取 GtkTextBuffer 和 文件内容 “contents”。`start_iter` 和 `end_iter` 是缓冲区的迭代器用于表示 buffer 中的位置，我现在不想解释它们，因为这需要很多时间，可以暂时记住这些术语。
- 23-27：将 “contents” 写入文件，如果失败，则输出错误消息
- 28：释放 “contents”

## tfe1.c 代码

下面是完整的 `tfe1.c`：

~~~C
  1 #include <gtk/gtk.h>
  2 
  3 /* Define TfeTextView Widget which is the child object of GtkTextView */
  4 
  5 #define TFE_TYPE_TEXT_VIEW tfe_text_view_get_type ()
  6 G_DECLARE_FINAL_TYPE (TfeTextView, tfe_text_view, TFE, TEXT_VIEW, GtkTextView)
  7 
  8 struct _TfeTextView
  9 {
 10   GtkTextView parent;
 11   GFile *file;
 12 };
 13 
 14 G_DEFINE_TYPE (TfeTextView, tfe_text_view, GTK_TYPE_TEXT_VIEW);
 15 
 16 static void
 17 tfe_text_view_init (TfeTextView *tv) {
 18 }
 19 
 20 static void
 21 tfe_text_view_class_init (TfeTextViewClass *class) {
 22 }
 23 
 24 void
 25 tfe_text_view_set_file (TfeTextView *tv, GFile *f) {
 26   tv -> file = f;
 27 }
 28 
 29 GFile *
 30 tfe_text_view_get_file (TfeTextView *tv) {
 31   return tv -> file;
 32 }
 33 
 34 GtkWidget *
 35 tfe_text_view_new (void) {
 36   return GTK_WIDGET (g_object_new (TFE_TYPE_TEXT_VIEW, NULL));
 37 }
 38 
 39 /* ---------- end of the definition of TfeTextView ---------- */
 40 
 41 static gboolean
 42 before_close (GtkWindow *win, gpointer user_data) {
 43   GtkWidget *nb = GTK_WIDGET (user_data);
 44   GtkWidget *scr;
 45   GtkWidget *tv;
 46   GFile *file;
 47   char *pathname;
 48   GtkTextBuffer *tb;
 49   GtkTextIter start_iter;
 50   GtkTextIter end_iter;
 51   char *contents;
 52   unsigned int n;
 53   unsigned int i;
 54 
 55   n = gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb));
 56   for (i = 0; i < n; ++i) {
 57     scr = gtk_notebook_get_nth_page (GTK_NOTEBOOK (nb), i);
 58     tv = gtk_scrolled_window_get_child (GTK_SCROLLED_WINDOW (scr));
 59     file = tfe_text_view_get_file (TFE_TEXT_VIEW (tv));
 60     tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 61     gtk_text_buffer_get_bounds (tb, &start_iter, &end_iter);
 62     contents = gtk_text_buffer_get_text (tb, &start_iter, &end_iter, FALSE);
 63     if (! g_file_replace_contents (file, contents, strlen (contents), NULL, TRUE, G_FILE_CREATE_NONE, NULL, NULL, NULL)) {
 64       pathname = g_file_get_path (file);
 65       g_print ("ERROR : Can't save %s.", pathname);
 66       g_free (pathname);
 67     }
 68     g_free (contents);
 69   }
 70   return FALSE;
 71 }
 72 
 73 static void
 74 app_activate (GApplication *app, gpointer user_data) {
 75   g_print ("You need to give filenames as arguments.\n");
 76 }
 77 
 78 static void
 79 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint, gpointer user_data) {
 80   GtkWidget *win;
 81   GtkWidget *nb;
 82   GtkWidget *lab;
 83   GtkNotebookPage *nbp;
 84   GtkWidget *scr;
 85   GtkWidget *tv;
 86   GtkTextBuffer *tb;
 87   char *contents;
 88   gsize length;
 89   char *filename;
 90   int i;
 91 
 92   win = gtk_application_window_new (GTK_APPLICATION (app));
 93   gtk_window_set_title (GTK_WINDOW (win), "file editor");
 94   gtk_window_maximize (GTK_WINDOW (win));
 95 
 96   nb = gtk_notebook_new ();
 97   gtk_window_set_child (GTK_WINDOW (win), nb);
 98 
 99   for (i = 0; i < n_files; i++) {
100     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, NULL)) {
101       scr = gtk_scrolled_window_new ();
102       tv = tfe_text_view_new ();
103       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
104       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
105       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
106 
107       tfe_text_view_set_file (TFE_TEXT_VIEW (tv),  g_file_dup (files[i]));
108       gtk_text_buffer_set_text (tb, contents, length);
109       g_free (contents);
110       filename = g_file_get_basename (files[i]);
111       lab = gtk_label_new (filename);
112       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
113       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
114       g_object_set (nbp, "tab-expand", TRUE, NULL);
115       g_free (filename);
116     } else if ((filename = g_file_get_path (files[i])) != NULL) {
117         g_print ("No such file: %s.\n", filename);
118         g_free (filename);
119     } else
120         g_print ("No valid file is given\n");
121   }
122   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0) {
123     g_signal_connect (win, "close-request", G_CALLBACK (before_close), nb);
124     gtk_widget_show (win);
125   } else
126     gtk_window_destroy (GTK_WINDOW (win));
127 }
128 
129 int
130 main (int argc, char **argv) {
131   GtkApplication *app;
132   int stat;
133 
134   app = gtk_application_new ("com.github.ToshioCP.tfe1", G_APPLICATION_HANDLES_OPEN);
135   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
136   g_signal_connect (app, "open", G_CALLBACK (app_open), NULL);
137   stat =g_application_run (G_APPLICATION (app), argc, argv);
138   g_object_unref (app);
139   return stat;
140 }
~~~

- 107：将指向 GFile 的指针设置为 TfeTextView，`files[i]` 是一个指向 GFile 结构的指针，它将被系统释放，所以你需要复制它。`g_file_dup` 复制给定的 GFile 结构。
- 123：连接 “close-request” 信号和 “before_close” 处理程序，第四个参数称为用户数据，它被提供给信号处理程序，因此，`nb` 被作为第二个参数提供给 `before_close`。

现在编译并运行它，`tfe` 目录中有一个示例文件，键入`./a.out taketori.txt`，修改内容并关闭窗口，确保文件已修改来测试关闭时的自动保存效果。

现在我们有了一个非常简单的编辑器，我们需要更多的功能，如打开、保存、另存为、更改字体等，我们将在下一节及之后添加这些功能。

主页：[教程介绍](../README.md)，上一节：[第07节](sec07.md)，下一节：[第09节](sec09.md)
