FROM ubuntu:20.04

ARG DEBIAN_FRONTEND=noninteractive

ENV TZ=America/Detriot

RUN apt-get update && apt-get install -y \
    apt-transport-https \
    ca-certificates \
    cdpr \
    curl \
    dnsutils \
    dsniff \
    ipcalc \
    iperf \
    iperf3 \
    fping \
    git-all \
    gnupg \
    gsutil \
    ifenslave \
    inetutils-traceroute \
    iputils-* \
    libkrb5-dev \
    lldpd \
    locales \
    mtr \
    nano \
    net-tools \
    netplan.io \
    openssh-server \
    python3 \
    python3-pip \
    snapd \
    sudo \
    tzdata \
    ufw \
    vim \
    wget

RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

RUN echo "deb https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

COPY requirements.txt requirements.txt

COPY requirements.yml requirements.yml

RUN pip3 install -r requirements.txt

RUN apt-get update && apt-get install -y google-cloud-cli

RUN ansible-galaxy collection install -r requirements.yml --force

RUN echo 'alias python="python3"' >> ~/.bashrc

RUN echo 'alias pip="pip3"' >> ~/.bashrc

RUN curl -fsSL https://get.pulumi.com | sh

LABEL maintainer="Mitch Vaughan <mitch@arista.com>" \
      version="2.0.1"
