FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest
WORKDIR /app
COPY . .
ENV SETUPTOOLS_SCM_PRETEND_VERSION=5.2.0
ENV PYTHONDONTWRITEBYTECODE=1
RUN pip install --no-cache-dir "Cython==3.1.6" "pytest==8.4.2" "pytest-timeout==2.4.0" "pytest-mock==3.15.1" \
 && pip install --no-cache-dir --no-build-isolation -e . \
 && chmod -R a+rwX /app
CMD ["/bin/bash"]
