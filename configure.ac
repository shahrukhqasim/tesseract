# -*-Shell-script-*-
#
# Copyright (c) Luc Vincent
# ----------------------------------------
# Initialization
# ----------------------------------------
AC_PREREQ([2.59])
AC_INIT([tesseract], [4.00.00dev], [https://github.com/tesseract-ocr/tesseract/issues])
AC_PROG_CXX([g++ clang++])
AC_LANG([C++])
AC_LANG_COMPILER_REQUIRE
CXXFLAGS=${CXXFLAGS:-""}
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([config])
AC_CONFIG_SRCDIR([api/tesseractmain.cpp])
AC_PREFIX_DEFAULT([/usr/local])
# Define date of package, etc. Could be useful in auto-generated
# documentation.
PACKAGE_YEAR=2016
PACKAGE_DATE="11/11"
abs_top_srcdir=`AS_DIRNAME([$0])`
gitrev="`git --git-dir=${abs_top_srcdir}/.git --work-tree=${abs_top_srcdir} describe --always --tags`"
if test -n "${gitrev}" ; then
    AC_REVISION(["${gitrev}"])
    AC_DEFINE_UNQUOTED([GIT_REV], ["${gitrev}"], [Define to be the git revision])
    echo "Using git revision: ${gitrev}"
fi
AC_DEFINE_UNQUOTED([PACKAGE_NAME], ["${PACKAGE_NAME}"], [Name of package])
AC_DEFINE_UNQUOTED([PACKAGE_VERSION], ["${PACKAGE_VERSION}"], [Version number])
AC_DEFINE_UNQUOTED([PACKAGE_YEAR], ["$PACKAGE_YEAR"], [Official year for this release])
AC_DEFINE_UNQUOTED([PACKAGE_DATE], ["$PACKAGE_DATE"], [Official date of release])
AC_SUBST([PACKAGE_NAME])
AC_SUBST([PACKAGE_VERSION])
AC_SUBST([PACKAGE_YEAR])
AC_SUBST([PACKAGE_DATE])
GENERIC_LIBRARY_NAME=tesseract
# Release versioning
GENERIC_MAJOR_VERSION=4
GENERIC_MINOR_VERSION=0
GENERIC_MICRO_VERSION=0
# API version (often = GENERIC_MAJOR_VERSION.GENERIC_MINOR_VERSION)
GENERIC_API_VERSION=$GENERIC_MAJOR_VERSION.$GENERIC_MINOR_VERSION
GENERIC_LIBRARY_VERSION=$GENERIC_MAJOR_VERSION:$GENERIC_MINOR_VERSION
AC_SUBST([GENERIC_API_VERSION])
AC_SUBST([GENERIC_MAJOR_VERSION])
AC_SUBST([GENERIC_LIBRARY_VERSION])
PACKAGE=$GENERIC_LIBRARY_NAME
AC_SUBST([GENERIC_LIBRARY_NAME])
GENERIC_VERSION=$GENERIC_MAJOR_VERSION.$GENERIC_MINOR_VERSION.$GENERIC_MICRO_VERSION
GENERIC_RELEASE=$GENERIC_MAJOR_VERSION.$GENERIC_MINOR_VERSION
AC_SUBST([GENERIC_RELEASE])
AC_SUBST([GENERIC_VERSION])
# ----------------------------------------
# Automake configuration
# ----------------------------------------
# Do not require README file (we use README.md)
AM_INIT_AUTOMAKE([foreign])
AC_CONFIG_HEADERS([config_auto.h:config/config.h.in])
AM_MAINTAINER_MODE
# default conditional
AM_CONDITIONAL([T_WIN], false)
AM_CONDITIONAL([MINGW], false)
AM_CONDITIONAL([OSX], false)
AM_CONDITIONAL([GRAPHICS_DISABLED], false)
AC_SUBST([AM_CPPFLAGS])
OPENCL_INC="/opt/AMDAPP/include"
OPENCL_LIBS="-lOpenCL -ltiff"
#############################
#
# Platform specific setup
#
#############################
AC_CANONICAL_HOST
case "${host_os}" in
    mingw*)
        AC_DEFINE_UNQUOTED([MINGW], 1, [This is a MinGW system])
        AM_CONDITIONAL([T_WIN], true)
        AM_CONDITIONAL([MINGW], true)
        AM_CONDITIONAL([ADD_RT], false)
        AC_SUBST([AM_LDFLAGS], ['-Wl,-no-undefined -Wl,--as-needed'])
        ;;
    cygwin*)
        AM_CONDITIONAL([ADD_RT], false)
        AC_SUBST([AM_LDFLAGS], ['-Wl,-no-undefined -Wl,--as-needed'])
        ;;
    solaris*)
        LIBS="-lsocket -lnsl -lrt -lxnet"
        AM_CONDITIONAL([ADD_RT], true)
        ;;
    *darwin*)
        OPENCL_LIBS=""
        OPENCL_INC=""
        AM_CONDITIONAL([ADD_RT], false)
        ;;
    powerpc-*-darwin*)
        OPENCL_LIBS=""
        ;;
    *)
        # default
        AM_CONDITIONAL([ADD_RT], true)
        ;;
esac
## Checks for supported compiler options.
AM_CONDITIONAL([AVX_OPT], false)
AM_CONDITIONAL([SSE41_OPT], false)
# The current implementation for AVX uses 64 bit code.
AC_CHECK_SIZEOF([void *])
#if test "$ac_cv_sizeof_void_p" = "8"; then
#    AX_CHECK_COMPILE_FLAG([-mavx], [avx=true], [avx=false])
#    if $avx; then
#        AM_CONDITIONAL([AVX_OPT], true)
#    fi
#fi
#AX_CHECK_COMPILE_FLAG([-msse4.1], [sse41=true], [sse41=false])
if $sse41; then
    AM_CONDITIONAL([SSE41_OPT], true)
fi
includedir="${includedir}/tesseract"
AC_ARG_WITH([extra-includes],
            [AS_HELP_STRING([--with-extra-includes=DIR],
                       [Define an additional directory for include files])],
            [if test -d "$withval" ; then
               CFLAGS="$CFLAGS -I$withval"
             else
               AC_MSG_ERROR([Cannot stat directory $withval])
             fi])
AC_ARG_WITH([extra-libraries],
            [AS_HELP_STRING([--with-extra-libraries=DIR],
                       [Define an additional directory for library files])],
            [if test -d "$withval" ; then
              LDFLAGS="$LDFLAGS -L$withval"
             else
               AC_MSG_ERROR([Cannot stat directory $withval])
             fi])
AC_MSG_CHECKING([--enable-graphics argument])
AC_ARG_ENABLE([graphics],
 [AS_HELP_STRING([--enable-graphics],[enable graphics (ScrollView) (default)])
AS_HELP_STRING([--disable-graphics],[disable graphics (ScrollView)])],
    [enable_graphics=$enableval],
    [enable_graphics="yes"])
AC_MSG_RESULT([$enable_graphics])
if test "$enable_graphics" = "no"; then
  AC_DEFINE([GRAPHICS_DISABLED], [], [Disable graphics])
  AM_CONDITIONAL([GRAPHICS_DISABLED], true)
fi
# check whether to build embedded version
AC_MSG_CHECKING([--enable-embedded argument])
AC_ARG_ENABLE([embedded],
    [  --enable-embedded       enable embedded build (default=no)],
    [enable_embedded=$enableval],
    [enable_embedded="no"])
AC_MSG_RESULT([$enable_embedded])
AM_CONDITIONAL([EMBEDDED], [test "$enable_embedded" = "yes"])
if test "$enable_embedded" = "yes"; then
  AM_CPPFLAGS="-DEMBEDDED $AM_CPPFLAGS"
fi
# check whether to build OpenMP support
AC_OPENMP
# check whether to build opencl version
AC_MSG_CHECKING([--enable-opencl argument])
AC_ARG_ENABLE([opencl],
    [  --enable-opencl         enable opencl build (default=no)],
    [enable_opencl=$enableval],
    [enable_opencl="no"])
AC_MSG_RESULT([$enable_opencl])
# check for opencl header
have_opencl=false
AC_CHECK_HEADERS([CL/cl.h], [have_opencl=true], [
    AC_CHECK_HEADERS(OpenCL/cl.h, have_opencl=true, have_opencl=false)
])
have_tiff=false
AC_CHECK_HEADERS([tiffio.h], [have_tiff=true], [have_tiff=false])
# https://lists.apple.com/archives/unix-porting/2009/Jan/msg00026.html
m4_define([MY_CHECK_FRAMEWORK],
  [AC_CACHE_CHECK([if -framework $1 works],[my_cv_framework_$1],
     [save_LIBS="$LIBS"
     LIBS="$LIBS -framework $1"
     AC_LINK_IFELSE([AC_LANG_PROGRAM([],[])],
             [my_cv_framework_$1=yes],
            [my_cv_framework_$1=no])
     LIBS="$save_LIBS"
    ])
   if test "$my_cv_framework_$1"="yes"; then
     AC_DEFINE(AS_TR_CPP([HAVE_FRAMEWORK_$1]), 1,
            [Define if you have the  $1 framework])
     AS_TR_CPP([FRAMEWORK_$1])="-framework $1"
     AC_SUBST(AS_TR_CPP([FRAMEWORK_$1]))
   fi]
)
have_opencl_lib=false
OPENCL_CPPFLAGS=''
OPENCL_LDFLAGS=''
case "${host_os}" in
  *darwin* | *-macos10*)
    echo "checking for OpenCL framework"
    MY_CHECK_FRAMEWORK([OpenCL])
    if test $my_cv_framework_OpenCL = yes; then
       have_opencl_lib=true
    fi
    if test "$enable_opencl" = "yes"; then
      if !($have_opencl_lib); then
        AC_MSG_ERROR([Required OpenCL library not found!])
      fi
      AM_CPPFLAGS="-DUSE_OPENCL $AM_CPPFLAGS"
      OPENCL_CPPFLAGS=""
      OPENCL_LDFLAGS="-framework OpenCL"
    fi
    ;;
  *)
    # default
    AC_CHECK_LIB([OpenCL], [clGetPlatformIDs],
                 [have_opencl_lib=true], [have_opencl_lib=false])
    if test "$enable_opencl" = "yes"; then
        if !($have_opencl); then
            AC_MSG_ERROR([Required OpenCL headers not found!])
        fi
        if !($have_opencl_lib); then
            AC_MSG_ERROR([Required OpenCL library not found!])
        fi
        if !($have_tiff); then
            AC_MSG_ERROR([Required TIFF headers not found! Try to install libtiff-dev?? package.])
        fi
        AM_CPPFLAGS="-DUSE_OPENCL $AM_CPPFLAGS"
        OPENCL_CPPFLAGS="-I${OPENCL_INC}"
        OPENCL_LDFLAGS="${OPENCL_LIBS}"
    fi
    ;;
esac
AM_CONDITIONAL([USE_OPENCL], [test "$enable_opencl" = "yes"])
AC_SUBST([OPENCL_CPPFLAGS])
AC_SUBST([OPENCL_LDFLAGS])
# check whether to build tesseract with -fvisibility=hidden -fvisibility-inlines-hidden
# http://gcc.gnu.org/wiki/Visibility
# http://groups.google.com/group/tesseract-dev/browse_thread/thread/976645ae98189127
AC_MSG_CHECKING([--enable-visibility argument])
AC_ARG_ENABLE([visibility],
    [AS_HELP_STRING([--enable-visibility],[enable experimental build with fvisibility (default=no)])],
    [enable_visibility=$enableval],
    [enable_visibility="no"])
AC_MSG_RESULT([$enable_visibility])
AM_CONDITIONAL([VISIBILITY], [test "$enable_visibility" = "yes"])
# check whether to build multiple libraries
AC_MSG_CHECKING([--enable-multiple-libraries argument])
AC_ARG_ENABLE([multiple-libraries],
    [AS_HELP_STRING([--enable-multiple-libraries],[enable multiple libraries (default=no)])],
    [enable_mlibs=$enableval],
    [enable_mlibs="no"])
AC_MSG_RESULT([$enable_mlibs])
AM_CONDITIONAL([USING_MULTIPLELIBS], [test "$enable_mlibs" = "yes"])
# Check if tessdata-prefix is disabled
AC_MSG_CHECKING([whether to use tessdata-prefix])
AC_ARG_ENABLE([tessdata-prefix],
    [AS_HELP_STRING([--disable-tessdata-prefix],
            [don't set TESSDATA-PREFIX during compile])],
    [tessdata_prefix="no"], [tessdata_prefix="yes"])
AC_MSG_RESULT([$tessdata_prefix])
AM_CONDITIONAL([NO_TESSDATA_PREFIX], [test "$tessdata_prefix" = "no"])
# Check whether enable debuging
AC_MSG_CHECKING([whether to enable debugging])
AC_ARG_ENABLE([debug],
    [AS_HELP_STRING([--enable-debug],
        [turn on debugging (default=no)])],
    [debug=$enableval],
    [debug="no"])
AC_MSG_RESULT([$debug])
if test x"$debug" = x"yes"; then
    AM_CXXFLAGS="$AM_CXXFLAGS -g -Wall -O0 -DDEBUG"
    AM_CPPFLAGS="$AM_CPPFLAGS -g -Wall -O0 -DDEBUG"
else
    AM_CXXFLAGS="$AM_CXXFLAGS -O2 -DNDEBUG"
    AM_CPPFLAGS="$AM_CPPFLAGS -O2 -DNDEBUG"
fi
#localedir='${prefix}/share/locale'
# Always look into a "gnu" directory.
curwd=`pwd`
if test -d $curwd/gnu/include ; then
   CPPFLAGS="$CPPFLAGS -I$curwd/gnu/include"
fi
if test -d $curwd/gnu/lib ; then
   LDFLAGS="$LDFLAGS -L$curwd/gnu/lib"
fi
# ----------------------------------------
# Check Compiler Characteristics and
# configure automake. The two appear to
# be intimately linked...
# ----------------------------------------
AC_PROG_LIBTOOL
# ----------------------------------------
# Additional checking of compiler characteristics
# ----------------------------------------
# Check Endianness. If Big Endian, this will define WORDS_BIGENDIAN
# See also at end of this file, where we define INTEL_BYTE_ORDER
# or MOTOROLA_BYTE_ORDER.
AC_C_BIGENDIAN
# ----------------------------------------
# Check for programs we need
# ----------------------------------------
# Check where all the following programs are and set
# variables accordingly:
LT_INIT
# ----------------------------------------
# C++ related options
# ----------------------------------------
AC_LANG_CPLUSPLUS
AC_MSG_CHECKING([if compiling with clang])
AC_COMPILE_IFELSE(
[AC_LANG_PROGRAM([], [[
#ifndef __clang__
       not clang
#endif
]])],
[CLANG=yes], [CLANG=no])
AC_MSG_RESULT([$CLANG])
dnl ********************
dnl turn on c++11
dnl ********************
OLD_CXXFLAGS=$CXXFLAGS
AC_MSG_CHECKING([whether compiler supports C++11])
CXXFLAGS="$CXXFLAGS -std=c++11"
snprintfworks=no
AC_COMPILE_IFELSE(
[
  AC_LANG_SOURCE([[
    #if (__cplusplus < 201103L)
    #error C++ 11 is unsupported
    #endif
  ]])
], [
     AC_MSG_RESULT(yes)
   ],
   [
     AC_MSG_RESULT(no)
     AC_MSG_ERROR([Your compiler does not have the necessary c++11 support! Cannot proceed.])
   ])
AC_CHECK_FUNCS([snprintf],, [snprintfworks=yes])
CXXFLAGS="$OLD_CXXFLAGS"
# set c++11 support based on platform/compiler
case "${host_os}" in
  cygwin*)
    CXXFLAGS="$CXXFLAGS -std=gnu++11"
    ;;
  *-darwin* | *-macos10*)
     if test "x$CLANG" = "xyes"; then
       CXXFLAGS="$CXXFLAGS -std=c++11
