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
    pkg-config \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /rtl-sdr
RUN git clone https://github.com/rtlsdrblog/rtl-sdr-blog && \
    cd rtl-sdr-blog && \
    dpkg-buildpackage -b --no-sign && \
    cd .. && \
    rm -rf rtl-sdr-blog && \
    dpkg -i librtlsdr0_*.deb && \
    dpkg -i librtlsdr-dev_*.deb

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
LABEL maintainer="georgedot@gmail.com" \
    vcs-ref="${rtl433GitRevision}"

RUN apt-get update && apt-get install -y \
    libusb-1.0-0 \
    '^libsoapysdr0\.[6-8]$' \
    '^libssl(1\.1|3)$' \
    '^soapysdr0\.[6-8]-module-all$' \
 && rm -rf /var/lib/apt/lists/*

COPY --from=builder /rtl-sdr/librtlsdr0_*.deb /tmp/
RUN dpkg -i /tmp/librtlsdr0_*.deb && \
    rm -rf /tmp/librtlsdr0_*.deb


WORKDIR /root
COPY --from=builder /build/root/ /

ENTRYPOINT ["/usr/local/bin/rtl_433"]
