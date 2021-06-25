cmake_minimum_required(VERSION 3.7)
project(zbar VERSION 0.21.0)

if("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")
  message(
    FATAL_ERROR
      "In-source builds are not permitted. Make a separate \
folder for building:\nmkdir build; cd build; cmake ..\nBefore that, remove the \
files already created:\nrm -rf CMakeCache.txt CMakeFiles")
endif("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")

set(VERSION ${PROJECT_VERSION})
set(ZBAR_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(ZBAR_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(ZBAR_VERSION_PATCH ${PROJECT_VERSION_PATCH})

option(ZBAR_ENABLE_CODABAR "build support for CODABAR  symbolgy" ON)
option(ZBAR_ENABLE_CODE128 "build support for CODE128  symbolgy" ON)
option(ZBAR_ENABLE_CODE39 "build support for CODE39   symbolgy" ON)
option(ZBAR_ENABLE_CODE93 "build support for CODE93   symbolgy" ON)
option(ZBAR_ENABLE_DATABAR "build support for DATABAR  symbolgy" ON)
option(ZBAR_ENABLE_EAN "build support for EAN      symbolgy" ON)
option(ZBAR_ENABLE_I25 "build support for I25      symbolgy" ON)
option(ZBAR_ENABLE_PDF417 "build support for PDF417   symbolgy" ON)
option(ZBAR_ENABLE_QRCODE "build support for QRCODE   symbolgy" ON)
option(ZBAR_ENABLE_SQCODE "build support for SQCODE   symbolgy" ON)

set(ENABLE_CODABAR ${ZBAR_ENABLE_CODABAR})
set(ENABLE_CODE128 ${ZBAR_ENABLE_CODE128})
set(ENABLE_CODE39 ${ZBAR_ENABLE_CODE39})
set(ENABLE_CODE93 ${ZBAR_ENABLE_CODE93})
set(ENABLE_DATABAR ${ZBAR_ENABLE_DATABAR})
set(ENABLE_EAN ${ZBAR_ENABLE_EAN})
set(ENABLE_I25 ${ZBAR_ENABLE_I25})
set(ENABLE_PDF417 ${ZBAR_ENABLE_PDF417})
set(ENABLE_QRCODE ${ZBAR_ENABLE_QRCODE})
set(ENABLE_SQCODE ${ZBAR_ENABLE_SQCODE})

# TODO: actually check for video support
set(HAVE_VIDEO 0)
set(HAVE_V4L1 0)
set(HAVE_V4L2 0)

# TODO: actually check for x11 support
if(${WIN32})
  set(X_DISPLAY_MISSING 1)
else()
  set(HAVE_X 0)
endif()

# TODO: actually check for xv support
set(HAVE_XV 0)

set(INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/include ${PROJECT_SOURCE_DIR}/zbar)

set(SOURCE_FILES
    ${PROJECT_SOURCE_DIR}/include/zbar.h
    ${PROJECT_SOURCE_DIR}/zbar/debug.h
    ${PROJECT_SOURCE_DIR}/zbar/config.c
    ${PROJECT_SOURCE_DIR}/zbar/error.h
    ${PROJECT_SOURCE_DIR}/zbar/error.c
    ${PROJECT_SOURCE_DIR}/zbar/symbol.h
    ${PROJECT_SOURCE_DIR}/zbar/symbol.c
    ${PROJECT_SOURCE_DIR}/zbar/image.h
    ${PROJECT_SOURCE_DIR}/zbar/image.c
    ${PROJECT_SOURCE_DIR}/zbar/convert.c
    ${PROJECT_SOURCE_DIR}/zbar/processor.c
    ${PROJECT_SOURCE_DIR}/zbar/processor.h
    ${PROJECT_SOURCE_DIR}/zbar/processor/lock.c
    ${PROJECT_SOURCE_DIR}/zbar/refcnt.h
    ${PROJECT_SOURCE_DIR}/zbar/refcnt.c
    ${PROJECT_SOURCE_DIR}/zbar/timer.h
    ${PROJECT_SOURCE_DIR}/zbar/mutex.h
    ${PROJECT_SOURCE_DIR}/zbar/event.h
    ${PROJECT_SOURCE_DIR}/zbar/thread.h
    ${PROJECT_SOURCE_DIR}/zbar/window.h
    ${PROJECT_SOURCE_DIR}/zbar/window.c
    ${PROJECT_SOURCE_DIR}/zbar/video.h
    ${PROJECT_SOURCE_DIR}/zbar/video.c
    ${PROJECT_SOURCE_DIR}/zbar/img_scanner.h
    ${PROJECT_SOURCE_DIR}/zbar/img_scanner.c
    ${PROJECT_SOURCE_DIR}/zbar/scanner.c
    ${PROJECT_SOURCE_DIR}/zbar/decoder.h
    ${PROJECT_SOURCE_DIR}/zbar/decoder.c
    # ${PROJECT_SOURCE_DIR}/zbar/misc.h ${PROJECT_SOURCE_DIR}/zbar/misc.c
)

set(EXTRA_SOURCES ${PROJECT_SOURCE_DIR}/zbar/svg.h
                  ${PROJECT_SOURCE_DIR}/zbar/svg.c)

set(STATIC_LIBS "")

if(ANDROID)
  message(STATUS "CMAKE_ANDROID_ARCH_ABI=${CMAKE_ANDROID_ARCH_ABI}")
  set(INCLUDE_DIRS
      ${INCLUDE_DIRS}
      ${PROJECT_SOURCE_DIR}/android/libjpeg/${CMAKE_ANDROID_ARCH_ABI}/install/include
      ${PROJECT_SOURCE_DIR}/android/libiconv/${CMAKE_ANDROID_ARCH_ABI}/install/include
  )
  set(SOURCE_FILES
      ${SOURCE_FILES}
      ${PROJECT_SOURCE_DIR}/android/libjpeg/${CMAKE_ANDROID_ARCH_ABI}/install/include/jconfig.h
      ${PROJECT_SOURCE_DIR}/android/libjpeg/${CMAKE_ANDROID_ARCH_ABI}/install/include/jerror.h
      ${PROJECT_SOURCE_DIR}/android/libjpeg/${CMAKE_ANDROID_ARCH_ABI}/install/include/jmorecfg.h
      ${PROJECT_SOURCE_DIR}/android/libjpeg/${CMAKE_ANDROID_ARCH_ABI}/install/include/jpeglib.h
      ${PROJECT_SOURCE_DIR}/android/libiconv/${CMAKE_ANDROID_ARCH_ABI}/install/include/iconv.h
      ${PROJECT_SOURCE_DIR}/android/libiconv/${CMAKE_ANDROID_ARCH_ABI}/install/include/libcharset.h
      ${PROJECT_SOURCE_DIR}/android/libiconv/${CMAKE_ANDROID_ARCH_ABI}/install/include/localcharset.h
  )

  add_library(libiconv STATIC IMPORTED)
  set_target_properties(
    libiconv
    PROPERTIES
      IMPORTED_LOCATION
      ${PROJECT_SOURCE_DIR}/android/libiconv/${CMAKE_ANDROID_ARCH_ABI}/install/lib/libiconv.a
  )

  add_library(libcharset STATIC IMPORTED)
  set_target_properties(
    libcharset
    PROPERTIES
      IMPORTED_LOCATION
      ${PROJECT_SOURCE_DIR}/android/libiconv/${CMAKE_ANDROID_ARCH_ABI}/install/lib/libcharset.a
  )

  add_library(libjpeg STATIC IMPORTED)
  set_target_properties(
    libjpeg
    PROPERTIES
      IMPORTED_LOCATION
      ${PROJECT_SOURCE_DIR}/android/libjpeg/${CMAKE_ANDROID_ARCH_ABI}/install/lib/libjpeg.a
  )

  set(HAVE_ICONV 1)
  set(HAVE_LIBJPEG 1)
  set(HAVE_JPEG 1)

  list(APPEND STATIC_LIBS libiconv libcharset libjpeg)

endif()

include_directories(${INCLUDE_DIRS})

message(STATUS "Include directories:")
get_property(
  dirs
  DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  PROPERTY INCLUDE_DIRECTORIES)
foreach(dir ${dirs})
  message(STATUS "dir='${dir}'")
endforeach()

message(STATUS "Checking include files.")
include(CheckIncludeFile)
include(CheckIncludeFiles)

check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(sys/stat.h HAVE_SYS_STAT_H)
check_include_file(sys/time.h HAVE_SYS_TIME_H)
check_include_file(sys/times.h HAVE_SYS_TIMES_H)
check_include_files("stdlib.h;stddef.h" HAVE_STDC_HEADERS)
check_include_file(stdlib.h HAVE_STDLIB_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(strings.h HAVE_STRINGS_H)
check_include_file(inttypes.h HAVE_INTTYPES_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(unistd.h HAVE_UNISTD_H)
check_include_file(poll.h HAVE_POLL_H)
check_include_file(errno.h HAVE_ERRNO_H)
check_include_file(features.h HAVE_FEATURES_H)
check_include_file(memory.h HAVE_MEMORY_H)
check_include_file(float.h HAVE_FLOAT_H)
check_include_file(MagickWand/MagickWand.h HAVE_MAGICKWAND_MAGICKWAND_H)
check_include_file(linux/videodev2.h HAVE_LINUX_VIDEODEV2_H)
check_include_file(linux/videodev.h HAVE_LINUX_VIDEODEV_H)

# check_include_file(iconv.h HAVE_ICONV) check_include_file(jpeglib.h HAVE_JPEG)

configure_file(${PROJECT_SOURCE_DIR}/include/config.h.in
               ${PROJECT_SOURCE_DIR}/include/config.h)

message(STATUS "ENABLE_EAN=${ENABLE_EAN}")
if(${ENABLE_EAN})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/decoder/ean.h
                   ${PROJECT_SOURCE_DIR}/zbar/decoder/ean.c)
endif()

message(STATUS "ENABLE_DATABAR=${ENABLE_DATABAR}")
if(${ENABLE_DATABAR})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/decoder/databar.h
                   ${PROJECT_SOURCE_DIR}/zbar/decoder/databar.c)
endif()

message(STATUS "ENABLE_CODE128=${ENABLE_CODE128}")
if(${ENABLE_CODE128})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/decoder/code128.h
                   ${PROJECT_SOURCE_DIR}/zbar/decoder/code128.c)
endif()

message(STATUS "ENABLE_CODE39=${ENABLE_CODE39}")
if(${ENABLE_CODE39})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/decoder/code39.h
                   ${PROJECT_SOURCE_DIR}/zbar/decoder/code39.c)
endif()

message(STATUS "ENABLE_CODE93=${ENABLE_CODE93}")
if(${ENABLE_CODE93})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/decoder/code93.h
                   ${PROJECT_SOURCE_DIR}/zbar/decoder/code93.c)
endif()

message(STATUS "ENABLE_CODABAR=${ENABLE_CODABAR}")
if(${ENABLE_CODABAR})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/decoder/codabar.h
                   ${PROJECT_SOURCE_DIR}/zbar/decoder/codabar.c)
endif()

message(STATUS "ENABLE_I25=${ENABLE_I25}")
if(${ENABLE_I25})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/decoder/i25.h
                   ${PROJECT_SOURCE_DIR}/zbar/decoder/i25.c)
endif()

message(STATUS "ENABLE_PDF417=${ENABLE_PDF417}")
if(${ENABLE_PDF417})
  set(SOURCE_FILES
      ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/decoder/pdf417.h
      ${PROJECT_SOURCE_DIR}/zbar/decoder/pdf417.c
      ${PROJECT_SOURCE_DIR}/zbar/decoder/pdf417_hash.h)
endif()

message(STATUS "ENABLE_QRCODE=${ENABLE_QRCODE}")
if(${ENABLE_QRCODE})
  set(SOURCE_FILES
      ${SOURCE_FILES}
      ${PROJECT_SOURCE_DIR}/zbar/decoder/qr_finder.h
      ${PROJECT_SOURCE_DIR}/zbar/decoder/qr_finder.c
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/qrdec.h
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/qrdec.c
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/qrdectxt.c
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/rs.h
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/rs.c
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/isaac.h
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/isaac.c
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/bch15_5.h
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/bch15_5.c
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/binarize.h
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/binarize.c
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/util.h
      ${PROJECT_SOURCE_DIR}/zbar/qrcode/util.c)
endif()

message(STATUS "ENABLE_SQCODE=${ENABLE_SQCODE}")
if(${ENABLE_SQCODE})
  set(SOURCE_FILES
      ${SOURCE_FILES}
      ${PROJECT_SOURCE_DIR}/zbar/sqcode.h
      ${PROJECT_SOURCE_DIR}/zbar/sqcode.c
      ${PROJECT_SOURCE_DIR}/zbar/decoder/sq_finder.h
      ${PROJECT_SOURCE_DIR}/zbar/decoder/sq_finder.c)
endif()

if(${WIN32})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/processor/win.c
                   ${PROJECT_SOURCE_DIR}/zbar/libzbar.rc)
else()
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/processor/posix.h
                   ${PROJECT_SOURCE_DIR}/zbar/processor/posix.c)
endif()

message(STATUS "HAVE_V4L1=${HAVE_V4L1}")
if(${HAVE_V4L1})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/video/v4l1.c)
  message(STATUS "HAVE_V4L2=${HAVE_V4L2}")
  if(${HAVE_V4L2})
    set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/video/v4l2.c)
  endif()
endif()

message(STATUS "WIN32=${WIN32}")
if(${WIN32})
  if(${HAVE_VIDEO})
    message(STATUS "TODO: WIN32 HAVE_VIDEO NOT IMPLEMENTED IN CMAKE")
  endif()
endif()

message(STATUS "HAVE_VIDEO=${HAVE_VIDEO}")
if(NOT ${HAVE_VIDEO})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/video/null.c)
endif()

message(STATUS "HAVE_JPEG=${HAVE_JPEG}")
if(${HAVE_JPEG})
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/jpeg.c)
endif()

message(STATUS "HAVE_X=${HAVE_X}")
if(${HAVE_X})
  set(SOURCE_FILES ${SOURCE_FILES}
                   # ${PROJECT_SOURCE_DIR}/zbar/jpeg.c TODO
  )
  message(STATUS "HAVE_XV=${HAVE_XV}")
  if(${HAVE_XV})
    set(SOURCE_FILES ${SOURCE_FILES}
                     # ${PROJECT_SOURCE_DIR}/zbar/jpeg.c TODO
    )
  endif()

elseif(${WIN32})
  message(STATUS "TODO: WIN32 WINDOW SYTEM NOT IMPLEMENTED IN CMAKE")
  set(SOURCE_FILES
      ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/window/win.h
      ${PROJECT_SOURCE_DIR}/zbar/window/win.c
      ${PROJECT_SOURCE_DIR}/zbar/window/dib.c)
  list(APPEND STATIC_LIBS gdi32 winnm)
else()
  message(STATUS "processor/null.c and window/null.c added to source files.")
  set(SOURCE_FILES ${SOURCE_FILES} ${PROJECT_SOURCE_DIR}/zbar/processor/null.c
                   ${PROJECT_SOURCE_DIR}/zbar/window/null.c)
endif()

message(STATUS "Source files:")
foreach(file ${SOURCE_FILES})
  message(STATUS "file='${file}'")
endforeach()

add_library(zbar SHARED ${SOURCE_FILES})
target_include_directories(zbar PUBLIC ${INCLUDE_DIRS})
target_link_libraries(zbar ${STATIC_LIBS})

message(STATUS "CMAKE_SHARED_LINKER_FLAGS=${CMAKE_SHARED_LINKER_FLAGS}")
