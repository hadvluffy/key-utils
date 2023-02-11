cmake_minimum_required(VERSION 3.5)

project(key-utils VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(QT NAMES Qt6 Qt5 COMPONENTS Widgets REQUIRED)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Widgets REQUIRED)

add_subdirectory(contrib/bitcoin-core)

set(PROJECT_SOURCES
        main.cpp
        mainwindow.cpp
        mainwindow.h
        mainwindow.ui
        xprivDialog.cpp
        xprivDialog.h
        xprivDialog.ui
        verifiedDialog.cpp
        verifiedDialog.h
        verifiedDialog.ui
        tapsigner_utils.h
        tapsigner_utils.cpp
)

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
    elseif (APPLE)
        # NOTE_: Don't include the path in MACOSX_BUNDLE_ICON_FILE -- this is
        # the property added to Info.plist
        set(MACOSX_BUNDLE_ICON_FILE Icon.icns)

        # And this part tells CMake where to find and install the file itself
        set(APP_ICON_MACOSX ${CMAKE_CURRENT_SOURCE_DIR}/Icon.icns)
        set_source_files_properties(${APP_ICON_MACOSX} PROPERTIES
               MACOSX_PACKAGE_LOCATION "Resources")
        add_executable(${PROJECT_NAME} MACOSX_BUNDLE
            ${PROJECT_SOURCES}
            resources.qrc
        )
        # set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/Info.plist")
    else ()
        add_executable(${PROJECT_NAME} WIN32
            ${PROJECT_SOURCES}
            resources.qrc
        )
    endif()
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE Qt${QT_VERSION_MAJOR}::Widgets bitcoin-core)

if (APPLE)
    SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE TRUE)
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER io.nunchuk.macos
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(${PROJECT_NAME})
endif()
