Minimal environment for running Erlang applications on Alpine Linux
=====

This image contains the basic dependencies for running Erlang applications.

Adds the following libraries to the original alpine image:

- ncurses-libs

Image size: **5.676 MB** 

>  **Notice:** This image does not contain Erlang. If you need Erlang installed, see [msaraiva/erlang](https://registry.hub.docker.com/u/msaraiva/erlang/)


## Usage

```
FROM msaraiva/alpine-erlang-base:3.2

RUN apk --update add erlang && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]

```
