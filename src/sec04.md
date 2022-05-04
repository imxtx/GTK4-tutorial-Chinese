主页：[教程介绍](../README.md)，上一节：[第03节](sec03.md)，下一节：[第05节](sec05.md)

# 控件介绍 (1)

## GtkLabel, GtkButton 和 GtkBox

### GtkLabel

在上一节中，我们创建了一个窗口并将其显示在屏幕上。接下来我们在此窗口中添加控件。最简单的控件是 GtkLabel。它是一个带有文本的控件。

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app, gpointer user_data) {
 5   GtkWidget *win;
 6   GtkWidget *lab;
 7 
 8   win = gtk_application_window_new (GTK_APPLICATION (app));
 9   gtk_window_set_title (GTK_WINDOW (win), "lb1");
10   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
11 
12   lab = gtk_label_new ("Hello.");
13   gtk_window_set_child (GTK_WINDOW (win), lab);
14 
15   gtk_widget_show (win);
16 }
17 
18 int
19 main (int argc, char **argv) {
20   GtkApplication *app;
21   int stat;
22 
23   app = gtk_application_new ("com.github.ToshioCP.lb1", G_APPLICATION_FLAGS_NONE);
24   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
25   stat =g_application_run (G_APPLICATION (app), argc, argv);
26   g_object_unref (app);
27   return stat;
28 }
29 
~~~

将程序保存为 `lb1.c`，编译执行：

    $ comp lb1
    $ ./a.out

窗口中间出现了一个字符串 "Hello."：

![Screenshot of the label](image/screenshot_lb1.png)

程序 `pr4.c` 和 `lb1.c` 只有细微的差别，我们可以用 `diff` 命令来查看文件的差别：

~~~
$ cd misc; diff pr4.c lb1.c
5a6
>   GtkWidget *lab;
8c9
<   gtk_window_set_title (GTK_WINDOW (win), "pr4");
---
>   gtk_window_set_title (GTK_WINDOW (win), "lb1");
9a11,14
> 
>   lab = gtk_label_new ("Hello.");
>   gtk_window_set_child (GTK_WINDOW (win), lab);
> 
18c23
<   app = gtk_application_new ("com.github.ToshioCP.pr4", G_APPLICATION_FLAGS_NONE);
---
>   app = gtk_application_new ("com.github.ToshioCP.lb1", G_APPLICATION_FLAGS_NONE);
~~~

结果显示:

- 新增了一个 `lab` 变量
- 窗口的标题（title）变了
- 创建了一个标签（label）控件，并且设置为窗口的子控件

函数 `gtk_window_set_child (GTK_WINDOW (win), lab)` 使标签 `lab` 成为窗口 `win` 的子控件。注意子控件不同于子对象。对象有父子关系，控件也有父子关系。但这两种关系是完全不同的，不要混淆。 在程序“lb1.c”中，“lab”是“win”的子控件。子控件始终位于屏幕上的父控件中，例如上图中的序窗口包括了标签。

窗口 `win` 没有任何父控件，我们称这样的窗口为顶级窗口，一个应用程序可以有多个顶级窗口。

### GtkButton

下一个介绍的控件是 GtkButton，GtkButton 在屏幕上显示一个按钮，按钮上面可以有标签或者图标。在下一个小节我们会创建一个带标签的按钮。当按钮被点击时，它会发送一个 “clicked” 信号。下面的程序演示了如何捕获该信号，然后执行对应的 handler。

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 click_cb (GtkButton *btn, gpointer user_data) {
 5   g_print ("Clicked.\n");
 6 }
 7 
 8 static void
 9 app_activate (GApplication *app, gpointer user_data) {
10   GtkWidget *win;
11   GtkWidget *btn;
12 
13   win = gtk_application_window_new (GTK_APPLICATION (app));
14   gtk_window_set_title (GTK_WINDOW (win), "lb2");
15   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
16 
17   btn = gtk_button_new_with_label ("Click me");
18   gtk_window_set_child (GTK_WINDOW (win), btn);
19   g_signal_connect (btn, "clicked", G_CALLBACK (click_cb), NULL);
20 
21   gtk_widget_show (win);
22 }
23 
24 int
25 main (int argc, char **argv) {
26   GtkApplication *app;
27   int stat;
28 
29   app = gtk_application_new ("com.github.ToshioCP.lb2", G_APPLICATION_FLAGS_NONE);
30   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
31   stat =g_application_run (G_APPLICATION (app), argc, argv);
32   g_object_unref (app);
33   return stat;
34 }
35 
~~~

查看第 17 到 19 行，首先，它创建一个带有“Click me”标签的 GtkButton 实例 `btn`。然后，将按钮添加到窗口 `win` 作为子控件。最后，将按钮的“clicked”信号连接到处理程序 `click_cb`。因此，如果点击了`btn`，就会调用函数`click_cb`，后缀“cb”表示“回调”。

将程序保存为 `lb2.c` 然后编译执行：

![Screenshot of the label](image/screenshot_lb2.png)

屏幕上出现了一个带有按钮的窗口，单击按钮（它是一个大按钮，您可以在窗口中的任何位置单击），字符串“Clicked” 会出现在终端中，表明 handler 是通过单击按钮调用的。

在调试的时候，我们可以使用 `g_print` 检查 handler 是否被调用，我们继续更改处理程序，让按钮实现关闭窗口的功能，下面的代码是`lb3.c`：

~~~C
 1 static void
 2 click_cb (GtkButton *btn, gpointer user_data) {
 3   GtkWindow *win = GTK_WINDOW (user_data);
 4   gtk_window_destroy (win);
 5 }
 6 
 7 static void
 8 app_activate (GApplication *app, gpointer user_data) {
 9   GtkWidget *win;
10   GtkWidget *btn;
11 
12   win = gtk_application_window_new (GTK_APPLICATION (app));
13   gtk_window_set_title (GTK_WINDOW (win), "lb3");
14   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
15 
16   btn = gtk_button_new_with_label ("Quit");
17   gtk_window_set_child (GTK_WINDOW (win), btn);
18   g_signal_connect (btn, "clicked", G_CALLBACK (click_cb), win);
19 
20   gtk_widget_show (win);
21 }
~~~

程序 `lb2.c` 和 `lb3.c` 差别如下：

~~~
$ cd misc; diff lb2.c lb3.c
5c5,6
<   g_print ("Clicked.\n");
---
>   GtkWindow *win = GTK_WINDOW (user_data);
>   gtk_window_destroy (win);
14c15
<   gtk_window_set_title (GTK_WINDOW (win), "lb2");
---
>   gtk_window_set_title (GTK_WINDOW (win), "lb3");
17c18
<   btn = gtk_button_new_with_label ("Click me");
---
>   btn = gtk_button_new_with_label ("Quit");
19c20
<   g_signal_connect (btn, "clicked", G_CALLBACK (click_cb), NULL);
---
>   g_signal_connect (btn, "clicked", G_CALLBACK (click_cb), win);
29c30
<   app = gtk_application_new ("com.github.ToshioCP.lb2", G_APPLICATION_FLAGS_NONE);
---
>   app = gtk_application_new ("com.github.ToshioCP.lb3", G_APPLICATION_FLAGS_NONE);
35d35
< 
~~~

有三处变化：

- 删掉了函数 `g_print`，新加了两行代码
- 按钮 `btn` 的标签从 "Click me" 改为 "Quit"
- 函数 `g_signal_connect` 的第四个参数由 `NULL` 改为 `win`

最重要的变化是 `g_signal_connect` 的第四个参数，此参数在 [GObject API Reference](https://docs.gtk.org/gobject/func.signal_connect.html) 中的 `g_signal_connect` 定义中被描述为“要传递给处理程序的数据”。因此，指向 GtkApplicationWindow 的指针 `win` 作为第二个参数 `user_data` 传递给处理程序。然后处理程序将其转换为指向 GtkWindow 的指针并调用 gtk_window_destroy 来销毁顶级窗口，然后应用程序退出。

### GtkBox

GtkWindow 和 GtkApplicationWindow 只能有一个子控件，如果要在一个窗口中添加两个或多个控件，则需要一个容器控件，而 GtkBox 便是容器之一。它将两个或多个子控件排列成一行或一列，以下步骤显示了在窗口中添加两个便按钮的方法：

- 创建一个 GtkApplicationWindow 实例
- 创建一个 GtkBox 实例并将其作为子控件添加到 GtkApplicationWindow 中
- 创建一个 GtkButton 实例并将其附加到 GtkBox
- 创建另一个 GtkButton 实例并将其附加到 GtkBox

控件的连接关系如下图所示：

![Parent-child relationship](image/box.png)

下面是对应的程序 `lb4.c`：

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 click1_cb (GtkButton *btn, gpointer user_data) {
 5   const gchar *s;
 6 
 7   s = gtk_button_get_label (btn);
 8   if (g_strcmp0 (s, "Hello.") == 0)
 9     gtk_button_set_label (btn, "Good-bye.");
10   else
11     gtk_button_set_label (btn, "Hello.");
12 }
13 
14 static void
15 click2_cb (GtkButton *btn, gpointer user_data) {
16   GtkWindow *win = GTK_WINDOW (user_data);
17   gtk_window_destroy (win);
18 }
19 
20 static void
21 app_activate (GApplication *app, gpointer user_data) {
22   GtkWidget *win;
23   GtkWidget *box;
24   GtkWidget *btn1;
25   GtkWidget *btn2;
26 
27   win = gtk_application_window_new (GTK_APPLICATION (app));
28   gtk_window_set_title (GTK_WINDOW (win), "lb4");
29   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
30 
31   box = gtk_box_new (GTK_ORIENTATION_VERTICAL, 5);
32   gtk_box_set_homogeneous (GTK_BOX (box), TRUE);
33   gtk_window_set_child (GTK_WINDOW (win), box);
34 
35   btn1 = gtk_button_new_with_label ("Hello.");
36   g_signal_connect (btn1, "clicked", G_CALLBACK (click1_cb), NULL);
37 
38   btn2 = gtk_button_new_with_label ("Quit");
39   g_signal_connect (btn2, "clicked", G_CALLBACK (click2_cb), win);
40 
41   gtk_box_append (GTK_BOX (box), btn1);
42   gtk_box_append (GTK_BOX (box), btn2);
43 
44   gtk_widget_show (win);
45 }
46 
47 int
48 main (int argc, char **argv) {
49   GtkApplication *app;
50   int stat;
51 
52   app = gtk_application_new ("com.github.ToshioCP.lb4", G_APPLICATION_FLAGS_NONE);
53   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
54   stat =g_application_run (G_APPLICATION (app), argc, argv);
55   g_object_unref (app);
56   return stat;
57 }
~~~

我们来看一下函数 `app_activate`，在创建 GtkApplicationWindow 实例之后，接着创建了一个 GtkBox：

    box = gtk_box_new(GTK_ORIENTATION_VERTICAL, 5);
    gtk_box_set_homogeneous (GTK_BOX (box), TRUE);

第一个参数表示 GtkBox 的子控件应该竖着排列，第二个参数表示自控间之间的距离。函数 `gtk_box_set_homogeneous` 表示子控件有同样的大小。

接着创建了两个按钮 `btn1` 和 `btn2`，然后连接了它们的“clicked”信号，接着把这两个按钮附加到了 `box` 上。

![Screenshot of the box](image/screenshot_lb4.png)

按钮 `btn1` 的功能是切换它显示的标签。按钮 `btn2` 的功能是销毁顶级窗口并退出程序。

主页：[教程介绍](../README.md)，上一节：[第03节](sec03.md)，下一节：[第05节](sec05.md)
