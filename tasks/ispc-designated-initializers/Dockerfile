FROM public.ecr.aws/d3j8x8q7/olympus-base-cpp:latest

RUN apt-get -o Acquire::Retries=5 update \
    && DEBIAN_FRONTEND=noninteractive apt-get -o Acquire::Retries=5 install -y --no-install-recommends \
        libclang-21-dev libc6-dev-i386 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

RUN cmake -S . -B build -G Ninja \
        -DCMAKE_BUILD_TYPE=Release \
        -DLLVM_CONFIG_EXECUTABLE=/usr/lib/llvm-21/bin/llvm-config \
        -DISPC_INCLUDE_EXAMPLES=OFF \
        -DISPC_INCLUDE_RT=OFF \
        -DISPC_INCLUDE_UTILS=OFF \
        -DISPC_INCLUDE_BENCHMARKS=OFF \
        -DISPC_LIBRARY=OFF \
        -DISPC_LIBRARY_JIT=OFF \
        -DARM_ENABLED=OFF \
    && cmake --build build --target ispc ispc-opt generic-target-bc stdlibs-bc stdlib-headers -j8

RUN chown -R 1000:1000 /app

CMD ["/bin/bash"]
