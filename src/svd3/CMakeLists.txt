set(SVD3_SOURCES svd3_cuda/svd3.cu)
set(SVD3_HEADESR svd3.h svd3_cuda/svd3_cuda.h)


CUDA_ADD_LIBRARY(SVD3 ${SVD3_SOURCES} ${SVD3_HEADESR})

# the ".." is needed to build as dependency to be able to include as <SVD3/....>
target_include_directories(SVD3 PUBLIC
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src>
        $<INSTALL_INTERFACE:include/>)

set_target_properties(SVD3
        PROPERTIES SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
        VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")

# Targets:
#   * header location after install: <prefix>/include/
install(TARGETS SVD3
        EXPORT "${targets_export_name}"
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION "${include_install_dir}/svd3")

install(DIRECTORY ${PROJECT_SOURCE_DIR}/src/svd3
        DESTINATION "${include_install_dir}/svd3"
        FILES_MATCHING PATTERN "*.hpp" PATTERN "*.h")
