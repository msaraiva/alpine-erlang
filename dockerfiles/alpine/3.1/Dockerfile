FROM alpine:3.1
MAINTAINER Marlus Saraiva <marlus.saraiva@gmail.com>

RUN apk --update add ncurses-libs && rm -rf /var/cache/apk/*

RUN echo 'http://dl-4.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories

CMD ["/bin/sh"]
