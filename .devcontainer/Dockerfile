# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.158.0/containers/python-3/.devcontainer/base.Dockerfile

# [Choice] Python version: 3, 3.9, 3.8, 3.7, 3.6
ARG VARIANT="3"
FROM mcr.microsoft.com/vscode/devcontainers/python:0-${VARIANT}

# [Option] Install Azure CLI
ARG INSTALL_AZURE_CLI="true"

# [Option] Install Azure Bicep
ARG INSTALL_AZURE_BICEP="true"

# [Option] Install zsh
ARG INSTALL_ZSH="true"

# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="false"

# [Option] Enable non-root Docker access in container
ARG ENABLE_NONROOT_DOCKER="true"

# [Option] Use the OSS Moby CLI instead of the licensed Docker CLI
ARG USE_MOBY="true"

ARG LINUX_VERSION=10
ARG LINUX_NAME=debian

# [Option] Install Node.js
ARG INSTALL_NODE="true"
ARG NODE_VERSION="lts/*"
RUN if [ "${INSTALL_NODE}" = "true" ]; then su vscode -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"; fi

# Install .NET 5.0
RUN if [ "${INSTALL_AZURE_BICEP}" = "true" ]; then wget https://packages.microsoft.com/config/${LINUX_NAME}/${LINUX_VERSION}/packages-microsoft-prod.deb -O packages-microsoft-prod.deb && \
    sudo dpkg -i packages-microsoft-prod.deb; fi

RUN if [ "${INSTALL_AZURE_BICEP}" = "true" ]; then sudo apt-get update && \
    sudo apt-get install -y apt-transport-https && \
    sudo apt-get update && \
    sudo apt-get install -y dotnet-sdk-5.0; fi

RUN if [ "${INSTALL_AZURE_BICEP}" = "true" ]; then sudo apt-get update && \
    sudo apt-get install -y apt-transport-https && \
    sudo apt-get update && \
    sudo apt-get install -y aspnetcore-runtime-5.0; fi

# Install Azure Bicep packages.
RUN if [ "${INSTALL_AZURE_BICEP}" = "true" ]; then curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-linux-x64 && \
    chmod +x ./bicep && \
    sudo mv ./bicep /usr/local/bin/bicep && \
    bicep --help; fi

# Install sndfile-tools to process .wav files
RUN sudo apt-get update && \
    sudo apt-get install -y sndfile-tools

# Install ffmpeg libsm6 libxext6 to use OpenCV libraries
# RUN sudo apt-get update && \
#     sudo apt-get install -y ffmpeg libsm6 libxext6

# Install needed packages and setup non-root user. Use a separate RUN statement to add your
# own dependencies. A user of "automatic" attempts to reuse an user ID if one already exists.
ARG USERNAME=automatic
ARG USER_UID=1000
ARG USER_GID=$USER_UID
COPY library-scripts/*.sh /tmp/library-scripts/

# Install Azure CLI packages.
RUN if [ "${INSTALL_AZURE_CLI}" = "true" ]; then bash /tmp/library-scripts/azcli-debian.sh \
    && apt-get clean -y; fi

RUN apt-get update \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    # Use Docker script from script library to set things up
    && /bin/bash /tmp/library-scripts/docker-debian.sh "${ENABLE_NONROOT_DOCKER}" "/var/run/docker-host.sock" "/var/run/docker.sock" "${USERNAME}" \
    # Clean up
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts/

# Upgrade pip.
RUN python3 -m pip install --upgrade pip 

# Install ipykernel.
RUN pip install -U ipykernel

# Install nbconvert.
RUN git clone https://github.com/jupyter/nbconvert.git && \
    pip install -e ./nbconvert && \
    pip install nbconvert[test]

# Setting the ENTRYPOINT to docker-init.sh will configure non-root access 
# to the Docker socket. The script will also execute CMD as needed.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>
