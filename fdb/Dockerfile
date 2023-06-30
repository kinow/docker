# -----------------------------------------------------------------------------
# --- ecbuild, eccodes, eckit, metkit, fdb.
FROM debian:bookworm-slim AS deps

ARG ECBUILD_VERSION=3.7.2
ARG ECKIT_VERSION=1.24.2
ARG METKIT_VERSION=1.10.13
ARG FDB_VERSION=5.11.17
ARG ECCODES_VERSION=2.30.4

# Install build-time dependencies.
RUN apt-get update && \
    apt-get install --yes --no-install-suggests --no-install-recommends \
      build-essential \
      ca-certificates \
      cmake \
      gfortran \
      git \
      libaec0 \
      libnetcdf-dev && \
    rm -rf /var/lib/apt/lists/*

# Install ecbuild.
RUN cd /opt/ && \
    git clone --depth 1 --branch $ECBUILD_VERSION https://github.com/ecmwf/ecbuild.git

ENV CMAKE_PREFIX_PATH="/opt/ecbuild/"

# Install eccodes.
RUN cd /usr/src/ && \
    git clone --depth 1 --branch $ECCODES_VERSION https://github.com/ecmwf/eccodes.git && \
    cd eccodes && \
    mkdir build && \
    cd build && \
    cmake ../ -DCMAKE_INSTALL_PREFIX=/opt/eccodes && \
    make -j4 && \
    ctest && \
    make install

# Install eckit.
RUN cd /usr/src/ && \
    git clone --depth 1 --branch $ECKIT_VERSION https://github.com/ecmwf/eckit.git && \
    cd eckit && \
    mkdir build && \
    cd build && \
    /opt/ecbuild/bin/ecbuild --prefix=/opt/eckit -- /usr/src/eckit/ && \
    make -j4 && \
    make install && \
    /opt/eckit/bin/eckit-version

# Install metkit.
RUN cd /usr/src/ && \
    git clone --depth 1 --branch $METKIT_VERSION https://github.com/ecmwf/metkit.git && \
    cd metkit && \
    mkdir build && \
    cd build && \
    /opt/ecbuild/bin/ecbuild --prefix=/opt/metkit -- -DECKIT_PATH=/opt/eckit/ /usr/src/metkit/ && \
    make -j4 && \
    make install

# Install fdb.
RUN cd /usr/src/ && \
    git clone --depth 1 --branch $FDB_VERSION https://github.com/ecmwf/fdb.git && \
    cd fdb && \
    mkdir build && \
    cd build && \
    /opt/ecbuild/bin/ecbuild --prefix=/opt/fdb -- -DCMAKE_INSTALL_PREFIX=/opt/fdb /usr/src/fdb/ && \
    make -j4 && \
    make install

# -----------------------------------------------------------------------------
# --- Final base image.
FROM debian:bookworm-slim

ARG AUTHOR
ARG BUILD_DATE
ARG IMAGE_VERSION
ARG ECBUILD_VERSION
ARG ECKIT_VERSION
ARG METKIT_VERSION
ARG FDB_VERSION
ARG ECCODES_VERSION

# Labels.
LABEL org.label-schema.schema-version="1.0"
LABEL org.label-schema.name="kinow/fdb"
LABEL org.label-schema.version="$IMAGE_VERSION"
LABEL org.label-schema.maintainer="$AUTHOR"
LABEL org.label-schema.build-date="$BUILD_DATE"
LABEL org.label-schema.description="Docker image with ECMWF FDB. \
**This is not an official image**. The versions of \
FDB and dependencies are pinned and available as \
labels of the image as well."
LABEL org.label-schema.docker.cmd="docker run -ti --rm kinow/fdb /bin/bash"

LABEL org.label-schema.url="https://github.com/ecmwf/fdb"
LABEL org.label-schema.vcs-url="https://github.com/kinow/docker/tree/master/fdb"
LABEL br.eti.kinoshita.ecbuild_version="$ECBUILD_VERSION"
LABEL br.eti.kinoshita.eckit_version="$ECKIT_VERSION"
LABEL br.eti.kinoshita.metkit_version="$METKIT_VERSION"
LABEL br.eti.kinoshita.fdb_version="$FDB_VERSION"
LABEL br.eti.kinoshita.eccodes_version="$ECCODES_VERSION"

# TODO: pyfdb is under development, so we are using a commit for now
# ARG PYFDB_VERSION=02692523ae83b6667dbfa189bbc49d37c3a14a0f

COPY --from=deps /opt/ecbuild /opt/ecbuild
COPY --from=deps /opt/eccodes /opt/eccodes
COPY --from=deps /opt/eckit /opt/eckit
COPY --from=deps /opt/metkit /opt/metkit
COPY --from=deps /opt/fdb /opt/fdb

USER root

# Install run-time dependencies.
RUN apt-get update && \
    apt-get install --yes --no-install-suggests --no-install-recommends \
      ca-certificates \
      libaec0 \
      libcurl3-gnutls \
      libeccodes-tools \
      libssl3 && \
    rm -rf /var/lib/apt/lists/*

# Create the localroot and make it writable (otherwise fdb commands may
# fail with "No writable roots available...).
RUN mkdir --parents /app/localroot

ENV PATH="$PATH:/opt/ecbuild/bin/:/opt/eckit/bin/:/opt/fdb/bin"
ENV LD_LIBRARY_PATH="/opt/eccodes/lib:/opt/eckit/lib/:/opt/metkit/lib/:/opt/fdb/lib/"
ENV FDB5_CONFIG_FILE=/app/config.yaml

# Copy the test data from pyfdb.
COPY schema config.yaml myrequest test.py *.grib /app/

# The schema file used in this image is from the tests of pyfdb:
# https://github.com/ecmwf/pyfdb/blob/${PYFDB_VERSION}/tests/default_fdb_schema
# Here we print the schema:
RUN fdb-schema

# The data used is also from pydfb:
# https://github.com/ecmwf/pyfdb/tree/${PYFDB_VERSION}/tests
# The data is in the GRIB format. Here we write the GRIB data into the FDB:
RUN find /app -name "*.grib" -exec fdb-write {} \;

# Here we list the data loaded. It must contain at least one data row.
RUN fdb-list class=rd,expver=xxxx,stream=oper,time=0000 | grep "class=rd,expver=xxxx"

# If everything worked, this image should have the following installed and ready to be used:
# - ecbuild (why not?)
# - eccodes
# - eckit
# - metkit
# - fdb
#
# To test, use FDB commands like:
# - fdb-schema
# - fdb-dump class=rd,expver=xxxx,stream=oper,time=0000
# - fdb list class=rd,expver=xxxx,stream=oper,time=0000

WORKDIR /app/

# You can modify the CMD statement as needed....
CMD ["/bin/bash"]
