## Building Boost for emscripten

We provide the pre-built Boost libraries, but if the compiler version is changed, the boost libraries must be rebuilt. 
Here is how to build Boost 1.58.0 (same version as core uses):

1. Install and activate the emscripten desired tools (as explained above)

2. Build Boost:

2.1. Download boost 1.58.0 and navigate to the root folder, usually `boost_1_58_0`

2.2. Changes to Boost code:

- Force the macro `BOOST_HAS_SCHED_YIELD` to be defined in `boost/config/posix_features.hpp`.

2.3. Prepare the build environment. Until Boost 1.64 it is only supported Python2:

```
./bootstrap.sh --with-python=python2 --prefix=path/to/installation/prefix
```

where `--prefix` is the path where you want `boost` to be installed (it can be anywhere)

2.4. Override the default `gcc` build environment with the `emscripten` one:

In project-config.jam:

```
Replace `using gcc ;` 
with `using gcc : : "em++" ;`
```

In tools/build/src/tools/gcc.jam:

```
Replace `toolset.flags gcc.archive .AR $(condition) : $(archiver[1]) ;`
with `toolset.flags gcc.archive .AR $(condition) : "emar" ;`

Replace `toolset.flags gcc.archive .RANLIB $(condition) : $(ranlib[1]) ;`
with `toolset.flags gcc.archive .RANLIB $(condition) : "emranlib" ;`
```

2.5 Build boost.
Notes: 
- Adjust -j12 to 2*[number of your processor's cores] for faster builds.
- if your system provides `bzlib.h`, you can remove the NO_BZIP2 flag to enable also support for bzlib.
- If Boost supports C++17 (boost has already removed `std::auto_ptr` usages), the C++14 flags can be removed.
- `--build-dir` is the directory where the build files will be placed (usually next to the `--prefix` one above).
- `--stagedir` is the directory where the output library files will be placed.
- context and coroutine libraries are not supported on asmjs.
- python gives some sort of error about invalid LONG_BIT definition, so removed also.
- The `-Wno-enum-constexpr-conversion` is needed because the compiler is too strict for this version of boost.

```
./b2 -j12 cxxflags="-std=c++14 -Wno-enum-constexpr-conversion" linkflags=-std=c++14 variant=release threading=multi link=static --without-python --without-context --without-coroutine -sNO_BZIP2=1 --build-dir=path/to/build/dir --stagedir=path/to/libs/output stage
```

2.6 Copy all needed files.

- Copy all the `--stagedir/lib` libraries into the project's `lib` directory.
- Copy the `boost_1_58_0/boost` directory into the project's `include` directory.

