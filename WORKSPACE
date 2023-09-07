# Copyright 2023 The Confidential Federated Compute Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http:#www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

# Install a hermetic LLVM toolchain. LLVM was chosen somewhat arbitrarily; GCC
# should work as well.
http_archive(
    name = "com_grail_bazel_toolchain",
    sha256 = "d3d218287e76c0ad28bc579db711d1fa019fb0463634dfd944a1c2679ef9565b",
    strip_prefix = "bazel-toolchain-0.8",
    url = "https://github.com/grailbio/bazel-toolchain/archive/refs/tags/0.8.tar.gz",
)

load("@com_grail_bazel_toolchain//toolchain:deps.bzl", "bazel_toolchain_dependencies")

bazel_toolchain_dependencies()

load("@com_grail_bazel_toolchain//toolchain:rules.bzl", "llvm_toolchain")

llvm_toolchain(
    name = "llvm_toolchain",
    llvm_version = "14.0.0",
)

load("@llvm_toolchain//:toolchains.bzl", "llvm_register_toolchains")

llvm_register_toolchains()

# Pin a newer version of gRPC than the one provided by Tensorflow.
# 1.50.0 is used as it is the gRPC version used by FCP. It's not a requirement
# for the versions to stay in sync if we need a feature of a later gRPC version,
# but simplifies the configuration of the two projects if we can use the same
# version.
http_archive(
    name = "com_github_grpc_grpc",
    sha256 = "76900ab068da86378395a8e125b5cc43dfae671e09ff6462ddfef18676e2165a",
    strip_prefix = "grpc-1.50.0",
    urls = ["https://github.com/grpc/grpc/archive/refs/tags/v1.50.0.tar.gz"],
)

# TensorFlow pins an old version of upb that's compatible with their old
# version of gRPC, but not with the newer version we use. Pin the version that
# would be added by gRPC 1.50.0.
http_archive(
    name = "upb",
    sha256 = "017a7e8e4e842d01dba5dc8aa316323eee080cd1b75986a7d1f94d87220e6502",
    strip_prefix = "upb-e4635f223e7d36dfbea3b722a4ca4807a7e882e2",
    urls = [
        "https://storage.googleapis.com/grpc-bazel-mirror/github.com/protocolbuffers/upb/archive/e4635f223e7d36dfbea3b722a4ca4807a7e882e2.tar.gz",
        "https://github.com/protocolbuffers/upb/archive/e4635f223e7d36dfbea3b722a4ca4807a7e882e2.tar.gz",
    ],
)

# Initialize hermetic python prior to loading Tensorflow deps. See
# https://github.com/tensorflow/tensorflow/blob/v2.14.0-rc0/WORKSPACE#L6
http_archive(
    name = "rules_python",
    sha256 = "5868e73107a8e85d8f323806e60cad7283f34b32163ea6ff1020cf27abef6036",
    strip_prefix = "rules_python-0.25.0",
    url = "https://github.com/bazelbuild/rules_python/releases/download/0.25.0/rules_python-0.25.0.tar.gz",
)

load("@rules_python//python:repositories.bzl", "python_register_toolchains")

python_register_toolchains(
    name = "python",
    python_version = "3.10",
)

# Bazel Skylib is needed to load @python//:defs.bzl below.
http_archive(
    name = "bazel_skylib",
    sha256 = "74d544d96f4a5bb630d465ca8bbcfe231e3594e5aae57e1edbf17a6eb3ca2506",
    urls = [
        "https://storage.googleapis.com/mirror.tensorflow.org/github.com/bazelbuild/bazel-skylib/releases/download/1.3.0/bazel-skylib-1.3.0.tar.gz",
        "https://github.com/bazelbuild/bazel-skylib/releases/download/1.3.0/bazel-skylib-1.3.0.tar.gz",
    ],
)

load("@python//:defs.bzl", "interpreter")
load("@rules_python//python:pip.bzl", "package_annotation", "pip_parse")

# Create a central repo that knows about the dependencies needed from
# requirements.txt.
pip_parse(
    name = "pypi_deps",
    python_interpreter_target = interpreter,
    requirements_lock = "//tff_worker:requirements.txt",
)

# Load the starlark macro, which will define pypi dependencies.
load("@pypi_deps//:requirements.bzl", "install_deps")

# Call it to define repos for requirements.
install_deps()

# Use pre-release version of Tensorflow because it is compatible with hermetic
# Python.
# Tensorflow v2.14.0-rc0
http_archive(
    name = "org_tensorflow",
    sha256 = "b54cb7ac94a74bbab4ffc40e362d684e9b08b4a10a307022f24cb80706765367",
    strip_prefix = "tensorflow-2.14.0-rc0",
    urls = [
        "https://github.com/tensorflow/tensorflow/archive/refs/tags/v2.14.0-rc0.tar.gz",
    ],
)

# The following is copied from TensorFlow's own WORKSPACE, see
# https://github.com/tensorflow/tensorflow/blob/v2.14.0-rc0/WORKSPACE#L68
load("@org_tensorflow//tensorflow:workspace3.bzl", "tf_workspace3")

tf_workspace3()

load("@org_tensorflow//tensorflow:workspace2.bzl", "tf_workspace2")

tf_workspace2()

load("@org_tensorflow//tensorflow:workspace1.bzl", "tf_workspace1")

tf_workspace1(with_rules_cc = False)

load("@org_tensorflow//tensorflow:workspace0.bzl", "tf_workspace0")

tf_workspace0()

load("@com_github_grpc_grpc//bazel:grpc_python_deps.bzl", "grpc_python_deps")

grpc_python_deps()

load("@rules_proto//proto:repositories.bzl", "rules_proto_dependencies", "rules_proto_toolchains")

rules_proto_dependencies()

rules_proto_toolchains()

load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")

git_repository(
    name = "federated-compute",
    commit = "04653d706de4ea1206dbbb2aec49f4e012c837c4",
    remote = "https://github.com/google/federated-compute.git",
)

# We must build TFF protos from source as they are not included in the version
# of TFF released as a python package.
git_repository(
    name = "tensorflow-federated",
    # Note that we also depend on TFF from a pypi dependency in requirements.txt
    # If the version used here for protos is incompatible with the version used
    # for the rest of TFF, it could cause issues.
    commit = "4a52f23c5974fdcc8e295c9553dcdf36f33d26a7",
    patches = [
        "//third_party/tensorflow_federated:BUILD.patch",
    ],
    remote = "https://github.com/tensorflow/federated.git",
)