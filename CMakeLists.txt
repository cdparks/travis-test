project(TRAVIS-TEST)

cmake_minimum_required(VERSION 2.8.0)

# ALL LIBRARIES -- will concatenate to this
set(MY_LIBS "")

# setup Fortran
enable_language(Fortran)
if (NOT DEFINED CMAKE_Fortran_COMPILER)
  message(FATAL_ERROR "Fortran compiler not found!")
endif()

if(${CMAKE_Fortran_COMPILER_ID} STREQUAL "Intel")
  set(MY_LIBS ${MY_LIBS} ifcore)
else()
  set(MY_LIBS ${MY_LIBS} gfortran)
endif()


if (DEFINED USER_LAPACK_LIB)
  message("Using user supplied LAPACK=${USER_LAPACK_LIB}")
  set(MY_LIBS ${MY_LIBS} ${USER_LAPACK_LIB})
# auto find LAPACK
else()
  find_package(LAPACK)
  set(MY_LIBS ${MY_LIBS} ${LAPACK_LIBRARIES})
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${LAPACK_LINKER_FLAGS}")
  if(${LAPACK_FOUND})
    message("FOUND LAPACK LIBS: " ${LAPACK_LIBRARIES})
  else()
    message(FATAL_ERROR "Could not find LAPACK library. Run `./configure --help`  for assistance.")
  endif()
endif()

if (DEFINED USER_BLAS_LIB)
  message("Using user supplied BLAS=${USER_BLAS_LIB}")
  set(MY_LIBS ${MY_LIBS} ${USER_BLAS_LIB})
# auto find BLAS
else()
  find_package(BLAS)
  include_directories(${BLAS_INCLUDE_DIR})
  set(MY_LIBS ${MY_LIBS} ${BLAS_LIBRARIES})
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${BLAS_LINKER_FLAGS}")
  if(${BLAS_FOUND})
    message("FOUND BLAS LIBS: " ${BLAS_LIBRARIES})
  else()
    message(FATAL_ERROR "Could not find BLAS library. Run `./configure --help`  for assistance.")
  endif()
endif()


add_definitions(-D_GNU_SOURCE)

set(C_DEFINES "${C_DEFINES}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${C_DEFINES}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -funroll-loops")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ftree-vectorize")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstrict-aliasing")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")

file(GLOB MY_SOURCES main.c)
add_executable(test_bin ${MY_SOURCES})
set_target_properties(test_bin PROPERTIES OUTPUT_NAME travis-test)
target_link_libraries(test_bin ${MY_LIBS})


# turn on testing
enable_testing()
add_test(dgemm travis-test)
