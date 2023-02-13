# dockgpu-template

Template repository to create a container with GPU support and CUDA, cuDNN, TensorRT libraries. The container is based on the [nvidia/cuda](https://hub.docker.com/r/nvidia/cuda) image.

> **Note:** This template is intended to be used with `vscode-dev-containers` extension.

## Included Features

ZSH is installed and made as the default shell. Some useful plugins are installed such as:

    - Syntax-Highlighting
    - Autosuggestions
    - Interactive-cd
    - Zoxide

Other installed tools are:

    - Git, gh
    - gcc, g++, make
    - jdk, jre, maven
    - openssl, curl, wget
    - mamba, conda

## Usage

To use this template edit theese files as described:

- `.env`: Set the container, image and user name.
- `devcontainer.json`: Substitute `dockgpu` as you want and `dock` username with the one you set in `.env` file. Theese are needed to set correctly the vscode remote container extension.
- `Dockerfile`: Substitute `dock` username with the one you set in `.env` file.

### Notes

Inside the dockerfile you can find a `RUN` command to launch a script that downloads a file froma a OneDrive link. This is a workaraund to get the `cudnn.h` file that seems to be not available on the official nvidia/cuda image.

Installed python packages are listed in the last `RUN` command of the dockerfile. You can add or remove packages as you need.
