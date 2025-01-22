cmake_minimum_required(VERSION 3.0.2)
project(Lite_motion)

## 可以在这里设置默认编译为 Release（可选）
set(CMAKE_BUILD_TYPE Release)

## 设置C++11标准、屏蔽警告（-w 可选）
add_definitions(-std=c++11 -w)

## 通过外部参数决定平台类型，例如：
##   catkin build --cmake-args -DBUILD_PLATFORM=arm
## 或   cmake .. -DBUILD_PLATFORM=arm
SET(BUILD_PLATFORM "x86" CACHE STRING "选择编译CPU类型：arm 或 x86")
if (BUILD_PLATFORM STREQUAL "arm")
  message(STATUS "===> This is arm platform <===")
  # 若需要在同一台 x86 主机上做交叉编译，可在此设置交叉编译工具链：
  SET(CMAKE_C_COMPILER   "aarch64-linux-gnu-gcc")
  SET(CMAKE_CXX_COMPILER "aarch64-linux-gnu-g++")
else()
  message(STATUS "===> This is x86 platform <===")
endif()

## ROS 依赖：根据需要添加其他组件，例如 std_msgs、sensor_msgs 等
find_package(catkin REQUIRED COMPONENTS
  roscpp
)

###################################
## catkin_package 声明本包信息
###################################
catkin_package(
  # LIBRARIES Lite_motion
  CATKIN_DEPENDS roscpp
  # DEPENDS system_lib
)

###################################
## 包含头文件目录
###################################
include_directories(
  include
  include/common
  lib/eigen3
  ${catkin_INCLUDE_DIRS}
)

## 如果需要链接某些库所在路径
link_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/lib
)

###################################
## 处理源码文件
###################################
# 方式1：自动搜集 src/ 下所有 .cpp
file(GLOB SRC_LIST 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
)

# 也可根据需要手动列出需要编译的 cpp 文件
# set(SRC_LIST
#     src/some_file.cpp
#     src/another_file.cpp
# )

###################################
## 生成可执行文件
###################################
add_executable(${PROJECT_NAME}
    src/main.cpp
    ${SRC_LIST}
)

###################################
## 链接库
###################################
if (BUILD_PLATFORM STREQUAL "arm")
  target_link_libraries(${PROJECT_NAME}
    ${catkin_LIBRARIES}
    # 注意如果你的库名是带前缀的完整 .so，就写完整名字
    # 如果是 libdeeprobotics_legged_sdk_aarch64.so，可省略 lib 和 .so 部分
    deeprobotics_legged_sdk_aarch64
    pthread
    m
    rt
    dl
    stdc++
  )
else()
  target_link_libraries(${PROJECT_NAME}
    ${catkin_LIBRARIES}
    deeprobotics_legged_sdk_x86_64
    pthread
    m
    rt
    dl
    stdc++
  )
endif()

###################################
## 可选：安装配置，用于安装到ROS环境
###################################
install(TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)

