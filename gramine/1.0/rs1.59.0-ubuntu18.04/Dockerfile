FROM sammyne/gramine:1.0-ubuntu18.04

RUN RUST_VERSION=1.59.0                                               &&\
  apt update                                                          &&\
  apt install -y --reinstall systemd                                  &&\
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs |           \
  sh -s -- -y --default-toolchain ${RUST_VERSION}                     &&\
  . $HOME/.cargo/env                                                  &&\
  rustup component add rust-src rls rust-analysis                     &&\
  echo 'source $HOME/.cargo/env' >> ~/.bashrc                         &&\
  rm -rf /root/.cargo/registry                                        &&\
  rm -rf /root/.cargo/git
