Erlang & OTP
=====

Erlang & OTP minimal environment based on Alpine Linux. 

This image contains the basic dependencies for running Elixir applications.

The following packages are pre-installed:

- ncurses-libs
- erlang-kernel
- erlang-stdlib
- erlang-compiler
- erlang-crypto
- erlang-syntax-tools
- erlang

> **Notice:** In order to keep images as compact as possible, Erlang libraries for Alpine Linux are split into many different packages. To see a full list of the packages, run `docker run --rm msaraiva/alpine:3.1 apk --update search 'erlang' | sort`

## Usage

```
FROM msaraiva/erlang:17.5

RUN apk --update add elixir && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]

```

