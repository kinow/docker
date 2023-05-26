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

FROM mambaorg/micromamba:bullseye-slim as micromamba

FROM debian:bookworm-slim

COPY --from=deps /opt/ecbuild /opt/ecbuild
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

RUN /usr/local/bin/_dockerfile_initialize_user_accounts.sh && \
    /usr/local/bin/_dockerfile_setup_root_prefix.sh

RUN apt-get update && apt-get install -y ca-certificates fdb5

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
	eccodes==2.30.0 \
    python==3.10 \
    git && \
    micromamba clean --all --yes

RUN python3 -m pip install --upgrade \
    git+https://github.com/ecmwf/pyfdb.git@02692523ae83b6667dbfa189bbc49d37c3a14a0f

ENV PATH="$PATH:/opt/ecbuild/bin/:/opt/eckit/bin/:/opt/fdb/bin"
# ENV LD_LIBRARY_PATH="/opt/eckit/lib/:/opt/fdb/lib/"
ENV FDB5_CONFIG_FILE=/app/config.yaml

WORKDIR /
