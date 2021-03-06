# jni

## using in Android studio

1.新建一个CMakeLists.txt 

### 注意 一定要加SHARED选项，不然不会生成so动态库

```
add_library(name SHARED cppname.cpp)
```

2.修改build.gradle

defaultConfig中添加

```

 ndk {
      abiFilters "armeabi-v7a"
    }
    
```

android中添加

```
externalNativeBuild {
    cmake {
      path "CMakeLists.txt"
    }
  }
```

整体示例如下：
```
android {
  compileSdkVersion 25
  buildToolsVersion "25.0.2"
  defaultConfig {
    minSdkVersion 21
    targetSdkVersion 25
    ndk {
      abiFilters "armeabi-v7a"
    }
  }
  externalNativeBuild {
    cmake {
      path "CMakeLists.txt"
    }
  }
}
```

3.新建cpp和.h文件
函数名称为包名加类名，例如：

可以用javah命令自动生成
在terminal进入到src/main/java目录，执行``javah org.ros2.examples.android.talker.TestJni``,生成的头文件在当前的src/main/java目录中


.h ``这里cpp编译要加extern "c"`` 2.要包含jni.h头文件

```
#include <jni.h>

extern "C" {
JNIEXPORT jstring JNICALL Java_org_ros2_examples_android_talker_jniUtile_getJniString
        (JNIEnv *, jclass);
}
```
.cpp
```
JNIEXPORT jstring JNICALL Java_org_ros2_examples_android_talker_jniUtile_getJniString
        (JNIEnv * env, jclass zz)
{
    return env-> NewStringUTF("hello jni...");
}

```

4.java文件中加载so库 这个库名称就是cmake中指定的名称

```
 static {
        System.loadLibrary("jiantu");
    }

```
整体效果如下：

```
package org.ros2.examples.android.listener;

public class JniUtils {

    static {
        System.loadLibrary("jiantu");
    }



    public static native String getJniString();
}


```

5.jni的cpp文件中调用第三方的so库方法

参考：https://developer.android.com/studio/projects/configure-cmake

修改CMakeLists.txt

```
cmake_minimum_required(VERSION 3.6)
add_library(jniCpp SHARED src/main/cpp/jniCpp.cpp)
link_directories(/Users/liushichao/workspace/ros2/ros2_android/ros2_android_ws/src/ros2_java/ros2_android_examples/ros2_talker_android/src/main/jniLibs/armeabi-v7a)
add_library( my_package
        SHARED
        IMPORTED )
set_target_properties( # Specifies the target library.
        my_package

        # Specifies the parameter you want to define.
        PROPERTIES IMPORTED_LOCATION

        # Provides the path to the library you want to import.
        /Users/liushichao/workspace/ros2/ros2_android/ros2_android_ws/src/ros2_java/ros2_android_examples/ros2_talker_android/src/main/jniLibs/armeabi-v7a/libmy_package.so )

include_directories( src/main/cpp/ )

target_link_libraries(jniCpp my_package)
```

通过上边的步骤就能完成编译了，但是要将预编译的so文件打包到apk中还需要进行下边的配置

在build.gradle中添加sourceSets.main字段，样例如下：

```
android {
    ...
    defaultConfig {
        ...
        sourecSets.main{
            //可以指定多个目录，但是每个目录里一定要定义abi支持的文件名称，例如../src/main/jniLibs/armeabi-v7a/xxx.so
            jniLibs.srcDirs = ['../src/main/jniLibs', '/Users/liushichao/workspace/ros2/ros2_android/ros2_android_ws/src/ros2_example/thirdparty']
        }
        ...
    }
    ...
}
```



6.jni打印日志

```
//1.包含头文件
#include <android/log.h>
//2.log定义
#define  LOG    "JNILOG" // 这个是自定义的LOG的TAG
#define  LOGD(...)  __android_log_print(ANDROID_LOG_DEBUG,LOG,__VA_ARGS__) // 定义LOGD类型
#define  LOGI(...)  __android_log_print(ANDROID_LOG_INFO,LOG,__VA_ARGS__) // 定义LOGI类型
#define  LOGW(...)  __android_log_print(ANDROID_LOG_WARN,LOG,__VA_ARGS__) // 定义LOGW类型
#define LOGE(...)  __android_log_print(ANDROID_LOG_ERROR,LOG,__VA_ARGS__) // 定义LOGE类型
#define LOGF(...)  __android_log_print(ANDROID_LOG_FATAL,LOG,__VA_ARGS__) // 定义LOGF类型
//3.修改CMakeLists.txt
find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       native-lib

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

### libc++

```
注意：libc++ 不是系统库。如果使用 libc++_shared.so，必须将其包含在 APK 中。如果使用 Gradle 构建应用，此步骤会自动完成。
```


### errno 以及 strerror

```
包含
#include <errno.h>
可以在程序中直接读取
errno，可以获取最近一次的错误代码

包含
#include <string.h>
可以调用
strerror(errno)
获取errno对应的字符串含义

完整例子：

 LOGE("again, open failed ... errno is : %d, means : %s", errno, strerror(errno));



```
