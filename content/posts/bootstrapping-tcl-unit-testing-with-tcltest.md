---
title: "Bootstrapping Tcl Unit Testing With Tcltest"
date: 2016-07-01
aliases:
  - /2016/07/01/bootstrapping-tcl-unit-testing-with-tcltest
---

If you are working with Tcl, and interested in adding unit tests to your project using tcltest, read on.

Usually there is some bootstrap work required to run your unit tests.
For example, every test should load the relevant libraries before using them in a test. There is also a need in a test runner that would execute the test suite and report results.
Since there are several ways to implement the above, and there are some gotchas along the way, I thought it would be useful to provide a reference implementation.

You can get the code for this blog post from a GitHub [gist](https://gist.github.com/deedd951add38ade5e09cba561864acb.git). The dashes ("-") in file names below symbolize path separators (e.g. "/" on Unix/Linux). I used this approach to make the gist structure flat and easy to follow.
Below is an overview of the files with some explanation about the implementation. The assumption is that you already familiar with [tcltest](http://www.tcl.tk/man/tcl/TclCmd/tcltest.htm) and [[incr Tcl]](http://incrtcl.sourceforge.net/itcl/).

**bootstrap.sh**

Please run this file once you clone the source code – it will create the intended directory structure from the flat files.
Then you can execute `tests/run` to see the DataManager test pass.

**tests-run** (i.e. `tests/run`)

This file sets `PROJECT_ROOT` environment variable pointing to the root of the project. The variable is used in `::tcltest::loadScript` to source all project libraries needed for unit tests execution. Then all unit tests under `$PROJECT_ROOT/tests` are discovered recursively and executed. It is possible to pass tcltest configuration arguments to `tests/run` to add or override existing options.

**tests-database-DataManager.test** (i.e. `tests/database/DataManager.test`)

An example unit test for `DataManager` class.

**database-DataManager.itcl** (i.e. `database/DataManager.itcl`)

An example `DataManager` class.

**Conclusion**

This implementation provides a good balance IMHO between the ability to run all tests in parallel, keeping a single main entry point for running the tests (as opposed to having multiple duplicate `all.tcl` files in each test directory), being able to run a single or selected tests (by providing `::tcltest::configure` options to `tests/run`) and keeping the `*.test` files lean in terms of locating and sourcing the libraries.
