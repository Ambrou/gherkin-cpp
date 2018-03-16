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

add_library(gherkin-cpp ${GHERKIN_SRCS} ${GHERKIN_HEADERS})
