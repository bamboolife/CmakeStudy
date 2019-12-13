cmake_minimum_required(VERSION 3.4.1)

################Camke语法练习############################
#set(var 123)
#message("var=${var}")
#
#set(list_var 1 2 3 4 5 )
#foreach(item IN LISTS list_var)
#    message( WARNING "item=${item}")
#endforeach(item)

message(WARNING "CMAKE_SOURCE_DIR = ${CMAKE_SOURCE_DIR}/.")
message(WARNING "PROJECT_SOURCE_DIR = ${PROJECT_SOURCE_DIR}")
message(WARNING "CMAKE_BINARY_DIR = ${CMAKE_BINARY_DIR}")
message(WARNING "CMAKE_CXX_FLAGS = ${CMAKE_CXX_FLAGS}")
message(WARNING "CMAKE_CXX_FLAGS = ${CMAKE_CXX_FLAGS} -L$-L${CMAKE_SOURCE_DIR}")




add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        native-lib.cpp)

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
        native-lib

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})