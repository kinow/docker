# -----------------------------------------------------------------------------
# --- ecbuild, eccodes, eckit, metkit, fdb.
FROM debian:bookworm-slim AS deps

ARG ECBUILD_VERSION=3.7.2
ARG ECKIT_VERSION=1.23.0
ARG METKIT_VERSION=1.10.11
ARG FDB_VERSION=5.11.10
ARG ECCODES_VERSION=2.30.2

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
# --- Micromamba / Conda (used later).
FROM mambaorg/micromamba:bullseye-slim as micromamba

# -----------------------------------------------------------------------------
# --- Final base image.
FROM debian:bookworm-slim

# TODO: pyfdb is under development, so we are using a commit for now
ARG PYFDB_VERSION=02692523ae83b6667dbfa189bbc49d37c3a14a0f

COPY --from=deps /opt/ecbuild /opt/ecbuild
COPY --from=deps /opt/eccodes /opt/eccodes
COPY --from=deps /opt/eckit /opt/eckit
COPY --from=deps /opt/metkit /opt/metkit
COPY --from=deps /opt/fdb /opt/fdb

USER root

# if your image defaults to a non-root user, then you may want to make the
# next 3 ARG commands match the values in your image. You can get the values
# by running: docker run --rm -it my/image id -a
ARG MAMBA_USER=mamba
ARG MAMBA_USER_ID=1000
ARG MAMBA_USER_GID=1000
ENV MAMBA_USER=$MAMBA_USER
ENV MAMBA_ROOT_PREFIX="/opt/conda"
ENV MAMBA_EXE="/bin/micromamba"

COPY --from=micromamba "$MAMBA_EXE" "$MAMBA_EXE"
COPY --from=micromamba /usr/local/bin/_activate_current_env.sh /usr/local/bin/_activate_current_env.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_shell.sh /usr/local/bin/_dockerfile_shell.sh
COPY --from=micromamba /usr/local/bin/_entrypoint.sh /usr/local/bin/_entrypoint.sh
COPY --from=micromamba /usr/local/bin/_activate_current_env.sh /usr/local/bin/_activate_current_env.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_initialize_user_accounts.sh /usr/local/bin/_dockerfile_initialize_user_accounts.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_setup_root_prefix.sh /usr/local/bin/_dockerfile_setup_root_prefix.sh
# Ensure shared libs installed by the previous step are available.
RUN set -ex \
    && /sbin/ldconfig

RUN /usr/local/bin/_dockerfile_initialize_user_accounts.sh && \
    /usr/local/bin/_dockerfile_setup_root_prefix.sh

# Install run-time dependencies.
RUN apt-get update && \
    apt-get install --yes --no-install-suggests --no-install-recommends \
      ca-certificates \
      libaec0 \
      libcurl4 \
      libcurl3-gnutls && \
    rm -rf /var/lib/apt/lists/*

# Create the localroot and make it writable (otherwise fdb commands may
# fail with "No writable roots available...).
RUN mkdir --parents /app/localroot && \
    chown --recursive $MAMBA_USER: /app

USER $MAMBA_USER

SHELL ["/usr/local/bin/_dockerfile_shell.sh"]

ENTRYPOINT ["/usr/local/bin/_entrypoint.sh"]
# Optional: if you want to customize the ENTRYPOINT and have a conda
# environment activated, then do this:
# ENTRYPOINT ["/usr/local/bin/_entrypoint.sh", "my_entrypoint_program"]

# You can modify the CMD statement as needed....
CMD ["/bin/bash"]

ARG MAMBA_DOCKERFILE_ACTIVATE=1

RUN micromamba install --yes --name base --channel conda-forge \
      git \
      python==3.10 && \
    micromamba clean --all --yes

RUN python3 -m pip install --upgrade \
    git+https://github.com/ecmwf/pyfdb.git@${PYFDB_VERSION}

ENV PATH="$PATH:/opt/ecbuild/bin/:/opt/eckit/bin/:/opt/fdb/bin"
ENV LD_LIBRARY_PATH="/opt/eccodes/lib:/opt/eckit/lib/:/opt/metkit/lib/:/opt/fdb/lib/"
ENV FDB5_CONFIG_FILE=/app/config.yaml
ENV \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Install the Python package for EC-Codes, and shelf-check to confirm
# both Python and system package are OK.
RUN python3 -m pip install eccodes && \
    python3 -m eccodes selfcheck

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

# Here we dump the table of contents of the data:
RUN fdb-dump class=rd,expver=xxxx,stream=oper,time=0000

# If everything worked, this image should have the following installed and ready to be used:
# - ecbuild (why not?)
# - eccodes
# - eckit
# - metkit
# - fdb
# - micromamba / conda, with
#   - python 3.10
#   - pyeccodes
#   - pyfdb (commit ${PYFDB_VERSION})
#
# To test, use FDB commands like:
# - fdb-schema
# - fdb-dump class=rd,expver=xxxx,stream=oper,time=0000
# - fdb list class=rd,expver=xxxx,stream=oper,time=0000

WORKDIR /app/
