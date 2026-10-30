FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

# Debian 12 (bookworm): gfortran 4:12.2.0-3 installs gfortran-12 at the version of
# the base image's gcc-12 (12.2.0-14+deb12u1).
RUN apt-get -o Acquire::Retries=10 update \
    && DEBIAN_FRONTEND=noninteractive apt-get -o Acquire::Retries=10 install -y --no-install-recommends \
        gfortran=4:12.2.0-3 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

ENV PYTHONDONTWRITEBYTECODE=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DEFAULT_TIMEOUT=60 \
    PIP_RETRIES=10

RUN pip install --only-binary=:all: ninja==1.11.1.1 cmake==3.25.2 \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
