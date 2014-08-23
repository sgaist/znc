project(modules)

include_directories("${PROJECT_BINARY_DIR}/../include")
include_directories("${PROJECT_SOURCE_DIR}/../include")

option(enable-tcl "enable modtcl" OFF)
if(enable-tcl)
    add_subdirectory(modtcl)
    list(APPEND module_srcs modtcl.cpp)
endif(enable-tcl)

option(enable-perl "enable perl" OFF)
if(enable-perl)
    add_subdirectory(modperl)
    list(APPEND module_srcs modperl.cpp)
endif(enable-perl)

option(enable-python "enable python" OFF)
if(enable-python)
    find_package(PythonLibs REQUIRED)
    include_directories(${PYTHON_INCLUDE_DIRS})
    add_subdirectory(modpython)
    list(APPEND module_srcs modpython.cpp)
endif(enable-python)

list(APPEND module_srcs adminlog.cpp
                        alias.cpp
                        autoattach.cpp
                        autocycle.cpp
                        autoop.cpp
                        autoreply.cpp
                        autovoice.cpp
                        awaynick.cpp
                        block_motd.cpp
                        blockuser.cpp
                        bouncedcc.cpp
                        buffextras.cpp
                        chansaver.cpp
                        clearbufferonmsg.cpp
                        clientnotify.cpp
                        controlpanel.cpp
                        crypt.cpp
                        dcc.cpp
                        disconkick.cpp
                        fail2ban.cpp
                        flooddetach.cpp
                        identfile.cpp
                        imapauth.cpp
                        keepnick.cpp
                        kickrejoin.cpp
                        lastseen.cpp
                        listsockets.cpp
                        log.cpp
                        missingmotd.cpp
                        modules_online.cpp
                        nickserv.cpp
                        notes.cpp
                        notify_connect.cpp
                        partyline.cpp
                        perform.cpp
                        q.cpp
                        raw.cpp
                        route_replies.cpp
                        sample.cpp
                        savebuff.cpp
                        schat.cpp
                        send_raw.cpp
                        shell.cpp
                        simple_away.cpp
                        stickychan.cpp
                        watch.cpp
                        webadmin.cpp
)

if(OPENSSL_FOUND)
    list(APPEND module_srcs awaystore.cpp
                            cert.cpp
                            certauth.cpp
                            ctcpflood.cpp)
endif(OPENSSL_FOUND)

if(HAVE_CYRUS_H)
    list(APPEND module_srcs cyrusauth.cpp
                            sasl.cpp)
endif(HAVE_CYRUS_H)

if(NOT DEFINED CMAKE_MACOSX_RPATH)
    set(CMAKE_MACOSX_RPATH 1)
endif()

foreach(source_file ${module_srcs})
    get_filename_component(source_name ${source_file} NAME_WE)
    add_library("${source_name}" SHARED ${source_file})
    target_link_libraries("${source_name}" ${libs_list})
endforeach(source_file)
