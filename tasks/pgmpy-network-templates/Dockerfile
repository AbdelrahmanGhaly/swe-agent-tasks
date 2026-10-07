FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

RUN pip install --no-cache-dir -e ".[tests]" \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
