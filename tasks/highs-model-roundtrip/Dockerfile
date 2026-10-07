FROM public.ecr.aws/d3j8x8q7/olympus-base-cpp:latest
WORKDIR /app
COPY . .
RUN cmake -S . -B build -GNinja -DCMAKE_BUILD_TYPE=Release -DALL_TESTS=ON \
    && cmake --build build -j \
    && chmod -R a+rwX /app
CMD ["/bin/bash"]
