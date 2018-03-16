PROJECT(gherkin CXX)
cmake_minimum_required(VERSION 3.0)

option(BUILD_GHERKIN_TESTS "Build all of gherkin's own tests." OFF)
option(BUILD_SHARED_LIBS "Build shared library." OFF)

# Install library & headers in your directory
#SET(CMAKE_INSTALL_PREFIX  "")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
LIST(APPEND GHERKIN_SRS
        src/compiler.cpp
        src/parser.cpp
        )

add_library(gherkin ${GHERKIN_SRS})
target_include_directories(gherkin PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include;${CMAKE_CURRENT_SOURCE_DIR}/src>")

if (BUILD_GHERKIN_TESTS)
  LIST(APPEND GENERATE_TOKEN
          src/token_formatter_builder.c
          src/gherkin_generate_tokens.c
          )
  LIST(APPEND GHERKIN_CLI
          src/gherkin_cli.c
          )

  add_executable(gherkinexe ${GHERKIN_CLI})
  target_link_libraries(gherkinexe gherkin)
  target_include_directories(gherkinexe PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/src)

  add_executable(gherkin_generate_tokens ${GENERATE_TOKEN})
  target_link_libraries(gherkin_generate_tokens gherkin)

  file(GLOB GOOD_FEATURE_FILES
          ${CMAKE_CURRENT_SOURCE_DIR}/testdata/good/*.feature
          )
  file(GLOB BAD_FEATURE_FILES
          ${CMAKE_CURRENT_SOURCE_DIR}/testdata/bad/*.feature
          )
  add_custom_target(invo DEPENDS invo)
  add_dependencies(invo gherkinexe)
  FOREACH(ENTITY ${GOOD_FEATURE_FILES})
      add_custom_command(
              TARGET gherkinexe
              POST_BUILD
              COMMAND gherkinexe  ${ENTITY}
              DEPENDS invo
      )
  ENDFOREACH()
endif()

set(include_install_dir "include")
set(lib_install_dir "lib/")
set(bin_install_dir "bin/")
set(config_install_dir "lib/cmake/")

set(project_config "${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")

set(namespace "${PROJECT_NAME}::")

configure_file("${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in" "${project_config}" @ONLY)

# Install target (will install the library to specified CMAKE_INSTALL_PREFIX variable)
install(
  TARGETS gherkin
  EXPORT ${targets_export_name}
  ARCHIVE DESTINATION ${lib_install_dir}
  LIBRARY DESTINATION ${lib_install_dir}
  RUNTIME DESTINATION ${lib_install_dir}
  INCLUDES DESTINATION ${include_install_dir})

install(
  DIRECTORY "${PROJECT_SOURCE_DIR}/include/"
  DESTINATION ${include_install_dir}
  FILES_MATCHING PATTERN "*.*h")

install(
    FILES "${project_config}"
    DESTINATION "${config_install_dir}")

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}")

add_custom_target(install_${PROJECT_NAME}
        $(MAKE) install
        DEPENDS gherkin
        COMMENT "Installing ${PROJECT_NAME}")
