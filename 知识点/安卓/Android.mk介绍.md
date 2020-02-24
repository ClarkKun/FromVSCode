
https://www.jianshu.com/p/47825e8179dc

1. makefile概念
&emsp;&emsp;makefile关系到了整个工程的编译规则。一个工程中的源文件不计数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作

2. Android.mk文件
&emsp;&emsp;Android.mk文件用来向编译系统描述如何编译你的源代码。更确切地说，该文件其实就是一个小型的Makefile。由于该文件会被NDK的编译工具解析多次，因此应该尽量减少源码中声明变量，因为这些变量可能会被多次定义从而影响到后面的解析
这个文件的语法允许把源代码组织成模块，每个模块属于下列类型之一：
- ·APK程序：一般的Android程序，编译打包生成apk文件。
- ·JAVA库：java类库，编译打包生成jar包文件。
- ·C\C++应用程序：可执行的C/C++应用程序。
- ·C\C++静态库：编译生产C/C++静态库，并打包成.a文件。
- ·C\C++共享库：编译生成共享库，并打包成.so文件，有且只有共享库才能被安装/复制到APK包中。

### 举例
有如下两个文件
> 1.sources/test/hello.c  
2.sources/test/Android.mk  

其中“hello.c”是一个JNI共享库，实现返回“helloworld”字符串的原生方法.
Android.mk文件内容如下
```
LOCAL_PATH:=$(callmy-dir)

include $(CLEAR_VARS)

LOCAL_MODULE:=hello

LOCAL_SRC_FILES:=hello.c

include $(BUILD_SHARED_LIBRARY)

LOCAL_PATH:=$(callmy-dir)

include $(CLEAR_VARS)

LOCAL_MODULE:=hello

LOCAL_SRC_FILES:=hello.c

include $(BUILD_SHARED_LIBRARY)
```
1. LOCAL_PATH:=$(callmy-dir)  
&emsp;&emsp;一个Android.mk文件首先必须定义好LOCAL_PATH变量，用于在开发树中查找源文件。在这个例子中，宏函数my-dir由编译系统提供，用于返回当前路径（即包含Android.mk文件的目录）。
2. include $(CLEAR_VARS)  
&emsp;&emsp;CLEAR_VARS由编译系统提供，指定让GNUMAKEFILE清除,除了LOCAL_PATH变量外的许多LOCAL_***变量（例如：LOCAL_MODULE、LOCAL_SRC_FILES等）。这是非常有必要的，因为所有的编译文件都在同一个GUNMKAE执行环境中，所有的变量都是全局变量，不清除容易引起解析错误。

3. LOCAL_MODULE:=hello  
&emsp;&emsp;LOCAL_MODULE变量必须定义，用来标识在Android.mk文件描述的每一个模块。而且名称必须是唯一的，并且不能包含空格。编译系统会自动产生合适的前缀和后缀，比如一个被命名为hello的共享库模块，将会生成libhello.so文件。如果把库命名为libhello，编译系统将不会添加任何lib前缀，也会生成libhello.so文件。

4. LOCAL_SRC_FILES:=hello.c  
&emsp;&emsp;LOCAL_SRC_FILES变量必须包含将要编译打包进模块中的源代码文件。

5. include $(BUILD_SHARED_LIBRARY)  
&emsp;&emsp;BUILD_SHARED_LIBRARY是编译系统提供的变量，指向一个GNUMakefile脚本（应该就是build/core目录下的shared_library.mk）负责收集自从上次调用include$(CLEAR_VARS)以来，定义在LOCAL_***变量中的所有信息，并且决定编译什么，如何正确地去做，并根据其规则生成静态库。

6. 解释一下Android.mk里变量定义字符":="。“:=”类似于c中的宏，即在定义处明确展开，完全进行文本替换。


