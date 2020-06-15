project(basisu)

cmake_minimum_required(VERSION 3.0)
option(BUILD_X64 "build 64-bit" TRUE)
option(STATIC "static linking" FALSE)
option(BUILD_AS_LIBRARY "build basisu as a library instead of as an executable" FALSE)
option(NO_ITERATOR_DEBUG_LEVEL "if false, set _SECURE_SCL and _ITERATOR_DEBUG_LEVEL to 1 on MSVC builds. This can cause problems when linking debug builds" TRUE)

message("Initial BUILD_X64=${BUILD_X64}")


if (BUILD_X64)
	message("Building 64-bit")
else()
	message("Building 32-bit")
endif(BUILD_X64)

# set the debug level for msvc
if(NO_ITERATOR_DEBUG_LEVEL)
    # if defined, the _ITERATOR_DEBUG_LEVEL for debug builds is set to 1 instead of 2.
    # libraries with differing ITERATOR_DEBUG_LEVELS can't be linked, therefore this setting
    # might break linkage in MSVC debug builds
    add_definitions(-DBASISU_NO_ITERATOR_DEBUG_LEVEL)
endif()

if (NOT MSVC)
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")
   set(CMAKE_C_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")

   set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
   set(CMAKE_C_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")

   set(CMAKE_CXX_FLAGS -std=c++11)
   set(GCC_COMPILE_FLAGS "-fvisibility=hidden -fPIC -fno-strict-aliasing -D_LARGEFILE64_SOURCE=1 -D_FILE_OFFSET_BITS=64 -Wall -Wextra -Wno-unused-local-typedefs -Wno-unused-value -Wno-unused-parameter -Wno-unused-variable")

   if (NOT BUILD_X64)
	  set(GCC_COMPILE_FLAGS "${GCC_COMPILE_FLAGS} -m32")
   endif()

   if (STATIC)
	  set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} ${GCC_LINK_FLAGS} -static-libgcc -static-libstdc++ -static")
   else()
	  set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} ${GCC_LINK_FLAGS} -Wl,-rpath .")
   endif(STATIC)

   set(CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} ${GCC_COMPILE_FLAGS}")
   set(CMAKE_C_FLAGS_RELEASE  "${CMAKE_C_FLAGS_RELEASE} ${GCC_COMPILE_FLAGS}")
   set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${GCC_COMPILE_FLAGS} -D_DEBUG")

   set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} ${GCC_COMPILE_FLAGS}")
   set(CMAKE_CXX_FLAGS_RELEASE  "${CMAKE_CXX_FLAGS_RELEASE} ${GCC_COMPILE_FLAGS}")
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${GCC_COMPILE_FLAGS} -D_DEBUG")
endif()

set(BASISU_INC_LIST
    basisu_astc_decomp.h
    basisu_backend.h
    basisu_basis_file.h
    basisu_bc7enc.h
    basisu_comp.h
    basisu_enc.h
    basisu_etc.h
    basisu_frontend.h
    basisu_global_selector_palette_helpers.h
    basisu_gpu_texture.h
    basisu_miniz.h
    basisu_pvrtc1_4.h
    basisu_resampler.h
    basisu_resampler_filters.h
    basisu_ssim.h
    basisu_uastc_enc.h
    transcoder/basisu.h
    transcoder/basisu_global_selector_palette.h
    transcoder/basisu_transcoder_uastc.h
    transcoder/basisu_file_headers.h
    transcoder/basisu_transcoder.h
    transcoder/basisu_global_selector_cb.h
    transcoder/basisu_transcoder_internal.h
    )

set(BASISU_SRC_LIST ${COMMON_SRC_LIST} 
	basisu_backend.cpp
	basisu_basis_file.cpp
	basisu_comp.cpp
	basisu_enc.cpp
	basisu_etc.cpp
	basisu_frontend.cpp
	basisu_global_selector_palette_helpers.cpp
	basisu_gpu_texture.cpp
	basisu_pvrtc1_4.cpp
	basisu_resampler.cpp
	basisu_resample_filters.cpp
	basisu_ssim.cpp
	basisu_astc_decomp.cpp
	basisu_uastc_enc.cpp
	basisu_bc7enc.cpp
	lodepng.cpp
	apg_bmp.c
	jpgd.cpp
	transcoder/basisu_transcoder.cpp
	)

set(BASISU_TOOL_SRC_LIST ${BASISU_SRC_LIST}
    basisu_tool.cpp
    )


if (APPLE)
   set(BIN_DIRECTORY "bin_osx")
else()
   set(BIN_DIRECTORY "bin")
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${BIN_DIRECTORY})

if(NOT BUILD_AS_LIBRARY)
    add_executable(basisu ${BASISU_TOOL_SRC_LIST})
else()
    add_library(basisu ${BASISU_SRC_LIST})
    set_target_properties(basisu PROPERTIES PUBLIC_HEADER "${BASISU_INC_LIST}")
endif()

if (NOT MSVC)
   target_link_libraries(basisu m pthread)
endif()

install(TARGETS basisu)

if(NOT MSVC)
    # strip the output
    set_target_properties(basisu PROPERTIES LINK_FLAGS_RELEASE -s)
endif()
