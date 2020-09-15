# 详解build.gradle文件
## 外层build.gradle文件：
>更详细的地址：https://www.cnblogs.com/steffen/p/9212765.html
```java
buildscript {
    repositories {
        google()
        //jcenter()是一个代码托管仓库，很多Android开源项目都会选择将代码托管到jcenter上，声明了这行配置之后，就可以在项目中轻松引用任何jcenter上的开源项目
        jcenter()
    }
    //dependencies 闭包中使用classpath 声明了一个用于构建Android项目的Gradle插件
    dependencies {
        classpath 'com.android.tools.build:gradle:3.4.1'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
## app目录下的build.gradle
```java
//com.android.application 表示这是一个应用程序模块，com.android.library 表示这是一个库模块
apply plugin: 'com.android.application'

//android闭包内配置项目构建的各种属性
android {
    // 程序在编译的时候会检查lint，有任何错误提示会停止build，我们可以关闭这个开关
    lintOptions {
        // 即使报错也不会停止打包
        abortOnError false
        // 打包release版本的时候是否进行检测
        checkReleaseBuilds false
    }

    compileSdkVersion 29
    buildToolsVersion "29.0.2"
    //default闭包配置项目的更多细节
    defaultConfig {
        //指定项目的包名
        applicationId "com.example.servicebestpractice"
        //最低兼容的Android版本
        minSdkVersion 19
        //做过充分测试的版本
        targetSdkVersion 29
        //项目版本号
        versionCode 1
        //项目版本名
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    //buildTypes闭包中用于指定生成安装文件的相关配置
    buildTypes {
        //通常有两个子闭包：release和debug
        release {
            //是否对项目代码进行混淆
            minifyEnabled false
            //混淆使用的规则文件，第一个文件(.txt)在Android SDK目录下的，里面是所有项目通用的混淆规则,第二个文件（.pro）当前项目的app目录下的，里面可以编写当前项目特有的混淆规则
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    //本地依赖声明：将libs目录下所有.jar后缀的文件都添加到项目的构建路径
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation("com.squareup.okhttp3:okhttp:4.3.1")
    implementation 'com.google.code.gson:gson:2.8.6'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}
```