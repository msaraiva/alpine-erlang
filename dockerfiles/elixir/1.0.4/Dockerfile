FROM msaraiva/erlang:17.5
MAINTAINER Marlus Saraiva <marlus.saraiva@gmail.com>

RUN apk --update add elixir wget git && rm -rf /var/cache/apk/*

RUN mix local.hex --force && \
    mix local.rebar --force

CMD ["/bin/sh"]
