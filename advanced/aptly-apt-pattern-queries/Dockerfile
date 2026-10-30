FROM public.ecr.aws/d3j8x8q7/olympus-base-go:latest
ENV GOPATH=/opt/go
ENV GOCACHE=/opt/go/cache
ENV PATH=/opt/go/bin:/usr/local/go/bin:${PATH}
ENV GOFLAGS=-buildvcs=false
ENV GOTOOLCHAIN=local
WORKDIR /app
COPY . .
RUN printf '%s' 1.6.2 > VERSION \
    && go mod download \
    && go build ./... \
    && go vet ./query/... ./deb/... ./cmd/... \
    && chmod -R a+rwX /app /opt/go
CMD ["/bin/bash"]
