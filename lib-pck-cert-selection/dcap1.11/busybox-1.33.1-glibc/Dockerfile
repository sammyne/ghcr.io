FROM ubuntu:20.04 AS builder

ENV DEBIAN_FRONTEND=noninteractive

#RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list &&\
#  sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list

RUN apt update && apt install -y openssl git build-essential

WORKDIR /pck

RUN git clone -b DCAP_1.11                                          \
    https://github.com/intel/SGXDataCenterAttestationPrimitives .

RUN cd tools/PCKCertSelection           &&\
  make -C PCKCertSelectionLib           &&\
  mkdir /output                         &&\
  cp out/libPCKCertSelection.so /output

WORKDIR /output

RUN cp /lib/x86_64-linux-gnu/libdl*   \
  /lib/x86_64-linux-gnu/libstdc++*    \
  /lib/x86_64-linux-gnu/libgcc_s.so.1 \
  .

FROM busybox:1.33.1-glibc

WORKDIR /pck

COPY --from=builder /output .
