cmake_minimum_required(VERSION 2.8)

ENABLE_LANGUAGE(CXX)

option(BUILD_SHARED "build shared libraries" ON)
option(BUILD_STATIC "build static libraries" OFF)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

if(MSVC)
	find_package(iconv)
	list(APPEND LIBRARIES_TO_LINK ${ICONV_LIBRARIES})
	list(APPEND CMAKE_REQUIRED_DEFINITIONS -D_CRT_NONSTDC_NO_WARNINGS)
	include_directories(${ICONV_INCLUDE_DIR})
endif()

include(CheckIncludeFile)
include(CheckTypeSize)
include(TestBigEndian)

check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(stdint.h    HAVE_STDINT_H)
check_include_file(stddef.h    HAVE_STDDEF_H)
check_type_size("void*" SIZEOF_VOID_P BUILTIN_TYPES_ONLY)
test_big_endian(endianess)

if(HAVE_SYS_TYPES_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_SYS_TYPES_H)
endif()
if(HAVE_STDINT_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_STDINT_H)
endif()
if(HAVE_STDDEF_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_STDDEF_H)
endif()
if(SIZEOF_VOID_P)
	list(APPEND CMAKE_REQUIRED_DEFINITIONS -DSIZEOF_VOID_P=${SIZEOF_VOID_P})
endif()
if(endianess)
	list(APPEND CMAKE_REQUIRED_DEFINITIONS -DWORDS_BIGENDIAN=1)
endif()

list(APPEND CMAKE_REQUIRED_DEFINITIONS -DALIGNOF_DOUBLE=4)
list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_CONFIG_H)

add_definitions(${CMAKE_REQUIRED_DEFINITIONS})

find_package(expat)
find_package(freetype)
find_package(libpng)
find_package(zlib)
find_package(iconv)


set(DIR ${CMAKE_CURRENT_LIST_DIR})
set(FC_ARCH 		${DIR}/fc-arch/fc-arch.c)
set(FC_CACHE		${DIR}/fc-cache/fc-cache.c)
set(FC_CASE			${DIR}/fc-case/fc-case.c)
set(FC_CAT			${DIR}/fc-cat/fc-cat.c)
set(FC_GLYPHNAME	${DIR}/fc-glyphname/fc-glyphname.c)
set(FC_LANG			${DIR}/fc-lang/fc-lang.c)
set(FC_LIST			${DIR}/fc-list/fc-list.c)
set(FC_MATCH		${DIR}/fc-match/fc-match.c)               
set(FC_QUERY		${DIR}/fc-query/fc-query.c)
set(FC_SCAN			${DIR}/fc-scan/fc-scan.c)
file(GLOB FONTCONFIG ${DIR}/src/*.c)
file(GLOB FCFG_HDRS ${DIR}/src/*.h)
set(FONTCONFIG ${FONTCONFIG} ${FCFG_HDRS})
set(DIRENT			${DIR}/win32/dirent.c)

include_directories(	${EXPAT_INCLUDE_DIRS} 
						${FREETYPE_INCLUDE_DIRS} 
						${ZLIB_INCLUDE_DIRS} 
						${ICONV_INCLUDE_DIRS} 
						${DIR}/src 
						${DIR}/win32
						${CMAKE_CURRENT_LIST_DIR}
					)

add_executable(fc-arch 			${FC_ARCH})
add_executable(fc-cache 		${FC_CACHE})
add_executable(fc-case 			${FC_CASE})
add_executable(fc-cat 			${FC_CAT})
add_executable(fc-glyphname 	${FC_GLYPHNAME})
add_executable(fc-lang 			${FC_LANG})
add_executable(fc-match 		${FC_MATCH})
add_executable(fc-query 		${FC_QUERY})
add_executable(fc-scan 			${FC_SCAN})

if(MSVC)
	add_library(dirent STATIC ${DIRENT})
	list(APPEND LIBRARIES_TO_LINK dirent)
endif()

list(APPEND LIBRARIES_TO_LINK ${EXPAT_LIBRARIES} ${FREETYPE_LIBRARIES} ${PNG_LIBRARY} ${ZLIB_LIBRARY} ${ICONV_LIBRARY})

if(BUILD_SHARED)
	add_library(fontconfig SHARED ${FONTCONFIG})
	target_link_libraries(fontconfig ${LIBRARIES_TO_LINK})
	set_target_properties(fontconfig PROPERTIES COMPILE_DEFINITIONS "FcPublic=__declspec(dllexport)")
endif()
if(BUILD_STATIC)
	add_library(fontconfig-static STATIC ${FONTCONFIG})
	target_link_libraries(fontconfig-static ${LIBRARIES_TO_LINK})
	set_target_properties(fontconfig-static PROPERTIES COMPILE_DEFINITIONS "-DDLL_EXPORT")
endif()

target_link_libraries(fc-arch fontconfig)
target_link_libraries(fc-cache fontconfig)
target_link_libraries(fc-case fontconfig)
target_link_libraries(fc-cat fontconfig)
target_link_libraries(fc-glyphname fontconfig)
target_link_libraries(fc-lang fontconfig)
target_link_libraries(fc-match fontconfig)
target_link_libraries(fc-query fontconfig)
target_link_libraries(fc-scan fontconfig)
