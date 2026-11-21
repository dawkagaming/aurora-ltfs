cmake_minimum_required(VERSION 3.28)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

project(aurora-ltfs
    VERSION 1.5.0
    LANGUAGES C
)

option(ENABLE_TESTS "Build tests" OFF)
option(INSTALL_EXAMPLES "Install examples" ON)
option(INSTALL_DOCS "Install documentation files" ON)

# Find libraries
find_package(PkgConfig)

pkg_check_modules(LIBFUSE REQUIRED IMPORTED_TARGET GLOBAL fuse)
pkg_check_modules(LIBXML2 REQUIRED IMPORTED_TARGET GLOBAL libxml-2.0)
# pkg_check_modules(LIBICUIO REQUIRED IMPORTED_TARGET GLOBAL icu-io) # Which ICU?
pkg_check_modules(LIBUUID REQUIRED IMPORTED_TARGET GLOBAL uuid)

# Compatibility with autoconf
include(CheckIncludeFile)
include(CheckSymbolExists)
include(CheckStructHasMember)
include(CheckTypeSize)

# Headers
check_include_file(fcntl.h HAVE_FCNTL_H)
check_include_file(limits.h HAVE_LIMITS_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(stdlib.h HAVE_STDLIB_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(unistd.h HAVE_UNISTD_H)

# Functions
check_symbol_exists(strdup "string.h" HAVE_STRDUP)
check_symbol_exists(strndup "string.h" HAVE_STRNDUP)
check_symbol_exists(strnlen "string.h" HAVE_STRNLEN)

# Structures
check_struct_has_member(
    "struct stat" st_blksize
    "sys/types.h;sys/stat.h"
    HAVE_STRUCT_STAT_ST_BLKSIZE
)

check_struct_has_member(
    "struct stat" st_rdev
    "sys/types.h;sys/stat.h"
    HAVE_STRUCT_STAT_ST_RDEV
)

# Types
check_type_size("time_t" SIZEOF_TIME_T)

check_symbol_exists(
    xmlTextReaderSetup
    "libxml/xmlreader.h"
    HAVE_XML_READER_SETUP
)

check_symbol_exists(
    XML_PARSE_HUGE
    "libxml/parser.h"
    HAVE_XML_PARSE_HUGE
)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/config.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/config.h"
)

include_directories("${CMAKE_CURRENT_BINARY_DIR}")

###

add_subdirectory(src)

if(ENABLE_TESTS)
    add_subdirectory(tests)
endif()

# Misc installs

include(GNUInstallDirs)

# Configuration files
add_subdirectory(conf)

# Manual pages
add_subdirectory(man)

# Examples
if(INSTALL_EXAMPLES)
    add_subdirectory(examples)
endif()

# Documentation
if(INSTALL_DOCS)
    add_subdirectory(docs)
endif()