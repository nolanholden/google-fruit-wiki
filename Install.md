### Dependencies

There are no library dependencies, but this project uses many C++11 features so you need to use a relatively recent compiler in C++11 mode (**not just to build Fruit itself, also to build any code that uses Fruit**).

The supported compiler/STL/OS combinations are:

*   GCC 4.8.3 or later on Linux and OS X
*   Clang 3.5 or later using stdlibc++ (GCC's STL) on Linux
*   Clang 3.5 or later using libc++ on Linux and OS X

That said, any C++11-compliant compiler should work on any platform.

Compilers known _not_ to work due to a lacking C++11 support and/or compiler bugs:

*   Visual Studio ([bug1](https://connect.microsoft.com/VisualStudio/Feedback/Details/2197169), [bug2](https://connect.microsoft.com/VisualStudio/Feedback/Details/2197110))
*   Intel compiler ([bug1](https://software.intel.com/en-us/comment/1862049), [bug2](https://software.intel.com/en-us/comment/1854501))

If you find any other compiler/platform where Fruit doesn't work, feel free to [contact me](mailto:poletti.marco@gmail.com) and I can take a look.

### Prebuilt packages

Prebuilt packages for Fedora, openSUSE, Ubuntu, Debian and various other distributions are available. These packages are built using GCC, and **they won't work if you want to compile your project with Clang or other compilers**. In that case, see the section below on how to compile Fruit manually.

**[Choose your distribution here](http://software.opensuse.org/download.html?project=home%3Apoletti_marco&package=libfruit)**

After following the instructions there, to install the headers (necessary to compile programs that use Fruit):

*   For Fedora, openSUSE, RHEL, CentOS, SLE: install the `libfruit-devel` package.
*   For *Ubuntu, Debian: install the `fruit-dev` package.
*   For Arch: nothing to do, the headers are already installed.

I will write packages or build files for other distributions on request. Or if you write one, please send it to me and I will publish it here.

### Building Fruit manually

In most cases you can (and should) use the prebuilt packages instead, see the previous section.

For building Fruit, you'll need to have `cmake` and `make` installed, together with a recent version of GCC/Clang (see above).

First, get the code from Github: [https://github.com/google/fruit/releases](https://github.com/google/fruit/releases)

To configure and build:

    cmake -DCMAKE_BUILD_TYPE=Release . && make -j

To install (under Linux uses `/usr/local`):

    sudo make install

To configure for installation in a specific directory, e.g. `/usr`:

    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr . && make -j

The above instructions are the simplest to get started, but out-of-source builds are also supported.

Since Fruit 2.0.0, building with [Bazel](http://bazel.io) is also supported.
