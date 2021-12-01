# Building for PyLith development

There are a few modifications to the usual process when using the installer to build for PyLith development.
There are several configure arguments relevant to using the installer for development:

`--with-pylith-repo=URL`
: Clone the PyLith source code from this URL. The URL corresponds to your fork of the PyLith repository. If you are accessing GitHub via SSH, the URL usually has the form `git@github.com:YOUR_GITHUB_USERNAME/pylith.git`. If you are accessing GitHub via HTTPS, the URL usually has the form `https://github.com/YOUR_GITHUB_USERNAME/pylith.git`. The default URL is the `geodynamics/pylith` repository, <https://github.com/geodynamics/pylith.git>.

`--with-pylith-git=BRANCH`
:  Build branch `BRANCH` of PyLith. If you are just starting development and have not created any feature branches, then use the `main` branch.

Other arguments commonly used when configuring for PyLith development include:

`--enable-swig`
: Build the SWIG library for generating the Python/C++ interface.

 `--enable-pcre`
 : Build the PCRE library for use by SWIG.

`--enable-debugging`
: Use debugging (generate debugging symbols and use low-level optimization) when building spatialdata, PETSc, and PyLith.

:::{tip}
We always have debugging turned on when we are developing.
We build the code in directories separate from the source code.
This makes it easy to build an optimized version in a different directory.
We use environment variables to set the paths to the desired build.
:::

## Setting up the environment

Prerequisites:

* git
* make
* fork of PyLith repository

We use the following directory structure:

```bash
pylith-developer/
├── pylith_installer-2.2.2-2 # source code for the installer
├── build
│   └── debug # top-level directory for building with debugging
└── dest
   ├── dependencies # directory where dependencies are installed by installer
   └── debug # directory where PyLith and other CIG code is installed by installer
```


```{code-block} console
---
caption: Setting up the installer for PyLith development.
---
# Set some variables to hold information about our installer setup.
# The PYLITH_REPO should be https://github.com/YOUR_GITHUB_USERNAME/pylith.git if using a personal
# access token or git@github.com:YOUR_GITHUB_USERNAME/pylith.git if using SSH.
$ PYLITH_DIR=$HOME/pylith-developer
$ PYLITH_REPO=git@github.com:YOUR_GITHUB_USERNAME/pylith.git
$ PYLITH_BRANCH=main

# Create a top-level directory for PyLith.
$ mkdir -p $PYLITH_DIR
$ cd $PYLITH_DIR

# Place the installer source code tarball in $PYLITH_DIR and then unpack the tarball.
$ tar -xf pylith_installer-2.2.2-2.tar.gz
```

## Linux

This configuration is for the Ubuntu 20.04 Linux distribution.
The setup is similar for other Linux distributions.

We install several packages in addition to those installed for a [user installation for Ubuntu 20.04](../configs/ubuntu.md).

```bash
apt-get install -y --no-install-recommends \
      uncrustify \
      gnupg2 \
      lcov \
      gdb \
      valgrind
```

We use `autopep8` and `uncrustify` to format the source code, `gdb` and `valgrind` for debugging, and `lcov` to measure code coverage of tests.

Configure the installer.

```bash
mkdir $PYLITH_DIR/build/debug
cd $PYLITH_DIR/build/debug
$PYLITH_DIR/pylith_installer-2.2.2-2/configure \
    --with-pylith-git=$PYLITH_BRANCH \
    --with-pylith-repo=$PYLITH_REPO \
    --enable-debugging \
    --with-fetch=curl \
    --with-make-threads=$(nproc) \
    --prefix=$PYLITH_DIR/dest/debug \
    --enable-swig \
    --enable-pcre \
```

Check the configure output to make sure it ran without errors and the configuration matches what you want.
Setup your environment by running `source setup.sh` (bash shell).
Just like in a user installation, you need to rerun `source setup.sh` in every terminal shell used for PyLith development, or transfer the environment settings to your `.bashrc`. 

First, we build the remaining dependencies.

```bash
cd $PYLITH_DIR/build/debug
make external_deps
```

Next, we configure and build pythia, nemesis, spatialdata, and PETSc.

```bash
cd $PYLITH_DIR/build/debug
make installed_pythia
make installed_nemesis
make installed_spatialdata
make installed_petsc
```

Finally, configure and build PyLith.

```bash
cd $PYLITH_DIR/build/debug
make installed_pylith
```

## macOS

:::{important}
You will need to install XCode or XCode command line tools before configuring the installer.
:::

This configuration is for Big Sur (11.6.x).
We use the Apple clang/clang++ compiler.
We use the installer to build autotools and Python 2.7.

Configure the installer.

```bash
${HOME}/src/pylith/pylith_installer-3.0.0-0/configure  \
    --with-pylith-git=$PYLITH_BRANCH \
    --with-pylith-repo=$PYLITH_REPO \
    --enable-debugging \
    --with-fetch=curl \
    --prefix=$PYLITH_DIR/dest/debug \
    --enable-force-install \
    --with-make-threads=4 \
    --with-fortran=no \
    --enable-autotools \
    --enable-mpi=mpich \
    --enable-openssl \
    --with-numpy-blaslapack=no \
    --enable-swig \
    --enable-pcre \
    --enable-cmake \
    CC=clang CXX=clang++
```

Check the configure output to make sure it ran without errors and the configuration matches what you want.
Setup your environment by running `source setup.sh` (bash shell).
Just like in a user installation, you need to rerun `source setup.sh` in every terminal shell used for PyLith development, or transfer the environment settings to your `.bashrc`. 

First, we build the dependencies.

```bash
cd $PYLITH_DIR/build/debug
make external_deps
```

Next, we configure and build pythia, nemesis, spatialdata, and PETSc.

```bash
cd $PYLITH_DIR/build/debug
make installed_pythia
make installed_nemesis
make installed_spatialdata
make installed_petsc
```

Finally, configure and build PyLith.

```bash
cd $PYLITH_DIR/build/debug
make installed_pylith
```
