FROM python:3.11-slim-bookworm as base

FROM base as builder
ENV PYTHONUNBUFFERED=1 DEBIAN_FRONTEND=noninteractive
ARG ARM
ARG LIB_ARCH=${ARM:+arm}
ARG RTSP_ARCH=${ARM:+armv7}
ARG FFMPEG_ARCH=${ARM:+armv7l}
# Robust apt install with no recommends and cleanup to avoid layer bloat
RUN apt-get update \
    && apt-get install -y --no-install-recommends tar unzip curl jq g++ git ca-certificates \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt /tmp/requirements.txt
# Upgrade pip and install deps; avoid cache to keep image smaller
RUN python3 -m pip install --upgrade --no-cache-dir pip \
    && pip3 install --disable-pip-version-check --no-cache-dir --prefix=/build/usr/local -r /tmp/requirements.txt
COPY *.lib /tmp/lib/
RUN apt-get update \
    && apt-get install -y --no-install-recommends ffmpeg \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /build/app /build/tokens /build/img \
    && RTSP_TAG=$(curl -s https://api.github.com/repos/aler9/rtsp-simple-server/releases/latest | jq -r .tag_name) \
    && echo -n $RTSP_TAG > /build/RTSP_TAG \
    && curl -L https://github.com/aler9/rtsp-simple-server/releases/download/${RTSP_TAG}/rtsp-simple-server_${RTSP_TAG}_linux_${RTSP_ARCH:-amd64}.tar.gz \
    | tar xzf - -C /build/app \
    && cp /tmp/lib/${LIB_ARCH:-amd}.lib /build/usr/local/lib/libIOTCAPIs_ALL.so \
    && rm -rf /tmp/*
COPY . /build/app/

FROM base
ENV DEBIAN_FRONTEND=noninteractive PYTHONUNBUFFERED=1 RTSP_PROTOCOLS=tcp RTSP_READTIMEOUT=20s RTSP_LOGLEVEL=warn RTSP_WEBRTCICEUDPMUXADDRESS=:8189 SDK_KEY=AQAAAIZ44fijz5pURQiNw4xpEfV9ZysFH8LYBPDxiONQlbLKaDeb7n26TSOPSGHftbRVo25k3uz5of06iGNB4pSfmvsCvm/tTlmML6HKS0vVxZnzEuK95TPGEGt+aE15m6fjtRXQKnUav59VSRHwRj9Z1Kjm1ClfkSPUF5NfUvsb3IAbai0WlzZE1yYCtks7NFRMbTXUMq3bFtNhEERD/7oc504b FLASK_APP=frontend
COPY --from=builder /build /
WORKDIR /app
CMD [ "flask", "run", "--host=0.0.0.0"]
