FROM ubuntu:20.04

# the matching driver version is v1.36
ENV DEBIAN_FRONTEND=noninteractive      \
  SDK_VERSION=2.12.100.3

RUN UBUNTU_CODENAME=$(cat /etc/os-release | grep UBUNTU_CODENAME | awk -F"=" '{print $2}') &&\
  apt update                                                          &&\
  apt install -y curl gnupg2                                          &&\
  curl -fsSL \
  https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | \ 
  apt-key add -                                                       &&\
  echo "deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu ${UBUNTU_CODENAME} main" | tee /etc/apt/sources.list.d/intel-sgx.list &&\
  apt update                                                          &&\
  apt install  -y libsgx-urts=${SDK_VERSION}-${UBUNTU_CODENAME}1      &&\
  # aesm service need this dir to create socket
  mkdir /var/run/aesmd                                                &&\
  apt clean                                                           &&\
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*   
