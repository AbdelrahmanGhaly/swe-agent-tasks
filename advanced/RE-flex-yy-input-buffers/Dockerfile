FROM public.ecr.aws/d3j8x8q7/olympus-base-cpp:latest

RUN apt-get update \
    && apt-get install -y --no-install-recommends flex=2.6.4-8.2 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

RUN cmake -S . -B build -DCMAKE_BUILD_TYPE=Release \
    && cmake --build build -j4 \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
