FROM mambaorg/micromamba:2.3.0-ubuntu22.04 AS app

# build and run as root users since micromamba image has 'mambauser' set as the $USER
USER root
# set workdir to default for building; set to /data at the end
WORKDIR /

# ARG variables only persist during build time
# had to include the v for some of these due to GitHub tags.
# using pangolin-data github tag, NOT what is in the GH release title "v1.2.133"
ARG PANGOLIN_VER
ARG PANGOLIN_DATA_VER
ARG SCORPIO_VER
ARG CONSTELLATIONS_VER
ARG USHER_VER

# metadata labels
LABEL base.image="mambaorg/micromamba:2.3.0-ubuntu22.04"
LABEL dockerfile.version="1"
LABEL software="pangolin"
LABEL software.version=${PANGOLIN_VER}
LABEL description="Conda environment for Pangolin. Pangolin: Software package for assigning SARS-CoV-2 genome sequences to global lineages."
LABEL website="https://github.com/cov-lineages/pangolin"
LABEL license="GNU General Public License v3.0"
LABEL license.url="https://github.com/cov-lineages/pangolin/blob/master/LICENSE.txt"
LABEL maintainer="Curtis Kapsak"
LABEL maintainer.email="kapsakcj@gmail.com"

# install dependencies; cleanup apt garbage
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget \
    ca-certificates \
    git \
    procps \
    curl \
    jq \
    bsdmainutils && \
    apt-get autoclean && rm -rf /var/lib/apt/lists/*

# Get the pangolin repo
RUN PANGOLIN_VER=$(curl -s https://api.github.com/repos/cov-lineages/pangolin/releases/latest | jq -r '.tag_name') \
    && echo "Using Pangolin version: $PANGOLIN_VER" \
    && wget -O pangolin.tar.gz "https://github.com/cov-lineages/pangolin/archive/refs/tags/${PANGOLIN_VER}.tar.gz" \
    && tar -xzf pangolin.tar.gz \
    && rm pangolin.tar.gz \
    && mv pangolin-* pangolin

# set the environment; PATH is unnecessary here, but leaving anyways. It's reset later in dockerfile
ENV PATH="$PATH" \
    LC_ALL=C.UTF-8

# modify environment.yml to pin specific versions during install
# pin specific versions of usher, scorpio, pangolin-data, constellations, and pulp
# create the conda environment using modified environment.yml
# line to remove "defaults" channel to ensure that it isn't used due to Anaconda's recent ToS changes
RUN sed -i "s|usher.*|usher=${USHER_VER}|" /pangolin/environment.yml && \
    sed -i "s|scorpio.git|scorpio.git@${SCORPIO_VER}|" /pangolin/environment.yml && \
    sed -i "s|pangolin-data.git|pangolin-data.git@${PANGOLIN_DATA_VER}|" /pangolin/environment.yml && \
    sed -i "s|constellations.git|constellations.git@${CONSTELLATIONS_VER}|" /pangolin/environment.yml && \
    sed -i "12 a\  - pulp=2.7.0" /pangolin/environment.yml && \
    sed -i '/.*defaults/d' /pangolin/environment.yml && \
    micromamba create -n pangolin -y -f /pangolin/environment.yml && \
    micromamba clean -a -y -f

# hardcode pangolin executable into the PATH variable
ENV PATH="${PATH}:/opt/conda/envs/pangolin/bin/" XDG_CACHE_HOME=/tmp

WORKDIR /pangolin

# run pip install step; download optional pre-computed assignment hashes for UShER (useful for running on large batches of samples)
# best to skip using the assigment-cache if running on one sample for speed
# print versions
RUN pip install . && \
    pangolin --add-assignment-cache && \
    mkdir /data && \
    pangolin --all-versions && \
    usher --version

# final working directory in "app" layer is /data for passing data in/out of container
WORKDIR /data

# default command is to pull up help options for pangolin; can be overridden of course
CMD ["pangolin", "-h"]
