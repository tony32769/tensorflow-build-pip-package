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
# Configure with default values, no CUDA or OpenCL or Google Cloud
$ ./configure
$ bazel build -s --verbose_failures --local_resources 2048,.5,1.0 -c opt //tensorflow/tools/pip_package:build_pip_package

$ bazel-bin/tensorflow/tools/pip_package/build_pip_package $(pwd)/tensorflow_pkg
```

Note:
 * *--verbose_failures*: added just in case of some error during the building.
 * *--local_resources 2048,.5,1.0*: limits the resource used by the building process.
 * *-s*: to output all bazel commands.

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
$ bazel build -s --verbose_failures -c opt //tensorflow/core/user_ops:DifferentialEvolutionOp.so
```

Copy the library where you want and load it from a Python script.

Note:

 * `error: cannot call constructor ‘tensorflow::TensorShape::TensorShape’ directly [-fpermissive]`: use directly TensorShape -> `TensorShape(...)`
