FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

ENV PYTHONDONTWRITEBYTECODE=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_NO_CACHE_DIR=1

# Editable install: the SCons package and the scons command always run the code in /app
RUN pip install lxml==6.1.3 psutil==7.2.2 ninja==1.13.2 \
    && pip install --no-deps -e .

RUN groupadd -g 1000 model && useradd -u 1000 -g model -m model \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
