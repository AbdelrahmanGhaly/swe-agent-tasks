FROM public.ecr.aws/d3j8x8q7/olympus-base-go:latest

WORKDIR /app

COPY . .

RUN GOBIN=/usr/local/bin go install github.com/jstemmer/go-junit-report/v2@v2.1.0

RUN go build ./... && go vet ./...

RUN chmod -R a+rwX /app

CMD ["/bin/bash"]
