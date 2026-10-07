FROM public.ecr.aws/d3j8x8q7/olympus-base-rust:latest
WORKDIR /app
COPY . .
RUN curl -fsSL https://nodejs.org/dist/v24.15.0/node-v24.15.0-linux-x64.tar.xz -o /tmp/node.tar.xz \
    && echo "472655581fb851559730c48763e0c9d3bc25975c59d518003fc0849d3e4ba0f6  /tmp/node.tar.xz" | sha256sum -c - \
    && tar -xJf /tmp/node.tar.xz -C /usr/local --strip-components=1 \
    && rm /tmp/node.tar.xz \
    && node --version \
    && npm --version \
    && npm install -g typescript@7.0.2 \
    && tsc --version \
    && rustup show \
    && cargo fetch \
    && cargo vendor --locked /opt/syncpack-vendor > /opt/syncpack-vendor.toml \
    && cargo build --workspace --tests \
    && chmod -R a+rwX /app /opt/cargo /opt/rustup \
    && chmod -R a+rX /opt/syncpack-vendor /opt/syncpack-vendor.toml
CMD ["/bin/bash"]
