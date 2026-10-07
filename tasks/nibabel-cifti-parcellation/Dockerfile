FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

ENV USER=olympus \
    LOGNAME=olympus \
    HOME=/home/olympus \
    MPLCONFIGDIR=/home/olympus/matplotlib \
    SETUPTOOLS_SCM_PRETEND_VERSION=5.5.0.dev0

# Dependencies are resolved from the repository's committed uv.lock, so every version is pinned and
# the container needs no network once built. nibabel itself is then installed in place with
# --no-deps: the suite imports the package under test, and an in-place install keeps /app as the
# code that actually runs, so source changes take effect without reinstalling.
RUN uv export --frozen --no-hashes --no-emit-project --all-extras --no-default-groups \
      -o /tmp/requirements.txt \
    && pip install --no-cache-dir --no-deps -r /tmp/requirements.txt \
    && pip install --no-cache-dir --no-deps -e . \
    && rm -f /tmp/requirements.txt \
    && mkdir -p /home/olympus/matplotlib \
    && chmod -R a+rwX /app /home/olympus

CMD ["/bin/bash"]
