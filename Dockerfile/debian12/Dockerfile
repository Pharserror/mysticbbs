FROM debian:12

# Prevent interactive prompts during build
ENV DEBIAN_FRONTEND=noninteractive

LABEL maintainer="opicron <opicron@gmail.com>"

WORKDIR /root

# Enable 32-bit packages. Mystic may require 32-bit libraries.
RUN dpkg --add-architecture i386

# Base packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    cron \
    p7zip-full \
    unzip \
    zip \
    mc \
    wget \
    procps \
    curl \
    dos2unix \
    nano \
    expect \
    make \
    git \
    atool \
    htop \
    logrotate \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# --------------------------------------------------------------------------
# Python 3.14.7
# --------------------------------------------------------------------------

# Dependencies required to build CPython
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libffi-dev \
    liblzma-dev \
    libgdbm-dev \
    libgdbm-compat-dev \
    libnss3-dev \
    uuid-dev \
    && rm -rf /var/lib/apt/lists/*

# Install pyenv
ENV PYENV_ROOT="/opt/pyenv"
ENV PATH="/opt/pyenv/shims:/opt/pyenv/bin:$PATH"

# Build Python with shared-library support
ENV PYTHON_CONFIGURE_OPTS="--enable-shared"

RUN git clone --depth 1 https://github.com/pyenv/pyenv.git "${PYENV_ROOT}" && \
    "${PYENV_ROOT}/bin/pyenv" install 3.14.7 && \
    "${PYENV_ROOT}/bin/pyenv" global 3.14.7 && \
    cp "${PYENV_ROOT}/versions/3.14.7/lib/libpython3.14.so"* /usr/local/lib/ && \
    echo "/usr/local/lib" > /etc/ld.so.conf.d/python3.14.conf && \
    ldconfig

# Make sure Python/pip are the expected versions
RUN python --version && \
    python -m pip --version

# Python packages
RUN python -m pip install --no-cache-dir \
    requests \
    pyfiglet \
    pillow

# --------------------------------------------------------------------------
# Mystic logrotate configuration
# --------------------------------------------------------------------------

RUN mkdir -p /etc/logrotate.d && \
    printf '%s\n' \
        '/mystic/logs/*.log {' \
        ' maxsize 2M' \
        ' daily' \
        ' rotate 5' \
        ' missingok' \
        ' compress' \
        ' notifempty' \
        ' copytruncate' \
        '}' \
        > /etc/logrotate.d/mystic

# --------------------------------------------------------------------------
# CryptLib
# --------------------------------------------------------------------------

WORKDIR /root/cl

# CryptLib 3.4.5
RUN wget http://www.mysticbbs.com/downloads/cl345.zip

# Debian 12 ships GCC 12. The original Dockerfile explicitly installed
# GCC 9 because it was being built on Ubuntu 24.04.
#
# GCC 12 is used here for Debian 12.
RUN unzip -a cl345.zip && \
    dos2unix tools/*.sh && \
    make shared && \
    mv libcl.so.3.4.5 /lib/libcl.so

# --------------------------------------------------------------------------
# Mystic
# --------------------------------------------------------------------------

EXPOSE 23/tcp 22/tcp 24554/tcp 8081/tcp

WORKDIR /mystic

# Default command, can be overridden
CMD ["/mystic/boot.sh", "mystic"]

# For debugging:
# ENTRYPOINT ["tail", "-f", "/dev/null"]