主页：[教程介绍](../README.md)，上一节：[第04节](sec04.md)，下一节：[第06节](sec06.md)

# 5 控件介绍 (2)

## GtkTextView, GtkTextBuffer 和 GtkScrolledWindow

### GtkTextView 和 GtkTextBuffer

GtkTextView 是一个用于多行编辑的控件，GtkTextBuffer 是一个文本缓冲器，用于保存文本，可以连接到 GtkTextView。下面是程序 `tfv1.c`：

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app, gpointer user_data) {
 5   GtkWidget *win;
 6   GtkWidget *tv;
 7   GtkTextBuffer *tb;
 8   gchar *text;
 9 
10   text =
11       "Once upon a time, there was an old man who was called Taketori-no-Okina. "
12       "It is a japanese word that means a man whose work is making bamboo baskets.\n"
13       "One day, he went into a mountain and found a shining bamboo. "
14       "\"What a mysterious bamboo it is!,\" he said. "
15       "He cut it, then there was a small cute baby girl in it. "
16       "The girl was shining faintly. "
17       "He thought this baby girl is a gift from Heaven and took her home.\n"
18       "His wife was surprized at his tale. "
19       "They were very happy because they had no children. "
20       ;
21   win = gtk_application_window_new (GTK_APPLICATION (app));
22   gtk_window_set_title (GTK_WINDOW (win), "Taketori");
23   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
24 
25   tv = gtk_text_view_new ();
26   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
27   gtk_text_buffer_set_text (tb, text, -1);
28   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
29 
30   gtk_window_set_child (GTK_WINDOW (win), tv);
31 
32   gtk_widget_show (win);
33 }
34 
35 int
36 main (int argc, char **argv) {
37   GtkApplication *app;
38   int stat;
39 
40   app = gtk_application_new ("com.github.ToshioCP.tfv1", G_APPLICATION_FLAGS_NONE);
41   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
42   stat = g_application_run (G_APPLICATION (app), argc, argv);
43   g_object_unref (app);
44   return stat;
45 }
~~~

在程序第25行创建了一个 GtkTextView 实例，并用指针 `tv` 保存了该实例。当一个 GtkTextView 创建时，一个 GtkTextBuffer 也会被创建出来，并自动连接到 GtkTextView 上面。我们可以将 GtkTextBuffer 简称为 buffer。接着下一行代码使用指针 `tb` 保存了该 buffer，随后 10 ~ 20 行的文本被传给了该 buffer，现在该 buffer 中已经有了文本，我们可以在 GtkTextView 上显示或者编辑该文本。

GtkTextView 有一个换行模式，当它设置为 `GTK_WRAP_WORD_CHAR` 时，文本会在单词之间换行，如果单词的长度已经超过一行，也会在字母间换行。

第30行，`tv` 被附加到了 `win` 作为一个子控件。

编译运行：

![GtkTextView](image/screenshot_tfv1.png)

可以看到窗口中有一个光标，您可以在 GtkTextView 上添加或删除任何字符，并且您的更改会被实时保存在 GtkTextBuffer 中。如果添加超出窗口限制的更多字符，则高度会增加并且窗口会扩展。如果高度大于显示屏的高度，您将没法再控制窗口的大小，这是程序中的一个 Bug。这可以通过在 GtkApplicationWindow 和 GtkTextView 之间添加一个 GtkScrolledWindow 来解决。

### GtkScrolledWindow

我们需要做的是：

- 创建一个 GtkScrolledWindow 并将其作为 GtkApplicationWindow 的子控件插入
- 将 GtkTextView 控件作为字控件插入 GtkScrolledWindow。

修改`tfv1.c`并保存为`tfv2.c`，这两个文件之间的差异很小：

~~~
$ cd tfv; diff tfv1.c tfv2.c
5a6
>   GtkWidget *scr;
24a26,28
>   scr = gtk_scrolled_window_new ();
>   gtk_window_set_child (GTK_WINDOW (win), scr);
> 
30c34
<   gtk_window_set_child (GTK_WINDOW (win), tv);
---
>   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
40c44
<   app = gtk_application_new ("com.github.ToshioCP.tfv1", G_APPLICATION_FLAGS_NONE);
---
>   app = gtk_application_new ("com.github.ToshioCP.tfv2", G_APPLICATION_FLAGS_NONE);
~~~

下面是完整的 `tfv2.c` 程序：

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app, gpointer user_data) {
 5   GtkWidget *win;
 6   GtkWidget *scr;
 7   GtkWidget *tv;
 8   GtkTextBuffer *tb;
 9   gchar *text;
10 
11   text =
12       "Once upon a time, there was an old man who was called Taketori-no-Okina. "
13       "It is a japanese word that means a man whose work is making bamboo baskets.\n"
14       "One day, he went into a mountain and found a shining bamboo. "
15       "\"What a mysterious bamboo it is!,\" he said. "
16       "He cut it, then there was a small cute baby girl in it. "
17       "The girl was shining faintly. "
18       "He thought this baby girl is a gift from Heaven and took her home.\n"
19       "His wife was surprized at his tale. "
20       "They were very happy because they had no children. "
21       ;
22   win = gtk_application_window_new (GTK_APPLICATION (app));
23   gtk_window_set_title (GTK_WINDOW (win), "Taketori");
24   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
25 
26   scr = gtk_scrolled_window_new ();
27   gtk_window_set_child (GTK_WINDOW (win), scr);
28 
29   tv = gtk_text_view_new ();
30   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
31   gtk_text_buffer_set_text (tb, text, -1);
32   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
33 
34   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
35 
36   gtk_widget_show (win);
37 }
38 
39 int
40 main (int argc, char **argv) {
41   GtkApplication *app;
42   int stat;
43 
44   app = gtk_application_new ("com.github.ToshioCP.tfv2", G_APPLICATION_FLAGS_NONE);
45   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
46   stat = g_application_run (G_APPLICATION (app), argc, argv);
47   g_object_unref (app);
48   return stat;
49 }
~~~

编译并运行，请注意，当您键入大量字符时，这次窗口不会扩展，它会滚动并显示一个滚动条。

主页：[教程介绍](../README.md)，上一节：[第04节](sec04.md)，下一节：[第06节](sec06.md)
