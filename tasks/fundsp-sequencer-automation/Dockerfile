FROM public.ecr.aws/d3j8x8q7/olympus-base-rust:latest
ENV CARGO_PROFILE_DEV_DEBUG=0
WORKDIR /app
COPY . .
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends libasound2-dev libdbus-1-dev \
    && rm -rf /var/lib/apt/lists/*
RUN cargo install cargo2junit --version 0.1.14 --locked \
    && cargo fetch \
    && cargo build --tests \
    && chmod -R a+rwX /app /opt/cargo
ENV CARGO_NET_OFFLINE=true
CMD ["/bin/bash"]
