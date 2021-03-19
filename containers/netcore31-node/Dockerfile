# Preconfigured deccontainer docker image for development with .NET Core 3.1 and/or Node LTE
# This container definition is base on Microsoft devcontainers: https://github.com/microsoft/vscode-dev-containers
# And also my experience working with VS Code Remote Development

FROM mcr.microsoft.com/dotnet/sdk:3.1

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create a dedicated user for nvm as it can be shared by vscode and root users
ARG USER_NVM_NAME=nvm
ARG USER_NVM_UID=1100
ARG USER_NVM_GID=$USER_NVM_UID

# [Optional] Version of Node.js to install.
ARG INSTALL_NODE="true"
ARG NODE_VERSION="lts/*"

# [Optional] Install the Azure CLI
ARG INSTALL_AZURE_CLI="true"

# Docker Compose version
ARG COMPOSE_VERSION=1.28.5

ENV \
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=1 \
    NVM_DIR=/home/$USER_NVM_NAME/.nvm \
    # Avoid questions during Angular npm install
    NG_ANALYTICS=0 \
    # Avoid warnings by switching to noninteractive
    DEBIAN_FRONTEND=noninteractive

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog rsync 2>&1 \
    ## install locales
    && apt-get -y install --no-install-recommends locales 2>&1 \
    && update-locale LANG=C.UTF-8 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git bzip2 openssh-client less iproute2 procps apt-transport-https gnupg2 curl lsb-release psmisc \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # Non sudo docker acces for USERNAME
    && groupadd -g 800 docker \
    && usermod -a -G docker $USERNAME \
    #
    # Create a non-root nvm for shared nvm user / root usage.
    && groupadd --gid $USER_NVM_GID $USER_NVM_NAME \
    && useradd -s /bin/bash --uid $USER_NVM_UID --gid $USER_NVM_GID -m $USER_NVM_NAME \
    && usermod -a -G $USER_NVM_NAME $USERNAME \
    && usermod -a -G $USER_NVM_NAME root \
    # Disable nvm login
    && passwd -l $USER_NVM_NAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    # Avoiding extension reinstalls on container rebuild (Remote Docker)
    && mkdir -p /home/$USERNAME/.vscode-server/extensions \
       /home/$USERNAME/.vscode-server-insiders/extensions \
       # For root also
       /root/.vscode-server/extensions \
       /root/.vscode-server-insiders/extensions \
    # Set owner to .vscode-serverXYZ folders
    && chown -R $USERNAME:$USERNAME \
        /home/$USERNAME \
    # Set dotnet tools user path
    && echo 'export PATH="~/.dotnet/tools:$PATH"'> /etc/profile.d/dotnet.sh \
    #
    # Install Docker CE CLI
    && apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common lsb-release \
    && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | (OUT=$(apt-key add - 2>&1) || echo $OUT) \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    #
    # Install Docker Compose
    && curl -sSL "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose \
    #
    # Install Dapr side car
    && wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash \
    #
    # [Optional] Install Node.js for ASP.NET Core Web Applicationss
    && if [ "$INSTALL_NODE" = "true" ]; then \
        #
        # Install nvm and Node
        mkdir -p ${NVM_DIR} \
        && chmod -R g+rwx /home/$USER_NVM_NAME \
        && curl -so- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash 2>&1 \
        && /bin/bash -c "source $NVM_DIR/nvm.sh \
            && nvm install ${NODE_VERSION} \
            && nvm alias default ${NODE_VERSION} \
            && npm install -g npm typescript yarn 2>&1" \
        # Set owner and file access to nvm folders
        && chown -R $USER_NVM_NAME:$USER_NVM_NAME /home/$USER_NVM_NAME \
        && chmod -R g+w "$NVM_DIR" \
        && chmod a+r "$NVM_DIR/nvm.sh" \
        && chmod a+r "$NVM_DIR/bash_completion" \
        && echo "export NVM_DIR='$NVM_DIR'" > /etc/profile.d/nvm.sh \
        && echo '[ -s "$NVM_DIR/nvm.sh" ] && source "$NVM_DIR/nvm.sh" || true' >> /etc/profile.d/nvm.sh \
        && echo '[ -s "$NVM_DIR/bash_completion" ] && source "$NVM_DIR/bash_completion" || true' >> /etc/profile.d/nvm.sh \
        && chmod 644 /etc/profile.d/nvm.sh \
        # Set node modules path
        && echo 'export PATH="./node_modules/.bin:$PATH"' > /etc/profile.d/node_modules.sh; \
        #
        # Install yarn
        # && curl -sS https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/pubkey.gpg | apt-key add - 2>/dev/null \
        # && echo "deb https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
        # && apt-get update \
        # && apt-get -y install --no-install-recommends yarn; \
    fi \
    #
    # [Optional] Install the Azure CLI
    && if [ "$INSTALL_AZURE_CLI" = "true" ]; then \
        echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/azure-cli.list \
        && curl -sL https://packages.microsoft.com/keys/microsoft.asc | apt-key add - 2>/dev/null \
        && apt-get update \
        && apt-get install -y azure-cli; \
    fi \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Debian containers have a bug supporting locales in containers, so we use C.UTF-8 because some app need it.
ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8
ENV LANGUAGE C.UTF-8

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog
