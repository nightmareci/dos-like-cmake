It's simplest to just clone the original
[dos-like](https://github.com/mattiasgustavsson/dos-like) repo right into this
source tree; it's not provided in this repo as a Git submodule, so you can
easily clone a fork. A regular `git clone` of `dos-like` into this source tree
will clone into a form the `CMakeLists.txt` script will work with by default:

```sh
git clone https://github.com/mattiasgustavsson/dos-like.git
cmake -B build
cmake --build build
cd dos-like
../build/stranded
```

Be sure to have the working directory in the `dos-like` source tree when
running the examples, or they won't work.

Alternatively, you can manually specify the root of the `dos-like` source tree,
but it must match the layout of the original one:
```sh
cmake -B build -DDOS_LIKE_SOURCE=/path/to/dos-like
```

This CMake script is intended for developers of the `dos-like` project, to make
it easy to develop and debug the library and examples in an IDE supported by
CMake.

This CMake script only supports Windows MSVC builds not using SDL2 nor GLEW, or
builds on non-Windows platforms using SDL2 and GLEW. Building with TinyC isn't
supported.
