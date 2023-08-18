cmake_minimum_required(VERSION 3.20)

# Set extension name here
set(TARGET_NAME h3)

set(EXTENSION_NAME ${TARGET_NAME}_extension)
project(${TARGET_NAME})

# Avoid building tooling we won't need for release
set(BUILD_BENCHMARKS OFF CACHE BOOL "" FORCE)
set(BUILD_FILTERS OFF CACHE BOOL "" FORCE)
set(BUILD_GENERATORS OFF CACHE BOOL "" FORCE)
set(BUILD_TESTING OFF CACHE BOOL "" FORCE)
set(BUILD_FUZZERS OFF CACHE BOOL "" FORCE)
set(ENABLE_DOCS OFF CACHE BOOL "" FORCE)
set(ENABLE_TESTING OFF CACHE BOOL "" FORCE)

# Build the core library as static
set(BUILD_SHARED_LIBS OFF)

add_subdirectory(h3)
add_library(h3_dep ALIAS h3)

# Build the rest (other than the core library dependency) as shared
set(BUILD_SHARED_LIBS ON)

include_directories(src/include)

set(EXTENSION_SOURCES
  src/h3_extension.cpp
  src/h3_common.cpp
  src/h3_functions/h3_indexing.cpp
  src/h3_functions/h3_inspection.cpp
  src/h3_functions/h3_hierarchy.cpp
  src/h3_functions/h3_traversal.cpp
  src/h3_functions/h3_vertex.cpp
  src/h3_functions/h3_directededge.cpp
)

add_library(${EXTENSION_NAME} STATIC ${EXTENSION_SOURCES})
target_link_libraries(${EXTENSION_NAME} h3_dep)

set(PARAMETERS "-warnings")
build_loadable_extension(${TARGET_NAME} ${PARAMETERS} ${EXTENSION_SOURCES})

# Not sure if the next line is needed
# target_link_libraries(${TARGET_NAME}_loadable_extension h3_dep)

install(
  TARGETS ${EXTENSION_NAME} h3
  EXPORT "${DUCKDB_EXPORT_SET}"
  LIBRARY DESTINATION "${INSTALL_LIB_DIR}"
  ARCHIVE DESTINATION "${INSTALL_LIB_DIR}")
