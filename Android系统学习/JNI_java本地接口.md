## JNI
JNI，全称为Java Native Interface，即Java本地接口，JNI是Java调用Native 语言的一种特性。通过JNI可以使得Java与C/C++机型交互。即可以在Java代码中调用C/C++等语言的代码或者在C/C++代码中调用Java代码。
## 如何实现JNI
第1步：在Java中先声明一个native方法
第2步：编译Java源文件javac得到.class文件
第3步：通过javah -jni命令导出JNI的.h头文件
第4步：使用Java需要交互的本地代码，实现在Java中声明的Native方法（如果Java需要与C++交互，那么就用C++实现Java的Native方法。）
第5步：将本地代码编译成动态库(Windows系统下是.dll文件，如果是Linux系统下是.so文件，如果是Mac系统下是.jnilib)
第6步：通过Java命令执行Java程序，最终实现Java调用本地代码。
