# Build on Linux

## Install Dependencies

Note **Ubuntu and Fedora Dockerfile with all dependencies are ready for quick testing.**

### Install from the pre-built binary package 

In short, if FreeCAD has been installed, all the runtime dependencies such OpenCASCADE, TBB, would have been installed. 

For example, on Ubuntu 18.04
```bash
# assume FreeCAD ppa has been added to your repository list
sudo add-apt-repository ppa:freecad-maintainers/freecad-stable
sudo apt update
sudo install freecad
sudo apt-get install libtbb2 libocct-foundation-7.3  libocct-data-exchange-7.3  libocct-modeling-data-7.3 libocct-modeling-algorithms-7.3  libocct-ocaf-7.3
# download the deb
sudo dpkg -i parallel-preprocessor*.deb
```

#### packages needed to build or develop

note: **package name are debian based**, see installation section for RedHat based operation systems

+ C++17 compiler is also needed. 
+ essential build tool: cmake, git, etc
+ `libtbb2-dev`: thread pool
+ OpenCASCADE (`libocct*-dev` and `occt-misc`) can be installed from freecad-daily PPA
  ubuntu has opencascade with dbgsym, but its version is too outdated. https://launchpad.net/ubuntu/+source/opencascade
+ `python3-dev`   pybind11 is now a submodule

#### Optional dependencies:
+ `boost`: if C++17 compiler is not available, `boost::any` and `boost::filesystem`
+ `freecad` or `freecad-daily`: to view result and test, using the freecad-python3
+ `doxygen`: for in-source documentation generation
+ `zlib` and zipiso++: for zipped stream IO: used by FreeCAD project, simulate java.zipfile API

Future dependencies:
+ `openmpi, libscotch, libhypre`, can be installed with fenics from Fenics PPA
+ `Qt` for GUI 3D viewer

If you follow the official guide and you can build FreeCAD-daily 0.19 (python3 is strongly recommended), you should get the OpenCASCADE 7.3 dev from their PPA repo.

There are some third-party libraries already integrated into the source code tree, see more details on [software design wiki page](./wiki/Design.md)

---

## Build from source

Tested compiler: g++ 7.x, g++8.x, clang 6 (needs CMake 3.13+ ) on ubuntu 18.04

To use clang tool chain, `-T llvm` cmake command option may be needed to specify the linker app: lld-link
```bash
# ubuntu 18.04, needs `-T llvm`
cmake -T llvm ..  -DPYTHON_EXECUTABLE:FILEPATH=$(which python3) -DCMAKE_BUILD_TYPE=Release

# ubuntu 20.04, clang++ is v10,  pybind11 must be v2.6 to work
CXX=clang++ cmake ..  -DPYTHON_EXECUTABLE:FILEPATH=$(which python3) -DCMAKE_BUILD_TYPE=Release

```

clang++ shares header files and libstdc++.so with G++

Error with pybind11 when compiled by clang++10, which has been fixed in latest pybind11 (v2.6).

```
/usr/include/pybind11/pybind11.h:1010:9: error: no matching function for call to 'operator delete'
        ::operator delete(p, s);
        ^~~~~~~~~~~~~~~~~
```

User can either sticks with g++ or upgrade pybind11 installation to fix this error. 

### Method 1: building on local Linux

####  Ubuntu 18.04 and 20.04, Debian 10

The dependency is similar as Ubuntu 18.04, except OpenCASCADE is not installed from FreeCAD PPA, but Ubuntu official repository (version 7.3.3). 

```bash
# this repo contains submodule:
# for user clone this repo, git should automatically download all submodules, 
# if not, or not sure, just run
git submodule update --init --recursive
# otherwise, you will be warmed by cmake to run this command

# install compulsory dependencies
sudo apt-get install git cmake doxygen ccache g++

# add freecad-stable or freecad-daily ppa (for ubuntu 18.04)
source /etc/lsb-release  # help to detect ubuntu version, 
# $(lsb_release -c -s)
if [ "$DISTRIB_CODENAME" == "bionic" ]; then
add-apt-repository ppa:freecad-maintainers/freecad-daily  &&  apt-get update
apt-get install -y freecad-daily-python3 
fi

if [ "$DISTRIB_CODENAME" == "focal" ]; then
echo "libocct 7.3 can be installed from official repo"
fi

if [ "$DISTRIB_CODENAME" == "buster" ]; then
echo "libocct 7.4 can be installed from official Debian repo"
fi

sudo apt-get install libocct*-dev occt*
sudo apt-get install python3-dev libtbb-dev

# libocct some module depends on Xwindows system, although
sudo apt-get install libx11-dev libxmu-dev libxi-dev

############ optional ######################
sudo apt-get install libboost-dev

# Qt interface,  see the gitlab CI scrypt for the most updated 

# pybind11 will be download if not found on system, there is no need to install
#if you can prefer a system-wide install, uncomment the following line
# sudo apt-get install python3-pybind11 pybind11-dev

# read zipped data folder,  cmake has FindZLIB, version 1.5, optional 
#sudo apt-get install zlib-dev  libzipiso++-dev

```


####  Fedora29+ and Centos8

Tested out on fedora 30, but it should work for fedora 29+ ,see [dockerfile for fedora](./Dockerfile_fedora)

Fedora 31 and 32 have OpenCASCADE in repository, there is no needs to compile from source.

1. install dependencies
```bash
#this script is extracted from dockerfile


# you must update before any yum install or add a repo
#fedora30 has the default python as python3
# `copr-cli` is needed to to add copr repo
yum install copr-cli -y && yum update -y 

# g++ 9.2 for f30 high enough, there is no need to install boost-devel
yum install g++ cmake make git doxygen -y

# python3
yum install python3 python3-devel  -y 

# optionally, install version 2.x of zipios++,  https://github.com/Zipios/Zipios
# not in use for the moment
# yum install zipios zipios-devel -y

############## optional ####################
# Qt5 is optional for GUI operation
yum install qt5-devel qt5-qtwebsockets-devel qt5-qtwebsockets -y
```

### Install OpenCASCADE 7.x

#### Option 1: Install opencascade 7.4 from package repository

For fedora 30+ since Jan 2020, there are freecad (python3) and opencascade 7.4 in repository to install

```bash
# opencascade 7.4 package  for fedora 30+ is available  
yum install opencascade-draw, opencascade-foundation,  opencascade-modeling,  opencascade-ocaf \
    opencascade-visualization opencascade-devel freecad -y
```

#### Option 2: Download the opencascade source code and compile from source.

 Compile opencascade if not available in package repository, e.g. centos 7/8,  see the "Dockerfile_centos" file for updated instructions.

To get the latest source code from [OCCT official website](https://www.opencascade.com/), you need register (free of charge). Registered user may setup public ssh key and get readonly access to the occt repo
`git clone -b V7_4_0p1 gitolite@git.dev.opencascade.org:occt occt`
<https://old.opencascade.com/doc/occt-7.4.0/overview/html/occt_dev_guides__git_guide.html>

To get the release source code, this can be downloaded by wget from a link 
`wget "http://git.dev.opencascade.org/gitweb/?p=occt.git;a=snapshot;h=V7_4_0p1;sf=tgz" `
"V7_4_0p1" is the tag version name, more version tags can be found on `http://git.dev.opencascade.org/gitweb/?p=occt.git`


```bash

wget "http://git.dev.opencascade.org/gitweb/?p=occt.git;a=snapshot;h=V7_4_0p1;sf=tgz" -O occt.tar.gz
tar -xzf occt.tar.gz
cd occt-*
mkdir build
cd build
cmake .. -DUSE_TBB=ON -DBUILD_MODULE_Draw=OFF
make -j$(nproc)
sudo make install
# by default install to the prefix: /usr/local/
```

### Compile parallel-preprocessor

CMake build system is employed to simplify cross-platform development

```bash
git clone https://github.com/UKAEA/parallel-preprocessor.git
git submodule update --init --recursive

#git clone (outside docker is easier for this non-public repo, to avoid trouble in authentication)

cd parallel-preprocessor
#
mkdir build
cd build
cmake .. -DPYTHON_EXECUTABLE:FILEPATH=$(which python3) -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
```


## Installation

### install by "sudo make install"

`sudo make install` to install to `/usr` or any  other defined by `CMAKE_INSTALL_PREFIX`
`sudo make uninstall` in the build folder to uninstall.

### Use after built the deb/rpm package
#### Generate deb/rpm package

After successfully build this software from source using cmake, `make package` will generate the platform binary package, deb or rpm (currently, only Ubuntu 18.04 and fedora 30+). see [./Packaging.md] for more details.

#### Install parallel processor package
Precompiled binary packages may be provided in the future.

Platform package rpm and deb has been generated in the build directory, install it.
`sudo rpm -Uhv parallel-preprocessor*`  on fedora/RedHat systems 
or `sudo dpkg -i parallel-preprocessor*` on debian/ubuntu

### Use without system-wide installation

Without installation, this software can be evaluated by running [geomPipeline.py path_to_geometry_file](./python/geomPipeline.py), after building from source.

Note: change directory to the folder containing geomPipeline.py is not necessary, if user has put full path of `parallel-preprocessor/build/bin/` folder into user path. For example, by editing PATH varialbe in `~/.bashrc` on Ubuntu, it will just work as installed program.

Just append the `bin` folder to user `path` in `~/.bashrc`, that is all.  you should be able to imprint geometry by `geomPipeline.py  input_geometry_path`.   And also append `build/lib` to `LD_LIBRARY_PATH`

This is the recommended way in this development stage, then it is easier to pull and build the latest source.  Super user privilege is needed.







