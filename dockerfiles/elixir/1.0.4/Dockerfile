FROM msaraiva/erlang:17.5
MAINTAINER Marlus Saraiva <marlus.saraiva@gmail.com>

RUN apk --update add wget git erlang17-crypto erlang17-syntax-tools erlang17-inets erlang17-ssl \
    erlang17-public-key erlang17-asn1 erlang17-sasl erlang17-erl-interface erlang17-dev && \
    rm -rf /var/cache/apk/*

RUN wget https://github.com/elixir-lang/elixir/releases/download/v1.0.4/Precompiled.zip && \
    mkdir -p /opt/elixir-1.0.4/ && \
    unzip Precompiled.zip -d /opt/elixir-1.0.4/ && \
    rm Precompiled.zip

ENV PATH $PATH:/opt/elixir-1.0.4/bin

RUN mix local.hex --force && \
    mix local.rebar --force

CMD ["/bin/sh"]
