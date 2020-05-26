# Copyright 2017 The Bazel Authors. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

workspace(
    name = "build_bazel_rules_nodejs",
    managed_directories = {
        "@npm": ["node_modules"],
        "@npm_node_patches": ["packages/node-patches/node_modules"],
    },
)

load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")
load("//:index.bzl", "BAZEL_VERSION", "SUPPORTED_BAZEL_VERSIONS")

#
# Nested package workspaces required to build packages & reference rules
#

load("//packages:index.bzl", "NESTED_PACKAGES")

[local_repository(
    name = "npm_bazel_%s" % name,
    path = "packages/%s/src" % name,
) for name in NESTED_PACKAGES]

#
# Install rules_nodejs dev dependencies
#

load("//:package.bzl", "rules_nodejs_dev_dependencies")

rules_nodejs_dev_dependencies()

#
# Setup rules_nodejs npm dependencies
#

load("@build_bazel_rules_nodejs//:index.bzl", "npm_install", "yarn_install")

yarn_install(
    name = "npm",
    data = [
        "//:patches/jest-haste-map+25.3.0.patch",
        "//internal/npm_install/test:postinstall.js",
    ],
    environment = {
        "SOME_USER_ENV": "yarn is great!",
    },
    # The @npm//:node_modules_filegroup generated by manual_build_file_contents
    # is used in the //packages/typescript/test/reference_types_directive:tsconfig_types
    # test. For now we're still supporting node_modules as a filegroup tho this may
    # change in the future. The default generated //:node_modules target is a node_module_library
    # rule which provides NpmPackageInfo but that rule is not yet in the public API and we
    # have not yet dropped support for filegroup based node_modules target.
    manual_build_file_contents = """
filegroup(
  name = "node_modules_filegroup",
  srcs = [
    "//@types/hammerjs:hammerjs__files",
    "//@types/jasmine:jasmine__files",
    "//typescript:typescript__files",
  ],
)
""",
    package_json = "//:package.json",
    yarn_lock = "//:yarn.lock",
)

npm_install(
    name = "npm_node_patches",
    package_json = "//packages/node-patches:package.json",
    package_lock_json = "//packages/node-patches:package-lock.json",
)

npm_install(
    name = "angular_deps",
    package_json = "//packages/angular:package.json",
    package_lock_json = "//packages/angular:package-lock.json",
)

# Install all Bazel dependencies needed for npm packages that supply Bazel rules
load("@npm//:install_bazel_dependencies.bzl", "install_bazel_dependencies")

install_bazel_dependencies()

#
# Install npm_bazel_typescript dependencies
#

# We use git_repository since Renovate knows how to update it.
# With http_archive it only sees releases/download/*.tar.gz urls
git_repository(
    name = "build_bazel_rules_typescript",
    commit = "10a5a86885f95ab788fd841ade47b6a16e0c13d6",
    patches = [
        "//:rules_typescript_pr_494.patch",
        "//:rules_typescript_pr_496.patch",
    ],
    remote = "http://github.com/bazelbuild/rules_typescript.git",
    shallow_since = "1582757372 -0800",
)

# We have a source dependency on build_bazel_rules_typescript
# so we must repeat its transitive toolchain deps
load("@build_bazel_rules_typescript//:package.bzl", "rules_typescript_dev_dependencies")

rules_typescript_dev_dependencies()

load("@bazel_gazelle//:deps.bzl", "gazelle_dependencies")
load("@io_bazel_rules_go//go:deps.bzl", "go_register_toolchains", "go_rules_dependencies")

gazelle_dependencies()

go_rules_dependencies()

go_register_toolchains()

load("@build_bazel_rules_typescript//internal:ts_repositories.bzl", "ts_setup_dev_workspace")

ts_setup_dev_workspace()

load("@npm_bazel_typescript//internal:ts_repositories.bzl", "ts_setup_workspace")

ts_setup_workspace()

#
# Install npm_bazel_karma dependencies
#

load("@npm_bazel_karma//:package.bzl", "npm_bazel_karma_dependencies")

npm_bazel_karma_dependencies()

# Setup the rules_webtesting toolchain
load("@io_bazel_rules_webtesting//web:repositories.bzl", "web_test_repositories")

web_test_repositories()

load("@io_bazel_rules_webtesting//web/versioned:browsers-0.3.2.bzl", "browser_repositories")

browser_repositories(
    chromium = True,
    firefox = True,
)

#
# Dependencies to run stardoc & generating documentation
#

load("@io_bazel_rules_sass//sass:sass_repositories.bzl", "sass_repositories")

sass_repositories()

load("@io_bazel_stardoc//:setup.bzl", "stardoc_repositories")

stardoc_repositories()

load("@com_google_protobuf//:protobuf_deps.bzl", "protobuf_deps")

protobuf_deps()

load("@rules_pkg//:deps.bzl", "rules_pkg_dependencies")

rules_pkg_dependencies()

# Needed for starlark unit testing
load("@bazel_skylib//:workspace.bzl", "bazel_skylib_workspace")

bazel_skylib_workspace()

#
# Setup local respositories & install npm dependencies for tests
#

local_repository(
    name = "internal_npm_package_test_vendored_external",
    path = "internal/pkg_npm/test/vendored_external",
)

yarn_install(
    name = "fine_grained_deps_yarn",
    data = [
        "//internal/npm_install/test:postinstall.js",
    ],
    environment = {
        "SOME_USER_ENV": "yarn is great!",
    },
    included_files = [
        "",
        ".js",
        ".d.ts",
        ".json",
        ".proto",
    ],
    package_json = "//:tools/fine_grained_deps_yarn/package.json",
    symlink_node_modules = False,
    yarn_lock = "//:tools/fine_grained_deps_yarn/yarn.lock",
)

npm_install(
    name = "fine_grained_deps_npm",
    data = [
        "//internal/npm_install/test:postinstall.js",
    ],
    environment = {
        "SOME_USER_ENV": "npm is cool!",
    },
    included_files = [
        "",
        ".js",
        ".d.ts",
        ".json",
        ".proto",
    ],
    package_json = "//:tools/fine_grained_deps_npm/package.json",
    package_lock_json = "//:tools/fine_grained_deps_npm/package-lock.json",
    symlink_node_modules = False,
)

yarn_install(
    name = "fine_grained_no_bin",
    package_json = "//:tools/fine_grained_no_bin/package.json",
    symlink_node_modules = False,
    yarn_lock = "//:tools/fine_grained_no_bin/yarn.lock",
)

yarn_install(
    name = "fine_grained_goldens",
    included_files = [
        "",
        ".js",
        ".jst",
        ".ts",
        ".map",
        ".d.ts",
        ".json",
        ".proto",
    ],
    manual_build_file_contents = """
filegroup(
  name = "golden_files",
  srcs = [
    "//:BUILD.bazel",
    "//:install_bazel_dependencies.bzl",
    "//:manual_build_file_contents",
    "//:WORKSPACE",
    "//@angular/core:BUILD.bazel",
    "//@gregmagolan:BUILD.bazel",
    "//@gregmagolan/test-a/bin:BUILD.bazel",
    "//@gregmagolan/test-a:BUILD.bazel",
    "//@gregmagolan/test-a:index.bzl",
    "//@gregmagolan/test-b:BUILD.bazel",
    "//ajv:BUILD.bazel",
    "//jasmine/bin:BUILD.bazel",
    "//jasmine:BUILD.bazel",
    "//jasmine:index.bzl",
    "//rxjs:BUILD.bazel",
    "//unidiff:BUILD.bazel",
    "//zone.js:BUILD.bazel",
  ],
)""",
    package_json = "//:tools/fine_grained_goldens/package.json",
    symlink_node_modules = False,
    yarn_lock = "//:tools/fine_grained_goldens/yarn.lock",
)

#
# RBE configuration
#

load("@bazel_toolchains//rules:rbe_repo.bzl", "rbe_autoconfig")

# Creates toolchain configuration for remote execution with BuildKite CI
# for rbe_ubuntu1604
rbe_autoconfig(
    name = "buildkite_config",
)

rbe_autoconfig(
    name = "rbe_default",
    bazel_version = BAZEL_VERSION,
)

load("@build_bazel_integration_testing//tools:repositories.bzl", "bazel_binaries")

# Depend on the Bazel binaries
bazel_binaries(versions = SUPPORTED_BAZEL_VERSIONS)

# Install labs dependencies
load("@npm_bazel_labs//:package.bzl", "npm_bazel_labs_dependencies")

npm_bazel_labs_dependencies()

load("@rules_proto//proto:repositories.bzl", "rules_proto_dependencies", "rules_proto_toolchains")

rules_proto_dependencies()

rules_proto_toolchains()
