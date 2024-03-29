cmake_minimum_required(VERSION 3.5)

project(Universal-Audio-Visualizer VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS OpenGLWidgets Gui Core)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS OpenGLWidgets Gui Core)

include_directories(Headers/
                    Shaders/
                    Sources/
                    img/
                    lib/
                    lib/aubio/include)

link_directories(lib/aubio/lib)

set(PROJECT_SOURCES
        Headers/mainwindow.h
        Headers/oglwidget.h
        Headers/audiosystem.h
        Headers/endpointmenu.h
        Headers/slider.h
        Headers/Shapes/Shape.h
        Headers/Shapes/Cube.h
        Headers/Shapes/Sphere.h
        Headers/Shapes/Prism.h
        Headers/stdafx.h
        Sources/mainwindow.ui
        Sources/main.cpp
        Sources/mainwindow.cpp
        Sources/oglwidget.cpp
        Sources/endpointmenu.cpp
        Sources/audiosystem.cpp
        Sources/stdafx.cpp
        Sources/slider.ui
        Sources/slider.cpp
        Shaders.qrc
        Shaders/vertex.glsl
        Shaders/fragment.glsl
        Images.qrc
        resources.rc
)

set(EXECUTABLE_OUTPUT_PATH ${RUNTIME_OUTPUT_DIRECTORY})

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(Universal-Audio-Visualizer
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET Universal-Audio-Visualizer APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(Universal-Audio-Visualizer SHARED
            ${PROJECT_SOURCES}
        )
# Define properties for Android with Qt 5 after find_package() calls as:
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(Universal-Audio-Visualizer
            ${PROJECT_SOURCES}
        )
    endif()
endif()

target_link_libraries(Universal-Audio-Visualizer PRIVATE Qt${QT_VERSION_MAJOR}::Widgets
    PRIVATE Qt${QT_VERSION_MAJOR}::OpenGLWidgets
    PRIVATE Qt${QT_VERSION_MAJOR}::Gui
    PRIVATE Qt${QT_VERSION_MAJOR}::Core)

unset(CMAKE_LINK_LIBRARY_SUFFIX)
target_link_libraries(Universal-Audio-Visualizer PRIVATE "libaubio.a")


set_target_properties(Universal-Audio-Visualizer PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

add_custom_command(TARGET Universal-Audio-Visualizer POST_BUILD
               COMMAND ${CMAKE_COMMAND} -E copy_if_different
               "${PROJECT_SOURCE_DIR}/lib/aubio/bin/libaubio-5.dll"
                $<TARGET_FILE_DIR:Universal-Audio-Visualizer>)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(Universal-Audio-Visualizer)
endif()
