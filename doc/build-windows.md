WINDOWS BUILD NOTES
====================

Below are some notes on how to build Bitcoin Core for Windows.

Most developers use cross-compilation from Ubuntu to build executables for
Windows. This is also used to build the release binaries.

Currently only building on Ubuntu Trusty 14.04 is supported.
Other versions are unsupported or known to be broken (e.g. Ubuntu Xenial 16.04).

While there are potentially a number of ways to build on Windows (for example using msys / mingw-w64),
using the Windows Subsystem For Linux is the most straightforward. If you are building with
another method, please contribute the instructions here for others who are running versions
of Windows that are not compatible with the Windows Subsystem for Linux.

Compiling with Windows Subsystem For Linux
-------------------------------------------

With Windows 10, Microsoft has released a new feature named the [Windows
Subsystem for Linux](https://msdn.microsoft.com/commandline/wsl/about). This
feature allows you to run a bash shell directly on Windows in an Ubuntu-based
environment. Within this environment you can cross compile for Windows without
the need for a separate Linux VM or server.

This feature is not supported in versions of Windows prior to Windows 10 or on
Windows Server SKUs. In addition, it is available [only for 64-bit versions of
Windows](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide).

To get the bash shell, you must first activate the feature in Windows.

1. Turn on Developer Mode
  * Open Settings -> Update and Security -> For developers
  * Select the Developer Mode radio button
  * Restart if necessary
2. Enable the Windows Subsystem for Linux feature
  * From Start, search for "Turn Windows features on or off" (type 'turn')
  * Select Windows Subsystem for Linux (beta)
  * Click OK
  * Restart if necessary
3. Complete Installation
  * Open a cmd prompt and type "bash"
  * Accept the license
  * Create a new UNIX user account (this is a separate account from your Windows account)

After the bash shell is active, you can follow the instructions below, starting
with the "Cross-compilation" section. Compiling the 64-bit version is
recommended but it is possible to compile the 32-bit version.

Cross-compilation
-------------------

These steps can be performed on, for example, an Ubuntu 14.04 VM. The depends system
will also work on other Linux distributions, however the commands for
installing the toolchain will be different.

In your configure.ac, make sure your libtool initialization looks like:

LT_INIT([win32-dll])



First install the toolchains:

sudo apt-get install git curl build-essential libtool autotools-dev automake pkg-config bsdmainutils libdb++-dev libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils python3


A host toolchain (`build-essential`) is necessary because some dependency
packages (such as `protobuf`) need to build host utilities that are used in the
build process.

## Install libsodium on Ubuntu 14.04

    sudo add-apt-repository ppa:chris-lea/libsodium
    sudo echo "deb http://ppa.launchpad.net/chris-lea/libsodium/ubuntu trusty main" >> sudo /etc/apt/sources.list
    sudo echo "deb-src http://ppa.launchpad.net/chris-lea/libsodium/ubuntu trusty main" >> sudo /etc/apt/sources.list
    sudo apt-get update && sudo apt-get install libsodium-dev

## Install libsodium on Ubuntu 16.04

    sudo apt install libsodium-dev

## Building for 64-bit Windows

To build executables for Windows 64-bit, install the following dependencies:

    sudo apt-get install g++-mingw-w64-x86-64 mingw-w64-x86-64-dev

Then build using:

    cd depends
    make HOST=x86_64-w64-mingw32
    cd ../src/secp256k1
    ./autogen.sh
    ./configure --prefix=`pwd`/depends/x86_64-w64-mingw32
    make HOST=x86_64-w64-mingw32
    cd ../..
    ./autogen.sh
    ./configure --host=x86_64-w64-mingw32 --target=x86_64-w64-mingw32 --prefix=`pwd`/depends/x86_64-w64-mingw32 --with-incompatible-bdb
    make HOST=x86_64-w64-mingw32

## Building for 32-bit Windows

To build executables for Windows 32-bit, install the following dependencies:

    sudo apt-get install g++-mingw-w64-i686 mingw-w64-i686-dev

Then build using:

    cd depends
    make HOST=i686-w64-mingw32
    cd ..
    ./autogen.sh # not required when building from tarball
    CONFIG_SITE=$PWD/depends/i686-w64-mingw32/share/config.site ./configure --prefix=/   
    make

## Depends system

For further documentation on the depends system see [README.md](../depends/README.md) in the depends directory.

Installation
-------------

After building using the Windows subsystem it can be useful to copy the compiled
executables to a directory on the windows drive in the same directory structure
as they appear in the release `.zip` archive. This can be done in the following
way. This will install to `c:\workspace\bitcoin`, for example:

    make install DESTDIR=/mnt/c/workspace/bitcoin
