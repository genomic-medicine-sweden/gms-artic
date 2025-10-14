FROM alpine:3.11.5

LABEL authors="Matt Bull" \
      description="Docker image containing all requirements for an Illumina ncov2019 pipeline"

# 1️⃣ Install Alpine dependencies
RUN apk add --no-cache \
    bash \
    curl \
    git \
    procps \
    bzip2 \
    xz \
    wget \
    ca-certificates \
    libstdc++

# 2️⃣ Install glibc (no ldconfig!)
ENV GLIBC_VERSION=2.31-r0
RUN wget -q -O /tmp/glibc.apk \
      https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-${GLIBC_VERSION}.apk && \
    apk add --allow-untrusted /tmp/glibc.apk && \
    rm /tmp/glibc.apk

# 3️⃣ Make glibc visible to the system
ENV LD_LIBRARY_PATH=/usr/glibc-compat/lib:$LD_LIBRARY_PATH

# 4️⃣ Install Miniforge (uses glibc now available)
ENV CONDA_DIR=/opt/conda
ENV PATH=$CONDA_DIR/bin:$PATH
RUN wget --quiet https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh \
      -O /tmp/miniforge.sh && \
    bash /tmp/miniforge.sh -b -p $CONDA_DIR && \
    rm /tmp/miniforge.sh && \
    conda update -y conda && \
    conda install -y mamba -c conda-forge && \
    conda update -y mamba

# 5️⃣ Copy environment file and create environment
COPY environments/illumina/environment.yml /environment.yml
RUN mamba env create -f /environment.yml

# 6️⃣ Set environment
ENV PATH=$CONDA_DIR/envs/artic-ncov2019-illumina/bin:$PATH
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

# 7️⃣ Activate environment by default
SHELL ["conda", "run", "-n", "artic-ncov2019-illumina", "/bin/bash", "-c"]

