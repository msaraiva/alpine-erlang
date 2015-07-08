Minimal environment for running Elixir applications on Alpine Linux
=====

This image contains the basic dependencies for running Elixir applications.

Image size: **17.49 MB**

The following packages are pre-installed:

- ncurses-libs
- erlang-kernel
- erlang-stdlib
- erlang-compiler
- erlang-crypto
- erlang-syntax-tools
- erlang

> **Notice:** In order to keep images as compact as possible, Erlang libraries for Alpine Linux are split into many different packages. The full list of Erlang packages available can be found [here](http://pkgs.alpinelinux.org/packages?package=erlang%25&repo=all&arch=x86_64)

## Usage

```
FROM msaraiva/alpine-elixir-base:18.0

RUN apk --update add elixir && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]

```
