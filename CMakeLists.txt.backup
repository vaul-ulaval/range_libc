cmake_minimum_required(VERSION 3.16)

# Options
option(WITH_CUDA "Compile CUDA ray cast variants" ON)

# Set languages based on CUDA option
if(WITH_CUDA)
    project(range_libc LANGUAGES CXX CUDA)
    set(CHUNK_SIZE 262144)
    set(CHUNK_THREADS 256)

    add_definitions(-DUSE_CUDA=1)
    add_definitions(-DCHUNK_SIZE=${CHUNK_SIZE})
    add_definitions(-DNUM_THREADS=${CHUNK_THREADS})
else()
    project(range_libc LANGUAGES CXX)
endif()

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Include headers
include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/includes
  ${CMAKE_CURRENT_SOURCE_DIR}/vendor
)

# Add vendor/gflags
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/vendor/gflags)

# Source files for the library
set(LIB_SRCS
  vendor/lodepng/lodepng.cpp
)

# Source files for the executable
set(EXEC_SRCS
  main.cpp
)

# CUDA sources
if(WITH_CUDA)
  enable_language(CUDA)
  list(APPEND LIB_SRCS includes/kernels.cu)
endif()

# Create shared library
add_library(range_libc SHARED ${LIB_SRCS})

# CUDA target properties for the library
if(WITH_CUDA)
  set_target_properties(range_libc PROPERTIES
    CUDA_SEPARABLE_COMPILATION ON
    CUDA_ARCHITECTURES 87
    POSITION_INDEPENDENT_CODE ON
  )
endif()

# Compile definitions for the library
target_compile_definitions(range_libc PRIVATE
  USE_CUDA=$<BOOL:${WITH_CUDA}>
  CHUNK_SIZE=${CHUNK_SIZE}
  NUM_THREADS=${CHUNK_THREADS}
)

# Link libraries for the shared library
target_link_libraries(range_libc
  gflags
  pthread
)

# Create executable that uses the library
add_executable(range_libc_bin ${EXEC_SRCS})
set_target_properties(range_libc_bin PROPERTIES OUTPUT_NAME "range_libc")

# Link the executable with the library
target_link_libraries(range_libc_bin
  range_libc
)

# Install headers
install(DIRECTORY includes/
  DESTINATION include/range_libc
)

install(DIRECTORY vendor/
  DESTINATION include/range_libc
)

# Install library and binary
install(TARGETS range_libc
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

install(TARGETS range_libc_bin
  RUNTIME DESTINATION bin
)