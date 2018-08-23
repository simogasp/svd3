cmake_minimum_required(VERSION 3.1.3)

project(SVD3 LANGUAGES C CXX VERSION 1.0)

option(SVD3_BUILD_SAMPLES "Build samples" ON)

find_package(CUDA 7.0 REQUIRED)

set(CUDA_SEPARABLE_COMPILATION ON)

set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode;arch=compute_30,code=sm_30")
set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode;arch=compute_35,code=sm_35")
set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode;arch=compute_50,code=sm_50")
set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode;arch=compute_52,code=sm_52")
set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode=arch=compute_52,code=compute_52")

if(NOT CUDA_VERSION VERSION_LESS 8.0) # cuda >= 8.0
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode=arch=compute_60,code=sm_60")
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode=arch=compute_61,code=sm_61")
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode=arch=compute_62,code=sm_62")
endif()

if(NOT CUDA_VERSION VERSION_LESS 7.0) # cuda >= 7.0
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-std=c++11;")
endif()

set(SVD3_SOURCES svd3_cuda/svd3.cu )
set(SVD3_HEADESR svd3.h svd3_cuda/svd3_cuda.h )


CUDA_ADD_LIBRARY(SVD3 ${SVD3_SOURCES} ${SVD3_HEADESR})

# the ".." is needed to build as dependency to be able to include as <SVD3/....>
target_include_directories(SVD3 PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>
        $<INSTALL_INTERFACE:include/>)

set_target_properties(SVD3
        PROPERTIES SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
        VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")

if(SVD3_BUILD_SAMPLES)
    add_executable(testCpu main_cpu.cpp)
    target_link_libraries(testCpu PUBLIC SVD3)

    add_executable(testGpu svd3_cuda/main.cpp)
    target_link_libraries(testGpu PUBLIC SVD3)

    find_package(glm QUIET)
    if(glm_FOUND)
        add_executable(testGlm main_glm.cpp)
        target_compile_definitions(testGlm PUBLIC -DGLM_ENABLE_EXPERIMENTAL)
        target_link_libraries(testGlm PUBLIC glm SVD3)
    else()
        message(STATUS "GLM not found - Sample will not be built")
    endif()
endif()


# EXPORTING THE LIBRARY
#
include(GNUInstallDirs)
# place to put the cmake-related files
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
# include directory for install
set(include_install_dir "include")

# build directory containing the generated files
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Note: major version number must be the same as requested
write_basic_package_version_file("${version_config}" COMPATIBILITY SameMajorVersion)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file("${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in"
        "${project_config}"
        INSTALL_DESTINATION "${config_install_dir}")


# Targets:
#   * header location after install: <prefix>/include/
install(TARGETS SVD3
        EXPORT "${targets_export_name}"
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION "${include_install_dir}/SVD3")

install(DIRECTORY ${PROJECT_SOURCE_DIR}/svd3_cuda DESTINATION "${include_install_dir}/SVD3" FILES_MATCHING PATTERN "*.hpp" PATTERN "*.h")
install(FILES svd3.h DESTINATION "${include_install_dir}/SVD3")


# Config
#   * <prefix>/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake
install(EXPORT "${targets_export_name}"
        NAMESPACE "${namespace}"
        DESTINATION "${config_install_dir}")

# Config
#   * <prefix>/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Config.cmake
#   * <prefix>/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}ConfigVersion.cmake
install(
        FILES "${project_config}" "${version_config}"
        DESTINATION "${config_install_dir}")

export(PACKAGE SVD3)

## Uninstall target
configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        IMMEDIATE @ONLY)

add_custom_target(SVD3_uninstall
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
