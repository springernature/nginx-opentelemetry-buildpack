# Cloud Foundry NGINX Buildpack

[![CF Slack](https://www.google.com/s2/favicons?domain=www.slack.com) Join us on Slack](https://cloudfoundry.slack.com/messages/buildpacks/)

A Cloud Foundry [buildpack](http://docs.cloudfoundry.org/buildpacks/) for apps requiring NGINX.

### Open Tracing fork

This fork adds the [Nginx Open Tracing plugin](https://github.com/opentracing-contrib/nginx-opentracing) and supporting libraries, for communication with a Jaeger sink.

The libraries are built on an Ubuntu 18.04 VM:

```
sudo apt install cmake build-essential autoconf libtool libpcre++ libssl-dev zlib1g-dev
mkdir dist

# OpenTracing lib
curl -L https://github.com/opentracing/opentracing-cpp/archive/refs/tags/v1.6.0.tar.gz | tar xzf -
cd $HOME/opentracing-cpp-1.6.0
mkdir .build
cd .build
cmake ..
make
sudo make install # also required to build plugin & jaegar client
cd ../..
tar czf dist/libopentracing.tar.gz --transform 's?.*/??g' opentracing-cpp-1.6.0/.build/output/libopentracing*

# Jaeger client lib
curl -L https://github.com/jaegertracing/jaeger-client-cpp/archive/refs/tags/v0.7.0.tar.gz | tar xzf -
cd jaeger-client-cpp-0.7.0
mkdir build
cd build
cmake ..
make
cd ../..
tar cf dist/libjaegertracing.tar --transform 's?.*/??g' jaeger-client-cpp-0.7.0/build/libjaegertracing.so*
for FILE in $(find $HOME/.hunter/_Base/Cellar -name 'libyaml-cpp*'); do tar rf dist/libjaegertracing.tar --transform 's?.*/??g' "$FILE"; done
gzip dist/libjaegertracing.tar

# OpenResty
curl -L https://github.com/opentracing-contrib/nginx-opentracing/archive/refs/tags/v0.19.0.tar.gz | tar xzf -
curl -L https://openresty.org/download/openresty-1.19.9.1.tar.gz | tar xzf -
cd $HOME/openresty-1.19.9.1
./configure --with-compat --add-dynamic-module=$HOME/nginx-opentracing-0.19.0/opentracing
make
cd ..
tar czf dist/ngx_http_opentracing_module.tar.gz -C openresty-1.19.9.1/build/nginx-1.19.9/objs ngx_http_opentracing_module.so

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

