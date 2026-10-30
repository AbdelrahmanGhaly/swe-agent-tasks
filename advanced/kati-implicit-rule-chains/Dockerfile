FROM public.ecr.aws/d3j8x8q7/olympus-base-rust:latest

ENV GOPATH=/opt/go \
    GOCACHE=/opt/go/cache \
    GOFLAGS=-buildvcs=false \
    GOTOOLCHAIN=local

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
       g++ golang-go ninja-build binutils xz-utils curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# kati's tests compare it with GNU make 4.2.1; Debian ships 4.3.
RUN set -eux; \
    case "$(dpkg --print-architecture)" in \
      amd64) pkg=http://archive.ubuntu.com/ubuntu/pool/main/m/make-dfsg/make_4.2.1-1.2_amd64.deb; \
             sum=41e534af98cdb6219bc98fa4276d9c928a0862b8b373d49ee1fbe0ae5db64dc2 ;; \
      arm64) pkg=http://ports.ubuntu.com/ubuntu-ports/pool/main/m/make-dfsg/make_4.2.1-1.2_arm64.deb; \
             sum=4856ecbb79f91dd33f46474fca70331cc0940f7ba79a182e4e93970473c5a3b0 ;; \
    esac; \
    mkdir -p /opt/make421/tmp; cd /opt/make421/tmp; \
    curl -sSfL -o make.deb "$pkg"; \
    echo "$sum  make.deb" | sha256sum -c -; \
    ar x make.deb; tar xf data.tar.xz; \
    mv usr/bin/make /opt/make421/make; \
    cd /opt/make421; rm -rf tmp; \
    /opt/make421/make --version | head -n 1
ENV PATH=/opt/make421:$PATH

RUN GOBIN=/usr/local/bin go install github.com/jstemmer/go-junit-report/v2@v2.1.0 \
    && cargo install cargo2junit --version 0.1.14 --locked

WORKDIR /app
COPY . .
RUN cargo fetch \
    && cargo build --tests \
    && make -j"$(nproc)" ckati ckati_tests \
    && go mod download \
    && go vet . \
    && chmod -R a+rwX /app /opt/cargo /opt/go

ENV CARGO_NET_OFFLINE=true
CMD ["/bin/bash"]
