#
# Dockerfile for youtube-dl
#

FROM alpine
MAINTAINER kev <noreply@easypi.pro>

RUN set -xe \
    && apk add --no-cache ca-certificates \
                          ffmpeg \
                          openssl \
                          python3 \
			  bash \
			  bash-doc \
			  bash-completion \
    && pip3 install youtube-dl you-get

WORKDIR /data

CMD ["/bin/sh"]
