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
git apply --stat fix_session_include.patch
git apply --check fix_session_include.patch
```

Apply patch:

```
git am --signoff < fix_session_include.patch
```

## Create the package

Building reference on the (official webpage)[https://www.tensorflow.org/versions/master/get_started/os_setup.html#installing-from-sources].

Commands used:
```
bazel build -c opt //tensorflow/tools/pip_package:build_pip_package

$ bazel-bin/tensorflow/tools/pip_package/build_pip_package $(pwd)/tensorflow_pkg
```

## Install the package

```
pip install -U tensorflow_pkg/tensorflow-0.11.0rc1-py3-none-any.whl 
```

## Python VM

The VM is managed with *virtualenvwrapper* but was created with the following command
directly inside *.virtualenvs* folder:

```
pyvenv TensorFlow
```

