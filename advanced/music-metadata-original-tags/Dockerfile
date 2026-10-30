FROM public.ecr.aws/d3j8x8q7/olympus-base-typescript:latest

RUN apt-get -o Acquire::Retries=10 update \
    && DEBIAN_FRONTEND=noninteractive apt-get -o Acquire::Retries=10 install -y --no-install-recommends wavpack flac ffmpeg \
    && rm -rf /var/lib/apt/lists/* \
    && wavpack --version 2>&1 | head -n 1 | grep -q "^wavpack 5\.6" \
    && flac --version | grep -q "^flac 1\.4\.2" \
    && ffmpeg -version | head -n 1 | grep -q "^ffmpeg version 5\.1"

ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0
ENV COREPACK_HOME=/opt/corepack

WORKDIR /app
COPY . .

RUN corepack enable \
    && yarn install --immutable \
    && yarn compile-src:dev \
    && chmod -R a+rwX /app /opt/corepack

CMD ["/bin/bash"]
