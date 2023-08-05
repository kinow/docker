FROM mambaorg/micromamba:1.4.9-bullseye-slim

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

RUN mkdir -pv "${AUTOSUBMIT_ROOT_DIR}/{logs,metadata}" && \
    chown -R "${MAMBA_USER}:${MAMBA_USER}" "${AUTOSUBMIT_ROOT_DIR}"

USER "${MAMBA_USER}"

SHELL ["/usr/local/bin/_dockerfile_shell.sh"]
ENTRYPOINT ["/usr/local/bin/_entrypoint.sh"]

WORKDIR "${AUTOSUBMIT_ROOT_DIR}"

ENV ENV_NAME=base
ENV PATH "$MAMBA_ROOT_PREFIX/bin:$PATH"

RUN micromamba install --yes --name base --channel conda-forge \
      python=3.9.16 && \
    micromamba clean --all --yes && \
    /usr/local/bin/_activate_current_env.sh

RUN pip install autosubmit==4.0.84

RUN autosubmit configure \
      -db "${AUTOSUBMIT_ROOT_DIR}/database/" \
      -dbf autosubmit.db \
      -lr "${AUTOSUBMIT_ROOT_DIR}/experiments/" && \
    autosubmit install

CMD ["autosubmit"]
