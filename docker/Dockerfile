# Base docker with miniconda
FROM docker.io/continuumio/miniconda3:latest

# Connect this docker image to the biobb_workflows repository
LABEL org.opencontainers.image.source https://github.com/mmb-irb/MDDB-workflow

# Define working dir
WORKDIR /app

# Install libgl1 for avoiding problems with VMD
RUN apt-get update && \
    apt-get install -y libgl1

# Copy repo source
COPY . /app/MDDB-workflow

# Enable libmamba as solver
RUN conda config --set solver libmamba

# Create new environment
RUN conda env create --file=/app/MDDB-workflow/envs/environment.yml && conda clean -afy

# Install mwf SW
WORKDIR /app/MDDB-workflow
RUN conda run -n mwf_env python -m pip install -e .

ENV MWF_ENV_PATH=/opt/conda/envs/mwf_env
RUN conda install -n mwf_env -y -c conda-forge \
      cmake>=3.28 make pocl gxx_linux-64 gcc_linux-64 perl libgomp && \
    conda clean -afy

RUN conda run --no-capture-output -n mwf_env bash -c '\
    set -e && \
    git clone --depth 1 --branch pj_mddb-json-proto-2026 \
      https://gitlab.com/gromacs/gromacs.git /tmp/gromacs && \
    mkdir /tmp/gromacs/build && cd /tmp/gromacs/build && \
    cmake .. \
      -DCMAKE_BUILD_TYPE=Release \
      -DSHARED_LIBS_DEFAULT=ON -DBUILD_SHARED_LIBS=ON -DGMX_PREFER_STATIC_LIBS=NO \
      -DGMX_BUILD_OWN_FFTW=OFF -DREGRESSIONTEST_DOWNLOAD=OFF \
      -DCMAKE_PREFIX_PATH="${CONDA_PREFIX}" -DCMAKE_INSTALL_PREFIX="${CONDA_PREFIX}" \
      -DCMAKE_INSTALL_BINDIR="bin/gromacs_mddb" -DCMAKE_INSTALL_LIBDIR="lib" \
      -DGMX_VERSION_STRING_OF_FORK="conda-forge" \
      -DGMXAPI=OFF -DGMX_INSTALL_LEGACY_API=OFF -DGMX_INSTALL_NBLIB_API=OFF \
      -DGMX_USE_RDTSCP=OFF -DGMX_DEFAULT_SUFFIX=OFF \
      -DGMX_BINARY_SUFFIX="_mddb" -DGMX_LIBS_SUFFIX="_mddb" && \
    make -j$(nproc) && make install && rm -rf /tmp/gromacs'

RUN conda run -n mwf_env gmx_mddb -version

# Define working dir
WORKDIR /app

# Export python path
ENV PYTHONPATH="${PYTHONPATH}:/app/MDDB-workflow"

# Code to run when container is started
RUN echo '#!/bin/sh' > entrypoint.sh && \
    echo 'conda run --no-capture-output -n mwf_env "$@"' >> entrypoint.sh && \
    chmod +x entrypoint.sh

# Serve the app
ENTRYPOINT ["/app/entrypoint.sh"]
