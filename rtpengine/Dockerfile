FROM debian:bullseye-slim

ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN echo 'Acquire::Retries "5";' > /etc/apt/apt.conf.d/80-retries && \
    apt-get update --fix-missing && \
    apt-get install -y --no-install-recommends \
        build-essential \
        git \
        ca-certificates \
        pkg-config \
        gperf \
        iptables \
        net-tools \
        iproute2 \
        # Core libs
        libglib2.0-dev \
        libpcap-dev \
        libxml2-dev \
        libcurl4-openssl-dev \
        libevent-dev \
        libevent-pthreads-2.1-7 \
        libpcre2-dev \
        libssl-dev \
        libwebsockets-dev \
        libjson-glib-dev \
        # FFmpeg/media libs
        libavcodec-dev \
        libavfilter-dev \
        libavformat-dev \
        libavutil-dev \
        libswresample-dev \
        libspandsp-dev \
        libopus-dev \
        # Networking libs
        libmnl-dev \
        libnftnl-dev \
        libiptc-dev \
        # Other
        libncurses-dev \
        libhiredis-dev \
        mariadb-client \
        default-libmysqlclient-dev \
        pandoc \
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Clone and build RTPengine - using master branch
RUN git clone https://github.com/sipwise/rtpengine.git /rtpengine && \
    cd /rtpengine && \
    git checkout master && \
    make && \
    make install && \
    cp /rtpengine/daemon/rtpengine /usr/local/bin/ && \
    ldconfig

# Create symbolic link to ensure it's in PATH
RUN ln -sf /usr/local/bin/rtpengine /usr/bin/rtpengine

# Verify the binary exists and works
RUN which rtpengine && rtpengine --version

# Expose ports (comments on separate lines)
EXPOSE 22222/tcp
EXPOSE 2223/udp
EXPOSE 16384/udp
EXPOSE 16484/udp

# Health check (optional)
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
    CMD ss -tln | grep -q 22222 || exit 1

# Entry point with proper modern arguments
ENTRYPOINT ["/usr/local/bin/rtpengine"]

# Use shell form for CMD
CMD "--interface=0.0.0.0 --listen-ng=0.0.0.0:22222 --listen-udp=0.0.0.0:2223 --port-min=16384 --port-max=16484 --foreground --log-level=6 --tos=184"