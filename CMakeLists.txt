cmake_minimum_required(VERSION 3.14...3.22)

set(THE_constexpr-to-string_VERSION 1.0.1)

if(constexpr-to-string_TEST_INSTALLED_VERSION)
    project(
    constexpr-to-string_test_installation
    VERSION ${THE_constexpr-to-string_VERSION}
    LANGUAGES CXX)

    find_package(constexpr-to-string ${THE_constexpr-to-string_VERSION} REQUIRED)

    message("constexpr-to-string_FOUND: ${constexpr-to-string_FOUND}")

    file(WRITE "${CMAKE_BINARY_DIR}/${PROJECT_NAME}dummy_target.cpp"
       "#include <constexpr-to-string/to_string.hpp>\nint main() {}\n")

    add_executable(dummy_target
                 "${CMAKE_BINARY_DIR}/${PROJECT_NAME}dummy_target.cpp")

    target_link_libraries(dummy_target
                        PRIVATE constexpr-to-string::constexpr-to-string)

    find_package(PkgConfig)

    pkg_check_modules(constexpr-to-string GLOBAL IMPORTED_TARGET
                    constexpr-to-string)

    file(WRITE "${CMAKE_BINARY_DIR}/${PROJECT_NAME}dummy_target2.cpp"
       "#include <constexpr-to-string/to_string.hpp>\nint main() {}\n")

    add_executable(dummy_target2
                 "${CMAKE_BINARY_DIR}/${PROJECT_NAME}dummy_target2.cpp")

    target_link_libraries(dummy_target2 PRIVATE PkgConfig::constexpr-to-string)
    return()
endif()

project(
  constexpr-to-string
  VERSION ${THE_constexpr-to-string_VERSION}
  LANGUAGES CXX)

# ---- Include guards ----

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
    message(
    FATAL_ERROR
      "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
  )
endif()

# ---- Add source files ----

# Note: globbing sources is considered bad practice as CMake's generators may
# not detect new files automatically. Keep that in mind when changing files, or
# explicitly mention them here.
file(
  GLOB_RECURSE headers
  RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
  CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/include/*.hpp")

# ---- Create library ----

add_library(${PROJECT_NAME} INTERFACE)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 14)

target_include_directories(
  ${PROJECT_NAME} INTERFACE $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                            $<INSTALL_INTERFACE:include>)

# ---- Installation logic ----

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/${PROJECT_NAME}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}Targets
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(
  EXPORT ${PROJECT_NAME}Targets
  DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME}/
  NAMESPACE ${PROJECT_NAME}::)

file(
  WRITE "${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake.in"
  "@PACKAGE_INIT@\ninclude(\$\{CMAKE_CURRENT_LIST_DIR\}/${PROJECT_NAME}Targets.cmake)"
)

configure_package_config_file(
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME})

install(FILES "${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME})

write_basic_package_version_file(
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion ARCH_INDEPENDENT)

install(FILES "${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME})

file(
  GENERATE
  OUTPUT "${CMAKE_BINARY_DIR}/${PROJECT_NAME}.pc"
  CONTENT
    "prefix=${CMAKE_INSTALL_PREFIX}\nincludedir=\$\{prefix\}/include\nName: ${PROJECT_NAME}\nDescription:  C++14 code to convert integers to strings at compile-time \nVersion: ${PROJECT_VERSION}\nCflags: -I\$\{includedir\}\n"
)

install(FILES "${CMAKE_BINARY_DIR}/${PROJECT_NAME}.pc"
        DESTINATION ${CMAKE_INSTALL_DATADIR}/pkgconfig)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    include(CPack)
endif()
