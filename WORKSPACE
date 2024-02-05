# Copyright 2023 Google LLC.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

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
    ignore_root_user_error = True,
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

http_archive(
    name = "federated-compute",
    sha256 = "95939a942202f893db391df1537fc64c45fe94db7ce6484f20bd436aee70054c",
    strip_prefix = "federated-compute-8d4b013673f87ccd35c9ec95a97749a5ff83f467",
    url = "https://github.com/google/federated-compute/archive/8d4b013673f87ccd35c9ec95a97749a5ff83f467.tar.gz",
)

load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")

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

http_archive(
    name = "oak",
    patches = [
        "//third_party/oak:BUILD.containers.patch",
    ],
    sha256 = "ee0337b88d6260897362a70da175a6931c4d319bbcd3b4dd7b65de20bcea336a",
    strip_prefix = "oak-704f81ed6310dfed448670b9fcdeb7ab6cfc78d1",
    url = "https://github.com/project-oak/oak/archive/704f81ed6310dfed448670b9fcdeb7ab6cfc78d1.tar.gz",
)

load("@oak//bazel:repositories.bzl", "oak_toolchain_repositories")

oak_toolchain_repositories()

http_archive(
    name = "googletest",
    sha256 = "8ad598c73ad796e0d8280b082cebd82a630d73e73cd3c70057938a6501bba5d7",
    strip_prefix = "googletest-1.14.0",
    urls = ["https://github.com/google/googletest/archive/refs/tags/v1.14.0.tar.gz"],
)

# TODO: Switch to using the official SQLite repo. This is used for convenience during prototyping since it's bazel-friendly.
SQLITE_BAZEL_COMMIT = "2a512f90dcdabfc7c3279cc324f1abd84af49911"

http_archive(
    name = "sqlite_bazel",
    patches = ["//third_party/sqlite_bazel:BUILD.patch"],
    sha256 = "079ae321db00b2a697dc8c27acc21e63e05c353b89bdb510bb6d6778d3a05866",
    strip_prefix = "sqlite-bazel-" + SQLITE_BAZEL_COMMIT,
    urls = ["https://github.com/rockwotj/sqlite-bazel/archive/%s.zip" % SQLITE_BAZEL_COMMIT],
)

http_archive(
    name = "com_google_absl",
    sha256 = "aabf6c57e3834f8dc3873a927f37eaf69975d4b28117fc7427dfb1c661542a87",
    strip_prefix = "abseil-cpp-98eb410c93ad059f9bba1bf43f5bb916fc92a5ea",
    urls = ["https://github.com/abseil/abseil-cpp/archive/98eb410c93ad059f9bba1bf43f5bb916fc92a5ea.zip"],
)

http_archive(
    name = "google_benchmark",
    sha256 = "6bc180a57d23d4d9515519f92b0c83d61b05b5bab188961f36ac7b06b0d9e9ce",
    strip_prefix = "benchmark-1.8.3",
    urls = ["https://github.com/google/benchmark/archive/refs/tags/v1.8.3.tar.gz"]
)

http_archive(
    name = "rules_oci",
    sha256 = "58b7a175ee90c12583afeca388523adf6a4e5a0528f330b41c302b91a4d6fc06",
    strip_prefix = "rules_oci-1.6.0",
    url = "https://github.com/bazel-contrib/rules_oci/releases/download/v1.6.0/rules_oci-v1.6.0.tar.gz",
)

load("@rules_oci//oci:dependencies.bzl", "rules_oci_dependencies")

rules_oci_dependencies()

load("@rules_oci//oci:repositories.bzl", "LATEST_CRANE_VERSION", "LATEST_ZOT_VERSION", "oci_register_toolchains")

oci_register_toolchains(
    name = "oci",
    crane_version = LATEST_CRANE_VERSION,
    zot_version = LATEST_ZOT_VERSION,
)

load("@rules_oci//oci:pull.bzl", "oci_pull")

oci_pull(
    name = "distroless_cc_debian12",
    digest = "sha256:6714977f9f02632c31377650c15d89a7efaebf43bab0f37c712c30fc01edb973",
    image = "gcr.io/distroless/cc-debian12",
    platforms = ["linux/amd64"],
)

# Install a hermetic GCC toolchain. This must be defined after rules_oci
# because it uses an older version of aspect_bazel_lib.
http_archive(
    name = "aspect_gcc_toolchain",
    sha256 = "3341394b1376fb96a87ac3ca01c582f7f18e7dc5e16e8cf40880a31dd7ac0e1e",
    strip_prefix = "gcc-toolchain-0.4.2",
    url = "https://github.com/aspect-build/gcc-toolchain/archive/refs/tags/0.4.2.tar.gz",
)

load("@aspect_gcc_toolchain//toolchain:repositories.bzl", "gcc_toolchain_dependencies")

gcc_toolchain_dependencies()

load("@aspect_gcc_toolchain//toolchain:defs.bzl", "ARCHS", "gcc_register_toolchain")

gcc_register_toolchain(
    name = "gcc_toolchain_x86_64",
    target_arch = ARCHS.x86_64,
)

# Stub out unneeded Java proto library rules used by various dependencies. This
# avoids needing to depend on a Java toolchain.
load("//:stub_repo.bzl", "stub_repo")

stub_repo(
    name = "io_grpc_grpc_java",
    rules = {":java_grpc_library.bzl": ["java_grpc_library"]},
)

stub_repo(
    name = "rules_java",
    rules = {"java:defs.bzl": ["java_lite_proto_library", "java_proto_library"]},
)
