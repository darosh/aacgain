include(ExternalProject)

IF(UNIX)
include(ProcessorCount)
ProcessorCount(N)

if(CMAKE_OSX_ARCHITECTURES STREQUAL "x86_64")
    set(AUTOTOOLS_HOST "--host=x86_64-apple-darwin")
    set(AUTOTOOLS_CFLAGS "CFLAGS=\"-arch x86_64 -std=gnu99\"")
    set(AUTOTOOLS_CXXFLAGS "CXXFLAGS=\"-arch x86_64\"")
elseif(CMAKE_OSX_ARCHITECTURES STREQUAL "arm64")
    set(AUTOTOOLS_HOST "--host=arm-apple-darwin")
    set(AUTOTOOLS_CFLAGS "CFLAGS=\"-arch arm64 -std=gnu99\"")
    set(AUTOTOOLS_CXXFLAGS "CXXFLAGS=\"-arch arm64\"")
else()
    set(AUTOTOOLS_HOST "")
    set(AUTOTOOLS_CFLAGS "CFLAGS=\"-std=gnu99\"")
    set(AUTOTOOLS_CXXFLAGS "")
endif()

ExternalProject_Add(faad2_proj
                    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/faad2"
                    CONFIGURE_COMMAND sh -c "cd <SOURCE_DIR> && rm -f config.status && autoreconf -i && cd <BINARY_DIR> && <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --enable-static ${AUTOTOOLS_HOST} ${AUTOTOOLS_CFLAGS}"
                    BUILD_COMMAND make -j ${N}
                    INSTALL_COMMAND make -j ${N} install
                   )

ExternalProject_Add(mp4v2_proj
                    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/mp4v2"
                    CONFIGURE_COMMAND sh -c "cd <SOURCE_DIR> && rm -f config.status && autoreconf -i && cd <BINARY_DIR> && <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --enable-static ${AUTOTOOLS_HOST} ${AUTOTOOLS_CFLAGS} ${AUTOTOOLS_CXXFLAGS}"
                    BUILD_COMMAND make -j ${N}
                    INSTALL_COMMAND make -j ${N} install
                   )

set(PREFIX_CLEAN_DIRS bin include lib share)
set_directory_properties(PROPERTIES ADDITIONAL_CLEAN_FILES "${PREFIX_CLEAN_DIRS}")
ENDIF(UNIX)

IF(WIN32)
ExternalProject_Add(faad2_proj
	SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/faad2"
		    CONFIGURE_COMMAND ""
		    BUILD_COMMAND msbuild project/msvc/faad2.sln -t:libfaad -p:Configuration=Release -p:Platform=x64 -maxcpucount
		    INSTALL_COMMAND ""
		    BUILD_IN_SOURCE 1
		   )
ExternalProject_Add(mp4v2_proj
		    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/mp4v2"
		    CONFIGURE_COMMAND ""
		    BUILD_COMMAND msbuild mp4v2-Win/mp4v2.sln -t:libmp4v2 -p:Configuration=Release -p:Platform=x64 -maxcpucount
		    INSTALL_COMMAND ""
		    BUILD_IN_SOURCE 1
		   )
ENDIF(WIN32)

add_library(faad2 STATIC IMPORTED GLOBAL)
add_library(mp4v2 STATIC IMPORTED GLOBAL)

IF(UNIX)
set(FAAD_LIB_PATH "${CMAKE_CURRENT_BINARY_DIR}/faad2_proj-prefix/lib/libfaad.a")
set(MP4V2_LIB_PATH "${CMAKE_CURRENT_BINARY_DIR}/mp4v2_proj-prefix/lib/libmp4v2.a")
ENDIF(UNIX)

if(WIN32)
set(FAAD_LIB_PATH "${PROJECT_SOURCE_DIR}/3rdparty/faad2/project/msvc/x64/Release/libfaad.lib")
set(MP4V2_LIB_PATH "${PROJECT_SOURCE_DIR}/3rdparty/mp4v2/mp4v2-Win/bin/x64/Release/libmp4v2.lib")
ENDIF(WIN32)

set_target_properties(faad2 PROPERTIES IMPORTED_LOCATION "${FAAD_LIB_PATH}")
set_target_properties(mp4v2 PROPERTIES IMPORTED_LOCATION "${MP4V2_LIB_PATH}")
