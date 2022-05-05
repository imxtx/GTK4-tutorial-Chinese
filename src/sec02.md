主页：[教程介绍](../README.md)，上一节：[第01节](sec01.md)，下一节：[第03节](sec03.md)

# 2 在 Linux 上安装 Gtk4

本节教程描述如何在 Linux 发行版上安装 Gtk4。

有三种方式安装 Gtk4：

- 使用 Linux 自带的包管理系统安装，例如 apt
- 从源文件编译安装
- 使用 gnome-boxes 安装 Gnome 40

## 使用包管理器安装

这是最简单的安装方式，也是推荐的安装方式。我已经在 Ubuntu 21.04 上安装了 Gtk4。

输入如下命令进行安装：

```bash
$ sudo apt-get install libgtk-4-bin libgtk-4-common libgtk-4-dev libgtk-4-doc
```

Fedora, Arch, Debian 和 OpenSUSE 等也可以使用对应的包管理器进行安装，请参考 [Installing GTK from packages](https://www.gtk.org/docs/installations/linux#installing-gtk-from-packages)。

以下表格列出了本身就支持 Gtk4 的发行版：

|   发行版    |           版本             |Gtk4 |    Gnome40    |
|:----------:|:-------------------------:|:---:|:-------------:|
|   Fedora   |            35             |4.4.2|    Gnome41    |
|   Ubuntu   |           21.10           | 4.4 |Gnome40 (40.4.0)|
|   Debian   |     bookworm(testing)     |4.6.2| Gnome40 (40.4) |
|    Arch    |      rolling release      |4.6.3|    Gnome42    |
|  OpenSUSE  |Tumbleweed(rolling release)|4.6.2|    Gnome42    |

如果你已经通过此方法安装了 Gtk4，那么无需再阅读下面的内容。

## 使用源码编译安装

如果你的操作系统没有预编译的 Gtk4 安装包，那么你可能需要编译源码来安装。另外如果想使用最新版的 Gtk4 也需编译源码安装。

我在2021年1月使用源码安装了 Gtk4。因此，以下信息是旧信息，尤其是每个软件的版本可能与目前有些出入。有关最新信息，请参阅 [Gtk API Reference, Building GTK](https://docs.gtk.org/gtk4/building.html)。


### Gtk4 安装的先决条件

- Linux操作系统。例如，Ubuntu 20.10 或 20.04LTS。其他发行版可能也没问题。
- gcc、meson、ninja、git、wget等开发包。
- 以下每个软件都需要这些开发包。

### 安装目标

我在 `$HOME/local` 目录下安装了 Gtk4。这是一个私人用户区。

如果要安装在系统区，`/opt/gtk4` 目录是不错的选择之一。[Gtk API Reference, Building GTK](https://docs.gtk.org/gtk4/building.html) 给出了在 `/opt/gtk4` 目录的安装示例。

不要将它安装到默认的 `/usr/local`。很多不是基于 Gtk4 构建的 Ubuntu 应用程序会使用此目录下系统自带的 Gtk 库。因此，安装在默认位置风险很高，很可能会发生不好的事情。

### 安装到 Ubuntu 20.10

大多数必需的库都包含在 Ubuntu 20.10 中。因此，可以使用 apt-get 命令来安装它们。您不需要从源码安装它们。您可以跳过下面有关依赖库安装（Glib、Pango、Gdk-pixbuf 和 Gtk-doc）的小节。

### Glib 安装

如果您的 Ubuntu 是 20.04LTS，则需要从源码安装。请检查您的库的版本，如果它低于必要的版本，请从源码安装。

例如，

```bash
$ pkg-config --modversion glib-2.0
2.64.6
```

必要的版本是 2.66.0 或更高版本。 因此，上面的示例表明您需要安装 Glib。我安装了 2.67.1，这是当时（2021年1月）的最新版本。下载 Glib 源文件，然后解压缩并提取文件:

```bash
$ wget https://download.gnome.org/sources/glib/2.67/glib-2.67.1.tar.xz
$ tar -Jxf glib-2.67.1.tar.xz
```

编译 Glib 可能需要额外的一些库，你可以使用 meson 来找到它们：

```bash
$ meson --prefix $HOME/local _build
```

使用 apt-get 来安装这些依赖：

```bash
$ sudo apt-get install -y  libpcre2-dev libffi-dev
```

装完依赖库之后即可编译 Glib：

```bash
$ rm -rf _build
$ meson --prefix $HOME/local _build
$ ninja -C _build
$ ninja -C _build install
```

由于后面的安装还需要 Glib，所以我们可以设置一些环境变量，以便后面的编译过程能够找到 Glib，把下面的内容保存为文件 `env.sh`：

```bash
# compiler
CPPFLAGS="-I$HOME/local/include"
LDFLAGS="-L$HOME/local/lib"
PKG_CONFIG_PATH="$HOME/local/lib/pkgconfig:$HOME/local/lib/x86_64-linux-gnu/pkgconfig"
export CPPFLAGS LDFLAGS PKG_CONFIG_PATH
# linker
LD_LIBRARY_PATH="$HOME/local/lib/x86_64-linux-gnu/"
PATH="$HOME/local/bin:$PATH"
export LD_LIBRARY_PATH PATH
# gsetting
export GSETTINGS_SCHEMA_DIR=$HOME/local/share/glib-2.0/schemas
```

然后使用 . (dot) 或者 source 命令将这些环境变量导入到当前的bash：

```bash
$ . env.sh
```

或者

```bash
$ source env.sh
```

上面的命令会执行 `env.sh` 中的命令来更改当前 shell 中的环境变量。

### 安装 Pango

下载后解压：

    $ wget https://download.gnome.org/sources/pango/1.48/pango-1.48.0.tar.xz
    $ tar -Jxf pango-1.48.0.tar.xz

可以使用 meson 确定依赖库，然后安装所有依赖库，然后编译安装 Pango：

    $ meson --prefix $HOME/local _build
    $ ninja -C _build
    $ ninja -C _build install

上面的命令将 Pango-1.0.gir 安装在 `$HOME/local/share/gir-1.0` 目录。如果您安装 Pango 时没有指定 `--prefix` 选项，那么它将位于 `/usr/local/share/gir-1.0`。此目录 (/usr/local/share) 由一些应用程序使用。这些应用程序通过环境变量 `XDG_DATA_DIRS` 找到该目录。它是一个文本文件，保存了“共享”目录的列表，如“/usr/share”、“usr/local/share”等。现在需要将 `$HOME/local/share` 添加到 `XDG_DATA_DIRS` 中，否则后面编译会出错。

    $ export XDG_DATA_DIRS=$HOME/local/share:$XDG_DATA_DIRS

### 安装 Gdk-pixbuf 和 Gtk-doc

下载和解压：

    $ wget https://download.gnome.org/sources/gdk-pixbuf/2.42/gdk-pixbuf-2.42.2.tar.xz
    $ tar -Jxf gdk-pixbuf-2.42.2.tar.xz
    $ wget https://download.gnome.org/sources/gtk-doc/1.33/gtk-doc-1.33.1.tar.xz
    $ tar -Jxf gtk-doc-1.33.1.tar.xz

和之前一样，安装依赖包，然后编译并安装它们。

Gtk-doc 的安装会将 `gtk-doc.pc` 放在 `$HOME/local/share/pkgconfig` 下。该文件由构建工具之一的 pkg-config 使用。该目录需要添加到环境变量`PKG_CONFIG_PATH`：

    $ export PKG_CONFIG_PATH="$HOME/local/share/pkgconfig:$PKG_CONFIG_PATH"

### 安装 Gtk4

如果你想安装最新的版本，可以克隆最新的仓库：

    $ git clone https://gitlab.gnome.org/GNOME/gtk.git

如果想要安装最新的稳定版本，那么可以从 [Gnome source website](https://download.gnome.org/sources/gtk/) 下载。目前最新版本是 4.6.3 (2022年5月3日)。

编译安装：

    $ meson --prefix $HOME/local _build
    $ ninja -C _build
    $ ninja -C _build install

更多资料请参考 [Gtk4 API Reference, Building GTK](https://docs.gtk.org/gtk4/building.html)。

### 修改 env.sh

在你退出登录之后，之前手动设置的临时环境变量会消失，需要重新添加，因此修改 `env.sh`：

    # compiler
    CPPFLAGS="-I$HOME/local/include"
    LDFLAGS="-L$HOME/local/lib"
    PKG_CONFIG_PATH="$HOME/local/lib/pkgconfig:$HOME/local/lib/x86_64-linux-gnu/pkgconfig:
    $HOME/local/share/pkgconfig"
    export CPPFLAGS LDFLAGS PKG_CONFIG_PATH
    # linker
    LD_LIBRARY_PATH="$HOME/local/lib/x86_64-linux-gnu/"
    PATH="$HOME/local/bin:$PATH"
    export LD_LIBRARY_PATH PATH
    # gir
    XDG_DATA_DIRS=$HOME/local/share:$XDG_DATA_DIRS
    export XDG_DATA_DIRS
    # gsetting
    export GSETTINGS_SCHEMA_DIR=$HOME/local/share/glib-2.0/schemas
    # girepository-1.0
    export GI_TYPELIB_PATH=$HOME/local/lib/x86_64-linux-gnu/girepository-1.0

在安装 Gtk4 库之前，使用 . (dot) 或 source 命令运行。

您可能认为可以将它们添加到您的 `.profile` 中。但这是一个错误的决定。永远不要将它们写入您的`.profile`。只有在编译和运行 Gtk4 应用程序时才需要上述环境变量，否则没有必要。如果您更改了上述环境变量并运行 Gtk3 应用程序，可能会导致严重损坏。

### 编译 Gtk4 应用程序

编译 Gtk4 程序之前，需要执行上述的 `env.sh` 导入相关环境变量：

    $ . env.sh

之后即可以直接编译。例如要编译 `sample.c`，请键入以下内容。

    $ gcc `pkg-config --cflags gtk4` sample.c `pkg-config --libs gtk4`

要了解如何编译 Gtk4 应用程序，请参阅第 3 节（GtkApplication 和 GtkApplicationWindow）及之后的部分。

## 使用 gnome-boxes 安装 Fedora 34

本节的最后一部分关于 Gnome40 和 gnome-boxes。Gnome 40 是 Gnome 桌面系统的新版本。并且 Gtk4 已经预先安装在发行版中。请先查看 [Gnome 40 网站](https://forty.gnome.org/) 了解更多信息。

*但是，编译和运行 Gtk4 应用程序不需要 Gnome40。*

目前有六种选择。

- Gnome OS
- Arch Linux
- Fedora 35
- openSUSE Tumbleweed
- Ubuntu 21.10
- Debian bookworm

我已经用 gnome-boxes 安装了 Fedora 34。当时我的操作系统是 Ubuntu 21.04。使用 Gnome-boxes 在 Ubuntu 中创建一个虚拟机，然后将 Fedora 将安装到该虚拟机上。

步骤如下：

1. 下载 Fedora 34 iso 文件。[Gnome 40 网站](https://forty.gnome.org/) 末尾有下载链接。
2. 使用 apt-get 命令安装 gnome-boxes。

        $ sudo apt-get 安装 gnome-boxes

3. 运行 gnome-boxes。
4. 点击左上角的 `+` 按钮并通过点击 `Create a Virtual Machine ...` 启动创建向导。然后出现一个对话框。单击 `Operationg System Image File` 并选择已下载的 iso 文件。
5. 然后，执行 Fedora 的安装程序。按照安装程序的说明进行操作。在安装结束时，安装程​​序会指示重新启动系统。单击标题栏右侧的 并选择重新启动或关闭。
6. 之后回到 gnome-boxes 的初始窗口，窗口的左上角有一个 `Fedora 34 Workstation` 按钮。单击按钮，Fedora 将被执行。
7. 之后会出现一个设置对话框。根据向导设置 Fedora。

现在您可以使用 Fedora，它已经包含 Gtk4 库，但是需要安装Gtk4开发包。可以使用 `dnf` 安装 `gtk4.x86_64` 包：

    $ sudo dnf install gtk4.x86_64

### Gtk4 编译测试

您可以通过编译基于 Gtk4 的源文件来测试 Gtk4 开发包是否安装正确。例如尝试编译将在第 21 节中编写的 `tfe` 文本编辑器。

1. 打开浏览器
2. 打开 [Gtk4-Tutorial](https://github.com/ToshioCP/Gtk4-tutorial)
3. 点击绿色的按钮 `Code`
4. 选在 `Download ZIP` 以下载源码
5. 解压下载的文件
6. 进入目录 `src/tfe7`.
7. 编译

        $ meson _build
        bash: meson: command not found...
        Install package 'meson' to provide command 'meson'? [N/y] y

        * Waiting in queue...
        The following packages have to be installed:
        meson-0.56.2-2.fc34.noarch    High productivity build system
        ninja-build-1.10.2-2.fc34.x86_64    Small build system with a focus on speed
        vim-filesystem-2:8.2.2787-1.fc34.noarch    VIM filesystem layout
        Proceed with changes? [N/y] y

        ... ...
        ... ...

        The Meson build system
        Version: 0.56.2

        ... ...
        ... ...

        Project name: tfe
        Project version: undefined
        C compiler for the host machine: cc (gcc 11.0.0 "cc (GCC) 11.0.0 20210210 (Red Hat 11.0.0-0)")
        C linker for the host machine: cc ld.bfd 2.35.1-38
        Host machine cpu family: x86_64
        Host machine cpu: x86_64
        Found pkg-config: /usr/bin/pkg-config (1.7.3)
        Run-time dependency gtk4 found: YES 4.2.0
        Found pkg-config: /usr/bin/pkg-config (1.7.3)
        Program glib-compile-resources found: YES (/usr/bin/glib-compile-resources)
        Program glib-compile-schemas found: YES (/usr/bin/glib-compile-schemas)
        Program glib-compile-schemas found: YES (/usr/bin/glib-compile-schemas)
        Build targets in project: 4

        Found ninja-1.10.2 at /usr/bin/ninja

        $ ninja -C _build
        ninja: Entering directory `_build'
        [12/12] Linking target tfe

        $ ninja -C _build install
        ninja: Entering directory `_build'
        [0/1] Installing files.
        Installing tfe to /usr/local/bin
        Installation failed due to insufficient permissions.
        Attempting to use polkit to gain elevated privileges...
        Installing tfe to /usr/local/bin
        Installing /home/<username>/Gtk4-tutorial-main/src/tfe7/com.github.ToshioCP.tfe.gschema.xml to /usr/local/share/glib-2.0/schemas
        Running custom install script '/usr/bin/glib-compile-schemas /usr/local/share/glib-2.0/schemas/'

8. 执行

        $ tfe

之后 `tfe` 文本编辑器会显示出来。说明编译和执行已经成功了。

主页：[教程介绍](../README.md)，上一节：[第01节](sec01.md)，下一节：[第03节](sec03.md)
