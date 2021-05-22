---
title: 'Using Catch2 in a project built with Bazel'
date: 2020-10-10
categories: [programming]
summary: "We go through the steps to get Catch2 working on a C++ project that uses Bazel as build system."
tags: [bazel, catch2, c++, cpp]
---


### Long story short

**1. Make Catch2 available to the project.** Add the following lines to the `WORKSPACE` file:
```
# Load `http_archive`
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

# Catch2
http_archive(
    name = "com_github_catchorg_catch2",
    urls = ["https://github.com/catchorg/Catch2/archive/v2.13.1.tar.gz"],
    strip_prefix = "Catch2-2.13.1",
    sha256 = "36bcc9e6190923961be11e589d747e606515de95f10779e29853cfeae560bd6c",
)
```

**2. Create `main`.** For example, we can locate this in `<workspace_root>/test_utils`.

`<workspace_root>/test_utils/catch-main.cpp`:
```cpp
#define CATCH_CONFIG_MAIN
#include <catch2/catch.hpp>
```

`<workspace_root>/test_utils/BUILD`:
```
cc_library(
    name = "catch-main",
    srcs = ["catch-main.cpp"],
    deps = ["@com_github_catchorg_catch2//:catch2"],
    testonly = True,
    visibility = ["//visibility:public"],
)
```

Note that you can use the field `copts` to add compilation flags.

**3. Add tests**

In `<mytest_dir>/my_test.cpp`:
```cpp
#include <catch2/catch.hpp>
// Other include directives

TEST_CASE("Penguins are elegant in their clumsiness", "[goals]") {
    //...
}
```

In `<mytest_dir>/BUILD`:
```
cc_test(
    name = "my_test",
    srcs = ["my_test.cpp"],
    deps = [
        "@com_github_catchorg_catch2//:catch2",
        "//test_utils:catch-main",
        #...
    ],
)
```

### Long story not that short

First thing we need to do is to make Catch2 available to our project. We can use Bazel\'s `http_archive` to get a particular Catch2 release by adding the following lines to our project\'s `WORKSPACE` file:
```
# Load `http_archive`
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

# Catch2
http_archive(
    name = "com_github_catchorg_catch2",
    urls = ["https://github.com/catchorg/Catch2/archive/v2.13.1.tar.gz"],
    strip_prefix = "Catch2-2.13.1",
    sha256 = "36bcc9e6190923961be11e589d747e606515de95f10779e29853cfeae560bd6c",
)
```

- `name` contains the name by which we will refer to Catch2 in our `BUILD` files (we will see this in action in a bit).
- We are using `strip_prefix` in order to make the `#include` directive for Catch2 version independent. This could also be done by providing our own `BUILD.catch2` file for Catch2 and specifying paths in there. However that would make it more difficult to mantain. For example, when switching the version of Catch2 we want to use, we would need to make changes in `WORKSPACE` and in `BUILD.catch2`. Also, we would be describing how to build Catch2 in our own project, instead of using the build description that Catch2\' authors provide.
- The value in `sha256` must match the SHA-256 of the downloaded archive, so keep that in mind when changing Catch2's version in use.

If we run `bazel build //...` in our project's root directory (workspace), Bazel will inform us that Catch2 is being fetched.

We can now go ahead and create our tests. Following the recommendation in Catch2's documentation, it is best to separate the test's entry point from the test code. Whether we are fine with the `main()` Catch2 provides or we need our own, keeping it separate from our test code makes it easier to organize test in different files and executables (`cc_test`s in the case of Bazel).

For example, we could have a package called `test_utils` containing a `cc_library` for that entry point. So we would need

`<workspace_root>/test_utils/catch-main.cpp`:
```cpp
#define CATCH_CONFIG_MAIN
#include <catch2/catch.hpp>
```

and `<workspace_root>/test_utils/BUILD`:
```
cc_library(
    name = "catch-main",
    srcs = ["catch-main.cpp"],
    deps = ["@com_github_catchorg_catch2//:catch2"],
    testonly = True,
    visibility = ["//visibility:public"],
)
```

Since we used `strip_prefix` when asking Bazel to get Catch2 for us, we can include its header with `#include <catch2/catch.hpp>`. Note that using `#include "external/com_github_catchorg_catch2/single_include/catch2/catch.hpp`, while more verbose, would be more appropriate, since Catch2 is an external dependency fetched by Bazel and not a system installed library.

We set `testonly` to `True` because we want `catch-main` to only be used in test code. Those artifacts with `testonly` set to `True` can only be depended upon by other artifacts also marked as `testonly` artifacts. For example, by default, `cc_test` sets `testonly` to `True` but `cc_library` and `cc_executable` set it to `False` (which is why we are specifying it here for the `cc_library` `catch-main`). We can also use the `testonly` attribute when writing our own matchers or any code that should be used only in tests, such as mock classes.

Since we are filtering the usage of the library `catch-main` with the attribute `testonly`, we can safely set its visibility to `public`.

In case that some tests in the project require personalized entry points, we can add with the same approach `catch-main-<special-conditions>`.

This is another good point to check that the changes we made are correct by running `bazel build //...` or `bazel build //test_utils:catch-main`.


And we can finally add our tests. We do this by using Bazel's C++ test rule `cc_test`. If we write our test code in `<mytest_dir>/my_test.cpp`, something along the lines of
```cpp
#include <catch2/catch.hpp>
// Other include directives

TEST_CASE("Penguins are elegant in their clumsiness", "[goals]") {
    //...
}
```

then the declaration of the test for Bazel can be added to `<mytest_dir>/BUILD` as
```
cc_test(
    name = "my_test",
    srcs = ["my_test.cpp"],
    deps = [
        "@com_github_catchorg_catch2//:catch2",
        "//test_utils:catch-main",
        #...
    ],
)
```
As mentioned before, `cc_test` sets `testonly` to `True` by default, so we can depend on `//test_utils:catch-main`. And to check that it all works, we can run `bazel test //...`, which will build the test executable and any of the dependencies that require building and run the test.

When the tests fails, Bazel command line output will point to the log file containing the test output. When using VSCode, for example, if you running Bazel commands in the editor\'s terminal, `Ctrl + left click`ing on the path will open in VSCode the test log, and then you can have it side by side with the test code - it's a great way to work on fixing failing tests! Or, more often, fixing the code being tested.
