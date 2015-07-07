FROM msaraiva/erlang:18.0.1
MAINTAINER Marlus Saraiva <marlus.saraiva@gmail.com>

RUN apk --update add elixir wget git && rm -rf /var/cache/apk/*

RUN mix local.hex --force && \
    mix local.rebar --force

CMD ["/bin/sh"]
