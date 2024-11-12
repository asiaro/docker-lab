FROM ubuntu:22.04 as build-stage

ENV DEBIAN_FRONTEND=noninteractive
COPY ./sources.list /etc/apt/sources.list

# Install prerequisites
RUN apt -y update && apt -y upgrade && \
    apt -y install autoconf automake autotools-dev curl python-is-python3 python3-venv libmpc-dev libmpfr-dev \
           libgmp-dev gawk build-essential bison flex texinfo gperf libtool \
           patchutils bc zlib1g-dev libexpat-dev git \
           pkg-config ninja-build \
           libglib2.0-dev libpixman-1-dev

# Build toolchain
WORKDIR /riscv/
RUN curl https://mirror.iscas.ac.cn/riscv-toolchains/git/riscv-collab/riscv-gnu-toolchain.sh | bash
WORKDIR /riscv/riscv-gnu-toolchain/
RUN ./configure --prefix=/opt/riscv
RUN make -j $(nproc) linux
RUN make -j $(nproc) build-qemu
RUN make install

# FROM ubuntu:22.04 as production-stage
FROM ghcr.io/spinalhdl/docker:master as production-stage

ENV DEBIAN_FRONTEND=noninteractive
COPY ./sources.list /etc/apt/sources.list

RUN apt -y update && apt -y upgrade && \
    apt -y install build-essential git clang libglib2.0-dev && \
    rm -rf /var/lib/apt/lists/*

COPY --from=build-stage /opt/riscv /opt/riscv
ENV PATH="/opt/riscv/bin:${PATH}"
ENV RISCV="/opt/riscv/"

# Test if requirements exists
RUN command -v riscv64-unknown-linux-gnu-gcc
CMD ["bash"]