FROM public.ecr.aws/d3j8x8q7/olympus-base-rust:latest

WORKDIR /app
COPY . .

# cargo2junit turns libtest's json event stream into JUnit XML, which keeps the real panic
# block for a failing test. cargo fetch primes the registry because there is no network at
# run time, and Cargo.lock is committed upstream, so the versions resolved here are the
# ones the offline build uses later. numbat-wasm is excluded from the workspace.
RUN cargo install cargo2junit && cargo fetch && cargo build --workspace

CMD ["/bin/bash"]
