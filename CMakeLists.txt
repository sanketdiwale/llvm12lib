cmake_minimum_required(VERSION 3.18)
# project name
project(llvm VERSION 13)
# platform specific build configuration
if(UNIX AND NOT APPLE)
    if(NOT (MSVC OR MSYS OR MINGW))
    message(STATUS "Using gold linker")
    set(LLVM_USE_LINKER gold)
    endif()
endif()
####### External dependencies
include(FetchContent)
set(FETCHCONTENT_QUIET FALSE)
# get and configure LLVM
set(EP_LLVM "LLVM")
message(STATUS "Build ${EP_LLVM} from source")
set(CMAKE_BUILD_TYPE Release)
# set(FETCHCONTENT_SOURCE_DIR_${EP_LLVM} "${PROJECT_SOURCE_DIR}/external/llvm-project-src")
FetchContent_Declare(
    ${EP_LLVM}
    GIT_REPOSITORY https://github.com/llvm/llvm-project.git
    GIT_TAG llvmorg-13.0.0
    GIT_SHALLOW ON
    BUILD_ALWAYS OFF
    SOURCE_SUBDIR llvm
    GIT_PROGRESS TRUE
    USES_TERMINAL_DOWNLOAD TRUE
)
set(LLVM_ENABLE_RTTI ON CACHE BOOL "" FORCE)
set(LIBCLANG_BUILD_STATIC ON CACHE BOOL "" FORCE)
set(LLVM_ENABLE_LIBCXX ON CACHE BOOL "" FORCE)
set(LLVM_STATIC_LINK_CXX_STDLIB ON CACHE BOOL "" FORCE)
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
set(LLVM_BUILD_LLVM_DYLIB OFF CACHE BOOL "" FORCE)
set(LLVM_LINK_LLVM_DYLIB OFF CACHE BOOL "" FORCE)
set(LLVM_ENABLE_PROJECTS "clang;clang-tools-extra;compiler-rt;libcxx;libcxxabi;libunwind;lld;libclc" CACHE STRING "" FORCE) # libc;clang-tools-extra;
# clang;clang-tools-extra;compiler-rt;libcxx;libcxxabi;libunwind;lld;openmp;libclc" CACHE STRING "" FORCE) #;debuginfo-tests;parallel-libs;polly;pstl;libc;libclc;

set(CMAKE_INSTALL_PREFIX "${PROJECT_BINARY_DIR}/dist" CACHE STRING "" FORCE)
if(APPLE)
# set(CMAKE_OSX_SYSROOT "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk" CACHE STRING "" FORCE)
set(LLDB_USE_SYSTEM_DEBUGSERVER ON CACHE BOOL "" FORCE)
endif()
# Make LLVM available
FetchContent_MakeAvailable(${EP_LLVM})
FetchContent_GetProperties(${EP_LLVM})
set(llvm_CMAKE_PATH "${llvm_BINARY_DIR}/lib/cmake/llvm")

if(MSVC)
if(WIN32)
  # prioritize Git Patch on Windows as other Patches may be very old and incompatible.
  find_package(Git)
  if(Git_FOUND)
    get_filename_component(GIT_DIR ${GIT_EXECUTABLE} DIRECTORY)
    get_filename_component(GIT_DIR ${GIT_DIR} DIRECTORY)
  endif()
endif()

find_program(PATCH
NAMES patch
HINTS ${GIT_DIR}
PATH_SUFFIXES usr/bin
)

if(NOT PATCH)
  message(FATAL_ERROR "Did not find GNU Patch")
endif()

set(in_file "${llvm_SOURCE_DIR}/compiler-rt/cmake/Modules/AddCompilerRT.cmake" CACHE STRING "" FORCE)
set(patch_file "${PROJECT_SOURCE_DIR}/update-addcompilerRT.cmake-for-windows.patch" CACHE STRING "" FORCE)
set(out_file "${llvm_SOURCE_DIR}/compiler-rt/cmake/Modules/AddCompilerRT1.cmake" CACHE STRING "" FORCE)
file(RENAME  ${in_file} ${out_file})
execute_process(COMMAND ${PATCH} ${out_file} --input=${patch_file} --output=${in_file} --ignore-whitespace
TIMEOUT 15
COMMAND_ECHO STDOUT
RESULT_VARIABLE ret
)
file(REMOVE ${out_file})
if(NOT ret EQUAL 0)
  message(FATAL_ERROR "Failed to apply patch ${patch_file} to ${in_file} with ${PATCH}")
endif()


endif()
