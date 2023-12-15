FROM mambaorg/micromamba:1.4.9

# Basic requirements
RUN micromamba install -y -n base -c conda-forge \
      git

# Close SUNSET repository
RUN git clone -o origin --depth 1 https://earth.bsc.es/gitlab/es/sunset.git

# Create mamba environment
RUN micromamba env create -f sunset/conda_installation/environment-sunset.yml 
