# Mac OS X Build Instructions and Notes

This guide will show you how to build `sibcoind` (headless client) and `sibcoin-qt` (GUI wallet) for OSX.

## Notes

* Tested on OS X 10.7 through 10.11 on 64-bit Intel processors only.

* All of the commands should be executed in a Terminal application. The
built-in one is located in `/Applications/Utilities`.

## Prerequisites

You need to install Xcode with all the options checked so that the compiler
and everything is available in `/usr`, not just in `/Developer` directory. You can find Xcode on your OS X installation media, or you download the
latest version from [developer.apple.com/xcode/](https://developer.apple.com/xcode/). Versions 4.3 or later of Xcode require you to install command line tools as a separate step by selecting "Xcode > Preferences > Downloads > Components" (you may need
to repeat this if you recently upgraded the Xcode).

To install library dependencies, you will need [Homebrew](http://brew.sh).

## Building Sibcoin from Source

### Installing Dependencies with Homebrew

    brew install autoconf automake berkeley-db4 libtool boost miniupnpc \
        openssl pkg-config protobuf libevent

### Building Sibcoin Daemon

1. Clone the GitHub tree to get the source code and go into the directory.

    	git clone https://github.com/ivansib/sib16.git
    	cd sibcoin

2.  Build Sibcoin Core. This will configure and build the headless sibcoin
    binaries as well as the gui (if Qt is found). You can disable the gui build
    by passing `--without-gui` to configure.

    	./autogen.sh
    	./configure --without-gui
    	make

3.  It is also a good idea to build and run the unit tests:

    	make check

4.  (Optional) You can also install sibcoind to your path:

    	make install

### Building Sibcoin GUI Wallet

In addition to the basic dependencies we installed with Homebrew, building the Sibcoin Wallet GUI requires Qt and `libqrencode`. The latter is optional
 (pass `--with-qrencode=no` option to `./configure` command to disable
it). To install `libqrencode` and associated headers:

    brew install qrencode

The recommended version of Qt for Mac builds is 5.6.1-1. Although 
Qt4 builds should work, they could result in a broken UI. Meanwhile, the versions of Qt5 5.7 and above require
C++11 (not yet supported by Sibcoin Core), and Qt5 5.6.2 has some other issues.
You can install the recommended version of Qt (5.6.1-1) using the following:

    brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/e6d954bab88e89c5582498157077756900865070/Formula/qt5.rb

This should link the Qt 5.6.1-1 package under `/usr/local/opt/qt5` (if this is
not the case, run `brew switch qt5 5.6.1-1`). To make sure you have the right
version of Qt in your path before configuring the Sibcoin build:

    export PATH="/usr/local/opt/qt5/bin:$PATH"
    export PKG_CONFIG_PATH="/usr/local/opt/qt5/lib/pkgconfig:$PKG_CONFIG_PATH"

Finally, configure and build:

    ./autogen.sh
    ./configure LDFLAGS="-L/usr/local/opt/qt5/lib -L/usr/local/opt/qrencode/lib" \
        CPPFLAGS="-I/usr/local/opt/qt5/include -I/usr/local/opt/qrencode/include" \
        --with-gui=qt5 --with-qrencode=yes
    make

### (Optional) Using Qt Creator as IDE

You can use Qt Creator as IDE, for debugging and for manipulating forms, etc.
Download Qt Creator from [www.qt.io/download/](https://www.qt.io/download/). The free Community
Edition is sufficient; you will need to install only Qt Creator (uncheck the rest during the installation
process).

1. Make sure you installed all prerequisites through Homebrew (see above)
2. Run `./configure --enable-debug`
3. In Qt Creator menu select `New Project > Import Project > Import Existing Project`
4. Type `sibcoin-qt` as project name, and specify `src/qt` as location
5. Leave the file selection as is
6. Confirm the "summary page"
7. Under the "Projects" tab select "Manage Kits..."
8. Select the default "Desktop" kit and select "Clang (x86 64bit in `/usr/bin`)" as compiler
9. Select LLDB as debugger (you may need to set the path to your installation)
10. Start debugging with Qt Creator

### (Optional) Creating a Release Build

You can ignore this section if you are building `sibcoind` for your own use.

Note: `sibcoind/sibcoin-cli` binaries are not included in the `Sibcoin-Qt.app` bundle.

If you are building `sibcoind` or `Sibcoin-Qt` for others, your build machine
should be set up as follows for maximum compatibility:

All dependencies should be compiled with these flags:

    -mmacosx-version-min=10.7
    -arch x86_64
    -isysroot $(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.7.sdk

Once dependencies are compiled, see
[doc/release-process.md](release-process.md) for how the Sibcoin Core bundle is
packaged and signed to create the .dmg disk image that is distributed.

## Running Sibcoin Daemon

### Setting Up

The Sibcoin Daemon available at `src/sibcoind` (with respect to package root). Before actually running it though, we have to create an RPC configuration file.

Run `sibcoind` to get the filename where we should place the file, or just try these commands:

    echo -e "rpcuser=sibcoinrpc\nrpcpassword=$(xxd -l 16 -p /dev/urandom)" \
        > "/Users/${USER}/Library/Application Support/Sibcoin/sibcoin.conf"
    chmod 600 "/Users/${USER}/Library/Application Support/Sibcoin/sibcoin.conf"

The next time you run the daemon, it will start downloading the blockchain, but it won't output anything while it's doing this. This process may take several hours, but you can monitor its progress by looking at the `debug.log` file:

    tail -f $HOME/Library/Application\ Support/Sibcoin/debug.log

### Useful Commands

    ./sibcoind -daemon    # to start the sibcoin daemon.
    ./sibcoin-cli --help  # for a list of command-line options.
    ./sibcoin-cli help    # When the daemon is running, to get a list of RPC commands
