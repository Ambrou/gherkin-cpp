cmake_minimum_required(VERSION 3.0)

PROJECT(gherkin CXX)

# Crée des variables avec les fichiers à compiler
set(GHERKIN_SRCS
        src/compiler.cpp
        src/parser.cpp
)

set(GHERKIN_HEADERS
    include/compiler.hpp
    include/gherkin.hpp
    include/parser.hpp
)

include_directories(${FMEM_INCLUDE})
include_directories(${GHERKIN_C_INCLUDE})
include_directories(./include)

link_directories(${FMEM_LIB})
link_directories(${GHERKIN_C_LIB})

add_library(gherkin-cpp ${GHERKIN_SRCS} ${GHERKIN_HEADERS})

target_link_libraries(gherkin-cpp fmem gherkin)

set(include_install_dir "include")
set(lib_install_dir "lib/")
set(bin_install_dir "bin/")

set(project_config "${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")

set(namespace "${PROJECT_NAME}::")

configure_file("${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in" "${project_config}" @ONLY)

install(
  TARGETS gherkin-cpp
  EXPORT ${targets_export_name}
  ARCHIVE DESTINATION ${lib_install_dir}
  LIBRARY DESTINATION ${lib_install_dir}
  RUNTIME DESTINATION ${bin_install_dir}
  INCLUDES DESTINATION ${include_install_dir})

install(
  DIRECTORY "${PROJECT_SOURCE_DIR}/include/"
  DESTINATION ${include_install_dir}
  FILES_MATCHING PATTERN "*.hpp")

add_custom_target(install_${PROJECT_NAME}
  $(MAKE) install
  DEPENDS gherkin-cpp
  COMMENT "Installing ${PROJECT_NAME}")