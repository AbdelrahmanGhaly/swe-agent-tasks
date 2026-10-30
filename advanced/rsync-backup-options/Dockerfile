FROM public.ecr.aws/d3j8x8q7/olympus-base-go:latest

# rsync 3.2.7 is the interoperability reference the tests compare against.
RUN apt-get update \
 && apt-get install -y --no-install-recommends "rsync=3.2.7-1+deb12u*" \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

ENV GOFLAGS=-buildvcs=false GOTOOLCHAIN=local GOCACHE=/tmp/gocache

RUN go mod download \
 && go build ./... \
 && (go vet ./... >/dev/null 2>&1 || true) \
 && groupadd -g 1000 model && useradd -u 1000 -g model -m model \
 && chmod -R a+rwX /app /tmp/gocache \
 && chmod -R a+rX "$(go env GOMODCACHE)"

CMD ["/bin/bash"]
