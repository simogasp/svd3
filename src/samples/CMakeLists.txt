add_executable(testCpu main_cpu.cpp)
target_link_libraries(testCpu PUBLIC SVD3)

add_executable(testGpu main_gpu.cpp)
target_link_libraries(testGpu PUBLIC SVD3)

find_package(glm QUIET)
if(glm_FOUND)
    add_executable(testGlm main_glm.cpp)
    target_compile_definitions(testGlm PUBLIC -DGLM_ENABLE_EXPERIMENTAL)
    target_link_libraries(testGlm PUBLIC glm SVD3)
else()
    message(STATUS "GLM not found - Sample will not be built")
endif()