# Stage 1: Build binary in container with Go installed
FROM golang:1.25-alpine3.23 AS builder

# We clone private dependencies via git
RUN apk update && apk upgrade && apk add git

# Ensure these are not fetched via the global Go proxies, as they might have
# issues with private repositories.
ENV GOPRIVATE=gitlab.inf.unibe.ch

# Copy .netrc file, which is passed in as a secret, to proper place.
# No longer needed, now that the asymmetric-quorums library is public.
# RUN --mount=type=secret,id=netrc cp /run/secrets/netrc /root/.netrc && chmod 600 /root/.netrc

COPY / /cometbft
WORKDIR /cometbft

RUN go build -o cometbft ./cmd/cometbft

# Stage 2: Container to run application in. Using same Alpine version as we
# used to build it above.
FROM alpine:3.23

# We fix those so we they are known, making it easier to wrangle with file
# permissions on mounted directories.
ARG UID=1000
ARG GID=1000

LABEL maintainer="michael.senn@unibe.ch"

ENV CMTHOME=/cometbft

RUN apk update && \
    apk upgrade && \
    apk --no-cache add bash && \
    addgroup -g "$GID" cometbft && \
    adduser -S -G cometbft -u "$UID" cometbft -h "$CMTHOME"

# Anything from now on is run as the non-privileged application user
USER cometbft
WORKDIR $CMTHOME

COPY --from=builder /cometbft/cometbft /usr/bin/cometbft
STOPSIGNAL SIGTERM
# P2P and RPC ports
EXPOSE 26656 26657

# Expose a volume in /cometbft where the config can be mounted later.
# Defining it in the Dockerfile will ensure that file permissions work out,
# even though it runs as a non-privileged user in the container.
VOLUME /cometbft

ENTRYPOINT ["/usr/bin/cometbft"]
CMD ["node"]
