Alpine Linux for Erlang & Elixir Applications
=====

Base image for Erlang/Elixir applications. 

Adds the following libraries to the original alpine image:

- ncurses-libs

## Usage

```
FROM msaraiva/alpine:3.1

RUN apk --update add erlang && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]

```

