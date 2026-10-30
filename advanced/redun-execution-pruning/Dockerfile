FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

ENV PYTHONDONTWRITEBYTECODE=1 \
    USER=redun \
    UV_HTTP_TIMEOUT=300 \
    UV_LINK_MODE=copy \
    UV_PYTHON_INSTALL_DIR=/opt/python \
    VIRTUAL_ENV=/opt/venv \
    PATH=/opt/venv/bin:$PATH

WORKDIR /app
COPY . .

RUN uv python install 3.11.15 \
    && uv venv --seed --python 3.11.15 /opt/venv \
    && uv export --frozen --extra glue --extra k8s --extra google-batch --extra azure --group dev --no-emit-project --no-hashes -o /tmp/requirements.txt \
    && uv pip install --no-cache -r /tmp/requirements.txt \
    && uv pip install --no-cache --no-deps -e . \
    && rm /tmp/requirements.txt \
    && git config --system --add safe.directory '*' \
    && if [ ! -d .git ]; then git init -q . && git add -A && git -c user.name=redun -c user.email=redun@localhost commit -q -m base; fi \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
