cmake_minimum_required(VERSION 3.5)

project(qtwebsocketsdemo VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(ANDROID_PACKAGE_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/android)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets LinguistTools Network Concurrent Charts Multimedia MultimediaWidgets Sql Svg)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets LinguistTools Network Concurrent Charts Multimedia MultimediaWidgets Sql Svg)

set(PROJECT_SOURCES
    src/main.cpp
    src/mainwindow.cpp
    src/mainwindow.h
    src/mainwindow.ui
    resource.qrc
)

qt5_add_resources(QT_RESOURCE_FILES ${PROJECT_DIR}/resource.qrc)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(${PROJECT_NAME}
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
else()
    if(ANDROID)
        add_library(${PROJECT_NAME} SHARED
            ${PROJECT_SOURCES}
        )
    set(ANDROID_MIN_SDK_VERSION 21)
else()
    add_executable(${PROJECT_NAME}
        ${PROJECT_SOURCES}
    )
endif()
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE Qt${QT_VERSION_MAJOR}::Widgets Qt${QT_VERSION_MAJOR}::Network Qt${QT_VERSION_MAJOR}::Sql Qt${QT_VERSION_MAJOR}::Svg
    Qt${QT_VERSION_MAJOR}::Concurrent Qt${QT_VERSION_MAJOR}::Charts Qt${QT_VERSION_MAJOR}::Multimedia Qt${QT_VERSION_MAJOR}::MultimediaWidgets)

set_target_properties(${PROJECT_NAME} PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

install(TARGETS ${PROJECT_NAME}
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(${PROJECT_NAME})
endif()

# =============================================


set_property(TARGET ${PROJECT_NAME} PROPERTY QT_ANDROID_PRIVATE_KEY_SKIP_VALIDATION ON)

if (WIN32)

elseif (ANDROID)
    find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS AndroidExtras)
    find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS AndroidExtras)
    target_link_libraries(${PROJECT_NAME} PRIVATE Qt${QT_VERSION_MAJOR}::AndroidExtras)

elseif (UNIX AND NOT APPLE AND NOT WIN32 AND NOT ANDROID)
    set(CMAKE_CXX_FLAGS "-fno-pie -no-pie ${CMAKE_CXX_FLAGS}")
    set(CMAKE_C_FLAGS "-fno-pie -no-pie ${CMAKE_CXX_FLAGS}")

endif()

add_definitions(-D_HAS_STD_BYTE=0)

# =============================================


set(QtWebsockets_include ${CMAKE_CURRENT_SOURCE_DIR}/../../output/include)
include_directories(${QtWebsockets_include})

if (WIN32)
    if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        set(QtWebsockets_platform "win32msvc")
    elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        set(QtWebsockets_platform "win32mingw")
    endif()
    set(QtWebsockets_lib_dir ${CMAKE_CURRENT_SOURCE_DIR}/../../output/lib/${QtWebsockets_platform}/QtWebsockets)
    target_link_directories(${PROJECT_NAME} PRIVATE ${QtWebsockets_lib_dir})
    target_link_libraries(${PROJECT_NAME} PRIVATE QtWebsockets)
elseif (ANDROID)
    set(QtWebsockets_platform "android")
    set(QtWebsockets_lib_dir ${CMAKE_CURRENT_SOURCE_DIR}/../../output/lib/${QtWebsockets_platform}/QtWebsockets)
    set(QtWebsockets_lib ${QtWebsockets_lib_dir}/libQtWebsockets_armeabi-v7a.so)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${QtWebsockets_lib}" "${CMAKE_CURRENT_BINARY_DIR}/android-build/libs/${ANDROID_ABI}/libQtWebsockets_armeabi-v7a.so")
    target_link_directories(${PROJECT_NAME} PRIVATE ${QtWebsockets_lib_dir})
    target_link_libraries(${PROJECT_NAME} PRIVATE "QtWebsockets_armeabi-v7a")

elseif (UNIX AND NOT APPLE AND NOT WIN32 AND NOT ANDROID)
    set(QtWebsockets_platform "linux64")
    set(QtWebsockets_lib_dir ${CMAKE_CURRENT_SOURCE_DIR}/../../output/lib/${QtWebsockets_platform}/QtWebsockets)
    target_link_directories(${PROJECT_NAME} PRIVATE ${QtWebsockets_lib_dir})
    target_link_libraries(${PROJECT_NAME} PRIVATE QtWebsockets)

endif()


# =============================================

if (ANDROID)
    set(qtandroidcls_include ${CMAKE_CURRENT_SOURCE_DIR}/include)
    include_directories(${qtandroidcls_include})

    set(qtandroidcls_platform "android")
    set(qtandroidcls_lib_dir ${CMAKE_CURRENT_SOURCE_DIR}/lib/${qtandroidcls_platform}/qtandroidcls)
    set(qtandroidcls_lib ${qtandroidcls_lib_dir}/libqtandroidcls_armeabi-v7a.so)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${qtandroidcls_lib}" "${CMAKE_CURRENT_BINARY_DIR}/android-build/libs/${ANDROID_ABI}/libqtandroidcls_armeabi-v7a.so")
    target_link_directories(${PROJECT_NAME} PRIVATE ${qtandroidcls_lib_dir})
    target_link_libraries(${PROJECT_NAME} PRIVATE "qtandroidcls_armeabi-v7a")
endif()

# =============================================

