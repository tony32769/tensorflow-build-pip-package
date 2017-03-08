# tensorflow-build-pip-package

TensorFlow python package to install, builded from source with bazel.

## Build info

* TensorFlow version:  0.11
* Commit hash: "eaa9dde98d95f843ad1d3d0f5956693991372e4a"
* Bazel:  0.3.2-homebrew
* NO CUDA
* Python 3.5

## Apply patch

Check what will happen:

```
$ git apply --stat fix_session_include.patch
$ git apply --check fix_session_include.patch
```

Apply patch:

```
$ git am --signoff < fix_session_include.patch
```

Note: path is useful also if you need to compile a plugin for TensorFlow under linux.

## Create the package

Building reference on the (official webpage)[https://www.tensorflow.org/versions/master/get_started/os_setup.html#installing-from-sources].

Commands used:
```
##
# Configure with default values, no CUDA or OpenCL or Google Cloud
$ ./configure
##
# Linux config
$ bazel build --verbose_failures --local_resources 2048,.5,1.0 -c opt --copt=-mavx --config=cuda --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" //tensorflow/tools/pip_package:build_pip_package
##
# MacOs config
$ bazel build --verbose_failures --local_resources 2048,.5,1.0 -c opt --copt=-mavx2 --copt=-msse4.2 --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" //tensorflow/tools/pip_package:build_pip_package
##
# Create the package
$ bazel-bin/tensorflow/tools/pip_package/build_pip_package $(pwd)/tensorflow_pkg
```

Note:
 * *--verbose_failures*: added just in case of some error during the building.
 * *--local_resources 2048,.5,1.0*: limits the resource used by the building process.
 * *-s*: to output all bazel commands.
 * *--config=cuda*: add support for GPU computing with Nvidia CUDA (feature previous enabled with `./configure`)
 * *--copt=-mavx --copt=-mavx2 --copt=-mfma*: supported set of instructions for 2500k
 * *--copt=-mavx2 --copt=-msse4.2* --copt=-mfma: supported set of instructions for i5-4258U
 * *--cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0"*: force the compiler to use the same ABI of binary package (current version 0.12 is compiled with gcc4, so if you have gcc5 you need this option).

## Install the package

```
$ pip install -U tensorflow_pkg/tensorflow-0.11.0rc1-py3-none-any.whl 
```

## Python VM

The VM is managed with *virtualenvwrapper* but was created with the following command
directly inside *.virtualenvs* folder:

```
$ pyvenv TensorFlow
```

## Note about matplotlib

If you can't render without a screen you will have this error:

```bash
_tkinter.TclError: no display name and no $DISPLAY environment variable
```

To solve this error do this:

```bash
python -c "import matplotlib; print(matplotlib.matplotlib_fname())"
```

Modify *matplotlibrc* changing **backend      : tkagg** in **backend : Agg**.

## Build a new OP with source

Copy your C/C++ code inside the folder `tensorflow/core/user_ops`. If the file added is `example.cpp` you have
to add in the same folder a file named **BUILD** with this snippet of code:

```
load("//tensorflow:tensorflow.bzl", "tf_custom_op_library")

tf_custom_op_library(
    name = "example.so",
    srcs = ["example.cpp"],
)
```

Bazel will load all needed dependencies and will compile your *.so* lib.

```
$ bazel build -s --verbose_failures -c opt //tensorflow/core/user_ops:example.so
```

Copy the library where you want and load it from a Python script.

Note:

 * `error: cannot call constructor ‘tensorflow::TensorShape::TensorShape’ directly [-fpermissive]`: use directly TensorShape -> `TensorShape(...)`

## Compile Tensorflow CC libs

```bash
$ bazel build -c opt --config=cuda --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" //tensorflow:libtensorflow_cc.so
```

Note:
 * *--cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0"*: force the compiler to use the same ABI of binary package.
