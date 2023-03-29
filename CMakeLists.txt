# 指定最低支持的 CMake 版本为 3.2
cmake_minimum_required(VERSION 3.2)

# 更新策略设置，以避免在当前cmake版本级别加载ament_cmake包时出错
# Update the policy setting to avoid an error when loading the ament_cmake package
# at the current cmake version level
if(POLICY CMP0057)
    cmake_policy(SET CMP0057 NEW)
endif()

project(groot)

# 查找并载入 Qt5 相关的组件
find_package(Qt5 COMPONENTS Core Widgets Gui OpenGL Xml Svg)
# 添加自定义的 CMake 模块路径
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}  "${CMAKE_CURRENT_LIST_DIR}/cmake")

# 如果 CMake 版本不低于 3.1，则设置 C++ 标准为 11，否则设置编译选项为 -std=c++11
if(NOT CMAKE_VERSION VERSION_LESS 3.1)
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

#############################################################
# 查找并载入 ament_cmake 和 catkin，
# 如果找到了 ament_cmake，则使用 ament_cmake 编译，否则使用 catkin 编译
find_package(ament_cmake QUIET)
find_package(catkin QUIET)

# 如果找到 ament_cmake，则会使用 ament_cmake 来编译软件包
# 如果没有 ament_cmake，则会使用 catkin      来编译软件包
if(ament_cmake_FOUND)
# ament_cmake 是一个 CMake 扩展，用于构建 ROS 2 软件包。
# 它提供了一组 CMake 宏和函数，用于自动化 ROS 2 软件包的构建和安装过程

# ament_index_cpp 是一个 CMake 扩展，用于查找和载入 ROS 2 软件包。
# 它提供了一组 CMake 宏和函数，用于自动化 ROS 2 软件包的构建和安装过程
  find_package(ament_index_cpp REQUIRED)
  find_package(behaviortree_cpp_v3 REQUIRED)

  set(dependencies
    behaviortree_cpp_v3
  )

  message(STATUS "---------------------")
  message(STATUS "Compiling with AMENT.")
  message(STATUS "---------------------")
elseif(catkin_FOUND OR CATKIN_DEVEL_PREFIX OR CATKIN_BUILD_BINARY_PACKAGE )
# http://answers.ros.org/question/230877/optionally-build-a-package-with-catkin/
  set(catkin_FOUND 1)
  # add_definitions( -DUSING_ROS )

  find_package(catkin REQUIRED COMPONENTS behaviortree_cpp_v3)

  message(STATUS "----------------------")
  message(STATUS "Compiling with CATKIN.")
  message(STATUS "----------------------")

  catkin_package(
    INCLUDE_DIRS 
    LIBRARIES qt_node_editor
    CATKIN_DEPENDS behaviortree_cpp_v3
    DEPENDS 
    )
endif()
#############################################################

# 启用自动 MOC、UIC 和 RCC，以及当前目录的包含路径
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

# 添加到编译器的定义中
add_definitions(${Qt5Widgets_DEFINITIONS} -DNODE_EDITOR_STATIC )

# Qt5Widgets_EXECUTABLE_COMPILE_FLAGS 是 Qt5Widgets 模块的编译标志，它包含了 Qt5Widgets 模块的所有编译选项
# CMAKE_CXX_FLAGS 是 C++ 编译器的标志，它用于指定编译器的行为，例如编译器的优化级别、警告级别等
# Qt5Widgets 模块的编译标志添加到 C++ 编译器的标志中，以便在编译时使用
set(CMAKE_CXX_FLAGS "${Qt5Widgets_EXECUTABLE_COMPILE_FLAGS}")

#set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-frame-pointer -fsanitize=address")
#set (CMAKE_LINKER_FLAGS_DEBUG "${CMAKE_STATIC_LINKER_FLAGS_DEBUG} -fno-omit-frame-pointer -fsanitize=address")

if(ament_cmake_FOUND)
# 将 include 和 ./QtNodeEditor/include 目录添加到编译器的包含路径中
  INCLUDE_DIRECTORIES(include ./QtNodeEditor/include ./)
else()
  INCLUDE_DIRECTORIES(${catkin_INCLUDE_DIRS} ./QtNodeEditor/include ./  )
endif()

# 将QtNodeEditor目录作为子目录添加到当前项目中
add_subdirectory( QtNodeEditor )

##########################################
# FIX ME in the future....

# 将BUILD_EXAMPLES变量设置为OFF，并将其缓存为BOOL类型
set(BUILD_EXAMPLES   OFF CACHE BOOL "")
set(BUILD_UNIT_TESTS OFF CACHE BOOL "")
set(BUILD_TOOLS OFF CACHE BOOL "")
set(BUILD_TESTING OFF CACHE BOOL "")
set(CATKIN_ENABLE_TESTING OFF CACHE BOOL "")

if(NOT catkin_FOUND AND NOT ament_cmake_FOUND)

    if(NOT EXISTS depend/BehaviorTree.CPP)
      message("Can't find the submodule BehaviorTree.CPP")
      message("Run [git submodule update --init --recursive]")
    endif()

    SET(BUILD_EXAMPLES OFF CACHE BOOL "Build tutorials and examples" FORCE)
    SET(BUILD_UNIT_TESTS OFF CACHE BOOL "Build the unit tests" FORCE)
    SET(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared libraries" FORCE)

    # look for BehaviorTree.CPP as submodule
    # use git submodule only if you are not compiling with catkin
    add_subdirectory( depend/BehaviorTree.CPP )
    include_directories( depend/BehaviorTree.CPP/include )

    message(STATUS "----------------------------------")
    message(STATUS "BehaviourTree.CPP submodule found.")
    message(STATUS "----------------------------------")
endif()

##########################################

set(APP_CPPS
    ./bt_editor/models/BehaviorTreeNodeModel.cpp
    ./bt_editor/models/SubtreeNodeModel.cpp

    ./bt_editor/mainwindow.cpp
    ./bt_editor/editor_flowscene.cpp
    ./bt_editor/utils.cpp
    ./bt_editor/bt_editor_base.cpp
    ./bt_editor/graphic_container.cpp
    ./bt_editor/startup_dialog.cpp

    ./bt_editor/sidepanel_editor.cpp
    ./bt_editor/sidepanel_replay.cpp
    ./bt_editor/custom_node_dialog.cpp

    ./bt_editor/XML_utilities.cpp
    )

set(RESOURCE_FILES
    ./bt_editor/resources/icons.qrc
    ./bt_editor/resources/style.qrc
    ./test_data/test_files.qrc
    )

set(FORMS_UI
  ./bt_editor/about_dialog.ui
  ./bt_editor/mainwindow.ui
  ./bt_editor/action_form.ui
  ./bt_editor/sidepanel_editor.ui
  ./bt_editor/sidepanel_replay.ui
  ./bt_editor/startup_dialog.ui
  ./bt_editor/custom_node_dialog.ui
  )

find_package(ZMQ)

if( ZMQ_FOUND )
    message(STATUS "ZeroMQ found.")
    # 启用实时监控功能，并指示是否找到了 ZeroMQ
    add_definitions( -DZMQ_FOUND )

    # 添加cpp和ui文件
    set(APP_CPPS ${APP_CPPS} ./bt_editor/sidepanel_monitor.cpp )
    set(FORMS_UI ${FORMS_UI} ./bt_editor/sidepanel_monitor.ui )

else()
    message(WARNING "ZeroMQ NOT found. Real-time monitoring disabled.")
endif()

# 从一组 Qt Designer .ui 文件生成 C++ 代码
# FORMS_HEADERS： 保存生成的头文件列表
# ${FORMS_UI}：要处理的 .ui 文件列表
QT5_WRAP_UI(FORMS_HEADERS ${FORMS_UI})

# 创建了一个名为 behavior_tree_editor 的共享库
# 将其与 ${APP_CPPS} 变量中列出的 C++ 源文件和 ${FORMS_HEADERS} 变量中列出的生成的头文件链接起来
add_library(behavior_tree_editor SHARED
    ${APP_CPPS}
    ${FORMS_HEADERS}
)

# 将 QtNodeEditor、curses、ncursesw 和 tinfo 添加到 GROOT_DEPENDENCIES 变量中
SET(GROOT_DEPENDENCIES QtNodeEditor curses ncursesw tinfo )

if(ament_cmake_FOUND)
    # 将 behavior_tree_editor 库与 ${dependencies} 中列出的依赖项链接起来
    ament_target_dependencies(behavior_tree_editor ${dependencies})
elseif( catkin_FOUND )
    SET(GROOT_DEPENDENCIES ${GROOT_DEPENDENCIES} ${catkin_LIBRARIES} )
else()
    SET(GROOT_DEPENDENCIES ${GROOT_DEPENDENCIES} behaviortree_cpp_v3 )
endif()

if( ZMQ_FOUND )
    # 若为苹果系统
    if (APPLE)
        find_package(cppzmq)
        SET(GROOT_DEPENDENCIES ${GROOT_DEPENDENCIES} cppzmq)
    else()
        SET(GROOT_DEPENDENCIES ${GROOT_DEPENDENCIES} zmq)
    endif()
endif()

# 这将链接目标与 behaviortree_cpp_v3 库，该库在文件的 find_package 部分中指定为依赖项
target_link_libraries(behavior_tree_editor ${GROOT_DEPENDENCIES} )

# 创建一个名为 Groot 的可执行文件，
# 并将 ./bt_editor/main.cpp 和 ${RESOURCE_FILES} 中列出的源文件链接到该可执行文件中
add_executable(Groot ./bt_editor/main.cpp  ${RESOURCE_FILES})
# 将 behavior_tree_editor 库链接到 Groot 可执行文件中，以便在运行时使用
target_link_libraries(Groot behavior_tree_editor )

add_subdirectory(test)

######################################################
# INSTALL
if(ament_cmake_FOUND)
    set( GROOT_LIB_DESTINATION   lib )
    set( GROOT_INC_DESTINATION   include )
    set( GROOT_BIN_DESTINATION   lib/${PROJECT_NAME} )
elseif(catkin_FOUND)
    set( GROOT_LIB_DESTINATION   ${CATKIN_PACKAGE_LIB_DESTINATION} )
    set( GROOT_INC_DESTINATION   ${CATKIN_PACKAGE_INCLUDE_DESTINATION} )
    set( GROOT_BIN_DESTINATION   ${CATKIN_PACKAGE_BIN_DESTINATION} )
else()
    set( GROOT_LIB_DESTINATION   lib )
    set( GROOT_INC_DESTINATION   include )
    set( GROOT_BIN_DESTINATION   bin )
    
    INSTALL( FILES ${CMAKE_CURRENT_SOURCE_DIR}/groot_icon.png
			 DESTINATION share/icons/hicolor/256x256)
			 
	INSTALL( FILES ${CMAKE_CURRENT_SOURCE_DIR}/Groot.desktop
			 DESTINATION share/applications)
    
endif()

INSTALL(TARGETS behavior_tree_editor LIBRARY DESTINATION ${GROOT_LIB_DESTINATION} )
INSTALL(TARGETS Groot RUNTIME DESTINATION ${GROOT_BIN_DESTINATION} )

if(ament_cmake_FOUND)
  ament_export_include_directories(include)
  ament_export_dependencies(${GROOT_DEPENDENCIES} ${dependencies})
  ament_export_libraries(behavior_tree_editor)
  ament_package()
endif()
