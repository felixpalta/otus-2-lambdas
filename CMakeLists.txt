cmake_minimum_required(VERSION 3.2)

project(lambdas VERSION 0.0.$ENV{TRAVIS_BUILD_NUMBER})

message("Version ${PROJECT_VERSION_MAJOR} ${PROJECT_VERSION_MINOR} ${PROJECT_VERSION_PATCH}")

find_package(Boost COMPONENTS unit_test_framework REQUIRED)

configure_file(version.h.in version.hpp @ONLY)

add_executable(lambdas_cli "main.cpp")
add_library(lambdas "lib.cpp")
add_executable(test_version_boost test_version_boost.cpp)

set_target_properties(lambdas_cli lambdas test_version_boost PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)
target_include_directories(lambdas
    PRIVATE "${CMAKE_BINARY_DIR}"
)
set_target_properties(test_version_boost PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)
target_link_libraries(lambdas_cli PRIVATE
    lambdas
)
target_link_libraries(test_version_boost
    ${Boost_LIBRARIES}
    lambdas
)

if (MSVC)
    target_compile_options(lambdas_cli PRIVATE
        /W4
    )
    target_compile_options(lambdas PRIVATE
        /W4
    )
    target_compile_options(test_version_boost PRIVATE
        /W4
    )
else ()
    target_compile_options(lambdas_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(lambdas PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(test_version_boost PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
endif()

install(TARGETS lambdas_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT "Felix Palta <my@email.com>")

include(CPack)

enable_testing()

add_test(test_version_boost test_version_boost)
