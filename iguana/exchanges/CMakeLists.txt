include_directories(${CMAKE_CURRENT_SOURCE_DIR})
set(MM_SOURCES mm.c ../mini-gmp.c ../groestl.c ../segwit_addr.c ../keccak.c)

set(MM_LIBS
        curl
        pthread
        libcrypto777
        libjpeg
        libsecp256k1
        dl
        crypto
        ssl
    )

if(WIN32)
link_directories(${CMAKE_SOURCE_DIR}/marketmaker_depends/curl/build_msvc_2015_win64/lib/Release ${CMAKE_SOURCE_DIR}/marketmaker_depends/pthread-win32/bin/x64_MSVC2015.Release ${CMAKE_SOURCE_DIR}/marketmaker_depends/nanomsg/build_msvc_2015_win64/Release)
set(MM_LIBS ${MM_LIBS} nanomsg)
endif()
add_library(marketmaker-lib STATIC ${MM_SOURCES})
include_directories(../../crypto777)
if(WIN32)
target_compile_definitions(marketmaker-lib PRIVATE)
else()
target_compile_definitions(marketmaker-lib PRIVATE USE_STATIC_NANOMSG)
endif()
if(UNIX)
    target_link_libraries(marketmaker-lib m)
endif()
if(WIN32)
add_definitions(-DNATIVE_WINDOWS)
add_definitions(-DIGUANA_LOG2PACKETSIZE=20)
add_definitions(-DIGUANA_MAXPACKETSIZE=1572864)
add_definitions(-D_CRT_SECURE_NO_WARNINGS)
include_directories("${CMAKE_SOURCE_DIR}/includes")
endif()
if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
target_link_libraries(marketmaker-lib -static-libgcc -static-libstdc++)
endif()
# Link with apple OS libraries
if (APPLE)
    find_library(CoreFoundation CoreFoundation)
    find_library(Security Security)
    target_link_libraries(marketmaker-lib ${CoreFoundation} ${Security} jemalloc)
endif()

target_link_libraries(marketmaker-lib ${MM_LIBS})

if(NOT DEFINED MM_VERSION)
    SET(MM_VERSION UNKNOWN)
endif()

if(NOT DEFINED ETOMIC)
    target_compile_definitions(marketmaker-lib PRIVATE -DNOTETOMIC)
endif()

target_compile_definitions(marketmaker-lib PRIVATE -DMM_VERSION="${MM_VERSION}")
