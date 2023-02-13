# Pull nvidia cuda image for drivers
FROM nvidia/cuda:12.0.1-devel-ubuntu22.04 as base

# Define environment variables
ARG username=dock
ARG uid=1000
ARG gid=1000
ENV USER=$username
ENV UID=$uid
ENV GID=$gid
ENV HOME=/home/$USER
ENV DEBIAN_FRONTEND=noninteractive

# Install packages and clean up
RUN apt update && apt install -q -y --no-install-recommends --fix-missing --allow-change-held-packages \
    bzip2 \
    chroma curl cmake \
    default-jdk  default-jre\
    fzf \
    gcc g++ git gh \
    linux-headers-$(uname -r) libnccl2 libnccl-dev libcudnn8 libcudnn8-dev libnvinfer8 \
    libnvinfer-dev libnvinfer-plugin8 libnvinfer-plugin-dev \
    make maven \
    nano neofetch \
    nvidia-cuda-toolkit\
    openssl openssh-client \
    procps \
    sudo \
    unzip \
    wget \
    zip zoxide \
    && \
    apt full-upgrade -y && apt autoremove -y && apt clean -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*/apt/lists/*

# Install LSD for ls substitute
RUN wget https://github.com/Peltoche/lsd/releases/download/0.23.1/lsd_0.23.1_amd64.deb -P /tmp && \
    dpkg -i /tmp/lsd_0.23.1_amd64.deb && \
    rm /tmp/lsd_0.23.1_amd64.deb

# Create non-root user with UID and GID
RUN groupadd -g ${gid} ${username} && \
    useradd -u ${uid} -g ${gid} -s /bin/bash -m ${username} && \
    mkdir -p /etc/sudoers.d && \
    echo ${username} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${username} &&\
    chmod 0440 /etc/sudoers.d/${username}

# Create workspace directories
RUN mkdir -p ${HOME}/zsh_history && \
    chown ${USER}:${USER} ${HOME}/zsh_history && \
    mkdir ${HOME}/workspace && \
    chown ${USER}:${USER} ${HOME}/workspace

# Copy user configuration files
COPY --chown=${USER}:${USER} ./config/.aliases.sh ${HOME}/
COPY --chown=${USER}:${USER} ./config/.bashrc ${HOME}/
COPY --chown=${USER}:${USER} ./config/.nanorc ${HOME}/
COPY --chown=${USER}:${USER} ./config/download_tars.sh ${HOME}/

# set up nvidia libs
RUN chmod +x ${HOME}/download_tars.sh && \
    /bin/bash ${HOME}/download_tars.sh && \
    tar -xf ${HOME}/cudnn8.tar.xz  && \
    rm -f ${HOME}/cudnn8.tar.xz && \
    rm -f ${HOME}/download_tars.sh && \
    cp -r cudnn-linux-x86_64-8.6.0.163_cuda11-archive/include/* /usr/local/cuda/include/ && \
    cp -r cudnn-linux-x86_64-8.6.0.163_cuda11-archive/lib/* /usr/local/cuda/lib64/ && \
    rm -rf cudnn-linux-x86_64-8.6.0.163_cuda11-archive && \
    chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*

# Install ZSH and Oh My ZSH with Powerlevel10k theme through script
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -p git \
    -p git-extras \
    -p git-flow \
    -p command-not-found \
    -p colorize \
    -p common-aliases \
    -p sudo \
    -p zoxide \
    -p zsh-interactive-cd \
    -p battery \
    -p https://github.com/chrissicool/zsh-256color \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions \
    -p https://github.com/zshzoo/cd-ls \
    -p https://github.com/zdharma-continuum/fast-syntax-highlighting \
    -a "zstyle ':omz:update' mode auto" \
    -a "HIST_STAMPS='dd.mm.yyyy'" \
    -a "typeset -g POWERLEVEL9K_MODE=nerdfont-complete" \
    -a "compinit" \
    -a 'eval "$(zoxide init zsh)"' \
    -a "source ~/.aliases.sh" && \
    chsh -s /usr/bin/zsh ${USER}

# Add Conda to PATH
ENV CONDA_DIR=${HOME}/conda
ENV PATH=${CONDA_DIR}/bin:${PATH}

# Install Mambaforge and conda packages, then clean up and initialize zsh
RUN wget "https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh" -O ~/miniforge.sh && \
    chmod +x ~/miniforge.sh && \
    /bin/bash ~/miniforge.sh -b -p ${CONDA_DIR} &&  \
    rm ~/miniforge.sh && \
    mamba update --all --yes && \
    mamba install -n base -y \
    bandit black \
    cudatoolkit cudnn \
    ipython ipykernel isort \
    jupyter \
    matplotlib matplotlib-inline mypy \
    numpy \
    pandas pylint \
    seaborn \
    && \
    conda clean --tarballs --index-cache --packages --yes && \
    find ${CONDA_DIR} -follow -type f -name '*.a' -delete && \
    find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete && \
    conda clean --force-pkgs-dirs --all --yes  && \
    mamba init zsh && \
    conda config --set auto_activate_base true

# Switch to non-root user
USER ${USER}

WORKDIR ${HOME}
ENTRYPOINT [ "/bin/zsh" ]
CMD [ "-l" ]