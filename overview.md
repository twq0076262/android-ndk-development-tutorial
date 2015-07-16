# Android NDK 开发教程二：概述

注意：在 Windows 上运行 NDK 需要有 [Cygwin](http://www.cygwin.com/) 支持，个人建议使用 Ubuntu 为好 。

**介绍：**

Android SDK 是一个允许 Android 应用开发人员使用 C 或 C++源文件编译并嵌入到本机源代码中的应用程序包的一组工 具。

**重要说明：**

Android NDK 只能用于 android 1.5 以上版本

**1. Android NDK 的目的：**

Android 虚拟机允许你的应用程序源代码通过 JNI 调用在本地实现的源代码，简单的说，这就意味着：

—-你的应用程序将声明一个或多个用’native’关键字的方法用来指明它们是通过本地代码实现的
例如：

```

    native byte[] loadFile(String filePath)

```

—-你必须提供包含实现这些方法的共享库（就是.so），将共享库打包到你的应用程序包 apk 中，这些库文件必须根据 标准的 Unix 约定来命名为 lib<something>.so，并且是需要包含一个标准的 JNI 的接口，例如

```

     libFileLoader.so

```


—-你的应用程序必须明确的装载这些库文件(.so文件)，比如，在程序的开始装载它，只需要简单的添加几句源代码：

```

    static {
    
    System.loadLibrary(“FileLoader”);
    
    }

```

注意：这里你不必再将前缀 lib 和后缀.so 写入。

Android NDK 对于 Android SDK 只是个组件，它可以帮你：

—-生成的 JNI 兼容的共享库可以在大于 Android1.5 平台的 ARM CPU 上运行

—-将生成的共享库拷贝到合适的程序工程路径的位置上，以保证它们自动的添加到你的 apk 包中（并且签名的）

—-在以后的版本中，我们将提供来帮助你的源代码通过远程 gdb 连接和尽可能多的源代码的信息。

而且，Android NDK 还提供：

—-一组交叉编译链（编译器、链接器等）来生成可以在 Linux，OS X 和 Windows (用 Cygwin )运行的二进制文件

—-一组与由 Android 平台提供的稳定的本地 API 列表的头文件

它们在 docs/STABLE-APIS.html 中有说明

重要提示：

记住，在以后的更新和发布平台中，Android 系统镜像中的大多数本地系统库并不是一成不变的，而是可以彻底改变， 甚至删除的

—-一个编译系统（build system）可以允许开发者写一个非常短的编译文件（build files）去描述哪个源代码需要编译，并且怎样编译。编译系统可以解决所有的 toolchain/platform/CPU/ABI 细节的问题。并且，较晚的 NDK 版 本中还添加了更多的可以不用改变开发者的编译文件的情况下的 toolchains，platforms，系统接口

**2. Android NDK 的缺点**

NDK 并不是一个可以编写通用的源代码并且可以在 Android 设备上运行的方法，你的应用程序还是需要使用 JAVA 程序，适当的处理系统事件来避免“应用程序没有反应”的对话框或者处理 Android 应用程序的生命周期

注意：可以适当的在源代码中写一个复杂的应用程序，用于启动/停止一个小型的“应用程序包”

强烈建议很好地理解的 JNI，因为许多操作在这种环境要求的开发人员，都采取具体的行动，不一定在常见典型的本机代码。这些措施包括：

—-不能通过指针直接访问 VM 的对象。比如：你不能安全的得到一个指向 String 对象的 16 位 char 数组的循环遍历

—-需要显示引用管理本机代码时候要保持处理 JNI 调用之间的 VM 对象

NDK 在 Android 平台仅仅提供了有限的本地 API 和库文件的支持的系统头文件，然而一个标准的 Android 系统镜像包括许多本地共享库，这些都应该被考虑在更新和发行版本的可以彻底改变的实现细节

如果 Android 系统库没有明确的被 NDK 明确的支持，然后应用程序不应该依赖于它提供的，或者打破了将来在各种设备上的无线系统更新

选定的系统库将逐渐被添加到稳定的 NDK API 中

**3. NDK开发实践**

下面将给出一个怎样用 Android NDK 开发本地代码的粗略的概述

（1） 把本地代码放在 $PROJECT/jni/…下，比如将 hello.c 放到 apps/hello/jni/目录下

（2） 在你的 NDK 编译系统中在 $PROJECT/jni/Android.mk 来描述你的源代码

（3） 可选：在 $PROJECT/jni/Application.mk 到你的编译系统中来详细描述你的项目，尽管你开始的话不一定需要它， 但是它允许你使用更多的 CPU 或者覆盖编译器/链接器的标记（看 docs/APPLICATION-MK.html 了解更多细节）

（4） 从你的项目的目录开始通过运行”$NDK/ndk-build”来编译你的代码，或者从子目录开始

（5） 最后一步可以 copy，万一成功，剥离共享库的应用层序需要你的应用程序的项目根目录。然后你通过通常的方法来 生成最终的 apk

现在，开始一些更 的细节

① 配置 NDK

以前的发行版本需要你运行“build/host-setup.sh”脚本来配置你的 NDK。从 release 4(NDK r)以后就完全去除了这一步

② 放置 C/C++ 代码

假如我们创建的是 test 目录，创建的代码 hello.c

把 hello.c 放到 test/jni 目录下

这个项目的位置相当于你的 Android 应用程序项目的路径

这样你就很轻松的组织起来了你想要的 jni 的目录，这里项目目录的名字和结构不会影响到最终生成的 apk，所以 你不必用类似于 com.<mycompany>.<myproject> 作为应用程序包名
 
注意，NDK 是支持 C 和 C++ 的，NDK 支持的 C++ 文件扩展名是’.cpp’，但是其他的扩展名也是可以被处理的     （看 docs/ANDROID-MK.html 了解更多）

它可以通过调整你的 Android.mk 文件来将源代码放在不同的位置

③ 创建一个 Android.mk 编译脚本

Android.mk 文件是一个小型的编译脚本，你可以在 NDK 编译系统中用它来描述你的源代码。更详细的描述在 docs/ANDROID-MK.html 中

总而言之，NDK 将你的源代码聚合到模块(modules)中，每个模块可以执行下列之一

—-一个静态库(lib<project>.a)

—-一个动态库(lib<project>.so)

你可以在 Android.mk 中定义多个模块，或者你可以编写多个 Android.mk 文件，每一个定义一个单独的模块

注意，单独的 Android.mk 也行被编译系统多次解析，以确定哪些变量没有被定义。

默认地，NDK 会通过如下的编译脚本去寻找

test/jni/Android.mk（存放位置）

如果你想定义 Android.mk 到子目录中，你需要在最高层的 Android.mk 中明确的包含它们，下面是一个帮助的方法可以实现这个功能。

```

    include $(call all-subdir-makefiles)

```

它会将所有的在子目录中的 Android.mk 文件加入到当前编译文件的路径中

④ 写一个 Application.mk 编译文件（可选）

在你的编译系统中有一个 Android.mk 文件描述模块的同时，Application.mk 文件藐视你的应用程序本身。请看 docs/APPLICATION-MK.html 文档来理解这个文件允许我们做什么。这包括

—-你的应用程序需要模块的准确清单

—-CPU 架构生成机器代码

—-可选信息，你是否需要一个 release 或者 debug build，特殊的 C/C++ 编译器标志和其他适用于所有模块的 build

这个文件是可选文件：默认地，NDK 会提供一个对于所有的在你的 Android.mk（所有的 makefiles 都在里面）中的所有模块的简单编译并且指定默认的 CPU ABI

使用 Application.mk 有两种方法：

—-把它放到 test/jni/Application.mk，它就会自动的被’ndk-build’脚本找出来

—-把它放在 NDK/<name>/Application.mk，也就是 NDK 安装的路径下，然后从 NDK 目录下执行”make APP=<name>”

这个方法是 Android NDK r4 以前的。现在仍然兼容。但是我们强烈建议你使用第一种方法，因为它更简单并且不用修改 NDK 安装树的目录。

再次看看 docs/APPLICATION-MK.html 对于它的完整说明

⑤ 调用 NDK 编译系统

用 NDK 编译成机器码的最好方法是使用”ndk-build”脚本，你还可以使用第二个，这取决于你早起常见的”$NDK/apps”子目录

在两种情况下，成功构建将 copy 应用程序所需的最终的已经剥离的二进制模块（即共享库）到应用程序的项目路径中（注意，未剥离的版本主要是用于调试目的，无需拷贝未剥离的二进制文件到设备中）

[1]：使用’ndk-build’命令

‘ndk-build’脚本位于NDK安装目录最顶层，可以直接被应用程序项目目录（你的AndroidManifest.xml文件所在位置）或者其他任何子目录

$ cd $PROJECT

$ $NDK/ndk-build(注意是 $NDK/ndkbuild，这是个命令)
将启动 NDK 的 build 脚本，它会自动探测您开发的系统和应用程序项目文件，以确定 build 设么

例如：

$ndk-build

$ndk-build clean à 清理生成的二进制文件

$ndk-build –B V=1 à 强制完全重新 build，显示命令

默认的，它期望的是可选文件 $PROJECT/jni/Application.mk 和必须的文件 $PROJECT/jni/Android.mk

成功的话，它讲话就复制生成的二进制模块（即共享库.so文件）到你的项目树中的适当位置。您可以在以后重新 build 完整的 Android 应用程序包或者通过“ant”命令，或者 ADT 插件。

可以看 docs/NDK-BUILD.html 来了解更多的信息

[2]：使用 $NDK/apps/<name>/Application.mk

这种 build 方法是在 Android NDK r4 版本之前的，不过依然兼容现在的。我们强烈建议您尽可能的使用’ndk-build’，因为我们可能会删除在以后的 NDK 发行版本中的支持

① 创建一个子目录为 $NDK/apps/<name>/

② 在 $NDK/apps/<name>/目录下写一个 Application.mk 文件，然后需要定义一个 APP_PROJECT_PATH 来执行你的应用程序项目的目录。

③ 进入到 NDK 安装目录，然后再输入如下的命令

```

    $cd $NDK

```

注意：输入 cd $NDK 后，会自动跳到你设置的 ndk 的目录中

$make APP=<name>

或

$make APP=<name> -B 表示重新编译

结果跟第一种方法一样，除了中间文件被放置到了 $NDK/out/apps/<name>/

**4. 从新 build 你的应用程序包**

在 NDK 生成的二进制文件后，你需要使用一般的方法来重新 build 你的 Android 应用程序包文件(apk)，或者用“ant”命令或者 ADT 插件

有关详细信息，请参阅 Android SDK 的文档，新的.apk 会嵌入到您的共享库中，他们将自动提取安装时由系统安装的软件包到你的 Android 设备上

**5. 调试支持**

NDK 提供了一个服务脚本，名字叫”ndk-gdb”，很容易推出一个应用程序的本地调试会话。

本机调试仅仅能运行在 Android 2.2 或者更高版本，并且不需要 root 权限或者特权访问，所以可以随意调试你的应用程序。

有关详细信息，请阅读 DOCS / NDK- GDB.html。总括而言，本机调试
遵循这个简单的计划：  
（1）确保您的应用程序调试（如设置机器人：调试“真”，在您的 AndroidManifest.xml）

（2） “NDK 构建”构建您的应用程序，然后安装在您的 设备/模拟器

（3）启动应用程序。

（4）运行“ndk-gdb”从你的应用程序项目目录。

你会得到一个 gdb 提示符。一个有用的列表，请参阅 GDB 用户手册命令。

Tags: [Android](http://www.imobilebbs.com/wordpress/archives/tag/android), [NDK](http://www.imobilebbs.com/wordpress/archives/tag/ndk)

