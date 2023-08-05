FROM mambaorg/micromamba:1.4.9-bullseye-slim AS micromamba

FROM debian:bullseye-slim

# Build:
# docker build -t kinow/autosubmit:4.0.84-bullseye-slim -t kinow/autosubmit:latest .
#
# Run:
# docker run --rm kinow/autosubmit:latest autosubmit --version
#
# Run with volumes:
#
# Create the directories to hold DB and experiments somewhere
# mkdir -pv /tmp/autosubmit/{database,experiments}
# 
# Create an external DB, for example:
# docker run --rm -v /tmp/autosubmit/database:/app/autosubmit/database -v /tmp/autosubmit/experiments:/app/autosubmit/experiments kinow/autosubmit:latest autosubmit install
#
# Create a dummy experiment
# docker run --rm -v /tmp/autosubmit/database:/app/autosubmit/database -v /tmp/autosubmit/experiments:/app/autosubmit/experiments kinow/autosubmit:latest autosubmit expid -H local -d test --dummy
#
# Confirm any container created with the image can access the experiments
# docker run --rm -v /tmp/autosubmit/database:/app/autosubmit/database -v /tmp/autosubmit/experiments:/app/autosubmit/experiments kinow/autosubmit:latest autosubmit describe
#
# To delete an experiment (-ti if you do not pass `-f`):
# docker run --rm -ti -v /tmp/autosubmit/database:/app/autosubmit/database -v /tmp/autosubmit/experiments:/app/autosubmit/experiments kinow/autosubmit:latest autosubmit delete -f a000


ARG AUTOSUBMIT_ROOT_DIR=/app/autosubmit/

USER root

# For:
# libmamba No CA certificates found on system
RUN apt update && \
    apt install -y ca-certificates && \
    update-ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# From micromamba docs, https://micromamba-docker.readthedocs.io/en/latest/advanced_usage.html#adding-micromamba-to-an-existing-docker-image
# --- x snip x ---

# if your image defaults to a non-root user, then you may want to make the
# next 3 ARG commands match the values in your image. You can get the values
# by running: docker run --rm -it my/image id -a
ARG MAMBA_USER=autosubmit
ARG MAMBA_USER_ID=1000
ARG MAMBA_USER_GID=1000
ENV MAMBA_USER=$MAMBA_USER
ENV MAMBA_ROOT_PREFIX="/opt/conda"
ENV MAMBA_EXE="/bin/micromamba"

COPY --from=micromamba "$MAMBA_EXE" "$MAMBA_EXE"
COPY --from=micromamba /usr/local/bin/_activate_current_env.sh /usr/local/bin/_activate_current_env.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_shell.sh /usr/local/bin/_dockerfile_shell.sh
COPY --from=micromamba /usr/local/bin/_entrypoint.sh /usr/local/bin/_entrypoint.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_initialize_user_accounts.sh /usr/local/bin/_dockerfile_initialize_user_accounts.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_setup_root_prefix.sh /usr/local/bin/_dockerfile_setup_root_prefix.sh

RUN /usr/local/bin/_dockerfile_initialize_user_accounts.sh && \
    /usr/local/bin/_dockerfile_setup_root_prefix.sh

USER $MAMBA_USER

SHELL ["/usr/local/bin/_dockerfile_shell.sh"]

ENTRYPOINT ["/usr/local/bin/_entrypoint.sh"]
# Optional: if you want to customize the ENTRYPOINT and have a conda
# environment activated, then do this:
# ENTRYPOINT ["/usr/local/bin/_entrypoint.sh", "my_entrypoint_program"]
# --- x snip x ---

# The directories the container will use for Autosubmit files.
USER root
RUN mkdir -pv "${AUTOSUBMIT_ROOT_DIR}/{logs,metadata}" && \
    chown -R "${MAMBA_USER}:${MAMBA_USER}" "${AUTOSUBMIT_ROOT_DIR}"
USER $MAMBA_USER

WORKDIR "${AUTOSUBMIT_ROOT_DIR}"

ENV ENV_NAME=base
ENV PATH "$MAMBA_ROOT_PREFIX/bin:$PATH"

# Install Python.
RUN micromamba install --yes --name base --channel conda-forge \
      python=3.9.16 && \
    micromamba clean --all --yes && \
    /usr/local/bin/_activate_current_env.sh

# Install Autosubmit.
RUN pip install autosubmit==4.0.84

# Configure Autosubmit.
RUN autosubmit configure \
      -db "${AUTOSUBMIT_ROOT_DIR}/database/" \
      -dbf autosubmit.db \
      -lr "${AUTOSUBMIT_ROOT_DIR}/experiments/" && \
    autosubmit install

CMD ["autosubmit"]
