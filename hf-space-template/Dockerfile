FROM golang:alpine AS builder

ARG GIT_REPO=https://github.com/YOUR_GITHUB_USERNAME/metatube-sdk-go.git
ARG GIT_REF=main

RUN apk add --no-cache git ca-certificates tzdata
WORKDIR /src

RUN git clone --depth 1 --branch ${GIT_REF} ${GIT_REPO} /src
RUN go mod download
RUN CGO_ENABLED=0 GO111MODULE=on go build -trimpath -ldflags="-w -s" -o /build/metatube-server ./cmd/server

FROM alpine:latest

RUN apk add --no-cache ca-certificates tzdata \
    && mkdir -p /data \
    && chmod 777 /data

ENV GIN_MODE=release
ENV PORT=8080
ENV DSN=/data/metatube.db
ENV DB_AUTO_MIGRATE=1
ENV REQUEST_TIMEOUT=1m
ENV MT_ACTOR_PROVIDER_PRIORITY_AV-LEAGUE=1000
ENV MT_ACTOR_PROVIDER_AV-LEAGUE__PRIORITY=1000

EXPOSE 8080

COPY --from=builder /build/metatube-server /metatube-server

ENTRYPOINT ["/metatube-server"]
