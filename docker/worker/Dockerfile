# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        git \
        gnupg-agent \
        python3-pip \
        software-properties-common

# Install Docker.
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" && \
    apt-get install -y docker-ce

# Install gcloud
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - && \
    apt-get update && apt-get install -y google-cloud-sdk

# Install gVisor.
RUN curl -fsSL https://gvisor.dev/archive.key | apt-key add - && \
    add-apt-repository "deb https://storage.googleapis.com/gvisor/releases master main" && \
    apt-get update && apt-get install -y runsc

# Make gVisor the default Docker runtime.
COPY daemon.json /etc/docker/daemon.json

# Use newer libgit2 to avoid a bug with cloning from Gerrit
# https://github.com/libgit2/libgit2/pull/5536
RUN apt-get install -y cmake build-essential wget libssl-dev && \
    wget https://github.com/libgit2/libgit2/releases/download/v1.1.0/libgit2-1.1.0.tar.gz && \
    tar xzf libgit2-1.1.0.tar.gz && \
    cd libgit2-1.1.0/ && \
    cmake . && \
    make && \
    make install && \
    apt-get remove --purge -y cmake build-essential wget libssl-dev
RUN ldconfig

RUN mkdir /work
WORKDIR /work
VOLUME /var/lib/docker

ENV PIP_NO_BINARY pygit2
COPY Pipfile Pipfile.lock /work/
RUN pip3 install pipenv && pipenv sync

COPY worker.py /usr/local/bin
COPY osv /usr/local/bin/osv/
RUN chmod 755 /usr/local/bin/worker.py

ENV GOOGLE_CLOUD_PROJECT oss-vdb
CMD ["pipenv", "run", "worker.py"]
