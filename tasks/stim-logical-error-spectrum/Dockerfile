FROM public.ecr.aws/d3j8x8q7/olympus-base-cpp:latest

RUN apt-get update \
    && apt-get install -y --no-install-recommends libgtest-dev ccache \
    && rm -rf /var/lib/apt/lists/*

ENV CCACHE_DIR=/opt/ccache
ENV CCACHE_MAXSIZE=8G
ENV CCACHE_SLOPPINESS=include_file_mtime,include_file_ctime

WORKDIR /app
COPY . .

# Warm the compiler cache, then throw the build tree away. Shipping a prebuilt tree
# lets `make` call a stale binary up to date when the sources it is handed do not
# have newer timestamps, so test.sh builds from scratch every run and leans on the
# cache (which keys on content, not timestamps) to stay quick.
RUN mkdir -p "$CCACHE_DIR" \
    && cmake -S . -B /tmp/warmup -DCMAKE_BUILD_TYPE=Release -DSIMD_WIDTH=128 \
         -DCMAKE_CXX_COMPILER_LAUNCHER=ccache \
    && cmake --build /tmp/warmup --target stim_test_o3 -j 4 \
    && rm -rf /tmp/warmup \
    && chmod -R a+rwX "$CCACHE_DIR"

CMD ["/bin/bash"]
