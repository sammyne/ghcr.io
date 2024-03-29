# https://grpc.io/docs/languages/cpp/quickstart/#build-and-install-grpc-and-protocol-buffers

FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive \
  GRPC_VERSION=v1.48.0

RUN apt update && apt install -y git cmake build-essential autoconf libtool pkg-config libssl-dev

WORKDIR /github.com/grpc/grpc

RUN git clone --recurse-submodules  \
    -b $GRPC_VERSION                \
    --depth 1                       \
    --shallow-submodules            \
    https://github.com/grpc/grpc .  &&\
  cmake -B build -DgRPC_INSTALL=ON  \
    -DgRPC_BUILD_TESTS=OFF          \
    -DCMAKE_INSTALL_PREFIX=/grpc    \
    -DgRPC_SSL_PROVIDER=package     &&\
  cmake --build build               &&\
  cmake --install build             &&\
  rm -rf build

ENV PATH=$PATH:/grpc/bin

ADD fix-protoc-plugins.sh .

RUN bash fix-protoc-plugins.sh
