FROM debian:trixie

ARG GO_VERSION=1.26.3
ARG ZIG_VERSION=0.16.0
ARG NODE_VERSION=24.16.0
ARG PYTHON_VERSION=3.14.5
ARG TARGETARCH

ENV DEBIAN_FRONTEND=noninteractive
ENV GOPATH=/go
ENV GOROOT=/usr/local/go
ENV VNC_DISPLAY=:1
ENV VNC_PORT=5901
ENV VNC_GEOMETRY=1440x900
ENV VNC_DEPTH=24
ENV VNC_PASSWORD=changeme
ENV NOVNC_PORT=6080
ENV UV_MANAGED_PYTHON=1
ENV UV_PYTHON_INSTALL_DIR=/opt/uv/python
ENV UV_PYTHON_BIN_DIR=/usr/local/bin
ENV PATH=/usr/local/go/bin:/go/bin:/opt/zig:/opt/node/bin:${PATH}

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        bash \
        build-essential \
        ca-certificates \
        cmake \
        curl \
        dbus-x11 \
        file \
        git \
        gnupg \
        make \
        novnc \
        openssh-client \
        pkg-config \
        tar \
        tigervnc-common \
        tigervnc-standalone-server \
        tigervnc-tools \
        unzip \
        websockify \
        x11-xserver-utils \
        xauth \
        xfce4 \
        xfce4-terminal \
        xz-utils \
        zip \
    && rm -rf /var/lib/apt/lists/*

COPY --from=ghcr.io/astral-sh/uv:0.11.16 /uv /uvx /usr/local/bin/

RUN set -eux; \
    uv python install "${PYTHON_VERSION}" --default --compile-bytecode; \
    uv --version; \
    python --version; \
    python3 --version; \
    uv cache clean

RUN set -eux; \
    arch="${TARGETARCH:-$(dpkg --print-architecture)}"; \
    case "$arch" in \
        amd64|x86_64) go_arch="amd64"; node_arch="x64"; zig_arch="x86_64" ;; \
        arm64|aarch64) go_arch="arm64"; node_arch="arm64"; zig_arch="aarch64" ;; \
        *) echo "Unsupported architecture: $arch" >&2; exit 1 ;; \
    esac; \
    curl -fsSL "https://go.dev/dl/go${GO_VERSION}.linux-${go_arch}.tar.gz" -o /tmp/go.tar.gz; \
    rm -rf /usr/local/go; \
    tar -C /usr/local -xzf /tmp/go.tar.gz; \
    mkdir -p /go /opt/node /opt/zig; \
    curl -fsSL "https://nodejs.org/download/release/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-${node_arch}.tar.xz" -o /tmp/node.tar.xz; \
    tar -C /opt/node --strip-components=1 -xJf /tmp/node.tar.xz; \
    curl -fsSL "https://ziglang.org/download/${ZIG_VERSION}/zig-${zig_arch}-linux-${ZIG_VERSION}.tar.xz" -o /tmp/zig.tar.xz; \
    tar -C /opt/zig --strip-components=1 -xJf /tmp/zig.tar.xz; \
    corepack enable; \
    go version; \
    node --version; \
    npm --version; \
    zig version; \
    rm -f /tmp/go.tar.gz /tmp/node.tar.xz /tmp/zig.tar.xz

COPY start-vnc /usr/local/bin/start-vnc

RUN chmod +x /usr/local/bin/start-vnc

WORKDIR /workspace

EXPOSE 5901 6080

CMD ["start-vnc"]
