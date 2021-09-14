<!--
# Copyright (c) 2021, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
-->

[![License](https://img.shields.io/badge/License-BSD3-lightgrey.svg)](https://opensource.org/licenses/BSD-3-Clause)

# Triton Client Libraries and Examples

To simplify communication with Triton, the Triton project provides
several client libraries and examples of how to use those
libraries. Ask questions or report problems in the main Triton [issues
page](https://github.com/triton-inference-server/server/issues).

The provided client libaries are:

* [C++ and Python APIs](#client-library-apis) that make it easy to
  communicate with Triton from your C++ or Python application. Using
  these libraries you can send either HTTP/REST or GRPC requests to
  Triton to access all its capabilities: inferencing, status and
  health, statistics and metrics, model repository management,
  etc. These libraries also support using system and CUDA shared
  memory for passing inputs to and receiving outputs from Triton.

* [Java API](#client-library-apis) (contributed by Alibaba Cloud PAI Team)
  that makes it easy to communicate with Triton from your Java application
  using HTTP/REST requests. For now, only a limited feature subset is supported.

* The [protoc
  compiler](https://developers.google.com/protocol-buffers/docs/tutorials)
  can generate a GRPC API in a large number of programming
  languages. See [src/grpc_generated/go](src/grpc_generated/go) for an
  example for the [Go programming language](https://golang.org/). See
  [src/grpc_generated/java](src/grpc_generated/java) for an example
  for the Java and Scala programming languages.

There are also many example applications that show how to use these
libraries. Many of these examples use models from the [example model
repository](https://github.com/triton-inference-server/server/blob/master/docs/quickstart.md#create-a-model-repository).

* C++ and Python versions of *image_client*, an example application
  that uses the C++ or Python client library to execute image
  classification models on Triton. See [Image Classification
  Example](#image-classification-example).

* Several simple [C++ examples](src/c%2B%2B/examples) show
  how to use the C++ library to communicate with Triton to perform
  inferencing and other task. The C++ examples demonstrating the
  HTTP/REST client are named with a *simple_http_* prefix and the
  examples demonstrating the GRPC client are named with a
  *simple_grpc_* prefix. See [Simple Example
  Applications](#simple-example-applications).

* Several simple [Python examples](src/python/examples)
  show how to use the Python library to communicate with Triton to
  perform inferencing and other task. The Python examples
  demonstrating the HTTP/REST client are named with a *simple_http_*
  prefix and the examples demonstrating the GRPC client are named with
  a *simple_grpc_* prefix. See [Simple Example
  Applications](#simple-example-applications).

* Several simple [Java
  examples](src/java/src/main/java/triton/client/examples) show how to
  use the Java API to communicate with Triton to perform inferencing
  and other task.

* A couple of [Python examples that communicate with Triton using a
  Python GRPC API](src/python/examples) generated by the
  [protoc compiler](https://grpc.io/docs/guides/). *grpc_client.py* is
  a simple example that shows simple API
  usage. *grpc_image_client.py* is functionally equivalent to
  *image_client* but that uses a generated GRPC client stub to
  communicate with Triton.

## Getting the Client Libraries And Examples

The easiest way to get the Python client library is to [use pip to
install the tritonclient
module](#download-using-python-package-installer-pip). You can also
download the C++, Python and Java client libraries from [Triton GitHub
release](#download-from-github), or [download a pre-built Docker image
containing the client libraries](#download-docker-image-from-ngc) from
[NVIDIA GPU Cloud (NGC)](https://ngc.nvidia.com).

It is also possible to build build the client libraries with
[cmake](#build-using-cmake).

### Download Using Python Package Installer (pip)

The GRPC and HTTP client libraries are available as a Python package
that can be installed using a recent version of pip.

```
$ pip install tritonclient[all]
```

Using *all* installs both the HTTP/REST and GRPC client
libraries. There are two optional packages available, *grpc* and
*http* that can be used to install support specifically for the
protocol. For example, to install only the HTTP/REST client library
use,

```
$ pip install tritonclient[http]
```

The components of the install packages are:

* http
* grpc [ `service_pb2`, `service_pb2_grpc`, `model_config_pb2` ]
* utils [ linux distribution will include `shared_memory` and `cuda_shared_memory`]

The Linux version of the package also includes the
[perf_analyzer](https://github.com/triton-inference-server/server/blob/master/docs/perf_analyzer.md)
binary. The perf_analyzer binary is built on Ubuntu 20.04 and may not
run on other Linux distributions. To run the perf_analyzer the
following dependency must be installed:

```bash
sudo apt update
sudo apt install libb64-dev
```

To reiterate, the installation on windows will not include perf_analyzer
nor shared_memory/cuda_shared_memory components.

### Download From GitHub

The client libraries and the perf_analyzer executable can be
downloaded from the [Triton GitHub release
page](https://github.com/triton-inference-server/server/releases)
corresponding to the release you are interested in. The client
libraries are found in the "Assets" section of the release page in a
tar file named after the version of the release and the OS, for
example, v2.3.0_ubuntu2004.clients.tar.gz.

The pre-built libraries can be used on the corresponding host system
or you can install them into the Triton container to have both the
clients and server in the same container.

```bash
$ mkdir clients
$ cd clients
$ wget https://github.com/triton-inference-server/server/releases/download/<tarfile_path>
$ tar xzf <tarfile_name>
```

After installing, the libraries can be found in lib/, the headers in
include/, the Python wheel files in python/, and the jar files in
java/.  The bin/ and python/ directories contain the built examples
that you can learn more about below.

The perf_analyzer binary is built on Ubuntu 20.04 and may not run on
other Linux distributions. To use the C++ libraries or perf_analyzer
executable you must install some dependencies.

```bash
$ apt-get update
$ apt-get install curl libcurl4-openssl-dev libb64-dev
```

### Download Docker Image From NGC

A Docker image containing the client libraries and examples is
available from [NVIDIA GPU Cloud
(NGC)](https://ngc.nvidia.com). Before attempting to pull the
container ensure you have access to NGC.  For step-by-step
instructions, see the [NGC Getting Started
Guide](http://docs.nvidia.com/ngc/ngc-getting-started-guide/index.html).

Use docker pull to get the client libraries and examples container
from NGC.

```bash
$ docker pull nvcr.io/nvidia/tritonserver:<xx.yy>-py3-sdk
```

Where \<xx.yy\> is the version that you want to pull. Within the
container the client libraries are in /workspace/install/lib, the
corresponding headers in /workspace/install/include, and the Python
wheel files in /workspace/install/python. The image will also contain
the built client examples.

### Build Using CMake

The client library build is performed using CMake. To build the client
libraries and examples with all features, first change directory to
the root of this repo and checkout the release version of the branch
that you want to build (or the *main* branch if you want to build the
under-development version).

```bash
$ git checkout main
```

Building on Windows vs. non-Windows requires different invocations
because Triton on Windows does not yet support all the build options.

#### Non-Windows

Use *cmake* to configure the build.

```
$ mkdir build
$ cd build
$ cmake -DCMAKE_INSTALL_PREFIX=`pwd`/install -DTRITON_ENABLE_CC_HTTP=ON -DTRITON_ENABLE_CC_GRPC=ON -DTRITON_ENABLE_PERF_ANALYZER=ON -DTRITON_ENABLE_PYTHON_HTTP=ON -DTRITON_ENABLE_PYTHON_GRPC=ON -DTRITON_ENABLE_JAVA_HTTP=ON -DTRITON_ENABLE_GPU=ON -DTRITON_ENABLE_EXAMPLES=ON -DTRITON_ENABLE_TESTS=ON ..
```

Then use *make* to build the clients and examples.

```
$ make cc-clients python-clients java-clients
```

When the build completes the libraries and examples can be found in
the install directory.

#### Windows

To build the clients you must install an appropriate C++ compiler and
other dependencies required for the build. The easiest way to do this
is to create the [Windows min Docker
image](https://github.com/triton-inference-server/server/blob/master/docs/build.md#windows-10-min-container)
and the perform the build within a container launched from that image.

```
> docker run  -it --rm win10-py3-min powershell
```

It is not necessary to use Docker or the win10-py3-min container for
the build, but if you do not you must install the appropriate
dependencies onto your host system.

Next use *cmake* to configure the build. If you are not building
within the win10-py3-min container then you will likely need to adjust
the CMAKE_TOOLCHAIN_FILE location in the following command.

```
$ mkdir build
$ cd build
$ cmake -DVCPKG_TARGET_TRIPLET=x64-windows -DCMAKE_TOOLCHAIN_FILE='/vcpkg/scripts/buildsystems/vcpkg.cmake' -DCMAKE_INSTALL_PREFIX=install -DTRITON_ENABLE_CC_GRPC=ON -DTRITON_ENABLE_PYTHON_GRPC=ON -DTRITON_ENABLE_GPU=OFF -DTRITON_ENABLE_EXAMPLES=ON -DTRITON_ENABLE_TESTS=ON ..
```

Then use msbuild.exe to build.

```
$ msbuild.exe cc-clients.vcxproj -p:Configuration=Release -clp:ErrorsOnly
$ msbuild.exe python-clients.vcxproj -p:Configuration=Release -clp:ErrorsOnly
```

When the build completes the libraries and examples can be found in
the install directory.

## Client Library APIs

The C++ client API exposes a class-based interface. The commented
interface is available in
[grpc_client.h](src/c%2B%2B/library/grpc_client.h),
[http_client.h](src/c%2B%2B/library/http_client.h),
[common.h](src/c%2B%2B/library/common.h).

The Python client API provides similar capabilities as the C++
API. The commented interface is available in
[grpc](src/python/library/tritonclient/grpc/__init__.py)
and
[http](src/python/library/tritonclient/http/__init__.py).

The Java client API provides similar capabilities as the Python API
with similar classes and methods.  For more information please refer
to the [Java client directory](src/java).

### GRPC Options

#### SSL/TLS

The client library allows communication across a secured channel using gRPC protocol.

For C++ client, see `SslOptions` struct that encapsulates these options in [grpc_client.h](src/c%2B%2B/library/grpc_client.h).

For Python client, look for the following options in [grpc/__init__.py](src/python/library/tritonclient/grpc/__init__.py):

* ssl
* root_certificates
* private_key
* certificate_chain

The [C++](src/c%2B%2B/examples/simple_grpc_infer_client.cc) and [Python](src/python/examples/simple_grpc_infer_client.py) examples
demonstrates how to use SSL/TLS settings on client side. For information on the corresponding server-side parameters, refer to the 
[server documentation](https://github.com/triton-inference-server/server/blob/main/docs/inference_protocols.md#ssltls)

### Compression

The client library also exposes options to use on-wire compression for gRPC transactions. 

For C++ client, see `compression_algorithm` parameter in the `Infer`, `AsyncInfer` and `StartStream` functions in [grpc_client.h](src/c%2B%2B/library/grpc_client.h). By default, the parameter is set as `GRPC_COMPRESS_NONE`.

Similarly, for Python client, see `compression_algorithm` parameter in `infer`, `async_infer` and `start_stream` functions in [grpc/__init__.py](src/python/library/tritonclient/grpc/__init__.py):

The [C++](src/c%2B%2B/examples/simple_grpc_infer_client.cc) and [Python](src/python/examples/simple_grpc_infer_client.py) examples demonstrates how to configure compression for clients. For information on the corresponding server-side parameters, refer to the [server documentation](https://github.com/triton-inference-server/server/blob/main/docs/inference_protocols.md#compression)

#### GRPC KeepAlive

Triton exposes GRPC KeepAlive parameters with the default values for both
client and server described [here](https://github.com/grpc/grpc/blob/master/doc/keepalive.md).

You can find a `KeepAliveOptions` struct/class that encapsulates these
parameters in both the [C++](src/c%2B%2B/library/grpc_client.h) and
[Python](src/python/library/tritonclient/grpc/__init__.py) client libraries.

There is also a [C++](src/c%2B%2B/examples/simple_grpc_keepalive_client.cc) and
[Python](src/python/examples/simple_grpc_keepalive_client.py) example
demonstrating how to setup these parameters on the client-side. For information
on the corresponding server-side parameters, refer to the 
[server documentation](https://github.com/triton-inference-server/server/blob/main/docs/inference_protocols.md#grpc-keepalive)

## Simple Example Applications

This section describes several of the simple example applications and
the features that they illustrate.

### Bytes/String Datatype

Some frameworks support tensors where each element in the tensor is
variable-length binary data. Each element can hold a string or an
arbitrary sequence of bytes. On the client this datatype is BYTES (see
[Datatypes](https://github.com/triton-inference-server/server/blob/master/docs/model_configuration.md#datatypes)
for information on supported datatypes).

The Python client library uses numpy to represent input and output
tensors. For BYTES tensors the dtype of the numpy array should be
'np.object_' as shown in the examples. For backwards compatibility
with previous versions of the client library, 'np.bytes_' can also be
used for BYTES tensors. However, using 'np.bytes_' is not recommended
because using this dtype will cause numpy to remove all trailing zeros
from each array element. As a result, binary sequences ending in
zero(s) will not be represented correctly.

BYTES tensors are demonstrated in the C++ example applications
simple_http_string_infer_client.cc and
simple_grpc_string_infer_client.cc.  String tensors are demonstrated
in the Python example application simple_http_string_infer_client.py
and simple_grpc_string_infer_client.py.

### System Shared Memory

Using system shared memory to communicate tensors between the client
library and Triton can significantly improve performance in some
cases.

Using system shared memory is demonstrated in the C++ example
applications simple_http_shm_client.cc and simple_grpc_shm_client.cc.
Using system shared memory is demonstrated in the Python example
application simple_http_shm_client.py and simple_grpc_shm_client.py.

Python does not have a standard way of allocating and accessing shared
memory so as an example a simple [system shared memory
module](src/python/library/tritonclient/utils/shared_memory)
is provided that can be used with the Python client library to create,
set and destroy system shared memory.

### CUDA Shared Memory

Using CUDA shared memory to communicate tensors between the client
library and Triton can significantly improve performance in some
cases.

Using CUDA shared memory is demonstrated in the C++ example
applications simple_http_cudashm_client.cc and
simple_grpc_cudashm_client.cc.  Using CUDA shared memory is
demonstrated in the Python example application
simple_http_cudashm_client.py and simple_grpc_cudashm_client.py.

Python does not have a standard way of allocating and accessing shared
memory so as an example a simple [CUDA shared memory
module](src/python/library/tritonclient/utils/cuda_shared_memory)
is provided that can be used with the Python client library to create,
set and destroy CUDA shared memory.

### Client API for Stateful Models

When performing inference using a [stateful
model](https://github.com/triton-inference-server/server/blob/master/docs/architecture.md#stateful-models),
a client must identify which inference requests belong to the same
sequence and also when a sequence starts and ends.

Each sequence is identified with a sequence ID that is provided when
an inference request is made. It is up to the clients to create a
unique sequence ID. For each sequence the first inference request
should be marked as the start of the sequence and the last inference
requests should be marked as the end of the sequence.

The use of sequence ID and start and end flags are demonstrated in the
C++ example applications simple_http_sequence_stream_infer_client.cc
and simple_grpc_sequence_stream_infer_client.cc.  The use of sequence
ID and start and end flags are demonstrated in the Python example
application simple_http_sequence_stream_infer_client.py and
simple_grpc_sequence_stream_infer_client.py.

## Image Classification Example

The image classification example that uses the C++ client API is
available at
[src/c++/examples/image_client.cc](src/c%2B%2B/examples/image_client.cc). The
Python version of the image classification client is available at
[src/python/examples/image_client.py](src/python/examples/image_client.py).

To use image_client (or image_client.py) you must first have a running
Triton that is serving one or more image classification models. The
image_client application requires that the model have a single image
input and produce a single classification output. If you don't have a
model repository with image classification models see
[QuickStart](https://github.com/triton-inference-server/server/blob/master/docs/quickstart.md)
for instructions on how to create one.

Once Triton is running you can use the image_client application to
send inference requests. You can specify a single image or a directory
holding images. Here we send a request for the inception_graphdef
model for an image from the
[qa/images](https://github.com/triton-inference-server/server/tree/master/qa/images).

```bash
$ image_client -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
```

The Python version of the application accepts the same command-line
arguments.

```bash
$ python image_client.py -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
     0.826384 (505) = COFFEE MUG
```

The image_client and image_client.py applications use the client
libraries to talk to Triton. By default image_client instructs the
client library to use HTTP/REST protocol, but you can use the GRPC
protocol by providing the -i flag. You must also use the -u flag to
point at the GRPC endpoint on Triton.

```bash
$ image_client -i grpc -u localhost:8001 -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
```

By default the client prints the most probable classification for the
image. Use the -c flag to see more classifications.

```bash
$ image_client -m inception_graphdef -s INCEPTION -c 3 qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
```

The -b flag allows you to send a batch of images for inferencing.
The image_client application will form the batch from the image or
images that you specified. If the batch is bigger than the number of
images then image_client will just repeat the images to fill the
batch.

```bash
$ image_client -m inception_graphdef -s INCEPTION -c 3 -b 2 qa/images/mug.jpg
Request 0, batch size 2
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
```

Provide a directory instead of a single image to perform inferencing
on all images in the directory.

```
$ image_client -m inception_graphdef -s INCEPTION -c 3 -b 2 qa/images
Request 0, batch size 2
Image '/opt/tritonserver/qa/images/car.jpg':
    0.819196 (818) = SPORTS CAR
    0.033457 (437) = BEACH WAGON
    0.031232 (480) = CAR WHEEL
Image '/opt/tritonserver/qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
Request 1, batch size 2
Image '/opt/tritonserver/qa/images/vulture.jpeg':
    0.977632 (24) = VULTURE
    0.000613 (9) = HEN
    0.000560 (137) = EUROPEAN GALLINULE
Image '/opt/tritonserver/qa/images/car.jpg':
    0.819196 (818) = SPORTS CAR
    0.033457 (437) = BEACH WAGON
    0.031232 (480) = CAR WHEEL
```

The [grpc_image_client.py](src/python/examples/grpc_image_client.py)
application behaves the same as the image_client except that instead
of using the client library it uses the GRPC generated library to
communicate with Triton.

## Ensemble Image Classification Example Application

In comparison to the image classification example above, this example
uses an ensemble of an image-preprocessing model implemented as a
[DALI
backend](https://github.com/triton-inference-server/dali_backend) and
a TensorFlow Inception model. The ensemble model allows you to send
the raw image binaries in the request and receive classification
results without preprocessing the images on the client.

To try this example you should follow the [DALI ensemble example
instructions](https://github.com/triton-inference-server/dali_backend/tree/main/docs/examples/inception_ensemble).
