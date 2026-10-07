FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

# Shared libraries for the headless Chrome that kaleido uses to export charts (tests/test_backtesting_charts.py).
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       fonts-liberation libasound2 libatk-bridge2.0-0 libatk1.0-0 libcairo2 libcups2 libdbus-1-3 libdrm2 libgbm1 \
       libglib2.0-0 libnss3 libpango-1.0-0 libx11-6 libxcb1 libxcomposite1 libxdamage1 libxext6 libxfixes3 \
       libxkbcommon0 libxrandr2 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

ENV TZ=UTC \
    HOME=/tmp/home \
    BROWSER_PATH=/opt/chrome/chrome-linux64/chrome

RUN uv export --frozen --no-hashes --no-emit-project --all-extras --no-group docs \
      -o /tmp/requirements.txt \
    && pip install --no-cache-dir --no-deps -r /tmp/requirements.txt \
    && pip install --no-cache-dir --no-deps -e . \
    && rm -f /tmp/requirements.txt \
    && mkdir -p /opt/chrome /tmp/home \
    && plotly_get_chrome -y --path /opt/chrome \
    && chmod -R a+rX /opt/chrome \
    && chmod -R a+rwX /app /tmp/home

CMD ["/bin/bash"]
