find_package(SWIG REQUIRED)
include(${SWIG_USE_FILE})

#FIND_PACKAGE(PythonLibs)
include_directories(${PYTHON_INCLUDE_PATH})

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

set(CMAKE_SWIG_FLAGS "")

list(APPEND modpython_files modpython.i cstring.i)

set_source_files_properties(${modpython_files} PROPERTIES CPLUSPLUS ON)
set_source_files_properties(${modpython_files} PROPERTIES SWIG_FLAGS "-includeall")
swig_add_module(modpython python ${modpython_files})

#set_source_files_properties(modpython.i PROPERTIES CPLUSPLUS ON)
#set_source_files_properties(modpython.i PROPERTIES SWIG_FLAGS "-includeall")
#swig_add_modules(modpython python modpython.i modpython.cxx)

swig_link_libraries(modpython ${PYTHON_LIBRARIES})
