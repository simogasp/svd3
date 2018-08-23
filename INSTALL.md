# Fast 3x3 SVD

## Building instructions

Required tools:
* CMake >= 3.7 
* Git
* C/C++ compiler (gcc or visual studio or clang) with C++11 support.

Dependencies

* CUDA >= 7.0

### Compile the project

```bash
git clone https://github.com/EncovGroup/svd3
mkdir build && cd build
cmake  -DCMAKE_INSTALL_PREFIX=$PWD/install ..
make -j 8
```

### CMake options

* `SVD3_BUILD_SAMPLES` (default `ON`) Build the samples 
* `SVD3_EXPORT_PACKAGE` (default `OFF`) Export the library as cmake package


## Using SVD3 as third party

When you install SVD3 (say in `<prefix>`) a file `SVD3Config.cmake` is installed in `<prefix>/lib/cmake/SVD3/` that allows you to import the library in your CMake project.
In your `CMakeLists.txt` file you can add the dependency in this way:

```cmake
# Find the package from the SVD3Config.cmake 
# in <prefix>/lib/cmake/SVD3/. Under the namespace SVD3::
# it exposes the target SVD3 that allows you to compile
# and link with the library
find_package(SVD3 CONFIG REQUIRED)
...
# suppose you want to try it out in a executable
add_executable(svd3test yourfile.cpp)
# add link to the library
target_link_libraries(svd3test PUBLIC SVD3::SVD3)
```

Then, in order to build just pass the location of `SVD3Config.cmake` from the cmake command line:

```bash
cmake .. -DSVD3_DIR=<prefix>/lib/cmake/SVD3/
```

Note that `target_include_directories()` is not necessary.