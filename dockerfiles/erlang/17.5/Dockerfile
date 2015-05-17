FROM msaraiva/alpine:3.1
MAINTAINER Marlus Saraiva <marlus.saraiva@gmail.com>

RUN echo 'http://dl-4.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories

RUN apk --update add erlang erlang-crypto erlang-syntax-tools erlang-sasl && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]
