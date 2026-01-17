FROM debian:bookworm-20260112

# 下载并添加 GPG 密钥
# 添加 LLVM 18 的 APT 源（Debian 12 代号为 bookworm）
RUN apt update                                                                  &&\
  apt install -y ca-certificates wget                                           &&\
  wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | \
    tee /etc/apt/trusted.gpg.d/llvm.asc                                         &&\
  echo "deb http://apt.llvm.org/bookworm/ llvm-toolchain-bookworm-18 main" | \
    tee /etc/apt/sources.list.d/llvm-18.list                                    &&\
  apt update                                                                    &&\
  apt install -y clang-18 lldb-18                                               &&\
  apt clean                                                                     &&\
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*                                 &&\
  ln -sf `which clang-18` /usr/bin/clang                                        &&\
  ln -sf `which lldb-18` /usr/bin/lldb 
