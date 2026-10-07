FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app

COPY . .

# pythran is an ahead-of-time compiler: the analysis under test emits C++ and
# hands it to the system toolchain, so the image needs a working C++ setup as
# well as the Python one. The toolchain is installed explicitly rather than
# relying on whatever the base image happens to carry.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential=12.9 \
        cmake=3.25.1-1 \
        ninja-build=1.11.1-2~deb12u1 \
    && rm -rf /var/lib/apt/lists/*

# The test patch is applied after the image is built, so pythran has to be
# imported from the source tree; a copying install would leave the patched
# sources unused.
RUN pip install --no-cache-dir \
        "ply==3.11" "gast==0.7.0" "beniget==0.5.0" "numpy==2.3.2" \
        "setuptools==80.9.0" "pytest==8.4.1" \
    && pip install --no-cache-dir --no-deps --no-build-isolation -e . \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
