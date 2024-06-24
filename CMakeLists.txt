cmake_minimum_required(VERSION 3.10)

# Set the target architecture.
# All modern x86/x64 processors support AVX2.
# Older x86/x64 processors may support SSE2 but not AVX2.
# Very old x86/x64 processors, or non x86/x64 
# processors, do not support any of the two.
set(ENABLE_SSE2 True)
set(ENABLE_AVX2 True)

#####################

# Use CMake -DLGPL=ON if you opt for the LGPL license
option(LGPL "LGPL" OFF)

# set the project name
project(cdt)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# add the executable
# add_executable(${PROJECT_NAME} 
#     src/giftWrap.cpp
#     src/PLC.cpp
#     src/delaunay.cpp
#     src/main.cpp
# )

add_library(cdt_tetra src/delaunay.cpp src/PLC.cpp src/giftWrap.cpp)

if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
# optimize for speed
	target_compile_options(cdt_tetra PRIVATE /Ot)
# grant IEEE 754 compliance
	target_compile_options(cdt_tetra PRIVATE "/fp:strict")
# use intrinsic functions
	target_compile_options(cdt_tetra PRIVATE "/Oi")
# turn off annoying warnings
	target_compile_options(cdt_tetra PRIVATE "/D _CRT_SECURE_NO_WARNINGS")
	# set target architecture
	if(ENABLE_AVX2)
		target_compile_options(cdt_tetra PRIVATE "/arch:AVX2")
	elseif(ENABLE_SSE2)
		target_compile_options(cdt_tetra PRIVATE "/arch:SSE2")
	endif()
# reserve enough stack size
	target_link_options(cdt_tetra PRIVATE "/STACK:8421376")
	if(NOT LGPL)
		target_compile_options(cdt_tetra PRIVATE "/D USE_MAROTS_METHOD")
	endif()
else()
# set standard optimization level
	target_compile_options(cdt_tetra PRIVATE -O2)
# grant IEEE 754 compliance
	target_compile_options(cdt_tetra PRIVATE -frounding-math)
	# set target architecture
	if(ENABLE_AVX2)
		target_compile_options(cdt_tetra PRIVATE "-mavx2")
                target_compile_options(cdt_tetra PRIVATE "-mfma")
	elseif(ENABLE_SSE2)
		target_compile_options(cdt_tetra PRIVATE "-msse2")
	endif()
# reserve enough stack size	
	target_compile_options(cdt_tetra PRIVATE -Wl,-z,stacksize=8421376)
	if(NOT LGPL)
		target_compile_options(cdt_tetra PRIVATE "-DUSE_MAROTS_METHOD")
	endif()
endif()

# Public include directory
target_include_directories(cdt_tetra PUBLIC include)

