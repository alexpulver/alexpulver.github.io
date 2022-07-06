---
title: "Using Couchbase Python SDK on Cloud Foundry"
date: 2016-06-09
aliases:
  - /2016/06/09/using-couchbase-python-sdk-on-cloud-foundry
---

I wanted to use Couchbase as a session store for my app instances, and specifically Couchbase Python SDK. The problem I faced was that Couchbase Python SDK (and some of Couchbase other non-C SDKs) requires Couchbase C SDK to be installed a priori – it is not installed by pip automatically.

Sadly, [Conda](http://conda.pydata.org/docs/) package manager doesn’t have Couchbase C SDK in its repository, so I could not use Cloud Foundry [Python Conda buildpack](https://github.com/ihuston/python-conda-buildpack/) to install it (it can be very useful to install SciPy for example, which is supported by Conda).
I wanted to avoid bundling of Couchbase C SDK in my Git repository (binary files ❌), so set out to look for a way of installing Couchbase C SDK dynamically during application push to Cloud Foundry.
Cloud Foundry documentation and Google were not so kind to me this time 😢 – it took a lot of poking around, but eventually I found a nifty feature in Cloud Foundry Python buildpack source code – **[pre and post compile](https://github.com/cloudfoundry/python-buildpack/tree/master/bin/steps/hooks/) hooks**!

The eventual solution was combined of several things (code snippets below):
* Adding `bin/precompile` script to my project, and using it to clone Couchbase C SDK code from official repository on GitHub, compile and install it. The clone was required because Couchbase `configure.pl` script runs some Git commands, and produces error messages unless Git directory is present. Also, since the `bin/precompile` script doesn’t run under root account, the library had to be installed into a non-default writable area – which I found to be `/app` (link to `/home/vcap/app`, just without the hardcoded username).
* Specifying the non-default location of the Couchbase C SDK in `requirements.txt`, so Couchbase Python SDK C library will compile successfully.
* Setting `LD_LIBRARY_PATH` in `manifest.yml` so that Couchbase C SDK will be found during runtime after deployment of the application. If you are curious – environment variable values specified in `manifest.yml` are being appended to whatever was defined by the system, and do not override it (which is a good thing).

**Note:** If you don’t feel like compiling the Couchbase C SDK dynamically each time, you can compile it locally (make sure it’s binary compatible with Cloud Foundry container). Then local installation can be compressed and stored somewhere, for example in Amazon Simple Storage Service (Amazon S3), and retrieved dynamically during the deployment.

**bin/pre_compile**

```bash
#!/usr/bin/env bash
 
_LIBCOUCHBASE_VER=2.5.8
_PROJECT_ROOT=/app
 
# Build and install Couchbase C SDK (core library only).
# Couchbase make code runs some Git commands, so requires making a clone.
git clone --depth 1 --branch $_LIBCOUCHBASE_VER \
    https://github.com/couchbase/libcouchbase.git /tmp/libcouchbase_$$
cd /tmp/libcouchbase_$$
./configure.pl \
    --prefix $_PROJECT_ROOT \
    --disable-tools \
    --disable-tests \
    --disable-plugins \
    --disable-couchbasemock
make
make install
```

**requirements.txt**

```
couchbase==2.0.8 --global-option='build_ext' \
                 --global-option='-L/app/lib' \
                 --global-option='-I/app/include'
```

**manifest.yml**

```
env:
  LD_LIBRARY_PATH: /app/lib
```