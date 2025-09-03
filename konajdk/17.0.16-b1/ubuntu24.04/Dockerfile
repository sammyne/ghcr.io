FROM busybox:1.37.0 AS repo

WORKDIR /output

RUN wget -O - https://github.com/Tencent/TencentKona-17/releases/download/TencentKona-17.0.16/TencentKona-17.0.16.b1-jdk_linux-x86_64.tar.gz | \
    tar -zxv &&\
    mv TencentKona-* TencentKona


FROM ubuntu:24.04

ENV JAVA_HOME=/opt/TencentKona

ENV PATH=$PATH:$JAVA_HOME/bin

COPY --from=repo /output/TencentKona $JAVA_HOME
