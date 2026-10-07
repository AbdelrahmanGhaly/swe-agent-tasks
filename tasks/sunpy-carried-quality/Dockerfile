FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest
WORKDIR /app
COPY . .
ENV MPLBACKEND=Agg
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONHASHSEED=0
ENV SUNPY_DOWNLOADDIR=/tmp/sunpy-downloads
ENV SETUPTOOLS_SCM_PRETEND_VERSION=8.1.dev96+gb9fb3034f
# The project itself is installed, not only its dependencies. This is required rather than
# stylistic: sunpy/map/__init__.py asks importlib.metadata for sunpy's own requirements at
# import time, so with no distribution metadata the package cannot be imported at all. The
# install is editable so that /app stays the single copy of the source and the patches applied
# to it take effect; a second copy in site-packages could shadow them. Its version is pinned by
# SETUPTOOLS_SCM_PRETEND_VERSION above, since there is no .git directory to derive one from.
RUN pip install --no-cache-dir \
        astropy==8.0.1 \
        astropy-iers-data==0.2026.9.7.0.56.14 \
        contourpy==1.3.3 \
        fsspec==2026.7.0 \
        hypothesis==6.168.0 \
        matplotlib==3.11.1 \
        mpl-animators==1.2.4 \
        numpy==2.5.3 \
        packaging==26.3 \
        parfive==2.3.1 \
        pyerfa==2.0.1.5 \
        pytest==9.1.1 \
        pytest-arraydiff==0.7.0 \
        pytest-doctestplus==1.7.1 \
        pytest-mock==3.15.1 \
        pytest-remotedata==0.4.2 \
        pytest-xdist==3.8.0 \
        reproject==0.21.0 \
        requests==2.34.2 \
        scipy==1.18.1 \
    && pip install --no-cache-dir --no-deps -e . \
    && chmod -R a+rwX /app
CMD ["/bin/bash"]
