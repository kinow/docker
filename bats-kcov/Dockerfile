# -----------------------------------------------------------------------------
# kcov is a coverage tool for binaries that works with Bash shell
# scripts too. It uses the Dwarf standard debugging library to
# retrieve coverage values, and produce the final report for CICD.
FROM kcov/kcov:latest AS kcov

# -----------------------------------------------------------------------------
# bats is an old (one of the oldest?) Shell unit test libraries. It
# contains extensions for assertions that should look similar to
# those familiar with libraries like unittest or JUnit.
# The official image does not include the extensions we install here.
FROM debian:bookworm-slim AS bats

# Install build-time dependencies.
RUN apt-get update && \
    apt-get install --yes --no-install-suggests --no-install-recommends \
      build-essential \
      ca-certificates \
      git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install Bats.

RUN git clone https://github.com/bats-core/bats-core.git && \
    cd bats-core && \
    ./install.sh /usr/local

# Install Bats support tools.

RUN mkdir /usr/lib/bats && \
    cd /usr/lib/bats && \
    git clone https://github.com/bats-core/bats-support.git && \
    git clone https://github.com/bats-core/bats-assert.git

# -----------------------------------------------------------------------------
# Final base image.
FROM debian:bookworm-slim

# Install run-time dependencies.

RUN apt-get update && \
    apt-get install --yes --no-install-suggests --no-install-recommends \
      libcurl4 \
      libdw1 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy bats.

COPY --from=bats /usr/local/bin/bats* /usr/local/bin/
COPY --from=bats /usr/local/lib/bats-core/ /usr/local/lib/bats-core/
COPY --from=bats /usr/local/libexec/bats-core/ /usr/local/libexec/bats-core/
COPY --from=bats /usr/lib/bats/bats-assert/ /usr/lib/bats/bats-assert/
COPY --from=bats /usr/lib/bats/bats-support/ /usr/lib/bats/bats-support/

# Copy kcov.

COPY --from=kcov/kcov:latest /usr/local/bin/kcov* /usr/local/bin/
COPY --from=kcov/kcov:latest /usr/local/share/doc/kcov /usr/local/share/doc/kcov

WORKDIR "/code"

# ENTRYPOINT ["/bin/sh", "-c"]
CMD ["/usr/local/bin/bats"]
