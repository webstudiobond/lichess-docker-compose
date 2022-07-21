FROM debian:stable-slim

ARG TZ=Europe/Kiev
ARG LC=en_US.UTF-8

SHELL ["/bin/bash", "-c"]

RUN useradd -ms /bin/bash lichess && \
    apt update && \
    apt install -y sudo && \
    # Disable sudo login for the new lichess user.
    echo "lichess ALL = NOPASSWD : ALL" >> /etc/sudoers

ENV TZ=$TZ
RUN sudo ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Run as a non-privileged user.
USER lichess

# additional packeges
RUN sudo apt update && \
    sudo apt install -y \
    gnupg \
    ca-certificates \
    apt-transport-https \
    apt-utils \
    curl \
    gnupg \
    ca-certificates \
    unzip \
    zip \
    parallel \
    git

# Use UTF-8 encoding.
# fix Cannot set LC to default locale: No such file or directory
RUN sudo apt update && \
    sudo apt install -y \
    locales && \
    echo "LC_ALL=en_US.UTF-8" | sudo tee -a /etc/environment && \
    echo "en_US.UTF-8 UTF-8" | sudo tee -a /etc/locale.gen && \
    echo "LANG=en_US.UTF-8" | sudo tee -a /etc/locale.conf && \
    sudo locale-gen en_US.UTF-8

ENV LANG "en_US.UTF-8"
ENV LC_CTYPE "en_US.UTF-8"
ENV LANGUAGE "en_US.UTF-8"

# nvm => node => yarn
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash && \
    export NVM_DIR="$HOME/.nvm" && \
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
    nvm install 16 && \
    nvm use 16 && \
    npm install --global yarn

# Java
# fix E: unable to locate
RUN sudo mkdir -p /usr/share/man/man1 /usr/share/man/man2
# install
RUN sudo apt update && \
    sudo apt install -y --no-install-recommends openjdk-17-jdk-headless openjdk-17-jre-headless
# Prints installed java version, just for checking
RUN java --version

# sbt
RUN echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list && \
    echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list && \
    curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add && \
    sudo apt update && \
    sudo apt install sbt

# Cleanup
RUN sudo apt autoremove -y && \
    sudo apt clean

# Silence the parallel citation warning
RUN sudo mkdir -p ~/.parallel && sudo touch ~/.parallel/will-cite

ENV HOME /home/lichess
WORKDIR /home/lichess
