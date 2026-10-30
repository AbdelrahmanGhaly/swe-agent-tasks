FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

RUN apt-get update \
    && apt-get install -y --no-install-recommends ffmpeg \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

RUN pip install --no-cache-dir -e . \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
