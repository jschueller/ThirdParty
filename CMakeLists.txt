#
# This file is part of the open source part of the
# Platform for Algorithm Development and Rendering (PADrend).
# Web page: http://www.padrend.de/
# Copyright (C) 2012-2013 Benjamin Eikel <benjamin@eikel.org>
# 
# PADrend consists of an open source part and a proprietary part.
# The open source part of PADrend is subject to the terms of the Mozilla
# Public License, v. 2.0. You should have received a copy of the MPL along
# with this library; see the file LICENSE. If not, you can obtain one at
# http://mozilla.org/MPL/2.0/.
#
cmake_minimum_required(VERSION 2.8.8)

execute_process(
	COMMAND ${CMAKE_C_COMPILER} -dumpmachine
	OUTPUT_VARIABLE ARCH_TRIPLET
	OUTPUT_STRIP_TRAILING_WHITESPACE
)
set(INSTALL_DIR "${CMAKE_CURRENT_SOURCE_DIR}/${ARCH_TRIPLET}")

set(TOOLCHAIN_FILE "${CMAKE_CURRENT_BINARY_DIR}/Toolchain.cmake")
configure_file(${CMAKE_CURRENT_LIST_DIR}/Toolchain.cmake.in ${TOOLCHAIN_FILE} @ONLY)

include(ExternalProject)

option(BUILD_ZLIB "Build the zlib library (strongly recommended; required by several other libraries)" ON)
if(BUILD_ZLIB)
ExternalProject_Add(EP_ZLIB
	URL http://download.sourceforge.net/project/libpng/zlib/1.2.10/zlib-1.2.10.tar.gz
	URL_MD5 "d9794246f853d15ce0fcbf79b9a3cf13"
	PATCH_COMMAND patch -p0 < ${CMAKE_CURRENT_LIST_DIR}/zlib-CMakeLists.txt-OUTPUT_NAME.patch
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release
)
endif()

if(WIN32)
option(BUILD_CPPUNIT "Build the CppUnit library (optional; used for unit testing)" ON)
if(BUILD_CPPUNIT)
ExternalProject_Add(EP_LIBCPPUNIT
	URL http://dev-www.libreoffice.org/src/cppunit-1.13.2.tar.gz
	URL_MD5 "d1c6bdd5a76c66d2c38331e2d287bc01"
	PATCH_COMMAND patch -p0 < ${CMAKE_CURRENT_LIST_DIR}/libcppunit-TestAssert.h-sprintf_s.patch
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static --disable-doxygen CC=${CMAKE_C_COMPILER} CXX=${CMAKE_CXX_COMPILER} LIBS=-lstdc++
	INSTALL_DIR ${INSTALL_DIR}
)
endif()
endif()

option(BUILD_CURL "Build the cURL library (recommended; used for Util's NetProvider)" ON)
if(BUILD_CURL)
ExternalProject_Add(EP_LIBCURL
	DEPENDS EP_ZLIB
	URL http://curl.haxx.se/download/curl-7.52.1.tar.gz
	URL_MD5 "4e1ef056e117b4d25f4ec42ac609c0d4"
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static --disable-manual --with-zlib=<INSTALL_DIR> CC=${CMAKE_C_COMPILER} LD=${CMAKE_LINKER}
	INSTALL_DIR ${INSTALL_DIR}
)
endif()

option(BUILD_DETRI "Build the Detri library (optional; only used for 3D Delaunay triangulations)" ON)
if(BUILD_DETRI)
ExternalProject_Add(EP_LIBDETRI
	URL ${CMAKE_CURRENT_LIST_DIR}/Detri_2.6.b.tar.gz
 	URL_MD5 "e8e6de7710e96c162e549e4b38433b1b"
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release
)
endif()

option(BUILD_GLEW "Build the GLEW library (required; needed for OpenGL functions and extensions)" ON)
if(BUILD_GLEW)
ExternalProject_Add(EP_LIBGLEW
	URL http://downloads.sourceforge.net/project/glew/glew/2.0.0/glew-2.0.0.tgz
	URL_MD5 "2a2cd7c98f13854d2fcddae0d2b20411"
	PATCH_COMMAND patch -p0 < ${CMAKE_CURRENT_LIST_DIR}/libglew-Makefile.patch
	CONFIGURE_COMMAND ""
	BUILD_COMMAND GLEW_DEST=<INSTALL_DIR> CC=${CMAKE_C_COMPILER} ${CMAKE_MAKE_PROGRAM}
	INSTALL_DIR ${INSTALL_DIR}
	INSTALL_COMMAND GLEW_DEST=<INSTALL_DIR> ${CMAKE_MAKE_PROGRAM} install
	BUILD_IN_SOURCE 1
)
endif()

#
# If the build fails with linker errors when building OpenAL (e.g.
# "undefined reference to `___sync_sub_and_fetch_4'"), add "-march=i586" to 
# CMAKE_C_FLAGS and try it again.
#
if(NOT APPLE)
option(BUILD_OPENAL "Build the OpenAL library (optional; used for audio output; if possible, use an OpenAL library provided by the operating system instead)" ON)
if(BUILD_OPENAL)
ExternalProject_Add(EP_LIBOPENAL
	URL http://kcat.strangesoft.net/openal-releases/openal-soft-1.17.2.tar.bz2
	URL_MD5 "1764e0d8fec499589b47ebc724e0913d"
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release -DUTILS:BOOL=OFF -DEXAMPLES:BOOL=OFF -DMMDEVAPI:BOOL=OFF -DCMAKE_C_FLAGS:STRING=${CMAKE_C_FLAGS}
)
endif()
endif()

option(BUILD_LPSOLVE "Build the lp_solve library (optional; only used for MAR)" ON)
if(BUILD_LPSOLVE)
ExternalProject_Add(EP_LIBLPSOLVE
	URL http://downloads.sourceforge.net/lpsolve/lp_solve_5.5.2.5_source.tar.gz
	URL_MD5 "3be57261fc41dd8e210f54017220d5f7"
	PATCH_COMMAND cp ${CMAKE_CURRENT_LIST_DIR}/lpsolve-CMakeLists.txt CMakeLists.txt && cp ${CMAKE_CURRENT_LIST_DIR}/lpsolve-lpsolveConfig.cmake.in lpsolveConfig.cmake.in
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release
)
endif()

option(BUILD_LIBPNG "Build the libpng library (recommended; used to load and save PNG images; required by FreeType and SDL2_image)" ON)
if(BUILD_LIBPNG)
ExternalProject_Add(EP_LIBPNG
	DEPENDS EP_ZLIB
	URL http://download.sourceforge.net/libpng/libpng-1.6.27.tar.gz
	URL_MD5 "58698519e9f6126c1caeefc28dbcbd5f"
	PATCH_COMMAND patch -p0 < ${CMAKE_CURRENT_LIST_DIR}/libpng-CMakeLists.txt.patch
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release -DPNG_STATIC:BOOL=OFF -DPNG_TESTS:BOOL=OFF -DSKIP_INSTALL_EXECUTABLES:BOOL=ON -DSKIP_INSTALL_FILES:BOOL=ON
)
endif()

option(BUILD_LIBFREETYPE "Build the FreeType library (recommended; used for rendering custom fonts)" ON)
if(BUILD_LIBFREETYPE)
ExternalProject_Add(EP_LIBFREETYPE
	DEPENDS EP_ZLIB EP_LIBPNG
	URL http://downloads.sourceforge.net/project/freetype/freetype2/2.7.1/freetype-2.7.1.tar.gz
	URL_MD5 "78701bee8d249578d83bb9a2f3aa3616"
	INSTALL_DIR ${INSTALL_DIR}
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static --without-png CC=${CMAKE_C_COMPILER} CXX=${CMAKE_CXX_COMPILER}
)
endif()

option(BUILD_LIBSDL2 "Build the SDL2, SDL2_image, and SDL2_net libraries (recommended; required by Util to create windows, load images, networking; required by Sound)" ON)
if(BUILD_LIBSDL2)

ExternalProject_Add(EP_LIBJPEG
	URL http://www.ijg.org/files/jpegsrc.v9b.tar.gz
	URL_MD5 "6a9996ce116ec5c52b4870dbcd6d3ddb"
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static CC=${CMAKE_C_COMPILER}
	INSTALL_DIR ${INSTALL_DIR}
)

ExternalProject_Add(EP_LIBTIFF
	DEPENDS EP_ZLIB EP_LIBJPEG
	URL http://download.osgeo.org/libtiff/tiff-4.0.7.tar.gz
	URL_MD5 "77ae928d2c6b7fb46a21c3a29325157b"
	PATCH_COMMAND cp libtiff/tif_unix.c libtiff/tif_win32.c
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static --disable-cxx --with-zlib-include-dir=<INSTALL_DIR>/include --with-zlib-lib-dir=<INSTALL_DIR>/lib --with-jpeg-include-dir=<INSTALL_DIR>/include --with-jpeg-lib-dir=<INSTALL_DIR>/lib CC=${CMAKE_C_COMPILER} CXX=${CMAKE_CXX_COMPILER}
	INSTALL_DIR ${INSTALL_DIR}
)

if(APPLE)
	set(SDL_FLAGS --disable-video-x11 --disable-video-carbon --enable-video-cocoa)
endif()

ExternalProject_Add(EP_LIBSDL2
	URL http://www.libsdl.org/release/SDL2-2.0.5.tar.gz
	URL_MD5 "d4055424d556b4a908aa76fad63abd3c"
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-directx ${SDL_FLAGS} CC=${CMAKE_C_COMPILER} CXX=${CMAKE_CXX_COMPILER}
	INSTALL_DIR ${INSTALL_DIR}
)

set(SDL_CFLAGS "-I<INSTALL_DIR>/include")

ExternalProject_Add(EP_LIBSDL2_IMAGE
	DEPENDS EP_LIBJPEG EP_LIBPNG EP_LIBTIFF EP_LIBSDL2
	URL http://www.libsdl.org/projects/SDL_image/release/SDL2_image-2.0.1.tar.gz
	URL_MD5 "d94b94555ba022fa249a53a021dc3606"
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static --disable-jpg-shared --disable-png-shared --disable-tif-shared --disable-webp --with-sdl-prefix=<INSTALL_DIR> CC=${CMAKE_C_COMPILER} CXX=${CMAKE_CXX_COMPILER} "CFLAGS=${SDL_CFLAGS}"
	INSTALL_DIR ${INSTALL_DIR}
)
# Copy SDL.dll for executing sdl2-config
if(WIN32)
	ExternalProject_Add_Step(EP_LIBSDL2_IMAGE EP_LIBSDL2_IMAGE_COPY_SDL_DLL
		COMMAND ${CMAKE_COMMAND} -E copy ${INSTALL_DIR}/bin/SDL2.dll <BINARY_DIR>
		DEPENDERS configure
	)
endif()

ExternalProject_Add(EP_LIBSDL2_NET
	DEPENDS EP_LIBSDL2
	URL http://www.libsdl.org/projects/SDL_net/release/SDL2_net-2.0.1.tar.gz
	URL_MD5 "5c1d9d1cfa63301b141cb5c0de2ea7c4"
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static --with-sdl-prefix=<INSTALL_DIR> CC=${CMAKE_C_COMPILER} CXX=${CMAKE_CXX_COMPILER} "CFLAGS=${SDL_CFLAGS}"
	INSTALL_DIR ${INSTALL_DIR}
)
# Copy SDL.dll for executing sdl2-config
if(WIN32)
	ExternalProject_Add_Step(EP_LIBSDL2_NET EP_LIBSDL2_NET_COPY_SDL_DLL
		COMMAND ${CMAKE_COMMAND} -E copy ${INSTALL_DIR}/bin/SDL2.dll <BINARY_DIR>
		DEPENDERS configure
	)
endif()
endif()

option(BUILD_SQLITE "Build the SQLite library (recommended; used for Util's DBFSProvider)" ON)
if(BUILD_SQLITE)
ExternalProject_Add(EP_LIBSQLITE
	URL http://www.sqlite.org/2017/sqlite-autoconf-3160200.tar.gz
	URL_MD5 "5a153ef1fd2fa5845ada74deabc68e32"
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static CC=${CMAKE_C_COMPILER} CXX=${CMAKE_CXX_COMPILER}
	INSTALL_DIR ${INSTALL_DIR}
)
endif()

option(BUILD_LIBSERIAL "Build the Serial library (optional; used for optional Util::Serial-functions; https://github.com/wjwwood/serial)" OFF)
if(BUILD_LIBSERIAL)
ExternalProject_Add(EP_LIBSERIAL
	URL https://github.com/wjwwood/serial/archive/1.2.1.tar.gz
	URL_MD5 "b6d9ebdf821654715656577652b61b64"
	PATCH_COMMAND cp ${CMAKE_CURRENT_LIST_DIR}/libserial-CMakeLists.txt CMakeLists.txt && cp ${CMAKE_CURRENT_LIST_DIR}/libserial-serialConfig.cmake.in serialConfig.cmake.in
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release
	INSTALL_DIR ${INSTALL_DIR}
)
endif()

option(BUILD_LIBXML2 "Build the libxml2 library (optional; alternative to the built-in XML parser and required by OpenCOLLADA)" ON)
if(BUILD_LIBXML2)
ExternalProject_Add(EP_LIBXML2
	DEPENDS EP_ZLIB
	URL ftp://xmlsoft.org/libxml2/libxml2-2.9.4.tar.gz
	URL_MD5 "ae249165c173b1ff386ee8ad676815f5"
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --host=${ARCH_TRIPLET} --disable-static --without-python --with-zlib=<INSTALL_DIR> CC=${CMAKE_C_COMPILER}
	INSTALL_DIR ${INSTALL_DIR}
)
endif()

option(BUILD_LIBOPENCOLLADA "Build the OpenCOLLADA library (optional; import of COLLADA files into MinSG; build takes very long)" ON)
if(BUILD_LIBOPENCOLLADA)
ExternalProject_Add(EP_LIBPCRE
	DEPENDS EP_ZLIB
	URL http://download.sourceforge.net/pcre/pcre-8.39.tar.gz
	URL_MD5 "26a76d97e04c89fe9ce22ecc1cd0b315"
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release -DBUILD_SHARED_LIBS:BOOL=ON -DPCRE_BUILD_PCRECPP:BOOL=OFF -DPCRE_SUPPORT_UTF:BOOL=ON -DPCRE_BUILD_PCREGREP:BOOL=OFF -DPCRE_BUILD_TESTS:BOOL=OFF -DCMAKE_CXX_STANDARD_LIBRARIES:STRING=-lstdc++
)

if(UNIX)
	set(OPENCOLLADA_FLAGS "-DUSE_SHARED:BOOL=ON")
else()
	set(OPENCOLLADA_FLAGS "-DUSE_SHARED:BOOL=OFF")
endif()

ExternalProject_Add(EP_LIBOPENCOLLADA
	DEPENDS EP_LIBPCRE EP_LIBXML2 EP_ZLIB
	URL https://github.com/KhronosGroup/OpenCOLLADA/archive/v1.6.36.tar.gz
	URL_MD5 "06134eb53d40648369a160c8bc0adcb9"
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release -DUSE_LIBXML:BOOL=ON -DUSE_EXPAT:BOOL=OFF ${OPENCOLLADA_FLAGS}
)
endif()

option(BUILD_LIBZIP "Build the libzip library (optional; used for Util's ZIPProvider)" ON)
if(BUILD_LIBZIP)
ExternalProject_Add(EP_LIBZIP
	DEPENDS EP_ZLIB
	URL https://nih.at/libzip/libzip-1.1.3.tar.gz
	URL_MD5 "898f02db185068c8ae45bafe4c5cc81c"
	PATCH_COMMAND patch -p0 < ${CMAKE_CURRENT_LIST_DIR}/libzip-CMakeLists.txt-zipconf.h.patch
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release
)
endif()

option(BUILD_LIBARCHIVE "Build the libarchive library (optional; used for Util's ArchiveProvider)" ON)
if(BUILD_LIBARCHIVE)
ExternalProject_Add(EP_LIBARCHIVE
	DEPENDS EP_ZLIB EP_LIBXML2
	URL http://www.libarchive.org/downloads/libarchive-3.2.2.tar.gz
	URL_MD5 "1ec00b7dcaf969dd2a5712f85f23c764"
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release -DENABLE_NETTLE:BOOL=OFF -DENABLE_ICONV:BOOL=OFF -DENABLE_TAR:BOOL=OFF -DENABLE_CPIO:BOOL=OFF -DENABLE_TEST:BOOL=OFF -DENABLE_ACL:BOOL=OFF
)
endif()

option(BUILD_LIBBULLET "Build the Bullet library (optional; used for MinSG's physics extension)" ON)
if(BUILD_LIBBULLET)
ExternalProject_Add(EP_LIBBULLET
	URL https://github.com/bulletphysics/bullet3/archive/2.85.1.tar.gz
	URL_MD5 "194cf1ef1eeb7fa9ac7dc580219d3c8b"
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release -DBUILD_SHARED_LIBS:BOOL=ON -DUSE_GRAPHICAL_BENCHMARK:BOOL=OFF -DBUILD_CPU_DEMOS:BOOL=OFF -DUSE_GLUT:BOOL=OFF -DBUILD_BULLET2_DEMOS:BOOL=OFF -DBUILD_OPENGL3_DEMOS:BOOL=OFF -DBUILD_EXTRAS:BOOL=OFF -DBUILD_UNIT_TESTS:BOOL=OFF -DINSTALL_LIBS:BOOL=ON -DCMAKE_EXE_LINKER_FLAGS:STRING=-lstdc++ -DCMAKE_MODULE_LINKER_FLAGS:STRING=-lstdc++ -DCMAKE_SHARED_LINKER_FLAGS:STRING=-lstdc++ -DCMAKE_CXX_STANDARD_LIBRARIES:STRING=-lstdc++
)
endif()

option(BUILD_OPENCL "Build the OpenCL ICD library (optional; used for OpenCL; may require installed OpenCL-SDK)" OFF)
if(BUILD_OPENCL)
ExternalProject_Add(EP_LIBOPENCL
	URL https://www.khronos.org/registry/cl/specs/opencl-icd-1.2.11.0.tgz
 	URL_MD5 "697bd9f3a66fd61ad58fb4a17fda66c7"
	PATCH_COMMAND patch -p0 < ${CMAKE_CURRENT_LIST_DIR}/opencl-CMakeLists.txt.patch && patch -p0 < ${CMAKE_CURRENT_LIST_DIR}/opencl-icd_dispatch.c.patch
	INSTALL_DIR ${INSTALL_DIR}
	CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE:FILEPATH=${TOOLCHAIN_FILE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE:STRING=Release
)
endif()

