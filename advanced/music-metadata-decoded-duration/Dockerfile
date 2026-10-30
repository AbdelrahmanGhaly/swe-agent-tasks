FROM public.ecr.aws/d3j8x8q7/olympus-base-typescript:latest

RUN apt-get update \
    && apt-get install -y --no-install-recommends ffmpeg \
    && rm -rf /var/lib/apt/lists/* \
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
