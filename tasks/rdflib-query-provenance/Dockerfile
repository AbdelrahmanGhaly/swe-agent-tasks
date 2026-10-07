FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest
WORKDIR /app
COPY . .
RUN pip install --no-build-isolation "pyparsing==3.3.2" \
    && pip install --no-build-isolation -e . \
    && chmod -R a+rwX /app
CMD ["/bin/bash"]
