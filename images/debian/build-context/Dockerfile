ARG debianVersion=latest
FROM debian:${debianVersion} as builder

RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    debhelper \
    git \
    libusb-1.0-0-dev \
    libsoapysdr-dev \
    libssl-dev \
    librtlsdr-dev \
    dpkg \
    dpkg-dev \
    pkg-config \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /build

ADD ./rtl_433 ./rtl_433
WORKDIR ./rtl_433
WORKDIR ./build
RUN cmake -DENABLE_OPENSSL=ON ..
RUN make
WORKDIR /build/root
WORKDIR /build/rtl_433/build
RUN make DESTDIR=/build/root/ install
RUN ls -lah /build/root

ARG debianVersion=latest
FROM debian:${debianVersion}

ARG rtl433GitRevision=master
LABEL maintainer="hockeymikey@hockeymikey.com" \
    vcs-ref="${rtl433GitRevision}"
    
RUN apt-get update && apt-get install -y \
    libusb-1.0-0 \
#    librtlsdr0 \
    '^libsoapysdr0\.[6-8]$' \
    '^libssl(1\.1|3)$' \
    '^soapysdr0\.[6-8]-module-all$' \
    debhelper \
    dpkg \
    dpkg-dev \
    pkg-config \
    build-essential \
    cmake \
    git \
    libusb-1.0-0-dev \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /build

# Add rtl-sdr-blog
RUN git clone https://github.com/rtlsdrblog/rtl-sdr-blog
WORKDIR /build/rtl-sdr-blog
RUN dpkg-buildpackage -b --no-sign
WORKDIR ..

# Install rtl-sdr-blog
RUN rm *.changes
RUN rm *.buildinfo
RUN dpkg -i librtlsdr0_*
RUN dpkg -i librtlsdr-dev_*
RUN dpkg -i rtl-sdr_*

WORKDIR /root
COPY --from=builder /build/root/ /

ENTRYPOINT ["/usr/local/bin/rtl_433"]
