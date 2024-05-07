
cmake_minimum_required(VERSION 3.5)

project(QtWebsockets VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_definitions(-DQT_BUILD_WEBSOCKETS_LIB)

set(QTWEBSOCKETS_PROJECT_SOURCES
    "./src/qsslserver_p.h"
    "./src/qsslserver.cpp"
    # "./qwebsocket_wasm_p.cpp"

    "./src/qwebsockets_global.h"
    "./src/qwebsocket.h"
    "./src/qwebsocketserver.h"
    "./src/qwebsocketprotocol.h"
    "./src/qwebsocketcorsauthenticator.h"
    "./src/qmaskgenerator.h"

    "./src/qwebsocket_p.h"
    "./src/qwebsocketserver_p.h"
    "./src/qwebsocketprotocol_p.h"
    "./src/qwebsockethandshakerequest_p.h"
    "./src/qwebsockethandshakeresponse_p.h"
    "./src/qwebsocketdataprocessor_p.h"
    "./src/qwebsocketcorsauthenticator_p.h"
    "./src/qwebsocketframe_p.h"
    "./src/qdefaultmaskgenerator_p.h"

    "./src/qwebsocket.cpp"
    "./src/qwebsocket_p.cpp"
    "./src/qwebsocketserver.cpp"
    "./src/qwebsocketserver_p.cpp"
    "./src/qwebsocketprotocol.cpp"
    "./src/qwebsockethandshakerequest.cpp"
    "./src/qwebsockethandshakeresponse.cpp"
    "./src/qwebsocketdataprocessor.cpp"
    "./src/qwebsocketcorsauthenticator.cpp"
    "./src/qwebsocketframe.cpp"
    "./src/qmaskgenerator.cpp"
    "./src/qdefaultmaskgenerator_p.cpp"

)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Network)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Network)

add_library(${PROJECT_NAME} SHARED ${QTWEBSOCKETS_PROJECT_SOURCES})

target_link_libraries(${PROJECT_NAME} PRIVATE Qt${QT_VERSION_MAJOR}::CorePrivate Qt${QT_VERSION_MAJOR}::Network)

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

if (WIN32)
    message("CMAKE_CXX_COMPILER_ID ${CMAKE_CXX_COMPILER_ID}")
    if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        ADD_CUSTOM_COMMAND(TARGET ${PROJECT_NAME} POST_BUILD COMMAND python "${CMAKE_SOURCE_DIR}/script/package.py" win32msvc) # PRE_BUILD | PRE_LINK| POST_BUILD
    elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        ADD_CUSTOM_COMMAND(TARGET ${PROJECT_NAME} POST_BUILD COMMAND python "${CMAKE_SOURCE_DIR}/script/package.py" win32mingw)
    endif ()
    set_target_properties(${PROJECT_NAME} PROPERTIES
        WINDOWS_EXPORT_ALL_SYMBOLS TRUE
    )

elseif (ANDROID)
    ADD_CUSTOM_COMMAND(TARGET ${PROJECT_NAME} POST_BUILD COMMAND python3 "${CMAKE_SOURCE_DIR}/script/package.py" android)
    set(ANDROID_ABI armeabi-v7a)
    set(ANDROID_NATIVE_API_LEVEL 21)
    # set_target_properties(${PROJECT_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}")
elseif (UNIX AND NOT APPLE AND NOT WIN32 AND NOT ANDROID)
    ADD_CUSTOM_COMMAND(TARGET ${PROJECT_NAME} POST_BUILD COMMAND python3 "${CMAKE_SOURCE_DIR}/script/package.py" linux64)
endif()

add_definitions(-D_HAS_STD_BYTE=0)
