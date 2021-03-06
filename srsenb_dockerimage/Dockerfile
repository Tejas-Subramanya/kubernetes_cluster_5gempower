## build : docker build -t srsenb .
## run :   docker run --net=host --rm --privileged -v /dev/bus/usb:/dev/bus/usb -it srsenb

FROM ubuntu:18.04
MAINTAINER Tejas Subramanya <t.subramanya@fbk.eu>
ENV DEBIAN_FRONTEND noninteractive

# Dependencies for the UHD driver for the USRP hardware
RUN apt-get update && \
    apt-get -yq dist-upgrade && \
    apt-get -yq install autoconf build-essential libusb-1.0-0-dev cmake wget pkg-config libboost-all-dev python-dev python-cheetah git subversion

# Dependencies for UHD image downloader script
RUN apt-get -yq install python-mako python-requests 

# Fetching the uhd 3.010.001 driver for our USRP SDR card 
RUN wget http://files.ettus.com/binaries/uhd/uhd_003.010.001.001-release/uhd-3.10.1.1.tar.gz && tar xvzf uhd-3.10.1.1.tar.gz && cd UHD_3.10.1.1_release && mkdir build && cd build && cmake ../ && make && make install && ldconfig && python /usr/local/lib/uhd/utils/uhd_images_downloader.py

# Dependencies for SRSeNB
RUN apt-get -yq --no-install-recommends --assume-yes install libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev

# Fetch and building empower-proto 
RUN git clone https://github.com/5g-empower/empower-enb-proto && cd empower-enb-proto && make && make install

# Fetch and building empower-agent
RUN git clone https://github.com/5g-empower/empower-enb-agent && cd empower-enb-agent && make && make install

# Fetching the latest repository from 5G-EMPOWER AGENT branch and Building the srslte eNB for USRP
RUN git clone -b agent https://github.com/5g-empower/empower-srsLTE.git
ADD CMakeLists.txt /empower-srsLTE/srsenb/CMakeLists.txt
RUN cd empower-srsLTE && mkdir build && cd build && cmake ../ && make && make install

# Add configuration files from SRS
ADD enb.conf /empower-srsLTE/build/srsenb/src/enb.conf
ADD drb.conf /empower-srsLTE/build/srsenb/src/drb.conf
ADD rr.conf /empower-srsLTE/build/srsenb/src/rr.conf
ADD sib.conf /empower-srsLTE/build/srsenb/src/sib.conf

# Run directly the eNodeB code from SRS
ENTRYPOINT ./empower-srsLTE/build/srsenb/src/srsenb /empower-srsLTE/build/srsenb/src/enb.conf
