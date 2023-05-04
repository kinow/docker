# --- ECBUILD, ECKIT, METKIT, FDB

FROM debian:bookworm-slim AS deps

ARG ECBUILD_VERSION=3.7.2
ARG ECKIT_VERSION=1.23.0
ARG METKIT_VERSION=1.10.11
ARG FDB_VERSION=5.11.10

RUN apt-get update && apt-get install -y \
    git \
    cmake \
    build-essential && \
    rm -rf /var/lib/apt/lists/*

# ecbuild layer
# https://github.com/ecmwf/ecbuild/tags

RUN cd /opt/ && \
	git clone --depth 1 --branch $ECBUILD_VERSION https://github.com/ecmwf/ecbuild.git

RUN cd /usr/src/ && \
	git clone --depth 1 --branch $ECKIT_VERSION https://github.com/ecmwf/eckit.git && \
	cd eckit && \
    mkdir build && \
    cd build && \
    /opt/ecbuild/bin/ecbuild --prefix=/opt/eckit -- /usr/src/eckit/ && \
    make -j4 && \
    make install && \
    /opt/eckit/bin/eckit-version

RUN cd /usr/src/ && \
	git clone --depth 1 --branch $METKIT_VERSION https://github.com/ecmwf/metkit.git && \
	cd metkit && \
    mkdir build && \
    cd build && \
    /opt/ecbuild/bin/ecbuild --prefix=/opt/metkit -- -DECKIT_PATH=/opt/eckit/ /usr/src/metkit/ && \
    make -j4 && \
    make install

RUN cd /usr/src/ && \
	git clone --depth 1 --branch $FDB_VERSION https://github.com/ecmwf/fdb.git && \
	cd fdb && \
    mkdir build && \
    cd build && \
    /opt/ecbuild/bin/ecbuild --prefix=/opt/fdb -- -DCMAKE_INSTALL_PREFIX=/opt/fdb /usr/src/fdb/ && \
    make -j4 && \
    make install

# --- FDB, eccodes
# FROM scratch

FROM mambaorg/micromamba:bullseye-slim

COPY --from=deps /opt/ecbuild /opt/ecbuild
COPY --from=deps /opt/eckit /opt/eckit
COPY --from=deps /opt/metkit /opt/metkit
COPY --from=deps /opt/fdb /opt/fdb

RUN micromamba install --yes --name base --channel conda-forge \
	eccodes==2.30.0

ENV PATH="$PATH:/opt/ecbuild/bin/:/opt/eckit/bin/"

ARG MAMBA_DOCKERFILE_ACTIVATE=1

WORKDIR /
