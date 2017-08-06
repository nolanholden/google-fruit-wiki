### Dependencies

Fruit's only dependency is on the Boost hashset/hashmap implementation. Even that is optional: you can switch to the
STL implementation by passing the flag `-DFRUIT_USES_BOOST=False` to CMake. In any case, the Boost library is only
required when building Fruit itself and not when building code that uses Fruit.

Also, Fruit uses many C++11 features so you need to use a relatively recent compiler in C++11 mode (or later).
**This is not just to build Fruit itself, but also to build any code that uses Fruit.**

The supported compiler/STL/OS combinations are:

*   GCC 5.0.0 or later on Linux and OS X
*   MinGW's GCC 5.0.0 or later on Windows
*   Clang 3.5 or later using stdlibc++ (GCC's STL) on Linux
*   Clang 3.5 or later using libc++ on Linux and OS X
*   MSVC 2017 or later on Windows

That said, any C++11-compliant compiler should work on any platform.

Compilers known _not_ to work due to a lacking C++11 support and/or compiler bugs:

*   Intel compiler ([bug1](https://software.intel.com/en-us/comment/1862049),
    [bug2](https://software.intel.com/en-us/comment/1854501))

If you find any other compiler/platform where Fruit doesn't work, feel free to
[contact me](mailto:poletti.marco@gmail.com) and I can take a look.

### Prebuilt packages

Prebuilt packages for Fedora, openSUSE, Ubuntu, Debian and various other distributions are available. These packages are
built using GCC, and **they may not work if you want to compile your project with Clang or other compilers**. In that
case, see the section below on how to compile Fruit manually.

**[Choose your distribution here](http://software.opensuse.org/download.html?project=home%3Apoletti_marco&package=libfruit)**

After following the instructions there, to install the headers (necessary to compile programs that use Fruit):

*   For Fedora, openSUSE, RHEL, CentOS, SLE: install the `libfruit-devel` package.
*   For *Ubuntu, Debian: install the `fruit-dev` package.
*   For Arch: nothing to do, the headers are already installed.

I will write packages or build files for other distributions on request. Or if you write one, please send it to me and I
will publish it here.

### Building Fruit manually

In most cases you can (and should) use the prebuilt packages instead, see the previous section.

#### With CMake, on Linux/OS X

For building Fruit, you'll need to have `cmake` and `make` installed, together with a recent version of GCC/Clang (see
above).

First, get the code from Github: [https://github.com/google/fruit/releases](https://github.com/google/fruit/releases)

To configure and build:

    cmake -DCMAKE_BUILD_TYPE=Release . && make -j

Or if you don't want to use Boost:

    cmake -DCMAKE_BUILD_TYPE=Release -DFRUIT_USES_BOOST=False . && make -j

To install (under Linux this uses `/usr/local`):

    sudo make install

To configure for installation in a specific directory, e.g. `/usr`:

    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr . && make -j

The above instructions are the simplest to get started, but out-of-source builds are also supported.

#### With MSVC, on Windows

You can import Fruit in Visual Studio (2017 and later) as a CMake project. You need to set the relevant CMake flags in
the `CMakeSettings.json` file that Visual Studio will create.
For example, if you installed Boost in `C:\boost\boost_1_62_0`, you can put this configuration in your
`CMakeSettings.json`:

    {
        "configurations": [
            {
                "name": "x64-Release",
                "generator": "Visual Studio 15 2017 Win64",
                "configurationType": "Release",
                "buildRoot": "${env.LOCALAPPDATA}\\CMakeBuild\\${workspaceHash}\\build\\${name}",
                "cmakeCommandArgs": "-DBOOST_DIR=C:\\boost\\boost_1_62_0 -DCMAKE_BUILD_TYPE=Release",
                "buildCommandArgs": "-m -v:minimal"
            }
        ]
    }

Or if you don't want to use Boost, you can use:

    {
        "configurations": [
            {
                "name": "x64-Release",
                "generator": "Visual Studio 15 2017 Win64",
                "configurationType": "Release",
                "buildRoot": "${env.LOCALAPPDATA}\\CMakeBuild\\${workspaceHash}\\build\\${name}",
                "cmakeCommandArgs": "-DFRUIT_USES_BOOST=False -DCMAKE_BUILD_TYPE=Release",
                "buildCommandArgs": "-m -v:minimal"
            }
        ]
    } 

You can now run CMake within Visual Studio (from the menu: CMake -> Cache -> Generate -> CMakeLists.txt) and build Fruit
(from the menu: CMake -> Build All).

For more information (e.g. how to run Fruit tests in MSVC) see the
[CONTRIBUTING.md](https://github.com/google/fruit/blob/master/CONTRIBUTING.md) file.


#### With Bazel, on Linux

Building with [Bazel](http://bazel.io) is also supported. You should import the `fruit/extras/bazel_root` directory,
not the Fruit root. See the [CONTRIBUTING.md](https://github.com/google/fruit/blob/master/CONTRIBUTING.md) file
for more details.
