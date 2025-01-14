FROM openjdk:8-jdk-slim-buster as builder

RUN apt-get update -y && \
    apt-get install -qq --no-install-recommends git curl gnupg

WORKDIR /code/rskj

ARG RSKJ_RELEASE="IRIS"
ARG RSKJ_VERSION="3.3.0"

RUN gitrev="${RSKJ_RELEASE}-${RSKJ_VERSION}" && \
    git init && \
    git remote add origin https://github.com/rsksmart/rskj.git && \
    git fetch --depth 1 origin tag "$gitrev" && \
    git checkout "$gitrev"

RUN gpg --keyserver https://secchannel.rsk.co/release.asc --recv-keys 1A92D8942171AFA951A857365DECF4415E3B8FA4 && \
    gpg --verify --output SHA256SUMS SHA256SUMS.asc && \
    sha256sum --check SHA256SUMS && \
    ./configure.sh && \
    ./gradlew --no-daemon clean build -x test


FROM openjdk:8-jdk as runner

ARG UID=1000
ARG HOME="/home/rsk"
RUN useradd -m -u "$UID" --home="$HOME" rsk

COPY --from=builder --chown="$UID" "/code/rskj/rskj-core/build/libs/rskj-core-*-all.jar" "$HOME/rskj-core.jar"

RUN mkdir -p "$HOME/.rsk"; chown "$UID" "$HOME/.rsk"
RUN mkdir -p "$HOME/logs"; chown "$UID" "$HOME/logs"

WORKDIR "$HOME"

USER rsk
