FROM mambaorg/micromamba:1.4.9

ARG MAMBA_DOCKERFILE_ACTIVATE=1

# Basic requirements
RUN micromamba install --yes --name base --channel conda-forge \
      git && \
    micromamba clean --all --yes

# Clone SUNSET repository
RUN git clone -o origin --depth 1 https://earth.bsc.es/gitlab/es/sunset.git

# Create mamba environment
RUN micromamba env create -f sunset/conda_installation/environment-sunset.yml
