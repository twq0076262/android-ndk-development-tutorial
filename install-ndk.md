# Android NDK 开发教程一：安装 NDK

Android OS 的基本框架为 Linux-Java ，在介绍 Android 开发时用到的 Android 结构图：



android 体系底层为 Linux 内核，之上提供一些 C/C++ 函数库，因此 Android 应用开发也可以使用 C /C++ 开发，这就是 Android NDK 开发包，但 Android 提供 NDK 开发包的主要目的并不是推荐开发人员使用 C（Native 代码）来编写一般的 Android 应用，而是要使用 Java 代码来编写 Android 应用来更好的处理 Android 应用生命周期(Life-cycle)相关的事件以避免出现“应用程序不响应（[ANR](http://www.imobilebbs.com/wordpress/wp-content/uploads/2011/04/anr.png)）”的对话框。

使用 NDK 主要是通过 [JNI](http://en.wikipedia.org/wiki/Java_Native_Interface) 使用从 Java 代码调用 C 代码，也就是使用 Native 编程主要是为上层 Java 代码提供库函数（动态库或是静态库的形式）而不是全部使用 Native C 代码编写整个 Android 应用（尽管借助于少量 Java 代码也是可以大部分使用 C 代码来实现的）。`使用 NDK 大部分情况是需要将一些已有的 C 函数库移植到 Android 平台的所选择的快捷方法，而不是作为提高代码效率的手段`

安装 Android NDK 的方法非常简单：打开网页 [http://developer.android.com/sdk/ndk/index.html](http://developer.android.com/sdk/ndk/index.html)

选择合适的 NDK 开发包，下载解压即可。注：安装 NDK 之前需先安装 SDK 开发包，参见 [Android 简明开发教程二：安装开发环境。
](http://www.imobilebbs.com/wordpress/?p=831)

Android NDK 的前两级目录如下：

├── build  
│   ├── awk  
│   ├── core  
│   ├── gmsl  
│   └── tools  
├── docs  
│   ├── ANDROID-ATOMICS.html    
│   ├── ANDROID-MK.html    
│   ├── APPLICATION-MK.html  
│   ├── CHANGES.html  
│   ├── CPLUSPLUS-SUPPORT.html  
│   ├── CPU-ARCH-ABIS.html  
│   ├── CPU-ARM-NEON.html  
│   ├── CPU-FEATURES.html  
│   ├── CPU-X86.html  
│   ├── DEVELOPMENT.html  
│   ├── HOWTO.html  
│   ├── IMPORT-MODULE.html  
│   ├── INSTALL.html  
│   ├── LICENSES.html  
│   ├── NATIVE-ACTIVITY.HTML  
│   ├── NDK-BUILD.html  
│   ├── NDK-GDB.html  
│   ├── NDK-STACK.html  
│   ├── openmaxal  
│   ├── opensles  
│   ├── OVERVIEW.html  
│   ├── PREBUILTS.html  
│   ├── sidenav.html  
│   ├── STABLE-APIS.html  
│   ├── STANDALONE-TOOLCHAIN.html  
│   ├── system  
│   └── SYSTEM-ISSUES.html  
├── documentation.html  
├── GNUmakefile  
├── ndk-build  
├── ndk-build.cmd  
├── ndk-gdb  
├── ndk-stack  
├── ndk.txt  
├── platforms  
│   ├── android-14  
│   ├── android-3  
│   ├── android-4  
│   ├── android-5  
│   ├── android-8  
│   └── android-9  
├── prebuilt  
│   └── linux-x86  
├── README.TXT  
├── RELEASE.TXT  
├── samples  
│   ├── bitmap-plasma  
│   ├── hello-gl2  
│   ├── hello-jni  
│   ├── hello-neon  
│   ├── module-exports  
│   ├── native-activity  
│   ├── native-audio  
│   ├── native-media  
│   ├── native-plasma  
│   ├── san-angeles  
│   ├── test-libstdc++  
│   └── two-libs  
├── sources  
│   ├── android  
│   ├── cpufeatures  
│   └── cxx-stl  
├── tests  
│   ├── awk  
│   ├── build  
│   ├── device  
│   ├── README  
│   ├── run-standalone-tests.sh  
│   ├── run-tests.sh  
│   └── standalone  
└── toolchains  
   ├── arm-linux-androideabi-4.4.3  
   └── x86-4.4.3  

在开发 NDK 之前，建议先看一下 doc 子目录下的文档，后面的博客也会有所介绍。

Tags: [Android](http://www.imobilebbs.com/wordpress/archives/tag/android), [NDK](http://www.imobilebbs.com/wordpress/archives/tag/ndk)

 