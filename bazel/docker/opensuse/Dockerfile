# Create a virtual environment with all tools installed
# ref: https://hub.docker.com/r/opensuse/tumbleweed
FROM opensuse/tumbleweed AS env

# Install system build dependencies using one repository refresh.
ENV PATH=/usr/local/bin:$PATH
# Retry transient Tumbleweed mirror errors before updating or installing packages.
RUN (zypper --non-interactive refresh --force \
  || { sleep 30 && zypper --non-interactive refresh --force; } \
  || { sleep 60 && zypper --non-interactive refresh --force; }) \
&& zypper --no-refresh update -y \
&& zypper --no-refresh install -y git wget gcc gcc-c++ zlib-devel \
 java-21-openjdk-devel maven \
 python3 python3-pip python3-devel \
&& zypper clean -a
ENV CC=gcc CXX=g++
ENV JAVA_HOME=/usr/lib64/jvm/java-21-openjdk

# Install Bazelisk
ARG TARGETARCH=amd64
RUN wget \
https://github.com/bazelbuild/bazelisk/releases/download/v1.27.0/bazelisk-linux-${TARGETARCH} \
&& chmod +x bazelisk-linux-${TARGETARCH} \
&& mv bazelisk-linux-${TARGETARCH} /usr/local/bin/bazel

FROM env AS devel
WORKDIR /home/project
COPY . .

FROM devel AS build
RUN bazel version
# Work around the GNU ld 2.47 crash when linking object groups (Ubuntu #2172236).
RUN bazel build --config=ci \
    --features=-supports_start_end_lib --host_features=-supports_start_end_lib //...

FROM build AS test
RUN bazel test --config=ci \
    --features=-supports_start_end_lib --host_features=-supports_start_end_lib //...
