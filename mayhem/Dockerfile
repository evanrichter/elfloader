# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN export RUSTFLAGS="-Cdebug-assertions -C codegen-units=1" && \
    cargo +nightly -Z sparse-registry build --release && \
    mv target/release/elfloader /elfloader

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /elfloader /
