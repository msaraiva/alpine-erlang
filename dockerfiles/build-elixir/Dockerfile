FROM msaraiva/alpine:3.1
MAINTAINER Marlus Saraiva <marlus.saraiva@gmail.com>

RUN apk --update add sudo git make erlang erlang-crypto erlang-syntax-tools \
    erlang-inets erlang-ssl erlang-public-key erlang-asn1 erlang-sasl \
    erlang-erl-interface erlang-dev erlang-parsetools erlang-eunit erlang-tools && \
    rm -rf /var/cache/apk/*

RUN adduser -D dev && \
    echo 'dev ALL=NOPASSWD: ALL' > /etc/sudoers.d/dev

USER dev

WORKDIR /home/dev

CMD ["/bin/sh"]
