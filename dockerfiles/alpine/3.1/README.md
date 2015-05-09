msaraiva/alpine
=====

Base image for Erlang/Elixir applications. 

Adds the following libraries to the original alpine image:

- ncurses-libs

## Usage

```
FROM msaraiva/alpine:3.1

RUN echo 'http://dl-4.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories

RUN apk --update add erlang && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]

```


