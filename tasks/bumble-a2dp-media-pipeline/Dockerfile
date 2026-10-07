FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest
WORKDIR /app
COPY . .

# Every dependency is pinned to an exact version. The project itself is then put
# on the path with --no-deps, so this line resolves nothing and installs nothing
# beyond the pins above. It is editable on purpose: the task is solved by editing
# the sources under /app, and a non-editable install would copy them into
# site-packages and hide those edits from the tests.
RUN pip install --no-cache-dir \
      "aiohappyeyeballs==2.7.1" \
      "aiohttp==3.14.3" \
      "aiosignal==1.4.0" \
      "attrs==26.1.0" \
      "cffi==2.0.0" \
      "click==8.4.1" \
      "coverage==7.15.4" \
      "cryptography==48.0.0" \
      "frozenlist==1.8.0" \
      "grpcio==1.83.0" \
      "humanize==4.16.0" \
      "idna==3.18" \
      "libusb-package==1.0.26.4" \
      "libusb1==3.4.0" \
      "multidict==6.7.1" \
      "platformdirs==4.10.0" \
      "prettytable==3.18.0" \
      "prompt_toolkit==3.0.53" \
      "propcache==0.5.2" \
      "protobuf==7.36.0" \
      "pycparser==3.0" \
      "pyee==14.0.0" \
      "pyserial==3.5" \
      "pyserial-asyncio==0.6" \
      "pytest==9.0.3" \
      "pytest-asyncio==1.4.0" \
      "pytest-html==4.2.0" \
      "pyusb==1.3.1" \
      "typing_extensions==4.15.0" \
      "wcwidth==0.8.2" \
      "websockets==17.1" \
      "yarl==1.24.5" && \
    pip install --no-cache-dir --no-deps -e .
CMD ["/bin/bash"]
