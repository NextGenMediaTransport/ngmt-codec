# ngmt-codec — VMX codec library (NextGenMediaTransport)
cmake_minimum_required(VERSION 3.15)

project(ngmt-codec VERSION 0.1.0 LANGUAGES CXX)

include(GNUInstallDirs)

find_package(Threads REQUIRED)

# Host architecture: x86_64 vs AArch64 (one native arch per build; no universal fat binaries here).
string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR}" _ngmt_codec_proc)
set(_ngmt_codec_is_x86_64 FALSE)
set(_ngmt_codec_is_aarch64 FALSE)
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  if(_ngmt_codec_proc MATCHES "^(arm64|aarch64)$")
    set(_ngmt_codec_is_aarch64 TRUE)
  elseif(_ngmt_codec_proc MATCHES "^(x86_64|amd64)$")
    set(_ngmt_codec_is_x86_64 TRUE)
  endif()
endif()

if(WIN32 AND CMAKE_GENERATOR_PLATFORM STREQUAL "ARM64")
  set(_ngmt_codec_is_x86_64 FALSE)
  set(_ngmt_codec_is_aarch64 TRUE)
elseif(WIN32 AND (CMAKE_GENERATOR_PLATFORM STREQUAL "Win32" OR CMAKE_GENERATOR_PLATFORM STREQUAL "x86"))
  message(FATAL_ERROR "ngmt-codec: 32-bit x86 is not supported by this build.")
endif()

if(NOT _ngmt_codec_is_x86_64 AND NOT _ngmt_codec_is_aarch64)
  message(
    FATAL_ERROR
    "ngmt-codec: unsupported architecture (CMAKE_SYSTEM_PROCESSOR=${CMAKE_SYSTEM_PROCESSOR}). "
    "Expected 64-bit x86_64 or AArch64.")
endif()

set(_ngmt_codec_sources src/vmxcodec.cpp)

if(_ngmt_codec_is_x86_64)
  list(APPEND _ngmt_codec_sources src/vmxcodec_x86.cpp src/vmxcodec_avx2.cpp)
endif()

if(_ngmt_codec_is_aarch64)
  list(APPEND _ngmt_codec_sources src/vmxcodec_arm.cpp)
endif()

add_library(ngmt_codec SHARED ${_ngmt_codec_sources})

set_target_properties(ngmt_codec PROPERTIES OUTPUT_NAME ngmt-codec)

target_compile_features(ngmt_codec PUBLIC cxx_std_17)

target_include_directories(
  ngmt_codec PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>"
                    "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")

target_link_libraries(ngmt_codec PRIVATE Threads::Threads)

if(_ngmt_codec_is_x86_64)
  if(MSVC)
    set_source_files_properties(src/vmxcodec_avx2.cpp PROPERTIES COMPILE_OPTIONS "/arch:AVX2")
  else()
    # GCC/Clang default to generic x86-64 (SSE2). SSE4.1 intrinsics in the x86 sources require
    # an explicit ISA level; README documents SSE4.2 as the minimum host CPU for x86_64.
    set_source_files_properties(
      src/vmxcodec.cpp
      src/vmxcodec_x86.cpp
      PROPERTIES COMPILE_OPTIONS "-msse4.2")
    set_source_files_properties(
      src/vmxcodec_avx2.cpp PROPERTIES COMPILE_OPTIONS "-mavx2;-mbmi;-mlzcnt")
  endif()
endif()

if(WIN32)
  target_sources(ngmt_codec PRIVATE src/dllmain.cpp libvmx.rc)
  target_link_options(ngmt_codec PRIVATE "/DEF:${CMAKE_CURRENT_SOURCE_DIR}/exports.def")
endif()

install(
  TARGETS ngmt_codec
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(FILES src/vmxcodec.h src/thread_tasks.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
