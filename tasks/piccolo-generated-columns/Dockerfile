FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest
WORKDIR /app
ENV PICCOLO_CONF=tests.sqlite_conf
COPY . .
RUN pip install --no-cache-dir aiosqlite pytest python-dateutil \
    && pip install --no-cache-dir -e . \
    && python -c "import aiosqlite, piccolo, pytest" \
    && chmod -R a+rwX /app
CMD ["/bin/bash"]
