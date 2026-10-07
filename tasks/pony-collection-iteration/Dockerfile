FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest
WORKDIR /app
RUN apt-get update \
    && apt-get install -y --no-install-recommends postgresql libnss-wrapper \
    && rm -rf /var/lib/apt/lists/*
COPY . .
RUN pip install --no-cache-dir pytest pymysql psycopg2-binary \
    && pip install --no-cache-dir -e . \
    && chmod -R a+rwX /app
CMD ["/bin/bash"]
