FROM continuumio/miniconda3:latest AS condabuild
LABEL authors="Matt Bull" \
      description="Docker image containing all requirements for an Illumina ncov2019 pipeline"

COPY environments/extras.yml /extras.yml
COPY environments/illumina/environment.yml /environment.yml
RUN /opt/conda/bin/conda update conda && \
/opt/conda/bin/conda install mamba -c conda-forge && \
/opt/conda/bin/conda update mamba -c conda-forge && \
/opt/conda/bin/mamba env create -f /environment.yml

FROM alpine:3.11.5 
RUN apk add --no-cache git procps bash
COPY --from=condabuild /opt/conda/envs/artic-ncov2019-illumina /opt/conda/envs/artic-ncov2019-illumina
ENV PATH=/opt/conda/envs/artic-ncov2019-illumina/bin:$PATH
ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8
