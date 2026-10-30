FROM public.ecr.aws/d3j8x8q7/olympus-base-rust:latest

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
       g++ golang-go ninja-build binutils xz-utils curl ca-certificates perl python3 \
    && rm -rf /var/lib/apt/lists/*

# GNU make 4.2.1 is the reference kati is compared with; Debian 12 ships 4.3.
RUN set -eux; \
    arch="$(dpkg --print-architecture)"; \
    case "$arch" in \
      amd64) url=http://archive.ubuntu.com/ubuntu/pool/main/m/make-dfsg/make_4.2.1-1.2_amd64.deb; \
             sha=41e534af98cdb6219bc98fa4276d9c928a0862b8b373d49ee1fbe0ae5db64dc2 ;; \
      arm64) url=http://ports.ubuntu.com/ubuntu-ports/pool/main/m/make-dfsg/make_4.2.1-1.2_arm64.deb; \
             sha=4856ecbb79f91dd33f46474fca70331cc0940f7ba79a182e4e93970473c5a3b0 ;; \
      *) echo "unsupported architecture $arch"; exit 1 ;; \
    esac; \
    mkdir -p /tmp/make421; \
    curl -sSfL -o /tmp/make421/make.deb "$url"; \
    echo "$sha  /tmp/make421/make.deb" >/tmp/make421/make.sha256; \
    sha256sum -c /tmp/make421/make.sha256; \
    cd /tmp/make421 && ar x make.deb && tar xf data.tar.xz; \
    install -m 0755 /tmp/make421/usr/bin/make /usr/local/bin/make; \
    cd /; rm -rf /tmp/make421; \
    make --version | head -n 1

ENV GOPATH=/opt/go \
    GOCACHE=/opt/go/cache \
    GOFLAGS=-buildvcs=false

RUN GOBIN=/usr/local/bin go install github.com/jstemmer/go-junit-report/v2@v2.1.0 \
    && cargo install cargo2junit --version 0.1.14 --locked \
    && for f in /opt/cargo/bin/*; do ln -sf "$f" /usr/local/bin/; done \
    && printf '%s\n' \
       'export CARGO_HOME=/opt/cargo RUSTUP_HOME=/opt/rustup' \
       'export GOPATH=/opt/go GOCACHE=/opt/go/cache GOFLAGS=-buildvcs=false' \
       'export CARGO_NET_OFFLINE=true GOPROXY=off' \
       >/etc/profile.d/kati-toolchain.sh

WORKDIR /app
COPY . .

# Makefile.ckati adds -march=native, so a ckati built here can stop with
# "Illegal instruction" on a machine with an older CPU.
RUN cargo fetch \
    && cargo build \
    && make -j"$(nproc)" ckati \
       KATI_CXXFLAGS='-g -W -Wall -MMD -MP -O2 -DNOLOG -D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_FAST -D_GLIBCXX_ASSERTIONS' \
    && go mod download \
    && chmod -R a+rwX /app /opt/cargo /opt/go \
    && chmod -R a+rX /opt/rustup

ENV CARGO_NET_OFFLINE=true \
    GOPROXY=off

CMD ["/bin/bash"]
