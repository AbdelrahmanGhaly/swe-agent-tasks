FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

ENV SETUPTOOLS_SCM_PRETEND_VERSION=0.7.3.dev0 \
    WITH_EXTENSION=1 \
    NOWEB=1

RUN pip install --no-cache-dir "pytest==9.0.3" "psutil==7.2.2" \
    && pip install --no-cache-dir -e . \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
