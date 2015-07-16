# Android NDK 開發教程五：Android.mk 文件

NDK 項目一個重要組成是它的 make 文件 –android.mk. 下面部分來自網路翻譯（省得我再翻譯了:-).

註：大部分情況只需參考 HelloJni 和 twoLibs 的 android.mk 文件即可，如果你想搞清楚 android.mk 中定義變數的具體含義，可以參考下面翻譯。

Android.mk 文件語法詳述

介绍：

----------
這篇文檔是用來描述你的 C或 C++源文件中 Android.mk 編譯文件的語法的，為了理解她們我們需要您先看完 
docs/OVERVIEW.html（http://hualang.iteye.com/blog/1135105）文件來了解它的作用

概覽：

----------

Android.mk 文件是用來描述 build system(編譯系統)的，更準確的說：

–該文件是一個微型的 GNU Makefile 片段，將由 build system 解析一次或者多次。這樣，您就可以盡量減少您聲明的變數，並且不要以為在解析過程中沒有任何定義。

- 這個文件但語法是用來允許你將源文件組織成模塊，這個模塊中含有：
 - 一個靜態庫(.a 文件)
 - 一個動態庫(.so 文件)  

只有動態庫才會被安裝/複製到你的應用程序包，儘管靜態庫可以被用來生成動態庫。你可以在每個模塊中  都定義一個 Android.mk 文件，你也可以讓多個模塊共用一個 Android.mk 文件。

- build system 可以為你處理許多細節，例如：你不許要在 Android.mk 文件中列出頭文件或者其他的依賴關係，這些 NDK 的 build system 會自動為你計算並處理。

這也意味著，當更新到新版本的 NDK 的時候，你應該得益於新的 toolchain/platform 的支持，而無需修改你的 Android.mk 文件。

注意：這些語法非常接近於分布在完整的開源的 Android 源代碼中的 Android.mk 文件，儘管是 build system 實現的，但是它們的用法是不同的。這樣故意設計的決定是為了讓應用程序開發者重用「外部」庫的源代碼更容易。

簡單實例：

----------

再詳細講解語法之前，讓我們先看看一個簡單的例子”hello JNI”,它在 apps/hello-jni/project 下

- ‘src’目錄下用於存放 java 源文件
- 『jni』目錄下用於存放本地源文件，例如”jni/hello-jni.c”

這個源文件實現了一個簡單的共享庫(shared library)：實現了一個本地方法，為 VM 應用程序返回一個字元串。

- 『jni/Android.mk』文件描述了如何生成一個共享庫，它的內容是：

—————–Android.mk————————  
LOCAL_PATH := $(call my-dir)  
include $(CLEAR_VARS)  

LOCAL_MODULE := hello-jni  
LOCAL_SRC_FILES := hello-jni.c  

include $(BUILD_SHARED_LIBRARY)  

—————————————————

現在，讓我們分別解釋這幾行

```

    LOCAL_PATH：=$(call my-dir)

```

Android.mk 文件必須以 LOCAL_PATH 變數開始，它用於在樹中定位文件。在這個例子中，宏功能’my-dir’是由 build system 提供的，用於返回當前目錄路徑（包括 Android.mk 文件本身）

```

    include $(CLEAR_VARS)

```


CLEAR_VARS 變數是由 build system 提供的，並且指明了一個 GNU makefile 文件，這個功能會清理掉所有以  LOCAL_ 開頭的內容（例如 LOCAL_MODULE,LOCAL_SRC_FILES,LOCAL_STATIC_LIBRARIES 等），除了 LOCAL_PATH，這句話是必須的，因為如果所有的變數都是全局變數的話，所有的可控的編譯文件都需要在一個單獨的 GNU 中被解析並執行

LOCAL_MODULE :=hello-jni

LOCAL_MODULE 變數必須被定義，用來區分 Android.mk 中的每一個模塊。文件名必須是唯一的，不能有空格。注意，這裡編譯器會為你自動加上一些前綴和後綴，來保證文件是一致的，比如：這裡表明一個動態連接庫模塊被命名為”hello-jni”，但是最後會生成為”libhello-jni.so”文件。但是在 Java 中裝載這個庫的時候還是使用”hello-jni”名稱。當然如果我們使用”IMPORTANT NOTE:”,編譯系統就不會為你加上前綴，但是為了支持 Android 平台源碼中的 Android.mk 文件，也同樣會生成 libhello-jni.so 這樣的文件。

重要提示：如果你將你的模塊命名為 ’libfoo’，編譯系統將不會將前綴 ’lib’ 加上去，並且也會生成 libfoo.so 文件。

```

    LOCAL_SRC_FILES := hello-jni.c

```

LOCAL_SRC_FILES 變數被需包括一個 C 和 C++ 源文件的列表，這些會編譯並聚合到一個模塊中。

注意：這裡並不需要你列出頭文件和被包含的文件，因為編譯系統會自動為你計算相關的屬性，源代碼中的列表會直接傳遞給編譯器。

C++ 默認文件的擴展名是「.cpp」，我們可以通過定義一個 LOCAL_DEFAULT_CPP_EXTENSION 變數來定義一個不同的 C 文件。不要忘記在初始化前面的「.」點（也就是說”.cpp”可以正常工作，但是 cpp 不能正常工作）

```

include $(BUILD_SHARED_LIBRARY)

```

BUILD_SHARED_LIBRARY 這個變數是由系統提供的，並且指定給 GNU Makefile 的腳本，它可以收集所有你定義的”include $(CLEAR_VARS)”中以 LOCAL_ 開頭的變數，並且決定哪些要被編譯，哪些應該做的更加準確。編譯生成的是以”lib<module_name>.so”的文件，這個就是共享庫了。我們同樣也可以使用 BUILD_STATIC_LIBRARY 編譯系統便會生成一個以”lib<module_name>.a”的文件來供動態庫調用。

在 samples 目錄下有很多複雜的例子，那裡的 Android.mk 文件可以供我們參考

參考：

----------

這是個變數的列表，你可以依賴或者定義它們到 Android.mk 中。你可以定義自己使用的其他變數，但是 NDK 辨析系統只保留了以下名字：

- 以 LOCAL_ 開頭的名稱（如：LOCAL_MODULE）
- 以 PRIVATE_、NDK_、或 APP_（內部使用）開始的名稱
- 小寫的名稱（例如’my-dir’，內部使用）

如果你需要在 Android.mk 中定義自己的變數的話，我們建議使用 MY-前綴，一個簡單的例子：

----------

```

    MY_SOURCES := foo.c
    ifneq($(MY_CONFIG_BAR),)
    MY_SOURCES += bar.c
    endif

    LOCAL_SRC_FILES +=$(MY_SOURCES)

```

我們繼續：

NDK 提供的變數：
在您的 Android.mk 文件被解析之前這些 GNU Make 變數由編譯系統定義，注意，在某些情況下，NDK 可能被解析幾次，每次以不同的變數的定義解析的

CLEAR_VARS

CLEAR_VARS 這個變數由系統提供，功能是清理掉所有以  LOCAL_ 開頭的內容，再開始一個新的模塊之前，你必須包括這段腳本

```

    include ($CLEAR_VARS)

    BUILD_SHARED_LIBRARY

```

在編譯腳本中收集所有以 LOCAL_ 開頭的信息並且決定從列出的源代碼中編譯一個目標共享庫。注意，你必須定義了 LOCAL_MODULE 和 LOCAL_SRC_FILES 變數，使用它的時候，可以這樣定義

```

include $(BUILD_SHARED_LIBRARY)

```


注意，我們會生成一個以 lib<module_name>.so 為名的文件

BUILD_STATIC_LIBRARY

用來構建一個靜態庫，該靜態庫將不會被拷貝到你的 project/packages 下，但是可以被用於動態庫
(看下面的 LOCAL_STATIC_LIBRARY 和 LOCAL_WHOLE_STATIC_LIBRARY 介紹)

例如：

```

include $(BUILD_STATIC_LIBRARY)

```


注意，這將生成一個 lib<module_name>.a 為名字的模塊

PREBUILD_SHARED_LIBRARY

在編譯腳本中用於指定一個預先編譯的動態庫，不像 BUILD_SHARED_LIBRARY 和 BUILD_STATIC_LIBRARY，LOCAL_SRC_FILES 的預先共享庫必須是一個單獨的路徑（如：foo/libfoo.so），而不是源文件。

你可以在另一個模塊中引用預編譯的庫（參見 docs/pribuilds.html）

PRIBUILD_STATIC_LIBRARY

這個變數類似於 PREBUILD_SHARED_LIBRARY，但是是針對靜態庫的，(詳見 docs/prebuilds.html)

TARGET_ARCH

TARGET_ARCH 指框架中 CPU 的名字已經被 Android 開源代碼明確指出了，這裡的 arm 包含了任何 ARM-獨立結構的架構，以及每個獨立的 CPU 版本

TARGET_PLATFORM

Android 平台的名字在 Android.mk 中被解析，比如”android-3″對應 Android 1.5 系統鏡像，對於平台的名稱對應 Android 系統的列表，請看 docs/STABLE-APIS.html

TARGET_ARCH_ABI

在 Android.mk 中被解析時指CPU+ABI的名字。

目前支持的兩個值  
    armeabi for ARMv5TE  
    armeabi-v7a

注意，到 Android NDK 1.6_r1，這個值被簡化為”arm”。然而，這個值被重定義可以更好的匹配 Android 平台內部使用的是什麼

更多的信息可以參見 docs/CPU-ARCH-ABIS.html

未來的 NDK 版本中得到支持，它們會有一個不同的名字，注意所有基於 ARM 的 ABI 都會有一個”TARGET_ARCH”被定義給 arm，但也有可能有不同的”TARGET_ARCH_ABI”

TARGET_ABI

目標平台和 ABI 的鏈接，這裡要定義$(TARGET_PLATFORM)-$(TARGET_ARCH_ABI)它們都非常有用，特別是當你想測試一下具體的系統鏡像在一個真實設備環境的時候

默認地，這個是”android-3-armeabi”

(到 Android NDK 16_R1 版本，使用”android-3-arm”作為默認)

NDK 提供的宏功能

----------

以下是使用 GNU make 的宏功能，必須通過使用”$(call <function>)”，返回一個文本信息。

my-dir

返回最後包含的 makefile 的路徑，這通常是當前 Android.mk 所在目錄的路徑，在 Android.mk 開始之前定義
LOCAL——PATH 是很有用的。

在 Android.mk 文件的開始位置定義

```
    
    LOCAL_PATH :=$(call my-dir)

```

…聲明一個模塊

```

    include $(LOCAL_PATH)/foo/Android.mk

    LOCAL_PATH :=($call my-dir)

```


…聲明另一個模塊
這裡的問題是第二次調用”my-dir”定義 LOCAL_PATH 替換 $PATH 為 $PATH/foo，由於在此之前執行過。

對於這個原因，最好是將額外的其他所有東西都在 Android.mk 中包含進來

```

    LOCAL_PATH :=$(call my-dir)

```


…聲明一個模塊

```

LOCAL_PATH :=$(call my-dir)

```

…聲明另一個模塊

 #在 Android.mk 的最後額外包括進來

```

include $(LOCAL_PATH)/foo/Android.mk

```


如果這樣不方便的話，保存第一個 my-dir 調用的值到另一個變數中，例如

```

    MY_LOCAL_PATH :=$(call my-dir)
    LOCAL_PATH :=$(MY_LOCAL_PATH)

```

…聲明一個模塊

```

    include $(LOCAL_PATH)/foo/Android.mk
    LOCAL_PATH :=$(MY_LOCAL_PATH)

```

…聲明另一個模塊

all-subdir-makefiles
返回一個 Android.mk 文件所在位置的列表，以及當前的 my-dir 的路徑。比如

sources/foo/Android.mk  
sources/foo/lib1/Android.mk  
sources/foo/lib2/Android.mk  

如果 sources/foo/Android.mk 包含了這行語句
include $(call all-subdir-makefiles)

那麼，它將會自動將 sources/foo/lib1/Android.mk 和 sources/foo/lib2/Android.mk 包含進來。

此功能可以用於提供深層嵌套的源代碼目錄 build system 的層次結構。請注意，默認情況下，NDK 只會尋找
sources/*Android.mk

this-makefile  
返回當前 makefile 的路徑（也就是那個功能被調用了）

parent-makefile  
返回 makefile 的包含樹，也就是包含 Makefile 當前的文件

grand-parent-makefile  
你猜？

import-module  
一個允許你通過名字找到並包含另一個模塊的的 Android.mk 的功能，例如

$(call import-module,<name>)

這將會找到通過 NDK_MODULE_PATH 環境變數引用的模塊<name>的目錄列表，並且將其自動包含到
Android.mk 中

詳細信息請參閱:docs/IMPORT-MODULE.html

模塊變數描述：

----------

下面的這些變數是用來描述怎樣用你的模塊來編譯系統的。你可以定義它們中的一些比如
“include $(CLEAR_VARS)”和”include $(BUILD_XXX)”，正如前面所寫的，$(CLEAR_VARS)是一個可以取消定義/清楚所有變數的腳本。

LOCAL_PATH  
這個變數是用來給出當前文件的路徑。您比系再您的 Android.mk 開始位置定義：

LOCAL_PATH :=$(call my-dir)  
注意，這個變數是不被$(CLEAR_VARS)清除的，其他的都要被清除（我們可以定義幾個模塊到一個文件中）

LOCAL_MODULE  
這個是你模塊的名稱，它在你的所有模塊中名稱必須是唯一的，並且不能包含空格。你必須在包含任何
$(BUILD-XXX)腳本之前定義它。

默認情況下，模塊的名稱決定了生成的文件的名稱，例如 lib<foo>.so，它是 foo 模塊的名字。

你可以用 LOCAL_MODULE_FILENAME 覆蓋默認的那一個

LOCAL_MODULE_FILENAME

這個變數是可選的，並且允許你重新定義生成文件的名字。默認的，模塊<foo>將始終生成 lib<foo>.a 或者 lib<foo>.so 文件，這是標準的 UNIX 公約

你可以通過 LOCAL_MODULE_FILENAME 覆蓋它

```

    LOCAL_MODULE :=foo-version-1
    LOCAL_MODULE_FILENAME :=libfoo

```

注意：你不能將文件路徑或者文件擴展名寫到 LOCAL_MODULE_FILENAME 里，這些將有 build system 自動處理。

LOCAL_SRC_FILES  
這是你模塊中將要編譯的源文件列表。只列出將被傳遞到編譯器的文件，因為 build system 自動為您計算了它們的依賴。

注意：源文件的名稱都是相對 LOCAL_PATH 的，您可以使用路徑組件，例如

```

    LOCAL_SRC_FILES :=foo.c\
    toto/bar.c

```


注意：在 build system 時請務必使用 UNIX 風格的斜杠(/)，windows 風格的斜杠將不會得到處理

LOCAL_CPP_EXTENSION  
這是個可選的變數，可以被定義為文件擴展名為 c++的源文件，默認是”.cpp”，但是你可以改變它，比如

```

    LOCAL_CPP_EXTENSION:=.cxx
    
    LOCAL_C_INCLUDES

```

可選的路徑列表，相對於 NDK 的根目錄，當編譯所有的源文件（C、C++、或者彙編）時將被追加到搜索路徑中
例如：  
LOCAL_C_INCLUDES:=sources/foo  
或者  
LOCAL_C_INCLUDES:=$(LOCAL_PATH)/../foo

這些都在任何相應列入標誌之前被放置在  
LOCAL_CFLAGS / LOCAL_CPPFLAGS

當用用 ndk-gdb 啟動本機調試時，LOCAL_C_INCLUDES 也會自動被使用到

LOCAL_CFLAGS

當編譯 C/C++源文件時傳遞一個可選的編譯器標誌。
這對於指定額外的宏定義或編譯選項很有用

重要提示：盡量不要改變 Android.mk 中的優化/調試級別，這個可以通過在 Application.mk 中設置相應的信息來自動為你處理，並且會會讓 NDK 生成在調試過程中使用的有用的數據文件。

注意：在 Android-ndk-1.5_r1 中，只使用於 C 源文件，而不適用於 C++源文件。在匹配所有 Android build system 的行為已經得到了糾正。（現在你可以為 C++源文件使用 LOCAL_CPPFLAGS 來指定標誌）

它可以用 LOCAL_CFLAGS += -I<path>來指定額外的包含路徑，然而，如果使用 LOCAL_C_INCLUDES 會更好，因為用 ndk-gdk 進行本地調試的時候，那些路徑依然是需要使用的

LOCAL_CXXFLAGS  
LOCAL_CPPFLAGS 的別名。請注意，這個標誌在 NDK 的未來的版本中將會消失

LOCAL_CPPFLAGS  
當只編譯 C++源代碼的時候，將傳遞一個可選的編譯器標誌。它們將會出現再 LOCAL_CFLAGS 之後。

注意：在 Android NDK-1.5_r1 版本中，相應的標誌可以應用於 C 或 C++ 源文件上。在配合完整的 Android build system 的時候，這已經得到了糾正。（你可以使用 LOCAL_CFLAGS 去指定 C 或  C++源文件）

LOCAL_STATIC_LIBRARIES

靜態庫模塊的列表(通過 BUILD_STATIC_LIBRARY 創建)應與此模塊鏈接。這僅僅是為了使動態庫敏感。

LOCAL_SHARED_LIBRARY 
共享庫的列表「模塊」，這個模塊依賴於運行時.這在鏈接的時候和在生成的文件中嵌入相應的信息是非常必要的

LOCAL_WHOLE_STATIC_LIBRARIES

LOCAL_WHOLE_STATIC_LIBRARIES 是一個用於表示相應的庫模塊被用作為「整個檔案」到鏈接程序的變數。

當幾個靜態庫之間有循環依賴關係的時候，通常是很有益的。注意，當用來編譯一個動態庫的時候，這將迫使你將所有的靜態庫中的對象文件添加到最終的二進位文件中。但生成可執行程序時，這是不確定的。

LOCAL_LDLIBS  
當額外的鏈接標誌列表被用於在編譯你的模塊時，通過用”-l”前綴的特定系統庫傳遞名字是很有用的。例如，下面的舊愛哪個告訴你生成一個在載入時鏈接到/system/lib/libz.so 的模塊。

LOCAL_LDLIBS :=-lz

LOCAL_ALLOW_UNDEFINED_SYMBOLS  
默認情況下，當試圖編譯一個共享庫的時候遇到任何未定義的引用都可能導致”未定義符號”(undefined symbol)的錯誤。這在你的源代碼中捕獲 bug 會很有用。

然而，但是由於某些原因，你需要禁用此檢查的話，設置變數為”true”即可。需要注意的是，相應的共享庫在運行時可能載入失敗。

LOCAL_ARM_MODE  
默認情況下，在”thumb”模式下會生成 ARM 目標二進位，其中每個指令都是 16 位寬。你可以定義這個變數為”arm”，如果你想在”arm”模式下（32 位指令）強迫模塊對象文件的生成。例如：

```

    LOCAL_ARM_MODE := arm
```

注意，你需要執行編譯系統為在 ARM 模式下通過文件的名字增加後綴的方式編譯指定的源文件。比如：

```

    LOCAL_SRC_FILES :=foo.c bar.c.arm

```

這會告訴編譯系統一直以 ARM 模式編譯”bar.c”,並且通過 LOCAL_ARM_MODE 的值編譯 foo.c。

注意：在 Application.mk 文件中設置 APP_OPTIM 為”debug”也會強制 ARM 二進位文件的生成。這是因為工具鏈調試其中的 bug 不會處理 thumb 代碼。

LOCAL_ARM_NEON

定義這個變數為”true”會允許在你的 C 或 C++源文件的 GCC 的內部函數中使用 ARM 高級 SIMD（又名 NEON），以及在聚合文件中的 NEON 指令。

當針對”armeabi-v7a”ABI對應的 ARMv7 指令集時你應該定義它。注意，並不是所有的 ARMv7 都是基於 NEON 指令集擴展的 CPU，你應該執行運行時來檢測在運行時中這段代碼的安全。

另外，你也可以指定特定的源文件，比如用支持 NEON”.neon”後綴的源文件也可以被編譯。

```

    LOCAL_SRC_FILES :=foo.c.neon bar.c zoo.c.arm.neon

```

在這個例子中，”foo.c”將會被編譯在 thumb+neon 模式中，”bar.c”以 thumb 模式編譯，zoo.c 以 arm+neon 模式編譯。

注意，如果你使用兩個的話，”.neon”後綴必須出現在”.arm”後綴之後
（就是 foo.c.arm.neon 可以工作，但是 foo.c.neon.arm 不工作）

LOCAL_DISABLE_NO_EXECUTE

Android NDK r4 開始添加了支持”NX位”安全功能特性。它是默認啟用的，如果你需要的話，可以通過設置變數為「true」來禁用它。

注意：此功能不修改 ABI，並且只在 ARMv6 及以上的 CPU 設備的內核上被啟用。

更多信息，可以參見：  
http://en.wikipedia.org/wiki/NX_bit  
http://www.gentoo.org/proj/en/hardened/gnu-stack.xml

LOCAL_EXPORT_CFLAGS

定義這個變數用來記錄 C/C++編譯器標誌集合，並且會被添加到其他任何以 LOCAL_STATIC_LIBRARIES 和 LOCAL_SHARED_LIBRARIES 的模塊的 LOCAL_CFLAGS 定義中。

例如：這樣定義”foo”模塊  
```

    include $(CLEAR_VARS)  
    LOCAL_MODULE :=foo 
    LOCAL_SRC_FILES :=foo/foo.c
    LOCAL_EXPORT_CFLAGS :=-DFOO=1
    include $(BUILD_STATIC_LIBRARY)

```

另一個模塊，叫做”bar”，並且依賴於上面的模塊

```

    include $(CLEAR_VARS)
    LOCAL_MODULE :=bar
    LOCAL_SRC_FILES :=bar.c
    LOCAL_CFLAGS:=-DBAR=2
    LOCAL_STATIC_LIBRARIES:=foo
    include $(BUILD_SHARED_LIBRARY)

```


然後，當編譯 bar.c 的時候，標誌”-DFOO=1 -DBAR=2″將被傳遞到編譯器。

輸出的標誌被添加到模塊的 LOCAL_CFLAGS 上，所以你可以很容易複寫它們。它們也有傳遞性：如果”zoo”依賴”bar”，「bar」依賴”foo”，那麼”zoo”也將繼承”foo”輸出的所有標誌。

最後，當編譯模塊輸出標誌的時候，這些標誌並不會被使用。在上面的例子中，當編譯 foo/foo.c 時，
-DFOO=1 將不會被傳遞給編譯器。

LOCAL_EXPORT_CPPFLAGS

類似 LOCAL_EXPORT_CFLAGS,但適用於 C++標誌。

LOCAL_EXPORT_C_INCLUDES

類似 LOCAL_EXPORT_C_CFLAGS,但是只有 C 能包含路徑，如果”bar.c”想包含一些由”foo”模塊提供的頭文件的時候這會很有用。

LOCAL_EXPORT_LDLIBS

類似於 LOCAL_EXPORT_CFLAGS,但是只用於鏈接標誌。注意，引入的鏈接標誌將會被追加到模塊的 LOCAL_LDLIBS，這是因為UNIX連接器的工作方式。

當模塊 foo 是一個靜態庫的時候並且代碼依賴於系統庫時會很有用的。LOCAL_EXPORT_LDLIBS 可以用於輸出依賴，例如：

```

    include $(CLEAR_VARS)
    LOCAL_MODULE := foo
    LOCAL_SRC_FILES := foo/foo.c
    LOCAL_EXPORT_LDLIBS := -llog
    include $(BUILD_STATIC_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE := bar
    LOCAL_SRC_FILES := bar.c
    LOCAL_STATIC_LIBRARIES := foo
    include $(BUILD_SHARED_LIBRARY)

```


這裡，在連接器命令最後，libbar.so 將以-llog 參數進行編譯來表明它依賴於系統日誌庫，因為它依賴於 foo。

LOCAL_FILTER_ASM

這個變數定義了一個 shell 命令，將用於過濾，從你的 LOCAL_SRC_FILES 中產生的或者聚合文件。

當它被定義了，將會出現如下的情況：

–任何 C 或 C++源文件將會生成到一個臨時的聚合的文件中（而不是被編譯成目標文件）
–任何臨時聚合文件，任何在 LOCAL_SRC_FILES 中列出的聚合文件將通過 LOCAL_FILER_ASM 命令生成另一個臨時聚合文件

–這些過濾聚合文件被編譯成目標文件。

換種說法，如果

```

    LOCAL_SRC_FILES  := foo.c bar.S
    LOCAL_FILTER_ASM := myasmfilter

```


foo.c –1–> $OBJS_DIR/foo.S.original –2–> $OBJS_DIR/foo.S –3–> $OBJS_DIR/foo.o  
bar.S –2–> $OBJS_DIR/bar.S –3–> $OBJS_DIR/bar.o

「1」對應的編譯器，「2」的過濾器，和「3」的彙編。過濾器必須是一個獨立的 shell 命令作為第一個參數輸入文件的名稱，和輸出的名稱第二，如文件為：

myasmfilter$ OBJS_DIR/ foo.S.original$ OBJS_DIR/ foo.S
myasmfilter bar.S$ OBJS_DIR/ bar.S

Tags: [Android](http://www.imobilebbs.com/wordpress/archives/tag/android), [NDK](http://www.imobilebbs.com/wordpress/archives/tag/ndk)


