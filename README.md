# OpenXcom

For information about the OpenXcom project please refer to
[SupSuper's repository](https://github.com/SupSuper/OpenXcom).
This repository is mainly about building OpenXcom with a MinGW-w64 toolchain,
building with SDL 1.2, SDL image 1.2, SDL mixer 1.2 and SDL gfx DLLs and
statically with yaml-cpp. Steps are provided for building both 32-bits and
64-bits. You can use either [Msys2](http://msys2.github.io/)
or [TDM-GCC-64](http://tdm-gcc.tdragon.net/) for instance.

## Bi-arch toolchain or two toolchains

Most MinGW-w64 toolchains target only 32-bits or 64-bits toolchains. This is
the case with Msys2, you will need to install two sets of compiler if you
want to build both 32-bits and 64-bits binaries.
Other toolchains are bi-arch toolchains and will let you do that with a single
compiler. An example is TDM-GCC-64.

### Switch toolchains

If you use two mono-arch toolchains, you will have to update the Path
environment variable so that the toolchain you want to use will be found.
Don't change the global variable environment. Either create two batch scripts
to select a toolchain, or manually set Path in two DOS consoles (because
each running DOS console "remember" its own environment variables).

If you are not sure, use this command to check your toolchain:

    where gcc

First line will give you the location of the gcc currently pointed by Path.

### Change target with bi-arch toolchain

* For 32-bits building use `-m32` option both for compiling and linking.
* For 64-bits building use `-m64` option both for compiling and linking.

TDM-GCC-64 defaults to `-m64`, but it does not hurt to add this option when
building 64-bits objects and binaries. The options can be used with a mono-arch
toolchain, they will simply leads to building error if you use one with the
wrong toolchain. That means adding them can be a good way to make sure you have
properly set the toolchain you want to use, so I recommend setting them.

## SDL

### SDL 1.2

This part is actually the more painful for 64-bits building.
Go to [SDL 1.2 downloads](https://www.libsdl.org/download-1.2.php) and grab:
* SDL-devel-1.2.15-mingw32.tar.gz
* SDL-1.2.15.zip
* SDL-1.2.15-win32-x64.zip

Exctract SDL-1.2.15.zip and use either [Msys](http://mingw.org/wiki/msys),
[Cygwin](http://www.cygwin.com/) or [Msys2](http://msys2.github.io/) to build
a 64-bits SDL 1.2. Keep only the generated `libSDL.dll.a` and `libSDL.a`.

As an example with Cygwin64 do the following: install Cygwin64.
Choose to install the following packages if they are not already selected:
* coreutils from Base
* base-cygwin from Base
* cygwin from Base
* autoconf from Devel
* make from Devel
* mingw64-x86_64-gcc-g++ from Devel (4.9.x version rather than latter one)
* accept all proposed dependencies

Now run a Cygwin64 terminal, go the folder where you extracted SDL-1.2.15.zip,
(if it is on disk D: go to `/cygfrive/d/<path>` where `<path>` is the directory
path without disk letter and with backslashes replaced with slashes) and run:

    ./autogen.sh
    ./configure --host=x86_64-w64-mingw32
    make

This should generate `libSDL.dll.a` and `libSDL.a` in `build\.libs` subfolder.
I prefer not to use other generated files as they are available for download.

Copy header files to `<project>\SDL-1.2\include`, 32-bits DLLs to
`<project>\SDL-1.2\bin32`, 64-bits DLLs to `<project>\SDL-1.2\bin64`,
32-bits `libSDL.dll.a` and `libSDL.a` to `<project>\SDL-1.2\lib32`,
and 64-bits `libSDL.dll.a` and `libSDL.a` to `<project>\SDL-1.2\lib64`.

### SDL image 1.2 and SDL_mixer 1.2

Go to
[SDL mixer 1.2](https://www.libsdl.org/projects/SDL_mixer/release-1.2.html)
and to
[SDL image 1.2](https://www.libsdl.org/projects/SDL_image/release-1.2.html)
to grab the following files:
* SDL_mixer-1.2.12-win32.zip
* SDL_mixer-1.2.12-win32-x64.zip
* SDL_mixer-devel-1.2.12-VC.zip
* SDL_image-1.2.12-win32.zip
* SDL_image-1.2.12-win32-x64.zip
* SDL_image-devel-1.2.12-VC.zip

Copy header files to `<project>\SDL-1.2\include`, 32-bits DLLs to
`<project>\SDL-1.2\bin32` and 64-bits DLLs to `<project>\SDL-1.2\bin64`.

### SDL gfx

Grab SDL_gfx-2.0.25.tar.gz from
[SDL_gfx website](http://www.ferzkopp.net/wordpress/2016/01/02/sdl_gfx-sdl2_gfx/).
Extract it to folder `<project>\Build_SDL_gfx`. Extract `Makefile` from
`<project>\Build_SDL_gfx\Other Builds\mingw32.zip` to folder
`<project>\Build_SDL_gfx`.

Update `Makefile` for 32-bits building:

    CFLAGS = -O3 -m32 -march=athlon-xp -mmmx -msse -m3dnow -DBUILD_DLL -DWIN32 -I../SDL-1.2/include
    LIBS = ../SDL-1.2/bin32/SDL.dll
    OBJS = SDL_framerate.o SDL_gfxBlitFunc.o SDL_gfxPrimitives.o SDL_imageFilter.o SDL_rotozoom.o
        gcc -m32 -s -shared -o $@ $^ -static-libgcc -Wl,--enable-auto-import,--export-all -Wl,--out-implib=$(IMPORT_LIB) $(LIBS)

Switch to 32-bits toolchain and run:

    mingw32-make

Move `libSDL_gfx.a` and `libSDL_gfx.dll.a` to `<project>\SDL-1.2\lib32`,
move `SDL_gfx.dll` to `<project>\SDL-1.2\bin32` and delete `.o` files.

Update `Makefile` for 64-bits building:

    CFLAGS = -O3 -m64 -march=athlon-xp -mmmx -msse -m3dnow -DBUILD_DLL -DWIN32 -I../SDL-1.2/include
    LIBS = ../SDL-1.2/bin32/SDL.dll
    OBJS = SDL_framerate.o SDL_gfxBlitFunc.o SDL_gfxPrimitives.o SDL_imageFilter.o SDL_rotozoom.o
        gcc -m64 -s -shared -o $@ $^ -static-libgcc -Wl,--enable-auto-import,--export-all -Wl,--out-implib=$(IMPORT_LIB) $(LIBS)

Switch to 64-bits toolchain and run:

    mingw32-make

Move `libSDL_gfx.a` and `libSDL_gfx.dll.a` to `<project>\SLD-1.2\lib64`,
move `SDL_gfx.dll` to `<project>\SDL-1.2\bin64` and delete `.o` files.
Copy header files to `<project>\SDL-1.2\include`.

## yaml-cpp

If you want to build OpenXCom from Supsuper's repository, use yaml-cpp 0.5.1
and stick to c++98 standard. If you want to build
[Meridian's OXCE](https://github.com/MeridianOXC/OpenXcom/tree/oxce2.9-plus-proto)
use yaml-cpp 0.5.3 with c++11 standard. Whatever you do, do not use
yaml-cpp 0.5.2 for OpenXCom.

### CMake

Easier way to build yaml-cpp is using [CMake](https://cmake.org/). 
Download and install. You will need to have CMake in Path to use it.

### yaml-cpp 0.5.1

Grab the sources of yaml-cpp 0.5.1 from
[GitHub](https://github.com/jbeder/yaml-cpp/releases/tag/release-0.5.1).
Grab [the last release boost source](http://www.boost.org/)
(version 1.60 should be fine, so I use it as an example).

Extract boost to folder `<project>\boost_1_60_0`. Extract yaml-cpp to folder
`<project>\Build_yaml-cpp-0.5.1`. Go to this folder.

Switch to 32-bits toolchain and run:

    mkdir build_x86
    cd build_x86
    cmake -G "MinGW Makefiles" -DBUILD_SHARED_LIBS=OFF -DCMAKE_CXX_FLAGS=-m32 -DBoost_INCLUDE_DIR=<project>\boost_1_60_0 ..
    mingw32-make

Copy generated `libyaml-cpp.a` into `<project>\yaml-cpp-0.5.1\bin32`.

Switch to 64-bits toolchain and run:

    mkdir build_x64
    cd build_x64
    cmake -G "MinGW Makefiles" -DBUILD_SHARED_LIBS=OFF -DCMAKE_CXX_FLAGS=-m64 -DBoost_INCLUDE_DIR=<project>\boost_1_60_0 ..
    mingw32-make

Copy generated `libyaml-cpp.a` into `<project>\yaml-cpp-0.5.1\bin64`.
Copy `include` folder into `<project>\yaml-cpp-0.5.1\`.

### yaml-cpp 0.5.3

Grab the sources of yaml-cpp 0.5.3 from
[GitHub](https://github.com/jbeder/yaml-cpp/releases/tag/release-0.5.3) and
extract to folder `<project>\Build_yaml-cpp-0.5.3`. Go to this folder.

switch to 32-bits toolchain and run:

    mkdir build_x86
    cd build_x86
    cmake -G "MinGW Makefiles" -DBUILD_SHARED_LIBS=OFF -DCMAKE_CXX_FLAGS=-m32 ..
    mingw32-make

Copy generated `libyaml-cpp.a` into `<project>\yaml-cpp-0.5.3\bin32`.

switch to 64-bits toolchain and run:

    mkdir build_x64
    cd build_x64
    cmake -G "MinGW Makefiles" -DBUILD_SHARED_LIBS=OFF -DCMAKE_CXX_FLAGS=-m64 ..
    mingw32-make

Copy generated `libyaml-cpp.a` into `<project>\yaml-cpp-0.5.3\bin64`.
Copy `include` folder into `<project>\yaml-cpp-0.5.3\`.

## Building OpenXcom

Clone SupSuper's repository in `<project>\OpenXcom` or any of its various forks
which interests you.
Get from this repository either `makefile-mingw64-yaml-cpp-0.5.1` or
`makefile-mingw64-yaml-cpp-0.5.3`. Rename this file as
`<project>\OpenXcom\makefile` and edit it.

Grab and build [AppDep](https://github.com/JDarthenay/AppDep).
I18N support for AppDep is not mandatory, so if you don't care translations
don't try to build them, AppDep will be fine.

You need to set `PATH_BUILD_X86` to your Path for 32-bits toolchain,
and `PATH_BUILD_X64` to your Path for 64-bits toolchain.
Use same value twice if you use a bi-arch toolchain.
Set `PATH_APPDEP` value so that it gives access to `AppDep.exe` and any one of
your toolchains.

Run:

    mingw32-make depends
    mingw32-make all

Instead of `all`, you can build a target such as `release_x64`
or `release_x86`.

You can check
[GNU make documentation](https://www.gnu.org/software/make/manual/make.html) to
understand how works the two makefiles I provide.