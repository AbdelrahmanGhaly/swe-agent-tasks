FROM public.ecr.aws/d3j8x8q7/olympus-base-go:latest
ENV GOFLAGS=-buildvcs=false
ENV GOTOOLCHAIN=local
WORKDIR /app
COPY . .
RUN go mod download \
    && go build ./... \
    && go vet ./... \
    && chmod -R a+rwX /app /opt/go
CMD ["/bin/bash"]
