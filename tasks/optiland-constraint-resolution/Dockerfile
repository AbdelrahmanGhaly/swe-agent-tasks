FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app

COPY . .

# PySide6 is a test dependency of the optiland_gui suite; these are the shared
# libraries its wheels link against, mirroring the repository's own CI setup.
RUN apt-get update && apt-get install -y --no-install-recommends \
      libegl1 \
      libgl1 \
      libdbus-1-3 \
      libxkbcommon-x11-0 \
      libxcb-cursor0 \
      libxcb-icccm4 \
      libxcb-image0 \
      libxcb-keysyms1 \
      libxcb-randr0 \
      libxcb-render-util0 \
      libxcb-shape0 \
    && rm -rf /var/lib/apt/lists/*

ENV HOME=/tmp \
    MPLCONFIGDIR=/tmp/matplotlib \
    MPLBACKEND=Agg \
    QT_QPA_PLATFORM=offscreen

RUN pip install --no-cache-dir "pytest>=8.3.5" "pytest-xdist>=3.6" PySide6 qtconsole \
    && pip install --no-cache-dir -e . \
    && mkdir -p /tmp/matplotlib \
    && chmod -R a+rwX /app /tmp/matplotlib

CMD ["/bin/bash"]
