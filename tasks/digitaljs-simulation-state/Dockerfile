FROM public.ecr.aws/d3j8x8q7/olympus-base-typescript:latest

WORKDIR /app

COPY . .

RUN npm ci --include=dev --ignore-scripts \
    && npm run build-lib \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
