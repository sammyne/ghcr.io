FROM ubuntu:18.04

# the matching driver version is v1.36
ENV DCAP_VERSION=1.9.100.3            \
  DEBIAN_FRONTEND=noninteractive      \
  SDK_VERSION=2.12.100.3

#RUN UBUNTU_CODENAME=$(cat /etc/os-release | grep UBUNTU_CODENAME | awk -F"=" '{print $2}') &&\
#  echo > /etc/apt/sources.list                                      &&\     
#  # change apt source list
#  echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ ${UBUNTU_CODENAME} main restricted universe multiverse" >> /etc/apt/sources.list          &&\
#  echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ ${UBUNTU_CODENAME}-updates main restricted universe multiverse" >> /etc/apt/sources.list  &&\
#  echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ ${UBUNTU_CODENAME}-security main restricted universe multiverse" >> /etc/apt/sources.list                    

RUN UBUNTU_CODENAME=$(cat /etc/os-release | grep UBUNTU_CODENAME | awk -F"=" '{print $2}') &&\
  apt update                                                          &&\
  apt install -y curl gnupg2                                          &&\
  curl -fsSL \
  https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | \ 
  apt-key add -                                                       &&\
  echo "deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu ${UBUNTU_CODENAME} main" | tee /etc/apt/sources.list.d/intel-sgx.list &&\
  apt update                                                          &&\
  apt install  -y                                                       \         
  libsgx-urts=${SDK_VERSION}-${UBUNTU_CODENAME}1                        \
  libsgx-qe3-logic=${DCAP_VERSION}-${UBUNTU_CODENAME}1                  \
  libsgx-pce-logic=${DCAP_VERSION}-${UBUNTU_CODENAME}1                  \
  libsgx-dcap-ql=${DCAP_VERSION}-${UBUNTU_CODENAME}1                    \
  libsgx-dcap-default-qpl=${DCAP_VERSION}-${UBUNTU_CODENAME}1         &&\
  # aesm service need this dir to create socket
  mkdir /var/run/aesmd                                                &&\
  # fix the forgotten linking
  cd /usr/lib/x86_64-linux-gnu/                                       &&\
  ln -s libsgx_dcap_quoteverify.so.1 libsgx_dcap_quoteverify.so       &&\
  apt clean                                                           &&\
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*   