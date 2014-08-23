cmake_minimum_required(VERSION 2.8)
project(Znc C CXX)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")
include(CheckFunctionExists)
include(CheckLibraryExists)
include(CheckSymbolExists)
include(CheckIncludeFiles)

option(disable-largefile "omit support for large files" OFF)
if(NOT disable-largefile)
    option(_LARGE_FILES "Define for large files, on AIX-style hosts." ON)
endif(NOT disable-largefile)

option(enable-debug "enable debugging" OFF)
if(NOT disable-debug)
    option(_DEBUG "Define for debugging" ON)
endif(NOT disable-debug)

option(disable-ipv6 "disable ipv6 support" OFF)
if(NOT disable-ipv6)
    option(HAVE_IPV6 "Enable IPV6" ON)
endif(NOT disable-ipv6)

option(disable-openssl "disable openssl" OFF)
if(NOT disable-openssl)
    find_package(OpenSSL)
    if(OPENSSL_FOUND)
        option(HAVE_LIBSSL "Enable OpenSSL support" ON)
        option(HAVE_LIBCRYPTO "Enable Libcrypto support" ON)
        include_directories(${OPENSSL_INCLUDE_DIRS})
        list(APPEND libs_list ${OPENSSL_LIBRARIES})
    endif(OPENSSL_FOUND)
endif(NOT disable-openssl)

option(disable-zlib "disable zlib" OFF)
if(NOT disable-zlib)
    find_package(zlib)
    if(zlib_FOUND)
        option(HAVE_ZLIB "Enable zlib support" ON)
        include_directories(${ZLIB_INCLUDE_DIRS})
        list(APPEND libs_list ${ZLIB_LIBRARIES})
    endif(zlib_FOUND)
endif(NOT disable-zlib)

option(enable-swig "Enable automatic generation of source files needed
                    for modperl/modpython. This value is ignored if perl
                    and python are disabled. Usually no need to enable it." OFF)

if(enable-swig)
    find_package(SWIG 2.0.8 REQUIRED)
endif(enable-swig)

option(enable-cyrus "enable cyrus" OFF)
if(enable-cyrus)
    CHECK_INCLUDE_FILES ("sasl/sasl.h" HAVE_CYRUS_H)
endif(enable-cyrus)

option(disable-optimization "Disable some compiler optimizations to decrease
                             memory usage while compiling" OFF)

option(disable-tdns "disable threads usage for DNS resolving" OFF)
if(NOT disable-tdns)
    option(HAVE_THREADED_DNS "Define if threaded DNS is enabled" ON)
endif(NOT disable-tdns)

option(enable-run-from-source "ZNC will be runnable without installation" OFF)
if(enable-run-from-source)
    option(RUN_FROM_SOURCE "Define if ZNC should be runnable without installation" ON)
endif(enable-run-from-source)

option(disable-poll "use select() instead of poll()" OFF)
if(NOT disable-poll)
    option(CSOCK_USE_POLL "Use poll() instead of select()" ON)
endif(NOT disable-poll)

option(disable-charset "disable ICU support" OFF)
if(NOT disable-charset)
    find_package(ICU)
    if(ICU_FOUND)
        option(HAVE_ICU "Enable ICU support" ON)
        option(U_USING_ICU_NAMESPACE "Disable ICU namespace C++ cluttering" ON)
        list(APPEND libs_list ${ICU_LIBRARIES})
        include_directories(${ICU_INCLUDE_DIRS})
    endif(ICU_FOUND)
endif(NOT disable-charset)

find_package(Threads)
if(Threads_FOUND)
  option(HAVE_PTHREAD "Enable pthreads support" ON)
  list(APPEND libs_list ${CMAKE_THREAD_LIBS_INIT})
  CHECK_SYMBOL_EXISTS(PTHREAD_CREATE_JOINABLE pthread.h HAVE_PTHREAD_CREATE_JOINABLE)
    if(NOT HAVE_PTHREAD_CREATE_JOINABLE)
        CHECK_SYMBOL_EXISTS(PTHREAD_CREATE_DETACHED pthread.h HAVE_PTHREAD_CREATE_DETACHED)
        if(HAVE_PTHREAD_CREATE_DETACHED)
        set(PTHREAD_CREATE_JOINABLE "PTHREAD_CREATE_DETACHED")
        endif(HAVE_PTHREAD_CREATE_DETACHED)
    endif(NOT HAVE_PTHREAD_CREATE_JOINABLE)

    CHECK_SYMBOL_EXISTS(PTHREAD_PRIO_INHERIT pthread.h HAVE_PTHREAD_PRIO_INHERIT)
endif(Threads_FOUND)

check_function_exists(getopt_long HAVE_GETOPT_LONG)
check_function_exists(getphassphrase HAVE_GETPHASSPHRASE)
check_function_exists(lstat HAVE_LSTAT)

check_library_exists(gnugetopt getopt_long "" HAVE_LIBGNUGETOPT)

configure_file (
  "${PROJECT_SOURCE_DIR}/include/znc/zncconfig.h.in"
  "${PROJECT_BINARY_DIR}/include/znc/zncconfig.h"
  )

set(DST ${CMAKE_CURRENT_BINARY_DIR}/version.cpp)
file(WRITE ${DST} "const char* ZNC_VERSION_EXTRA = \"${DESC}\";")

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
include_directories("${PROJECT_BINARY_DIR}/include")
include_directories ("${PROJECT_SOURCE_DIR}/include")

set(DATAROOTDIR ${CMAKE_INSTALL_PREFIX}/share)
set(DATADIR ${DATAROOTDIR}/znc)

add_definitions(-D_MODDIR_=\"${PROJECT_BINARY_DIR}\")
add_definitions(-D_DATADIR_=\"${DATADIR}\")

list(APPEND lib_srcs src/Buffer.cpp
                     src/Chan.cpp
                     src/Client.cpp
                     src/ClientCommand.cpp
                     src/Config.cpp
                     src/Csocket.cpp
                     src/FileUtils.cpp
                     src/HTTPSock.cpp
                     src/IRCNetwork.cpp
                     src/IRCSock.cpp
                     src/Listener.cpp
                     src/MD5.cpp
                     src/Modules.cpp
                     src/Nick.cpp
                     src/Query.cpp
                     src/SHA256.cpp
                     src/Server.cpp
                     src/Socket.cpp
                     src/Template.cpp
                     src/Threads.cpp
                     src/User.cpp
                     src/Utils.cpp
                     src/WebModules.cpp
                     src/ZNCDebug.cpp
                     src/ZNCString.cpp
                     src/znc.cpp
                     ${CMAKE_CURRENT_BINARY_DIR}/version.cpp
)

if(NOT DEFINED CMAKE_MACOSX_RPATH)
    set(CMAKE_MACOSX_RPATH 1)
endif()

add_library(znc SHARED ${lib_srcs})
target_link_libraries(znc ${libs_list})

add_executable(znc_app src/main.cpp)
list(APPEND libs_list znc)
target_link_libraries(znc_app ${libs_list})
set_target_properties(znc_app PROPERTIES OUTPUT_NAME znc)

add_subdirectory(modules)
