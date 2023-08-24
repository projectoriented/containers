# Author: Mei Wu, https://github.com/projectoriented
FROM ubuntu:20.04

# Set environment variables to prevent interactive prompts during installation
ENV DEBIAN_FRONTEND=noninteractive

# Define versions
ARG HTSLIB_VERSION=1.18
ARG SAMTOOLS_VERSION=1.18
ARG MINIMAP2_VERSION=2.26
ARG STROBEALIGN_VERSION=0.11.0
ARG LOCITYPER_COMMIT_HASH=e0dd049

# Install essential packages
RUN apt-get update && \
    apt-get install -y \
        gcc-9 g++-9 \
        cmake \
        build-essential \
        clang \
        wget curl \
        zlib1g-dev libcurl4-gnutls-dev liblzma-dev libncurses5-dev libncursesw5-dev libssl-dev libbz2-dev \
        autoconf \
        git \
        jellyfish && \
    # Update the gcc symlink to point to gcc-9
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 100 && \
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 100 && \
    apt-get clean

# Set environment variables for GCC
ENV CC=/usr/bin/gcc-9
ENV CXX=/usr/bin/g++-9

# Install htslib
WORKDIR /opt/htslib
RUN wget https://github.com/samtools/htslib/releases/download/${HTSLIB_VERSION}/htslib-${HTSLIB_VERSION}.tar.bz2 && \
    tar -xvf htslib-${HTSLIB_VERSION}.tar.bz2 && \
    rm -r htslib-${HTSLIB_VERSION}.tar.bz2 && \
    cd htslib-${HTSLIB_VERSION} && \
    ./configure && \
    make

# Install samtools
WORKDIR /opt/samtools
RUN wget https://github.com/samtools/samtools/releases/download/${SAMTOOLS_VERSION}/samtools-${SAMTOOLS_VERSION}.tar.bz2 && \
    tar xvf samtools-${SAMTOOLS_VERSION}.tar.bz2 && \
    rm -r samtools-${SAMTOOLS_VERSION}.tar.bz2 && \
    cd samtools-${SAMTOOLS_VERSION}/ && \
    autoheader && \
    autoconf -Wno-header && \
    ./configure --with-htslib=/opt/htslib/htslib-${HTSLIB_VERSION} && \
    make
ENV PATH="/opt/samtools/samtools-${SAMTOOLS_VERSION}/bin:$PATH"

# Install minimap2
WORKDIR /opt/minimap2
RUN wget https://github.com/lh3/minimap2/releases/download/v${MINIMAP2_VERSION}/minimap2-${MINIMAP2_VERSION}_x64-linux.tar.bz2 && \
    tar xvf minimap2-${MINIMAP2_VERSION}_x64-linux.tar.bz2 && \
    rm minimap2-${MINIMAP2_VERSION}_x64-linux.tar.bz2
ENV PATH="/opt/minimap2/minimap2-${MINIMAP2_VERSION}_x64-linux:$PATH"

# Install strobealign
WORKDIR /opt/strobealign
RUN wget https://github.com/ksahlin/strobealign/archive/refs/tags/v${STROBEALIGN_VERSION}.tar.gz && \
    tar xvf v${STROBEALIGN_VERSION}.tar.gz && \
    rm v${STROBEALIGN_VERSION}.tar.gz && \
    cd strobealign-${STROBEALIGN_VERSION} && \
    cmake -B "build" -DCMAKE_C_FLAGS="-march=native" -DCMAKE_CXX_FLAGS="-march=native" && \
    make -j -C build
ENV PATH="/opt/strobealign/strobealign-${STROBEALIGN_VERSION}/build:$PATH"

# Install Rust
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y

# Clone locityper repository at a specific commit hash
RUN git clone https://github.com/tprodanov/locityper.git /opt/locityper && \
    cd /opt/locityper && \
    git clone https://github.com/smarco/WFA2-lib.git WFA2 && \
    git checkout ${LOCITYPER_COMMIT_HASH} && \
    /root/.cargo/bin/cargo build --release && \
    /root/.cargo/bin/cargo install --path .

# Set the working directory
WORKDIR /work

# Command to run when the container starts
CMD ["/bin/bash"]
