主页：[教程介绍](../README.md)，上一节：[第08节](sec08.md)，下一节：[第10节](sec10.md)

# 9 UI 文件和 GtkBuilder

## 新建、打开和保存按钮

在节教程中，我们创建了一简单的编辑器。它能在 `app_open` 函数中打开文件，并能够在更改之后，在关闭窗口的时候写回磁盘。这些功能都能够正常工作，但是不太友好。

如果能够有新建、打开、保存和关闭按钮那就更好了，本节教程将为我们的编辑器添加这些功能按钮，之后我们会介绍相关的信号和处理函数（handler）。

![Screenshot of the file editor](image/screenshot_tfe2.png)

上面的截图显示了编辑器的最终效果。源代码文件 `tfe2.c` 中的 `app_open` 函数如下图所示：

~~~C
 1 static void
 2 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint, gpointer user_data) {
 3   GtkWidget *win;
 4   GtkWidget *nb;
 5   GtkWidget *lab;
 6   GtkNotebookPage *nbp;
 7   GtkWidget *scr;
 8   GtkWidget *tv;
 9   GtkTextBuffer *tb;
10   char *contents;
11   gsize length;
12   char *filename;
13   int i;
14 
15   GtkWidget *boxv;
16   GtkWidget *boxh;
17   GtkWidget *dmy1;
18   GtkWidget *dmy2;
19   GtkWidget *dmy3;
20   GtkWidget *btnn; /* button for new */
21   GtkWidget *btno; /* button for open */
22   GtkWidget *btns; /* button for save */
23   GtkWidget *btnc; /* button for close */
24 
25   win = gtk_application_window_new (GTK_APPLICATION (app));
26   gtk_window_set_title (GTK_WINDOW (win), "file editor");
27   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
28 
29   boxv = gtk_box_new (GTK_ORIENTATION_VERTICAL, 0);
30   gtk_window_set_child (GTK_WINDOW (win), boxv);
31 
32   boxh = gtk_box_new (GTK_ORIENTATION_HORIZONTAL, 0);
33   gtk_box_append (GTK_BOX (boxv), boxh);
34 
35   dmy1 = gtk_label_new(NULL); /* dummy label for left space */
36   gtk_label_set_width_chars (GTK_LABEL (dmy1), 10);
37   dmy2 = gtk_label_new(NULL); /* dummy label for center space */
38   gtk_widget_set_hexpand (dmy2, TRUE);
39   dmy3 = gtk_label_new(NULL); /* dummy label for right space */
40   gtk_label_set_width_chars (GTK_LABEL (dmy3), 10);
41   btnn = gtk_button_new_with_label ("New");
42   btno = gtk_button_new_with_label ("Open");
43   btns = gtk_button_new_with_label ("Save");
44   btnc = gtk_button_new_with_label ("Close");
45 
46   gtk_box_append (GTK_BOX (boxh), dmy1);
47   gtk_box_append (GTK_BOX (boxh), btnn);
48   gtk_box_append (GTK_BOX (boxh), btno);
49   gtk_box_append (GTK_BOX (boxh), dmy2);
50   gtk_box_append (GTK_BOX (boxh), btns);
51   gtk_box_append (GTK_BOX (boxh), btnc);
52   gtk_box_append (GTK_BOX (boxh), dmy3);
53 
54   nb = gtk_notebook_new ();
55   gtk_widget_set_hexpand (nb, TRUE);
56   gtk_widget_set_vexpand (nb, TRUE);
57   gtk_box_append (GTK_BOX (boxv), nb);
58 
59   for (i = 0; i < n_files; i++) {
60     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, NULL)) {
61       scr = gtk_scrolled_window_new ();
62       tv = tfe_text_view_new ();
63       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
64       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
65       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
66 
67       tfe_text_view_set_file (TFE_TEXT_VIEW (tv),  g_file_dup (files[i]));
68       gtk_text_buffer_set_text (tb, contents, length);
69       g_free (contents);
70       filename = g_file_get_basename (files[i]);
71       lab = gtk_label_new (filename);
72       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
73       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
74       g_object_set (nbp, "tab-expand", TRUE, NULL);
75       g_free (filename);
76     } else if ((filename = g_file_get_path (files[i])) != NULL) {
77         g_print ("No such file: %s.\n", filename);
78         g_free (filename);
79     } else
80         g_print ("No valid file is given\n");
81   }
82   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0) {
83     gtk_widget_show (win);
84   } else
85     gtk_window_destroy (GTK_WINDOW (win));
86 }
~~~

该函数的主要功能是创建窗口中的一些控件。

- 25-27: 创建了 GtkApplicationWindow 实例，并设置了标题和默认的大小。
- 29-30: 创建了 GtkBox 实例 `boxv`，这是一个竖直排列的 box，并且是 GtkApplicationWindow的子控件，并且它本身也拥有两个子控件（第一个子控件是一个横向排列的 box， 第二个子控件是一个 GtkNotebook
- 32-33: 创建了 GtkBox 实例 `boxh`，然后附加到了 `boxv` 作为其第一个子控件
- 35-40: 创建了 3 个用于站位的标签（label），标签 `dmy1` 和 `dmy3` 中字符的宽度为 10，标签 `dmy2` 设置了自动扩展宽度，能够使中间的标签尽可能地占据两组按钮之间的空白区域。
- 41-44: 创建了 4 个按钮
- 46-52: 将这些 GtkLabel 和 GtkButton 附加到 `boxh`
- 54-57: 创建了一个 GtkNotebook 实例并将竖直方向和水平方向的自动扩展属性设置为 True，意味着 GtkNotebook 会尽可能地扩展其高度和宽度，这个 GtkNotebook 被添加为 `boxv` 的第二个子控件。

我们可以看到，创建这些控件的代码一共有 33（57-25+1）行，此外我们还创建了很多变量（`boxv`, `boxh`, `dmy1`, ...），除了为了创建这些控件，然后添加到他们的父控件上，大多数这些变量都是不必要存在的。那么是否有办法来减少这些工作呢？

Gtk 提供了 GtkBuilder，它能够读取 UI 文件数据，然后自动创建所有窗口和子控件，它能够帮助我们大量减少不必要的繁杂的工作。

## UI 文件

首先我们来看看 UI 文件 `tfe3.ui`，我们将使用这个文件来定义编辑器控件的层次结构。

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <object class="GtkApplicationWindow" id="win">
 4     <property name="title">file editor</property>
 5     <property name="default-width">600</property>
 6     <property name="default-height">400</property>
 7     <child>
 8       <object class="GtkBox" id="boxv">
 9         <property name="orientation">GTK_ORIENTATION_VERTICAL</property>
10         <child>
11           <object class="GtkBox" id="boxh">
12             <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
13             <child>
14               <object class="GtkLabel" id="dmy1">
15                 <property name="width-chars">10</property>
16               </object>
17             </child>
18             <child>
19               <object class="GtkButton" id="btnn">
20                 <property name="label">New</property>
21               </object>
22             </child>
23             <child>
24               <object class="GtkButton" id="btno">
25                 <property name="label">Open</property>
26               </object>
27             </child>
28             <child>
29               <object class="GtkLabel" id="dmy2">
30                 <property name="hexpand">TRUE</property>
31               </object>
32             </child>
33             <child>
34               <object class="GtkButton" id="btns">
35                 <property name="label">Save</property>
36               </object>
37             </child>
38             <child>
39               <object class="GtkButton" id="btnc">
40                 <property name="label">Close</property>
41               </object>
42             </child>
43             <child>
44               <object class="GtkLabel" id="dmy3">
45                 <property name="width-chars">10</property>
46               </object>
47             </child>
48           </object>
49         </child>
50         <child>
51           <object class="GtkNotebook" id="nb">
52             <property name="hexpand">TRUE</property>
53             <property name="vexpand">TRUE</property>
54           </object>
55         </child>
56       </object>
57     </child>
58   </object>
59 </interface>
~~~

这个文件的结构是一个 XML 文件。其中以 `<` 开始并以 `>` 结束的称为标签（tag），XML 中有两种标签，一种是开始标签，一种是结束标签。例如 `<interface>` 就是一个开始标签，而 `</interface>` 是一个结束标签。UI 文件就是分别以这两个标签开始和结尾。此外一些标签，例如 object 标签，可以在开始标签中附带 class 和 id 属性。

- 1：文件的第一行是 XML 声明，它声明了 XML 的版本为 1.0，文件的编码为 UTF-8，如果没有第一行，GtkBuilder 也会从 ui 文件创建控件，但是 ui 文件必须使用 UTF-8 编码，否则 GtkBuilder 不能识别并且会抛出错误然后终止程序。
- 3-6：创建了一个 object，它的类属性为 `GtkApplicationWindow`，id 属性为 `win`，这是编辑器的顶级窗口。此外定义了该窗口的三个属性，`title` 即窗口标题为 "file editor"，`default-width` 是窗口的默认宽度为 600，`default-height` 是窗口的默认高度为 400。
- 7：child 标签的含义是其中的内容作为一个子控件。例如，第 7 行 id 为 "boxv" 的 GtkBox 对象是控件 `win` 的子控件。

我们可以对比一下 ui 文件和 `app_open` 函数中的 25-57 行，它们其实完成了同样的功能。

你可以使用 GTK 提供的 `gtk4-builder-tool` 工具检查你的 ui 文件是否符合要求。

- `gtk4-builder-tool validate <ui file name>` 检查 ui 文件，如果 ui 文件中有任何语法错误，`gtk4-builder-tool` 会输出错误信息。
- `gtk4-builder-tool simplify <ui file name>` 能够简化 ui 文件，并且输出简化后的结果。如果提供了 `--replace` 选项，那么该工具会将原来的 ui 文件替换为简化后的版本。如果 ui 文件中指定的某些属性本省就是默认值，那么简化程序会将这些属性移除。此外一些值也会被简化，例如 True 和 False 分别会被简化为 1 和 0，但是显然使用 True 和 False 更容易维护。我们可以在编写界面时使用尽可能容易维护的方式编写 ui 文件，在需要使用这些 ui 文件的时候可以简化之后再使用。

另外，在编译之前使用这个工具检查你的 ui 文件是非常有用且有必要的。

## GtkBuilder

如下面的程序所示，GtkBuilder 根据 ui 文件来构建控件。

~~~C
GtkBuilder *build;

build = gtk_builder_new_from_file ("tfe3.ui");
win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
~~~

`gtk_builder_new_from_file` 读取 ui 文件，然后根据 ui 文件中的内容创建控件，然后返回一个 GtkBuilder 对象，其中包含了所有的控件以及它们的层次结构、属性等相关信息。函数 `gtk_builder_get_object (build, "win")` 返回一个指向 `win` 的指针，这是 ui 文件中窗口的 id 属性。所有的控件都根据 ui 文件中的定义被自动组合在了一起，不再需要我们手动编写代码来管理控件的父子关系。

改用 ui 文件之后，我们在代码里面只需要 `win` 和 `nb` 这两个变量，而不再需要额外的变量。这么做使我们减小了源代码的体积。

下面是用 `diff` 命令查看了使用 ui 文件前后代码的变化：

~~~
$ cd tfe; diff tfe2.c tfe3.c
58a59
>   GtkBuilder *build;
60,103c61,65
<   GtkWidget *boxv;
<   GtkWidget *boxh;
<   GtkWidget *dmy1;
<   GtkWidget *dmy2;
<   GtkWidget *dmy3;
<   GtkWidget *btnn; /* button for new */
<   GtkWidget *btno; /* button for open */
<   GtkWidget *btns; /* button for save */
<   GtkWidget *btnc; /* button for close */
< 
<   win = gtk_application_window_new (GTK_APPLICATION (app));
<   gtk_window_set_title (GTK_WINDOW (win), "file editor");
<   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
< 
<   boxv = gtk_box_new (GTK_ORIENTATION_VERTICAL, 0);
<   gtk_window_set_child (GTK_WINDOW (win), boxv);
< 
<   boxh = gtk_box_new (GTK_ORIENTATION_HORIZONTAL, 0);
<   gtk_box_append (GTK_BOX (boxv), boxh);
< 
<   dmy1 = gtk_label_new(NULL); /* dummy label for left space */
<   gtk_label_set_width_chars (GTK_LABEL (dmy1), 10);
<   dmy2 = gtk_label_new(NULL); /* dummy label for center space */
<   gtk_widget_set_hexpand (dmy2, TRUE);
<   dmy3 = gtk_label_new(NULL); /* dummy label for right space */
<   gtk_label_set_width_chars (GTK_LABEL (dmy3), 10);
<   btnn = gtk_button_new_with_label ("New");
<   btno = gtk_button_new_with_label ("Open");
<   btns = gtk_button_new_with_label ("Save");
<   btnc = gtk_button_new_with_label ("Close");
< 
<   gtk_box_append (GTK_BOX (boxh), dmy1);
<   gtk_box_append (GTK_BOX (boxh), btnn);
<   gtk_box_append (GTK_BOX (boxh), btno);
<   gtk_box_append (GTK_BOX (boxh), dmy2);
<   gtk_box_append (GTK_BOX (boxh), btns);
<   gtk_box_append (GTK_BOX (boxh), btnc);
<   gtk_box_append (GTK_BOX (boxh), dmy3);
< 
<   nb = gtk_notebook_new ();
<   gtk_widget_set_hexpand (nb, TRUE);
<   gtk_widget_set_vexpand (nb, TRUE);
<   gtk_box_append (GTK_BOX (boxv), nb);
< 
---
>   build = gtk_builder_new_from_file ("tfe3.ui");
>   win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
>   gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
>   nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
>   g_object_unref(build);
138c100
<   app = gtk_application_new ("com.github.ToshioCP.tfe2", G_APPLICATION_HANDLES_OPEN);
---
>   app = gtk_application_new ("com.github.ToshioCP.tfe3", G_APPLICATION_HANDLES_OPEN);
~~~

`60,103c61,65` 表示 44 (=103-60+1) 行变为 5 (=65-61+1) 行，减少了 39 行不必要的代码。使用 ui 文件不仅能够减少 C 源文件的代码行数，而且 ui 文件比代码更具有层次结构，非常容易维护。

现在我们来看看 `tfe3.c` 中的 `app_open` 函数：

~~~C
 1 static void
 2 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint, gpointer user_data) {
 3   GtkWidget *win;
 4   GtkWidget *nb;
 5   GtkWidget *lab;
 6   GtkNotebookPage *nbp;
 7   GtkWidget *scr;
 8   GtkWidget *tv;
 9   GtkTextBuffer *tb;
10   char *contents;
11   gsize length;
12   char *filename;
13   int i;
14   GtkBuilder *build;
15 
16   build = gtk_builder_new_from_file ("tfe3.ui");
17   win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
18   gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
19   nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
20   g_object_unref(build);
21   for (i = 0; i < n_files; i++) {
22     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, NULL)) {
23       scr = gtk_scrolled_window_new ();
24       tv = tfe_text_view_new ();
25       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
26       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
27       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
28 
29       tfe_text_view_set_file (TFE_TEXT_VIEW (tv),  g_file_dup (files[i]));
30       gtk_text_buffer_set_text (tb, contents, length);
31       g_free (contents);
32       filename = g_file_get_basename (files[i]);
33       lab = gtk_label_new (filename);
34       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
35       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
36       g_object_set (nbp, "tab-expand", TRUE, NULL);
37       g_free (filename);
38     } else if ((filename = g_file_get_path (files[i])) != NULL) {
39         g_print ("No such file: %s.\n", filename);
40         g_free (filename);
41     } else
42         g_print ("No valid file is given\n");
43   }
44   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0) {
45     gtk_widget_show (win);
46   } else
47     gtk_window_destroy (GTK_WINDOW (win));
48 }
~~~

你可以在 [src/tfe](tfe) 查看完整的 `tfe3.c`。

### 使用 ui 字符创

GtkBuilder 可以直接使用字符串来创建控件，使用函数 `gtk_builder_new_from_string` 即可：

~~~C
char *uistring;

uistring =
"<interface>"
  "<object class="GtkApplicationWindow" id="win">"
    "<property name=\"title\">file editor</property>"
    "<property name=\"default-width\">600</property>"
    "<property name=\"default-height\">400</property>"
    "<child>"
      "<object class=\"GtkBox\" id=\"boxv\">"
        "<property name="orientation">GTK_ORIENTATION_VERTICAL</property>"
... ... ...
... ... ...
"</interface>";

build = gtk_builder_new_from_stringfile (uistring);
~~~

这种方法有利有弊，ui 结构直接写在了源码中，好处是程序运行是不需要额外的 ui 文件。缺点是在 C 语言中的编写很长的字符串会让程序难以维护，而且我们使用了很多双引号，并且需要转义属性的双引号。如果你想这么做的话，可以编写一个能够把 ui 文件转换为 C 字符创的程序，这样可以使用 ui 文件来设计界面，而程序又不需要在运行时用到 ui 文件。转换方法如下：

- 在每个双引号前面都添加反斜杠。
- 在每行 ui 文件的前后添加双引号（参考上面的例子）。

### 使用 Gresource

使用 Gresource 和使用字符创类似，但是 Gresource 是压缩的二进制数据，而不是文本文件。我们可以使用工具将 ui 文件编译为二进制资源文件，不仅是 ui 文件，而且还能将图片、音频等编译为二进制资源文件。在编译之后，这些文件会被打包为一个 Gresource 对象。

我们需要给资源编译器 `glib-compile-resources` 提供一个 XML 文件，用以描述用到的资源。

~~~xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <gresources>
3   <gresource prefix="/com/github/ToshioCP/tfe3">
4     <file>tfe3.ui</file>
5   </gresource>
6 </gresources>
~~~

- 2：`gresources`标签可以包含多个 gresources（gresource标签），这个 xml 只有一个 gresource。
- 3：gresource 有一个 prefex(前缀)属性`/com/github/ToshioCP/tfe3`，类似于资源的根目录。
- 4：gresource 有文件 `tfe3.ui`，该文件位于 `/com/github/ToshioCP/tfe3/tfe3.ui`，这里可以看到父标签前缀的作用了，如果要添加更多文件，可以在第 4 行和第 5 行之间插入它们。

将此 xml 文本保存到 `tfe3.gresource.xml`，可以使用参数 `--help` 显示 gresource 编译器 `glib-compile-resources`的用法。

~~~
$ LANG=C glib-compile-resources --help
Usage:
  glib-compile-resources [OPTION?] FILE

Compile a resource specification into a resource file.
Resource specification files have the extension .gresource.xml,
and the resource file have the extension called .gresource.

Help Options:
  -h, --help                   Show help options

Application Options:
  --version                    Show program version and exit
  --target=FILE                Name of the output file
  --sourcedir=DIRECTORY        The directories to load files referenced in FILE from (default: current directory)
  --generate                   Generate output in the format selected for by the target filename extension
  --generate-header            Generate source header
  --generate-source            Generate source code used to link in the resource file into your code
  --generate-dependencies      Generate dependency list
  --dependency-file=FILE       Name of the dependency file to generate
  --generate-phony-targets     Include phony targets in the generated dependency file
  --manual-register            Don?t automatically create and register resource
  --internal                   Don?t export functions; declare them G_GNUC_INTERNAL
  --external-data              Don?t embed resource data in the C file; assume it's linked externally instead
  --c-name                     C identifier name used for the generated source code

~~~

运行资源编译器：

    $ glib-compile-resources tfe3.gresource.xml --target=resources.c --generate-source

编译器生成了一个 C 源文件 `resources.c`，修改`tfe3.c`并保存为`tfe3_r.c`：

~~~C
#include "resources.c"
... ... ...
... ... ...
build = gtk_builder_new_from_resource ("/com/github/ToshioCP/tfe3/tfe3.ui");
... ... ...
... ... ...
~~~

编译并运行，窗口出现，与本页开头的屏幕截图相同。

主页：[教程介绍](../README.md)，上一节：[第08节](sec08.md)，下一节：[第10节](sec10.md)
