# Cloud Foundry NGINX Buildpack

[![CF Slack](https://www.google.com/s2/favicons?domain=www.slack.com) Join us on Slack](https://cloudfoundry.slack.com/messages/buildpacks/)

A Cloud Foundry [buildpack](http://docs.cloudfoundry.org/buildpacks/) for apps requiring NGINX.

### Open-telemetry fork

This fork adds the [OpenTelemetry nginx plugin](https://github.com/open-telemetry/opentelemetry-cpp-contrib/tree/main/instrumentation/nginx).

The libraries are built on an Ubuntu 18.04 VM:

```
sudo apt install cmake build-essential autoconf libtool libpcre++ libssl-dev zlib1g-dev clang libcurl4-openssl-dev git
mkdir dist

# Ubuntu 18.04 has cmake 3.10, which is too old to build some of the packages
sudo apt remove cmake
sudo snap install cmake --classic

# JSON Library Dependency
curl -L https://github.com/nlohmann/json/archive/refs/tags/v3.10.4.tar.gz | tar xzf -
cd json-3.10.4
mkdir build && cd build
cmake ..
make
sudo make install
cd ../..

# GRPC Library Dependency
# the grpc source archives don't include all the submodules, so we need to clone
git clone --recursive https://github.com/grpc/grpc.git
cd grpc
git reset --hard 635693ce624f3b3a89e5a764f0664958ef08b2b9 # 1.41.1
mkdir cmake/build && cd cmake/build
cmake ../.. -DgRPC_INSTALL=ON
make
sudo make install
cd ../../..

# Google Test Library Dependency
git clone https://github.com/google/googletest.git -b release-1.11.0
cd googletest
mkdir build && cd build
cmake ..
make
sudo make install
cd ../..

# Google Benchmark Library Dependency
git clone https://github.com/google/benchmark.git -b v1.6.0
cd benchmark
mkdir build && cd build
cmake -DBENCHMARK_ENABLE_TESTING=false -DBENCHMARK_DOWNLOAD_DEPENDENCIES=on -DCMAKE_BUILD_TYPE=Release ..
make
sudo make install
cd ../..

# OpenTelemetry Library Dependency
git clone --recursive https://github.com/open-telemetry/opentelemetry-cpp -b v1.0.1
cd opentelemetry-cpp
mkdir build && cd build
cmake -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DWITH_OTLP=ON -DBUILD_TESTING=OFF -DWITH_ABSEIL=ON -DWITH_OTLP_HTTP=OFF ..
make
sudo make install
cd ../..
# And finally, the plugin
git clone https://github.com/open-telemetry/opentelemetry-cpp-contrib.git
cd opentelemetry-cpp-contrib/instrumentation/nginx
git reset --hard 30e872e1d91cb696b48418300fddd8f9fc36b914 # no tags in the repo
mkdir build && cd build
cmake -DNGINX_VERSION=1.19.9 .. # this needs to make the OpenResty upstream nginx version
make
cp otel_ngx_module.so ../../../../dist/
cd ../../../..

cd dist
strip otel_ngx_module.so # the binary is massive
tar czf otel_ngx_module_nginx1.19.1_30e872e.tar.gz otel_ngx_module.so
cd ..


# Now copy dist/* to dist/ in the buildpack
```

### Buildpack User Documentation

Official buildpack documentation can be found at [here](https://docs.cloudfoundry.org/buildpacks/nginx/index.html).

To use this buildpack, you will need to include an `nginx.conf` file in your app. [Here's an example.](https://github.com/cloudfoundry/nginx-buildpack/tree/master/fixtures/mainline)


### Building the Buildpack

To build this buildpack, run the following command from the buildpack's directory:

1. Source the .envrc file in the buildpack directory.

   ```bash
   source .envrc
   ```
   To simplify the process in the future, install [direnv](https://direnv.net/) which will automatically source .envrc when you change directories.

1. Install buildpack-packager

    ```bash
    ./scripts/install_tools.sh
    ```

1. Build the buildpack

    ```bash
    buildpack-packager build [ -cached=(true|false) ] -any-stack
    ```

1. Use in Cloud Foundry

   Upload the buildpack to your Cloud Foundry and optionally specify it by name

    ```bash
    cf create-buildpack [BUILDPACK_NAME] [BUILDPACK_ZIP_FILE_PATH] 1
    cf push my_app [-b BUILDPACK_NAME]
    ```

### Testing

Buildpacks use the [Cutlass](https://github.com/cloudfoundry/libbuildpack/tree/master/cutlass) framework for running integration tests.

To test this buildpack, run the following command from the buildpack's directory:

1. Source the .envrc file in the buildpack directory.

   ```bash
   source .envrc
   ```
   To simplify the process in the future, install [direnv](https://direnv.net/) which will automatically source .envrc when you change directories.

1. Run unit tests

    ```bash
    ./scripts/unit.sh
    ```

1. Run integration tests

    ```bash
    ./scripts/integration.sh
    ```

More information can be found on Github [cutlass](https://github.com/cloudfoundry/libbuildpack/tree/master/cutlass).

### Contributing

Find our guidelines [here](./CONTRIBUTING.md).

### Help and Support

Join the #buildpacks channel in our [Slack community](http://slack.cloudfoundry.org/)

### Reporting Issues

Open an issue on this project

### Active Development

The project backlog is on [Pivotal Tracker](https://www.pivotaltracker.com/projects/1042066)

