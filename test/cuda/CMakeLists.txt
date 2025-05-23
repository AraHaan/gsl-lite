﻿# Copyright 2015-2019 by Martin Moene
# Copyright 2019-2025 by Moritz Beutel
#
# https://github.com/gsl-lite/gsl-lite
#
# This code is licensed under the MIT License (MIT).


cmake_minimum_required( VERSION 3.16 FATAL_ERROR )

project( gsl-lite-test-cuda LANGUAGES CXX CUDA )

message( STATUS "Subproject '${PROJECT_NAME}', programs 'gsl-lite-*'")

include(${CMAKE_CURRENT_SOURCE_DIR}/../MakeTestTarget.cmake)


function(make_cuda_test_targets nameRoot maxStd )
    if( HAS_CUDA14_FLAG AND NOT maxStd LESS 14 )
        make_test_target( ${nameRoot}-cuda14.t STD 14 ${ARGN} DEFAULTS_VERSION v1 CUDA )
    endif()
    if( HAS_CUDA17_FLAG AND NOT maxStd LESS 17 )
        make_test_target( ${nameRoot}-cuda17.t STD 17 ${ARGN} DEFAULTS_VERSION v1 CUDA )
    endif()
endfunction()


# Add test targets for all CUDA versions :

set( CUDA_SOURCES
    "cuda.t.cu"
)
set( SOURCES
    "../gsl-lite.t.cpp"
    "../assert.t.cpp"
)
set_property( SOURCE ${SOURCES} PROPERTY LANGUAGE CUDA ) # compile .cpp files with the CUDA compiler

make_cuda_test_targets( "gsl-lite" 99 SOURCES ${SOURCES} ${CUDA_SOURCES} )
make_cuda_test_targets( "gsl-lite-expt" 99 SOURCES ${SOURCES} ${CUDA_SOURCES} EXTRA_OPTIONS "--expt-relaxed-constexpr" )

if( CMAKE_CUDA_COMPILER_ID MATCHES "Clang" OR ( CMAKE_CUDA_COMPILER_ID MATCHES "NVIDIA" AND CMAKE_CUDA_COMPILER_VERSION VERSION_GREATER 11.3 ) )
    make_cuda_test_targets( "gsl-lite-assume" 14
        SOURCES ${SOURCES} ${CUDA_SOURCES}
        DEFAULTS_VERSION v1
        CONTRACT_CHECKING OFF
        UNENFORCED_CONTRACTS ASSUME
        COMPILE_ONLY
    )
endif()
