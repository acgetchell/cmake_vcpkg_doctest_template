cmake_minimum_required(VERSION 3.15...3.25)

project(CmakeConfigPackageTests LANGUAGES CXX)

# ---- Test as standalone project the exported config package ----

if(PROJECT_IS_TOP_LEVEL OR TEST_INSTALLED_VERSION)
  enable_testing()

  find_package(cmake_vcpkg_doctest_template CONFIG REQUIRED) # for intro, project_options, ...

  if(NOT TARGET cmake_vcpkg_doctest_template::project_options)
    message(FATAL_ERROR "Required config package not found!")
    return() # be strictly paranoid for Template Janitor github action! CK
  endif()
endif()

# ---- Dependencies ----

find_package(doctest CONFIG REQUIRED)

add_library(main OBJECT main.cpp)
target_link_libraries(main PUBLIC doctest::doctest)
target_link_libraries(main PRIVATE cmake_vcpkg_doctest_template::project_options)

# Provide a simple smoke test to make sure that the CLI works and can display a --help message
add_test(NAME cli.has_help COMMAND intro --help)

# Provide a test to verify that the version being reported from the application
# matches the version given to CMake. This will be important once you package
# your program. Real world shows that this is the kind of simple mistake that is easy
# to make, but also easy to test for.
add_test(NAME cli.version_matches COMMAND intro --version)
set_tests_properties(cli.version_matches PROPERTIES PASS_REGULAR_EXPRESSION "${PROJECT_VERSION}")

add_executable(tests tests.cpp)
target_link_libraries(tests PRIVATE cmake_vcpkg_doctest_template::project_warnings
                                    cmake_vcpkg_doctest_template::project_options main)

# Add a file containing a set of constexpr tests
add_executable(constexpr_tests constexpr_tests.cpp)
target_link_libraries(constexpr_tests PRIVATE cmake_vcpkg_doctest_template::project_options
                                              cmake_vcpkg_doctest_template::project_warnings main)

# Disable the constexpr portion of the test, and build again this allows us to have an executable that we can debug when
# things go wrong with the constexpr testing
add_executable(relaxed_constexpr_tests constexpr_tests.cpp)
target_link_libraries(relaxed_constexpr_tests PRIVATE cmake_vcpkg_doctest_template::project_options
                                                      cmake_vcpkg_doctest_template::project_warnings main)
target_compile_definitions(relaxed_constexpr_tests PRIVATE -DCATCH_CONFIG_RUNTIME_STATIC_REQUIRE)
