FROM rust:1.87.0-bookworm AS builder

WORKDIR /workspace

RUN git clone -b v0.10.0 https://github.com/mozilla/sccache.git .

RUN cargo build -r

WORKDIR /output

RUN mv /workspace/target/release/sccache .


FROM ubuntu:24.04

ENV PATH=/opt/sccache:$PATH

COPY --from=builder /output/sccache /opt/sccache/sccache
