FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

ENV PYTHONDONTWRITEBYTECODE=1 \
    USER=vivisect \
    LOGNAME=vivisect \
    QT_QPA_PLATFORM=offscreen

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libegl1 libgl1 libglib2.0-0 libfontconfig1 libdbus-1-3 libasound2 libnss3 libnspr4 \
        libxkbcommon0 libxkbcommon-x11-0 libxkbfile1 libxcb-cursor0 libxdamage1 libxcomposite1 \
        libxrandr2 libxtst6 libxi6 libxext6 libx11-xcb1 libxshmfence1 libgbm1 libcups2 \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir \
        pyasn1==0.6.4 \
        pyasn1-modules==0.4.2 \
        cxxfilt==0.3.0 \
        msgpack==1.1.2 \
        pycparser==3.0 \
        PyQt6==6.11.0 \
        PyQt6-Qt6==6.11.2 \
        PyQt6-sip==13.12.0 \
        PyQt6-WebEngine==6.11.0 \
        PyQt6-WebEngine-Qt6==6.11.2 \
        pytest==9.0.3 \
    && pip install --no-cache-dir --no-deps -e . \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
