# Build Stage
FROM ubuntu:22.04 as builder

## Dependencies
RUN apt update && apt install -y libtss2-dev curl clang-14 pkg-config cmake build-essential
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain nightly
RUN ${HOME}/.cargo/bin/cargo install -f cargo-fuzz

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN bash -c "pushd fuzz && ${HOME}/.cargo/bin/cargo +nightly -Z sparse-registry fuzz build --features mbed-crypto-provider && popd" && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/fuzz_service /fuzz_service

# Package Stage
FROM ubuntu:22.04

COPY --from=builder /fuzz_service /
COPY --from=builder /src/config.toml /run_config.toml
COPY --from=builder  \
            /lib/x86_64-linux-gnu/libtss2-esys.so.0 \
            /lib/x86_64-linux-gnu/libtss2-tctildr.so.0 \
            /lib/x86_64-linux-gnu/libtss2-mu.so.0 \
            /lib/x86_64-linux-gnu/libcrypto.so.3 \
            /lib/x86_64-linux-gnu/libtss2-sys.so.1 \
            /lib/x86_64-linux-gnu/
