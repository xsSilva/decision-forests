load("@org_tensorflow//tensorflow:tensorflow.bzl", "tf_gen_op_wrapper_py")
load("//tensorflow_decision_forests/tensorflow:utils.bzl", "rpath_linkopts_to_tensorflow","tf_custom_op_library_external")

load("//tensorflow_decision_forests/tensorflow:utils.bzl", "rpath_linkopts_to_tensorflow")

package(
    default_visibility = ["//visibility:public"],
    licenses = ["notice"],
)

# Libraries
# =========

# User-facing library to run models in TensorFlow.
py_library(
    name = "api_py",
    srcs = ["api.py"],
    data = [":inference.so"],
    srcs_version = "PY3",
    deps = [
        ":op_py",
        # absl/logging dep,
        # six dep,
        "@org_tensorflow//tensorflow:tensorflow_py",
        "@ydf//yggdrasil_decision_forests/dataset:data_spec_py_proto",
        "@ydf//yggdrasil_decision_forests/model:abstract_model_py_proto",
        "//tensorflow_decision_forests/tensorflow:check_version",
    ],
)

py_library(
    name = "test_utils",
    srcs = ["test_utils.py"],
    srcs_version = "PY3",
    deps = [
        # absl/logging dep,
        "@org_tensorflow//tensorflow/python",
        "//tensorflow_decision_forests/component/inspector:blob_sequence",
        "@ydf//yggdrasil_decision_forests/dataset:data_spec_py_proto",
        "@ydf//yggdrasil_decision_forests/model:abstract_model_py_proto",
        "@ydf//yggdrasil_decision_forests/model/decision_tree:decision_tree_py_proto",
        "@ydf//yggdrasil_decision_forests/model/gradient_boosted_trees:gradient_boosted_trees_py_proto",
        "@ydf//yggdrasil_decision_forests/model/random_forest:random_forest_py_proto",
        "@ydf//yggdrasil_decision_forests/utils:distribution_py_proto",
    ],
)

# TF Ops
# ======

# Op+kernel for dynamical linking.
tf_custom_op_library_external(
    name = "inference.so",
    deps = [":kernel_and_op"],
)

# Op+kernel for static linking.
cc_library(
    name = "kernel_and_op",
    deps = [
        ":kernel",
        ":op",
        "//tensorflow_decision_forests/tensorflow:canonical_models",
    ],
    alwayslink = 1,
)

alias(
    name = "op_py",
    actual = select({
        "//tensorflow_decision_forests:use_precompiled_wrappers": ":op_py_precompile",
        "//conditions:default": ":op_py_no_precompile",
    }),
)

# Python wrapper around the ops.
# This generates a python wrapper around the op.{so,dll}:
# bazel-genfiles/third_party/tensorflow_decision_forests/tensorflow/ops/inference/op.py
# This is the reason why tf needs to be entirely compiled in OSS setting.
tf_gen_op_wrapper_py(
    name = "op_py_no_precompile",
    out = "op.py",
    cc_linkopts = rpath_linkopts_to_tensorflow("op"),
    deps = [":op"],
)

# Pre-compiled version of "tf_gen_op_wrapper_py" above.
#
# Refresh file file with:
# bazel build -c opt //tensorflow_decision_forests/tensorflow/ops/inference:op_py_precompile
# cp bazel-genfiles/third_party/tensorflow_decision_forests/tensorflow/ops/inference/op.py third_party/tensorflow_decision_forests/tensorflow/ops/inference/op.py
py_library(
    name = "op_py_precompile",
    srcs = ["op.py"],
    srcs_version = "PY3",
    deps = [
        "@org_tensorflow//tensorflow/python",
    ],
)

# Declaration of the ops.
cc_library(
    name = "op",
    srcs = ["op.cc"],
    linkstatic = 1,
    deps = ["@org_tensorflow//tensorflow/core:framework_headers_lib"],
    alwayslink = 1,
)

# Definition of the ops i.e. the kernels.
cc_library(
    name = "kernel",
    srcs = ["kernel.cc"],
    deps = [
        "@com_google_absl//absl/status",
        "@com_google_absl//absl/strings",
        "@org_tensorflow//tensorflow/core:framework_headers_lib",
        "@ydf//yggdrasil_decision_forests/dataset:data_spec_cc_proto",
        "@ydf//yggdrasil_decision_forests/dataset:vertical_dataset",
        "@ydf//yggdrasil_decision_forests/model:abstract_model",
        "@ydf//yggdrasil_decision_forests/model:model_library",
        "@ydf//yggdrasil_decision_forests/utils:compatibility",
        "@ydf//yggdrasil_decision_forests/utils:distribution_cc_proto",
        "@ydf//yggdrasil_decision_forests/utils:status_macros",
        "@ydf//yggdrasil_decision_forests/utils:tensorflow",
    ],
    alwayslink = 1,
)

# Tests
# =====

py_test(
    name = "tf1_test",
    srcs = ["tf1_test.py"],
    data = [
        "@ydf//yggdrasil_decision_forests/test_data",
    ],
    python_version = "PY3",
    deps = [
        ":api_py",
        ":test_utils",
        # absl/flags dep,
        # absl/logging dep,
        # absl/testing:parameterized dep,
        "@org_tensorflow//tensorflow/python",
    ],
)

py_test(
    name = "tf2_test",
    srcs = ["tf2_test.py"],
    data = [
        "@ydf//yggdrasil_decision_forests/test_data",
    ],
    python_version = "PY3",
    deps = [
        ":api_py",
        ":test_utils",
        # absl/flags dep,
        # absl/logging dep,
        # absl/testing:parameterized dep,
        "@org_tensorflow//tensorflow/python",
    ],
)
