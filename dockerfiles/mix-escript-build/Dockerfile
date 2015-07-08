FROM msaraiva/alpine-elixir-base:18.0
MAINTAINER Marlus Saraiva <marlus.saraiva@gmail.com>

ONBUILD ADD config/  /app/config/
ONBUILD ADD lib/     /app/lib/
ONBUILD ADD mix.exs  /app/mix.exs

WORKDIR /app

ENV MIX_ENV prod

ONBUILD RUN apk --update add --virtual build-dependencies elixir wget git && \
    mix local.hex --force && \
    mix local.rebar --force && \
    mix escript.build && \
    APP_NAME=$(mix run -e  'IO.puts Mix.Project.config |> Keyword.get(:app)') && \
    echo "#!/bin/sh" > /app/run.sh && \
    echo "exec /app/$APP_NAME \$@" >> /app/run.sh && \
    chmod +x /app/run.sh && \
    apk del build-dependencies && \
    rm -rf /app/_build && \
    rm -rf /root/.mix && \
    rm -rf /etc/ssl && \
    rm -rf /var/cache/apk/*

ENTRYPOINT ["/app/run.sh"]
