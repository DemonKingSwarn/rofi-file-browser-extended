cmake_minimum_required(VERSION 3.16)

project(
    rofi-file-browser-extended
    VERSION 1.0
    LANGUAGES C
)

include(CheckSymbolExists)
include(GNUInstallDirs)

# --------------------------------------------------
# Dependencies
# --------------------------------------------------

find_package(PkgConfig REQUIRED)

pkg_check_modules(GLIB2 REQUIRED IMPORTED_TARGET glib-2.0)
pkg_check_modules(CAIRO REQUIRED IMPORTED_TARGET cairo)

pkg_get_variable(ROFI_PLUGINS_DIR rofi pluginsdir)

# --------------------------------------------------
# Sources
# --------------------------------------------------

file(GLOB SRC CONFIGURE_DEPENDS
    src/*.c
)

# --------------------------------------------------
# Feature detection: FTW_ACTIONRETVAL
# --------------------------------------------------

set(CMAKE_REQUIRED_DEFINITIONS -D_GNU_SOURCE)
check_symbol_exists(FTW_ACTIONRETVAL "ftw.h" HAVE_FTW_ACTIONRETVAL)
unset(CMAKE_REQUIRED_DEFINITIONS)

if(NOT HAVE_FTW_ACTIONRETVAL)
    list(APPEND SRC src/posix-compat/extended_nftw.c)
endif()

# --------------------------------------------------
# Plugin target
# --------------------------------------------------

add_library(filebrowser SHARED ${SRC})

set_target_properties(filebrowser PROPERTIES
    PREFIX ""
    C_STANDARD 99
    C_STANDARD_REQUIRED YES
)

target_include_directories(filebrowser PRIVATE
    include
)

target_link_libraries(filebrowser PRIVATE
    PkgConfig::GLIB2
    PkgConfig::CAIRO
)

if(HAVE_FTW_ACTIONRETVAL)
    target_compile_definitions(filebrowser PRIVATE
        _GNU_SOURCE
        HAVE_FTW_ACTIONRETVAL
    )
else()
    target_compile_definitions(filebrowser PRIVATE
        _XOPEN_SOURCE=700
    )
endif()

# --------------------------------------------------
# Install
# --------------------------------------------------

install(
    TARGETS filebrowser
    LIBRARY DESTINATION "${ROFI_PLUGINS_DIR}"
)

# --------------------------------------------------
# Manpage
# --------------------------------------------------

add_custom_command(
    OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/doc/rofi-file-browser-extended.1.gz"
    COMMAND gzip -kf "${CMAKE_CURRENT_SOURCE_DIR}/doc/rofi-file-browser-extended.1"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/doc/rofi-file-browser-extended.1"
    COMMENT "Packing manpage"
)

add_custom_target(manpage ALL
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/doc/rofi-file-browser-extended.1.gz"
)

install(
    FILES "doc/rofi-file-browser-extended.1.gz"
    DESTINATION "${CMAKE_INSTALL_MANDIR}/man1"
)

