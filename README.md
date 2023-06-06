# Cloud Foundry NGINX Buildpack

[![CF Slack](https://www.google.com/s2/favicons?domain=www.slack.com) Join us on Slack](https://cloudfoundry.slack.com/messages/buildpacks/)

A Cloud Foundry [buildpack](http://docs.cloudfoundry.org/buildpacks/) for apps requiring NGINX.

### Open-telemetry fork

This fork adds the [OpenTelemetry nginx plugin](https://github.com/open-telemetry/opentelemetry-cpp-contrib/tree/main/instrumentation/nginx).

The libraries are built on an Ubuntu 22.04 VM (after apt-get update && apt-get upgrade to sync to latest libraries):

```
sudo apt install cmake build-essential autoconf libtool libpcre++-dev libssl-dev zlib1g-dev clang libcurl4-openssl-dev nlohmann-json3-dev nginx git

git clone https://github.com/open-telemetry/opentelemetry-cpp-contrib.git -b webserver/v1.0.3
git clone --shallow-submodules --depth 1 --recurse-submodules -b v1.49.2 https://github.com/grpc/grpc
git clone --shallow-submodules --depth 1 --recurse-submodules -b v1.8.1 https://github.com/open-telemetry/opentelemetry-cpp.git

cd grpc/
mkdir -p cmake/build
cd cmake/build/
cmake -DgRPC_INSTALL=ON -DgRPC_BUILD_TESTS=OFF -DCMAKE_BUILD_TYPE=Release -DgRPC_BUILD_GRPC_NODE_PLUGIN=OFF -DgRPC_BUILD_GRPC_OBJECTIVE_C_PLUGIN=OFF -DgRPC_BUILD_GRPC_PHP_PLUGIN=OFF -DgRPC_BUILD_GRPC_PHP_PLUGIN=OFF -DgRPC_BUILD_GRPC_PYTHON_PLUGIN=OFF -DgRPC_BUILD_GRPC_RUBY_PLUGIN=OFF -DCMAKE_CXX_STANDARD=17 ../..
make -j 8
sudo make install
cd ~/

cd opentelemetry-cpp
mkdir build 
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DWITH_OTLP=ON  -DWITH_OTLP_GRPC=ON  -DWITH_OTLP_HTTP=OFF  -DBUILD_TESTING=OFF  -DWITH_EXAMPLES=OFF  -DCMAKE_CXX_STANDARD=17  -DCMAKE_POSITION_INDEPENDENT_CODE=ON  -DWITH_ABSEIL=ON ..
make -j 8
sudo make install
cd ~/

cd opentelemetry-cpp-contrib/instrumentation/nginx/
mkdir build
cd build/
cmake -DNGINX_VERSION=1.21.4 -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/share/nginx/modules  -DCMAKE_CXX_STANDARD=17 ..
make -j 8
sudo make install

# The "otel_ngx_module.so" is now in the current (build) directory

strip otel_ngx_module.so # the binary is massive (optional)
tar czf otel_ngx_module_nginx1.21.4_5c0cc9c8.tar.gz otel_ngx_module.so
cd ..

# It should be tar-gzipped and copied into the dist folder in the buildpack. The manifest should be updated accordingly
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

