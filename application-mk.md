# Android NDK 开发教程六： application.mk

配合 android.mk 使用的 make 文件还有一个 application.mk ，大部分情况无需修改该文件，下面也来自[网络翻译](http://www.iteye.com/topic/1113483)

Application.mk 文件

简介：

----------
要将 C\C++ 代码编译为 SO 文件，光有 Android.mk 文件还不行，还需要一个 Application.mk 文件。
本文档是描述你的 Android 应用程序中需要的本地模块的 Application.mk 的语法使用，要明白如下。

Application.mk 目的是描述在你的应用程序中所需要的模块(即静态库或动态库)。

Application.mk 文件通常被放置在 $PROJECT/jni/Application.mk 下，$PROJECT 指的是您的项目。

另一种方法是将其放在顶层的子目录下：
$NDK/apps 目录下，例如：
$NDK/apps/<myapp>/Application.mk

<myapp>是一个简称，用于描述你的 NDK 编译系统的应用程序（这个名字不会生成共享库或者最终的包）

下面是 Application.mk 中定义的几个变量。

APP_PROJECT_PATH  
这个变量是强制性的，并且会给出应用程序工程的根目录的一个绝对路径。这是用来复制或者安装一个没有任何版本限制的 JNI 库，从而给 APK 生成工具一个详细的路径。

APP_MODULES  
这个变量是可选的，如果没有定义，NDK 将由在 Android.mk 中声明的默认的模块编译，并且包含所有的子文件（makefile 文件）  
如果 APP_MODULES 定义了，它不许是一个空格分隔的模块列表，这个模块名字被定义在 Android.mk 文件中的 LOCAL_MODULE 中。注意 NDK 会自动计算模块的依赖

注意：NDK 在 R4 开始改变了这个变量的行为，再次之前：  
- 在您的Application.mk中，该变量是强制的
- 必须明确列出所有需要的模块

APP_OPTIM  
这个变量是可选的，用来定义“release”或”debug”。在编译您的应用程序模块的时候，可以用来改变优先级。

“release”模式是默认的，并且会生成高度优化的二进制代码。”debug”模式生成的是未优化的二进制代码，但可以检测出很多的 BUG，可以用于调试。

注意：如果你的应用程序是可调试的（即，如果你的清单文件中设置了 android:debuggable 的属性是”true”）。默认的是”debug”而不是”release”。这可以通过设置 APP_OPTIM 为”release”来将其覆盖。

注意：可以在”release”和”debug”模式下一起调试，但是”release”模式编译后将会提供更少的 BUG 信息。在我们清楚 BUG 的过程中，有一些变量被优化了，或者根本就无法被检测出来，代码的重新排序会让这些带阿弥变得更加难以阅读，并且让这些轨迹更加不可靠。

APP_CFLAGS  
当编译模块中有任何 C 文件或者 C++文件的时候，C 编译器的信号就会被发出。这里可以在你的应用中需要这些模块时，进行编译的调整，这样就不许要直接更改 Android.mk 为文件本身了

重要警告：+++++++++++++++++++++++++++++++++++++++++++++++ + +  
+
+ 在这些编制中，所有的路径都需要于最顶层的 NDK 目录相对应。  
+ 例如，如果您有以下设置：  
+
+sources/foo/Android.mk  
+sources/bar/ Android.mk  
+ 编译过程中，若要在 foo/Android.mk 中指定你要添加的路径到 bar 源代码中，  
+ 你应该使用  
+ APP_CFLAGS += -Isources/bar  
+ 或者交替：  
+ APP_CFLAGS += -I $（LOCAL_PATH )/../bar  
+
+ 使用’-l../bar/’将不会工作，以为它将等同于”-l$NDK_ROOT/../bar”
++++++++++++++++++++++++++++++++++++++++++++++++++ +++++++++++++++++++++

注意：在 Android 的 NDK 1.5_r1，只适用于 C 源文件，而不适合 C++。
这已得到纠正，以建立完整相匹配的 Andr​​oid 系统。

APP_CXXFLAGS  
APP_CPPFLAGS 的别名，已经考虑在将在未来的版本中废除了

APP_CPPFLAGS  
当编译的只有 C++源文件的时候，可以通过这个 C++编译器来设置

注意：在 Android NDK-1.5_r1 中，这个标志可以应用于 C 和 C++源文件中。并且得到了纠正，以建立完整的与系统相匹配的 Android 编译系统。你先可也可以使用 APP_CFLAGS 来应用于 C 或者 C++源文件中。
建议使用 APP_CFLAGS

APP_BUILD_SCRIPT  
默认情况下，NDK 编译系统会在 $(APP_PROJECT_PATH)/jni 目录下寻找名为 Android.mk 文件：
$(APP_PROJECT_PATH)/jni/Android.mk

如果你想覆盖此行为，你可以定义 APP_BUILD_SCRIPT 来指定一个备用的编译脚本。一个非绝对路径总是被解释为相对于 NDK 的顶层的目录。

APP_ABI  
默认情况下，NDK 的编译系统回味”armeabi”ABI生成机器代码。喜爱哪个相当于一个基于 CPU 可以进行浮点运算的 ARMv5TE。你可以使用 APP_ABI 来选择一个不同的 ABI。

比如：为了在 ARMv7 的设备上支持硬件 FPU 指令。可以使用  
APP_ABI := armeabi-v7a

或者为了支持 IA-32 指令集，可以使用  
APP_ABI := x86

或者为了同时支持这三种，可以使用  
APP_ABI := armeabi armeabi-v7a x86

APP_STL  
默认情况下，NDK 的编译系统为最小的 C++运行时库（/system/lib/libstdc++.so）提供 C++头文件。  
然而，NDK 的 C++的实现，可以让你使用或着链接在自己的应用程序中。  
例如：  
APP_STL := stlport_static    –> static STLport library  
APP_STL := stlport_shared    –> shared STLport library  
APP_STL := system            –> default C++ runtime library  

下面是一个 Application.mk 文件的示例：

```

    APP_PROJECT_PATH := <path to project>

```

Tags: [Android](http://www.imobilebbs.com/wordpress/archives/tag/android), [NDK](http://www.imobilebbs.com/wordpress/archives/tag/ndk)