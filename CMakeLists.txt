cmake_minimum_required(VERSION 3.16)

project(plasma-volume-control)

set(PROJECT_VERSION "5.23.3")
set(PROJECT_VERSION_MAJOR 5)

set(QT_MIN_VERSION "5.15.0")
set(KF5_MIN_VERSION "5.86")
set(KDE_COMPILERSETTINGS_LEVEL "5.82")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(ECM ${KF5_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/ ${ECM_MODULE_PATH})

add_definitions(-DTRANSLATION_DOMAIN=\"kcm_pulseaudio\" -DQT_NO_KEYWORDS)

include(FeatureSummary)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(KDEClangFormat)
include(KDEGitCommitHooks)
include(ECMSetupVersion)
include(ECMOptionalAddSubdirectory)
include(FindPkgConfig)

ecm_setup_version(${PROJECT_VERSION}
    VARIABLE_PREFIX PLASMAPA
    VERSION_HEADER ${CMAKE_BINARY_DIR}/version.h)

find_package(Qt5 ${QT_MIN_VERSION} REQUIRED COMPONENTS
    Core
    Gui
    DBus
    Quick
)
find_package(KF5 ${KF5_MIN_VERSION} REQUIRED COMPONENTS
    CoreAddons
    Declarative
    DocTools
    GlobalAccel
    Notifications
    I18n
    Plasma
)

find_package(Canberra REQUIRED)

option(USE_GCONF "Use legacy GConf instead of GSettings")

pkg_check_modules(LIBPULSE libpulse REQUIRED IMPORTED_TARGET)
pkg_check_modules(LIBPULSE_MAINLOOP libpulse-mainloop-glib REQUIRED IMPORTED_TARGET)
if(USE_GCONF)
    pkg_check_modules(GCONF gconf-2.0 REQUIRED IMPORTED_TARGET)
    pkg_check_modules(GOBJECT gobject-2.0 REQUIRED IMPORTED_TARGET)
else()
    pkg_check_modules(GIO gio-2.0 REQUIRED IMPORTED_TARGET)
    set(USE_GSETTINGS True)
endif()

find_package(CanberraPulse)
set_package_properties(CanberraPulse PROPERTIES
    DESCRIPTION "Pulseaudio backend for libcanberra"
    PURPOSE "Required for volume feedback sounds"
    TYPE RUNTIME
)

find_package(SoundThemeFreedesktop)
set_package_properties(SoundThemeFreedesktop PROPERTIES
    DESCRIPTION "The standard freedesktop sound theme"
    PURPOSE "Required for volume feedback sounds"
    URL "https://www.freedesktop.org/wiki/Specifications/sound-theme-spec/"
    TYPE RUNTIME
)

find_package(KF5Kirigami2 ${KF5_MIN_VERSION} CONFIG)
set_package_properties(KF5Kirigami2 PROPERTIES
    DESCRIPTION "A QtQuick based components set"
    PURPOSE "Required at runtime the KCM"
    TYPE RUNTIME
)

configure_file(config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)

add_subdirectory(applet)
add_subdirectory(src)
add_subdirectory(data)
add_subdirectory(doc)

# add clang-format target for all our real source files
file(GLOB_RECURSE ALL_CLANG_FORMAT_SOURCE_FILES *.cpp *.h)
kde_clang_format(${ALL_CLANG_FORMAT_SOURCE_FILES})
kde_configure_git_pre_commit_hook(CHECKS CLANG_FORMAT)

feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)

find_package(KF5I18n CONFIG REQUIRED)
ki18n_install(po)

  find_package(KF5DocTools CONFIG)
  if(KF5DocTools_FOUND)
    kdoctools_install(po)
  endif()
