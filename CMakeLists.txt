cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES ON)


if ("${CMAKE_BINARY_DIR}" STREQUAL "${CMAKE_SOURCE_DIR}")
  message(FATAL_ERROR "In-source builds are disabled.
    Please create a subfolder and use `cmake ..` inside it.
    NOTE: cmake will now create CMakeCache.txt and CMakeFiles/*.
          You must delete them, or cmake will refuse to work.")
endif()

project(fcitx5-configtool VERSION 5.1.10)

set(QT_MIN_VERSION "6.4.0")
set(QT_MAJOR_VERSION 6)

find_package(ECM 5.68.0 REQUIRED)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR})
find_package(Qt${QT_MAJOR_VERSION} ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS Core Gui Widgets Concurrent)

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings)
include(FeatureSummary)
include(ECMSetupVersion)
include(GenerateExportHeader)
option(ENABLE_KCM "Enable KDE Config Module" On)
option(ENABLE_CONFIG_QT "Enable fcitx5-config-qt" On)
option(ENABLE_TEST "Enable test" Off)
add_definitions(-DTRANSLATION_DOMAIN=\"kcm_fcitx5\")
add_definitions(-DFCITX_GETTEXT_DOMAIN=\"fcitx5-configtool\")
add_definitions(-DQT_NO_KEYWORDS)

set(CONFIG_QT_UNIQUE_APP_SUPPORT 0)
if (ENABLE_CONFIG_QT)
    find_package(KF${QT_MAJOR_VERSION}ItemViews REQUIRED)
    find_package(KF${QT_MAJOR_VERSION}DBusAddons 5.91)
    find_package(KF${QT_MAJOR_VERSION}WindowSystem 5.91)

    if (TARGET KF${QT_MAJOR_VERSION}::DBusAddons AND TARGET KF${QT_MAJOR_VERSION}::WindowSystem)
        set(CONFIG_QT_UNIQUE_APP_SUPPORT 1)
    endif()
endif()

if (ENABLE_KCM)
    find_package(Qt${QT_MAJOR_VERSION}Quick REQUIRED)
    find_package(Qt${QT_MAJOR_VERSION}QuickControls2 REQUIRED)
    find_package(KF${QT_MAJOR_VERSION}CoreAddons REQUIRED)
    find_package(KF${QT_MAJOR_VERSION}I18n REQUIRED)
    find_package(KF${QT_MAJOR_VERSION}Package REQUIRED)
    find_package(KF${QT_MAJOR_VERSION}Declarative REQUIRED)
    find_package(KF${QT_MAJOR_VERSION}IconThemes REQUIRED)
    find_package(XKBCommon REQUIRED COMPONENTS XKBCommon)
    find_package(KF${QT_MAJOR_VERSION}Kirigami REQUIRED)
    find_package(Plasma REQUIRED)
    find_package(KF${QT_MAJOR_VERSION}Svg REQUIRED)
    find_package(KF${QT_MAJOR_VERSION}KCMUtils REQUIRED)
endif()

find_package(Fcitx5Core 5.1.13 REQUIRED)
find_package(Fcitx5Config REQUIRED)
find_package(Fcitx5Utils REQUIRED)
find_package(Fcitx5Qt${QT_MAJOR_VERSION}DBusAddons 5.1.4 REQUIRED)
find_package(Fcitx5Qt${QT_MAJOR_VERSION}WidgetsAddons 5.0.2 REQUIRED)
find_package(Qt${QT_MAJOR_VERSION}Svg)
set_package_properties(Qt${QT_MAJOR_VERSION}Svg PROPERTIES
    DESCRIPTION "May be used for show icons"
    TYPE RUNTIME)

get_target_property(FCITX5_QT_GUI_WRAPPER Fcitx5Qt${QT_MAJOR_VERSION}::gui-wrapper IMPORTED_LOCATION)

find_package(KF${QT_MAJOR_VERSION}WidgetsAddons REQUIRED)
find_package(IsoCodes REQUIRED)
kde_enable_exceptions()

include("${FCITX_INSTALL_CMAKECONFIG_DIR}/Fcitx5Utils/Fcitx5CompilerSettings.cmake")

find_package(PkgConfig REQUIRED)
find_package(Gettext REQUIRED)
find_package(X11 REQUIRED)
find_package(XKeyboardConfig REQUIRED)
pkg_check_modules(XkbFile "xkbfile" REQUIRED IMPORTED_TARGET)

configure_file(config.h.in config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_subdirectory(po)
add_subdirectory(src)
add_subdirectory(layout)

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
