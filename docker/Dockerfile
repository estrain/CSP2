# Use Biocontainers Base Image
FROM biocontainers/base AS build

ARG CSP2_VER="0.9.0"
ARG BEDTOOLS_VER="2.31.1"
ARG MUMMER_VER="4.0.0"
ARG SKESA_VER="2.4.0"
ARG MASH_VER="2.3"
ARG BBMAP_VER="38.90"
ARG PYTHON_VER="3.8"

USER root  # Switch to root for installation

WORKDIR /build

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    zlib1g-dev \
    libghc-bzlib-dev \
    liblzma-dev \
    wget \
    ca-certificates \
    openjdk-8-jre-headless \
    curl \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set up Python Environment
RUN conda create -y -n csp2_env python=${PYTHON_VER} && \
    echo "source activate csp2_env" >> ~/.bashrc && \
    /opt/conda/envs/csp2_env/bin/pip install --no-cache-dir \
        pandas~=1.2.0 pybedtools refchooser scikit-learn

# Install BEDTools
RUN wget -qO- https://github.com/arq5x/bedtools2/archive/refs/tags/v${BEDTOOLS_VER}.tar.gz | tar xz && \
    cd bedtools2-${BEDTOOLS_VER} && make && make install

# Install MUMmer
RUN wget -qO- https://github.com/mummer4/mummer/releases/download/v${MUMMER_VER}rc1/mummer-${MUMMER_VER}rc1.tar.gz | tar xz && \
    cd mummer-${MUMMER_VER}rc1 && ./configure --prefix=/usr/local && make && make install

# Install Skesa
RUN mkdir -p /usr/local/bin/skesa && cd /usr/local/bin/skesa && \
    wget -q https://github.com/ncbi/SKESA/releases/download/${SKESA_VER}/skesa.centos.7.7 -O skesa && \
    wget -q https://github.com/ncbi/SKESA/releases/download/${SKESA_VER}/gfa_connector.centos7.7 -O gfa_connector && \
    wget -q https://github.com/ncbi/SKESA/releases/download/${SKESA_VER}/kmercounter.centos7.7 -O kmercounter && \
    chmod +x skesa gfa_connector kmercounter

# Install Mash
RUN wget -qO- https://github.com/marbl/Mash/releases/download/v${MASH_VER}/mash-Linux64-v${MASH_VER}.tar | tar x && \
    mv mash-Linux64-v${MASH_VER}/mash /usr/local/bin

# Install BBMap
RUN wget -O BBMap_${BBMAP_VER}.tar.gz https://sourceforge.net/projects/bbmap/files/BBMap_${BBMAP_VER}.tar.gz/download && \
    tar -xvf BBMap_${BBMAP_VER}.tar.gz && \
    mv bbmap/* /usr/local/bin

FROM biocontainers/base AS app

ARG CSP2_VER
ARG PYTHON_VER

USER biodocker  # Use biodocker user (from Biocontainers)

WORKDIR /app

# Copy built binaries and dependencies from build stage
COPY --from=build /usr/local/bin /usr/local/bin
COPY --from=build /usr/local/lib /usr/local/lib
COPY --from=build /usr/local/share /usr/local/share
COPY --from=build /opt/conda /opt/conda
COPY --from=build /opt/venv /opt/venv

# Set up environment variables
ENV PATH="/opt/venv/bin:/usr/local/bin:/usr/local/libexec/mummer:/skesa:$PATH" \
    LC_ALL=C \
    NXF_OFFLINE='true'

# Install Nextflow
RUN curl -fsSL https://get.nextflow.io | bash && \
    chmod +x nextflow && mv nextflow /usr/local/bin

# Add application files
ADD bin ./bin
ADD conf ./conf
ADD subworkflows ./subworkflows
ADD CSP2.nf ./CSP2.nf
ADD nextflow.config ./nextflow.config

FROM app AS release

# Set final environment variables
ENV CSP2_VER=${CSP2_VER}
ENV BEDTOOLS_VER=${BEDTOOLS_VER}
ENV MUMMER_VER=${MUMMER_VER}
ENV SKESA_VER=${SKESA_VER}
ENV MASH_VER=${MASH_VER}
ENV BBMAP_VER=${BBMAP_VER}
ENV PYTHON_VER=${PYTHON_VER}

# Default command: Open a bash shell
CMD ["/bin/bash"]
