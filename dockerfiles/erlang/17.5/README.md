msaraiva/erlang
=====

Erlang & OTP minimal environment based on Alpine Linux. 

This image contains the basic dependencies for running Elixir applications.

The following packages are pre-installed:

- ncurses
- ncurses-terminfo-base
- ncurses-libs
- erlang-kernel
- erlang-stdlib
- erlang-compiler
- erlang-crypto
- erlang-syntax-tools
- erlang


## Usage

```
FROM msaraiva/erlang:17.5

RUN apk --update add elixir && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]

```

