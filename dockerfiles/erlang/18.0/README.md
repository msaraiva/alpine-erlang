Erlang & OTP on Alpine Linux
=====

Erlang & OTP minimal environment based on Alpine Linux. 

Image size: **16.78 MB**

For more info and examples on creating minimal docker images, see ["Erlang/Elixir on Alpine Linux"](https://github.com/msaraiva/docker-alpine).

The following packages are pre-installed:

- ncurses-libs
- erlang-kernel
- erlang-stdlib
- erlang-compiler
- erlang

> **Notice:** In order to keep images as compact as possible, Erlang libraries for Alpine Linux are split into many different packages. The full list of Erlang packages available can be found [here](http://pkgs.alpinelinux.org/packages?package=erlang%25&repo=all&arch=x86_64)

## Usage

```
FROM msaraiva/erlang:18.0

RUN apk --update add elixir && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]

```
