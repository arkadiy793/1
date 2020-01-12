FROM alpine
# Раскомментировать строку ниже, если будут сообщения о недоступности cdn
# RUN echo -e "http://nl.alpinelinux.org/alpine/v3.11/main\nhttp://nl.alpinelinux.org/alpine/v3.11/community" > /etc/apk/repositories
RUN apk add --no-cache curl wget busybox-extras netcat-openbsd python py-pip && pip install awscli
RUN apk --purge -v del py-pip
CMD tail -f /dev/null
