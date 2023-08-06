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

# micromamba docs, https://micromamba-docker.readthedocs.io/en/latest/advanced_usage.html#adding-micromamba-to-an-existing-docker-image

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

# end micromamba

# micromamba docs, https://micromamba-docker.readthedocs.io/en/latest/advanced_usage.html#changing-the-user-id-or-name

ARG NEW_MAMBA_USER=autosubmit
ARG NEW_MAMBA_USER_ID=1000
ARG NEW_MAMBA_USER_GID=1000
USER root

RUN if grep -q '^ID=alpine$' /etc/os-release; then \
      # alpine does not have usermod/groupmod
      apk add --no-cache --virtual temp-packages shadow=4.13-r0; \
    fi && \
    usermod "--login=${NEW_MAMBA_USER}" "--home=/home/${NEW_MAMBA_USER}" \
        --move-home "-u ${NEW_MAMBA_USER_ID}" "${MAMBA_USER}" && \
    groupmod "--new-name=${NEW_MAMBA_USER}" \
        "-g ${NEW_MAMBA_USER_GID}" "${MAMBA_USER}" && \
    if grep -q '^ID=alpine$' /etc/os-release; then \
      # remove the packages that were only needed for usermod/groupmod
      apk del temp-packages; \
    fi && \
    # Update the expected value of MAMBA_USER for the
    # _entrypoint.sh consistency check.
    echo "${NEW_MAMBA_USER}" > "/etc/arg_mamba_user" && \
    :
ENV MAMBA_USER=$NEW_MAMBA_USER
USER $MAMBA_USER

# end micromamba

WORKDIR "${AUTOSUBMIT_ROOT_DIR}"

ENV ENV_NAME=base
ENV PATH "$MAMBA_ROOT_PREFIX/bin:/usr/bin:/usr/local/bin:$PATH"

# The directories the container will use for Autosubmit files.
USER root
RUN mkdir -pv "${AUTOSUBMIT_ROOT_DIR}/logs" && \
    mkdir -pv "${AUTOSUBMIT_ROOT_DIR}/metadata" && \
    chown -R "${MAMBA_USER}:${MAMBA_USER}" "${AUTOSUBMIT_ROOT_DIR}"

# update-ca-certificates is for: libmamba No CA certificates found on system.
# openssh-server is for: Autosubmit platforms.
RUN apt update && \
    apt install -y ca-certificates \
    openssh-server \
    sudo && \
    update-ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER $MAMBA_USER

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

# SSH
USER root

RUN chown -R "${MAMBA_USER}:${MAMBA_USER}" "/home/${MAMBA_USER}/" && \
    chmod 0755 "/home/${MAMBA_USER}/" && \
    mkdir -pv /var/run/sshd && \
    mkdir -pv "/home/$MAMBA_USER/.ssh" && \
    chmod 0700 "/home/$MAMBA_USER/.ssh" && \
    touch "/home/$MAMBA_USER/.ssh/authorized_keys" && \
    chmod 600 "/home/$MAMBA_USER/.ssh/authorized_keys" && \
    chown -R "${MAMBA_USER}:${MAMBA_USER}" "/home/$MAMBA_USER/.ssh" && \
    usermod -a -G sudo "${MAMBA_USER}" && \
    sed -i "s/^%sudo.*$/%sudo ALL=(ALL:ALL) NOPASSWD:ALL/g" /etc/sudoers && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config && \
    sed -i 's/#SyslogFacility/SyslogFacility/g' /etc/ssh/sshd_config && \
    sed -i 's/#LogLevel INFO/LogLevel VERBOSE/g' /etc/ssh/sshd_config && \
    env | egrep -v "^(HOME=|USER=|MAIL=|LC_ALL=|LS_COLORS=|LANG=|HOSTNAME=|PWD=|TERM=|SHLVL=|LANGUAGE=|_=)" >> /etc/environment

#COPY ssh-keys/ /keys/
#RUN cat /keys/ssh_test.pub >> /home/nf2/.ssh/authorized_keys

ARG MAMBA_EXE
ENV MAMBA_EXE="${MAMBA_EXE}"
USER $MAMBA_USER

EXPOSE 22

ENTRYPOINT sudo service ssh restart && /bin/bash
CMD autosubmit
