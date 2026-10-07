FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

ENV CARGO_HOME=/opt/cargo \
    PATH=/opt/rust/bin:/opt/cargo/bin:$PATH \
    SETUPTOOLS_SCM_PRETEND_VERSION=1.9.0

RUN curl -fsSL -o /tmp/rust.tar.gz \
      https://static.rust-lang.org/dist/2024-11-28/rust-1.83.0-x86_64-unknown-linux-gnu.tar.gz \
    && echo "bd9d53d09d4b60826288336de19fb9c5c7592081e4e4520d6de2f65ee8d79087  /tmp/rust.tar.gz" \
      | sha256sum -c - \
    && tar -xzf /tmp/rust.tar.gz -C /tmp \
    && /tmp/rust-1.83.0-x86_64-unknown-linux-gnu/install.sh --prefix=/opt/rust \
         --components=rustc,cargo,rust-std-x86_64-unknown-linux-gnu \
    && rm -rf /tmp/rust.tar.gz /tmp/rust-1.83.0-x86_64-unknown-linux-gnu \
    && uv export --frozen --no-hashes --no-emit-project --group dev -o /tmp/requirements.txt \
    && pip install --no-cache-dir -r /tmp/requirements.txt \
    && pip install --no-cache-dir --no-build-isolation -e . \
    && rm -f /tmp/requirements.txt \
    && mkdir -p /opt/cargo \
    && chmod -R a+rwX /app /opt/cargo \
    && chmod -R a+rX /opt/rust

CMD ["/bin/bash"]