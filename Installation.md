# Introduction #

This document will guide you through the installation of Pyrit and it's modules.

Pyrit compiles and runs on Linux, FreeBSD and MacOS. Windows is not (and probably never will be) supported; there are however some reports of successful installations on Windows with the help of [MinGW](http://www.mingw.org/).

Pyrit consists of basically two parts:

  * The main module features the commandline-client, the scheduling- and database-code and a basic extension-module that uses the CPU for computation. The main module is required for everyone...
  * There are currently two extension modules that add support for more advanced hardware. The extension modules for [Nvidia-CUDA](http://www.nvidia.com/object/cuda_home.html) and [OpenCL](http://www.khronos.org/opencl/) may be installed optionally and are used if available and supported by local hardware.

You can choose between OpenCL and CUDA if you have a compatible Nvidia-GPU; you may want to take a look at [this page](http://www.nvidia.com/object/cuda_learn_products.html) to find out if your hardware supports [Nvidia-CUDA](http://www.nvidia.com/object/cuda_home.html).
People with GPUs from ATI are supported through [AMD's OpenCL-implementation](http://developer.amd.com/gpu/AMDAPPSDK/Pages/default.aspx) and may find [this page](http://developer.amd.com/gpu/AMDAPPSDK/pages/DriverCompatibility.aspx) of interest; other possible OpenCL-platforms like [IBM's Cell B.E.](http://www-03.ibm.com/technology/cell/) (that powers the Playstation 3) should work but are untested at the moment.


# Compiling from sources #

Compiling from source-code is the preferred way of getting Pyrit onto your system. Linux users running a binary distribution may need to install the development packages for [Python](http://www.python.org) (e.g. _python-devel_), [OpenSSL](http://www.openssl.org/) (e.g. _openssl-devel_ or _libssl-dev_) and [Zlib](http://zlib.net/) (e.g. _zlib-devel_). You also need a C-compiler like _gcc_. Users of MacOS probably only need to have [XCode](http://developer.apple.com/TOOLS/Xcode/) installed.

From time to time Pyrit get's packed into (hopefully) stable packages. In general you should download, compile and install these source-code packages from the [Download](http://code.google.com/p/pyrit/downloads/list) area.
The more adventurous among you may instead want to try the latest source-code in Pyrit's repository. The code in [svn-trunk](http://pyrit.googlecode.com/svn/trunk/) may include more features and provide better performance but also may cause random problems or even not compile at all. Use the fixed packages when in doubt.

### Stable: Source-code from fixed packages ###

Download the source-code package for Pyrit and (optionally) a extension-module.

  * Pyrit (required): [Version 0.3.0](http://pyrit.googlecode.com/files/pyrit-0.3.0.tar.gz)
  * CPyrit-CUDA (optional, for Nvidia-hardware): [Version 0.3.0](http://pyrit.googlecode.com/files/cpyrit-cuda-0.3.0.tar.gz)
  * CPyrit-OpenCL (optional, for compatible hardware): [Version 0.3.0](http://pyrit.googlecode.com/files/cpyrit-opencl-0.3.0.tar.gz)

Now unpack the source-code into a new directory like this:
```
tar xvzf pyrit-0.3.0.tar.gz
tar xvzf cpyrit-cuda-0.3.0.tar.gz
```

Continue with the compiling as explained below.

### Adventurous: Source-code from svn-trunk ###

You need to install a _subversion_-client before you can use Pyrit's source-code repository; most Linux distributions provide a package for that. Do the initial checkout from svn-trunk like this:
```
svn checkout http://pyrit.googlecode.com/svn/trunk/ pyrit_svn
```

This will create a new directory _'pyrit\_svn'_ that holds all of Pyrit's latest source-code. Execute ` svn update ` inside that directory to keep track of changes.

### Compiling and installing ###

#### ... the main module ####

Switch to the main module's directory which should be 'Pyrit-0.2.4' (if you used a source-code package) or 'pyrit\_svn/pyrit' (if you're on svn). We use Python's _distutils_ to compile and install the code:

```
cd pyrit-0.3.0
python setup.py build
```

If everything went well and no errors are thrown at you, use _distutils_ again to install Pyrit:

```
sudo python setup.py install
```

You can now execute 'pyrit' from your commandline; leave the source-code's directory before doing so to prevent Python from getting confused with module-lookups.

#### ... support for Nvidia-CUDA ####

Get yourself a copy of the CUDA-**Toolkit** from http://www.nvidia.com/object/cuda_get.html. You need to modify either _$PATH_ and _ldconfig_ or setup.py if you choose not to install the Toolkit into either '/usr/local/cuda' or '/opt/cuda' so CPyrit-CUDA's installation routine can find Nvidia's compiler 'nvcc'. You also need to have Nvidia's proprietary hardware-drivers installed in the way that fits your OS.

Switch to the directory holding CPyrit-CUDA's source-code and compile and install it just like you did with Pyrit:

```
cd cpyrit-cuda-0.3.0
python setup.py build
sudo python setup.py install
```

Executing _'pyrit list\_cores'_ should list your GPUs. Please see [the troubleshooting-wiki](http://code.google.com/p/pyrit/wiki/Troubleshooting) if it doesn't work.

#### ... support for OpenCL ####

OpenCL is currently supported by Nvidia (_GeForce_ GPUs), AMD (_ATI Radeon_ GPUs and SSE3-capable CPUs) and IBM (_CELL B.E._ CPUs). You can get a copy of the SDKs that are required to build CPyrit-OpenCL from the following sites (registration required):

  * [Nvidia OpenCL SDK/Toolkit](http://developer.nvidia.com/object/opencl-download.html)
  * [ATI Stream SDK](http://developer.amd.com/gpu/ATIStreamSDK/Pages/default.aspx)


Please see the drivers' installation instruction for how to get everything up and running. The SDKs usually include simple demos and examples. First try to get those demos working and you'll most probably have no problems installing CPyrit-OpenCL.

Switch to the directory holding CPyrit-OpenCL's source-code and compile and install it just like you did with Pyrit:
```
cd cpyrit-opencl-0.3.0
python setup.py build
sudo python setup.py install
```

Executing _'pyrit list\_cores'_ should list your GPUs. Please see [the troubleshooting-wiki](http://code.google.com/p/pyrit/wiki/Troubleshooting) if it doesn't work

# Using binary packages #

Binary packages are not directly supported. The [Pentoo-](http://www.pentoo.ch/) and the [Backtrack4](http://www.remote-exploit.org/backtrack.html)-LiveCD include Pyrit as pre-build packages.