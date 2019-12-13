# CmakeStudy
在android studio 2.2及以上，构建原生库的默认工具是 CMake。

​ CMake是一个跨平台的构建工具，可以用简单的语句来描述所有平台的安装(编译过程)。能够输出各种各样的makefile或者project文件。Cmake 并不直接建构出最终的软件，而是产生其他工具的脚本（如Makefile ），然后再依这个工具的构建方式使用。

​ CMake是一个比make更高级的编译配置工具，它可以根据不同平台、不同的编译器，生成相应的Makefile或者vcproj项目。从而达到跨平台的目的。Android Studio利用CMake生成的是ninja，ninja是一个小型的关注速度的构建系统。我们不需要关心ninja的脚本，知道怎么配置cmake就可以了。从而可以看出cmake其实是一个跨平台的支持产出各种不同的构建脚本的一个工具。

CMake的脚本名默认是CMakeLists.txt


#cmake最低版本

cmake_minimum_required(VERSION 3.6.0)

#指定项目

project(Main)

#生成可执行文件 main

add_executable(main main.c)

#执行cmake . 生成makefile

#再执行make即可生成main程序

如果源文件很多，那么一个个写进去是一件很麻烦的事情，这时候可以：


cmake_minimum_required(VERSION 3.6.0)

project(Main)

#查找当前目录所有源文件 并将名称保存到 DIR_SRCS 变量

#不能查找子目录

aux_source_directory(. DIR_SRCS)

message(${DIR_SRCS})

#也可以

file(GLOB DIR_SRCS *.c)

add_executable(main ${DIR_SRCS})

如果在cmake中需要使用其他目录的cmakelist


cmake_minimum_required (VERSION 3.6.0)

project (Main)

aux_source_directory(. DIR_SRCS)

# 添加 child 子目录下的cmakelist

add_subdirectory(child)

# 指定生成目标

add_executable(main ${DIR_SRCS})

# 添加链接库

target_link_libraries(main child)

#===========================================================================================

#child目录下的cmake就是：

cmake_minimum_required (VERSION 3.6.0)

aux_source_directory(. DIR_LIB_SRCS)

# 生成链接库 默认生成静态库

add_library (child ${DIR_LIB_SRCS})

#指定编译为静态库

add_library (child STATIC ${DIR_LIB_SRCS})

#指定编译为动态库

add_library (child SHARED ${DIR_LIB_SRCS})

在上面的例子中都是生成可执行文件，让我们对cmakelist有了一定的了解。现在到android studio中使用cmakelist


#NDK中已经有一部分预构建库 ndk库已经是被配置为cmake搜索路径的一部分 所以可以

findLibrary(log-lib log)

target_link_libraries( native-lib

                      ${log-lib} )

#设置cflag和cxxflag

#定义预编译宏：TEST

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DTEST"  )

set(CMAKE_Cxx_FLAGS "${CMAKE_Cxx_FLAGS} -DTEST"  )                     

#其实直接这样就行

target_link_libraries( native-lib

                      log )

添加其他预编译库(已经提前编译好的库)


#使用 IMPORTED 标志告知 CMake 只希望将库导入到项目中

#如果是静态库则将shared改为static

add_library( imported-lib

            SHARED

            IMPORTED )

# 参数分别为：库、属性、导入地址、so所在地址

set_target_properties(

                      imported-lib

                      PROPERTIES

                      IMPORTED_LOCATION

                      ${CMAKE_SOURCE_DIR}/src/${ANDROID_ABI}/libimported-lib.so )

#为了确保 CMake 可以在编译时定位头文件

#这样就可以使用 #include <xx> 引入

#否则需要使用 #include "path/xx"

include_directories( imported-lib/include/ )

#native-lib 是自己编写的源码最终要编译出的so库

target_link_libraries(native-lib imported-lib)

#===========================================================================================

#添加其他预编译库还可以使用这种方式

#使用-L指导编译时库文件的查找路径

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Lxx")

#为了确保 CMake 可以在编译时定位您的标头文件

include_directories( imported-lib/include/ )

#native-lib 是自己编写的源码最终要编译出的so库

target_link_libraries(native-lib imported-lib)

常用指令：


#set命令表示声明一个变量source 变量的值是后面的可变参数

set（source a b c）

message(${source})

#逻辑判断 计较字符串

set(ANDROID_ABI "areambi-v7a")

if(${ANDROID_ABI} STREQUAL "areambi")

  message("armv5")

elseif(${ANDROID_ABI} STREQUAL "areambi-v7a")

message("armv7a")

else()

endif()


//还可以在gradle中使用 arguments  设置一些配置

externalNativeBuild {

      cmake {

        arguments "-DANDROID_TOOLCHAIN=clang",  //使用的编译器clang/gcc

                  "-DANDROID_STL=gnustl_static" //cmake默认就是 gnustl_static

        cFlags "" //这里也可以指定cflag和cxxflag,效果和之前的cmakelist里使用一样

        cppFlags ""

      }

    }

5.0及以下与6.0及以上的注意事项：

​ 存在两个动态库libhello-jni.so 与 libTest.so。

libhello-jni.so依赖于libTest.so (使用NDK下的ndk-depends可查看依赖关系)，则：


//<=5.0:

System.loadLibrary("Test");

System.loadLibrary("hello-jni");

//>=6.0:

System.loadLibrary("hello-jni");

Android.mk

​ 使用Android.mk在 >=6.0 设备上不能再使用预编译动态库(静态库没问题)：


LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := Test

#libTest.so放在当前文件同目录

LOCAL_SRC_FILES := libTest.so

#预编译库

include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)

#引入上面的Test模块

LOCAL_SHARED_LIBRARIES := Test

LOCAL_MODULE := hello-jni

LOCAL_SRC_FILES := hello-jni.c

include $(BUILD_SHARED_LIBRARY)

上面这段配置生成的libhllo-jni在>=6.0设备中无法执行。

CMake

​ 使用CMakeList.txt在 >=6.0 设备上引入预编译动态库:


cmake_minimum_required(VERSION 3.4.1)

file(GLOB SOURCE *.c )

add_library(

            hello-jni

            SHARED

            ${SOURCE} )

#这段配置在6.0依然没问题

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -L[SO所在目录]")

#这段配置只能在6.0以下使用 原因和android.mk一样

#add_library(Test SHARED IMPORTED)

#set_target_properties(Test PROPERTIES IMPORTED_LOCATION [SO绝对地址])

target_link_libraries(  hello-jni Test )

注意事项

​ 从6.0开始 使用Android.mk 如果来引入一个预编译动态库 有问题

在4.4上 如果load一个动态库 ，需要先将这个动态库的依赖的其他动态库load进来

在6.0以下 System.loadLibrary 不会自动为我们加载依赖的动态库

6.0以上 System.loadLibrary 会自动为我们加载依赖的动态库

