Elixir on Alpine Linux + basic development tools
=====

Elixir is a dynamic, functional language designed for building scalable and maintainable applications.

Image size: **48.12 MB**

> **Notice:** If you don't need to download dependencies or compile anything, you can use [msaraiva/elxir](https://registry.hub.docker.com/u/msaraiva/elixir/) which is much smaller.

The following packages are pre-installed:

- erlang + dependencies
- erlang-inets
- erlang-ssl
- erlang-public-key
- erlang-asn1
- erlang-sasl
- erlang-erl-interface
- erlang-dev
- elixir
- wget
- git + dependencies
- rebar
- hex

> **Notice:** In order to keep images as compact as possible, Erlang libraries for Alpine Linux are split into many different packages. The full list of Erlang packages available can be found [here](http://pkgs.alpinelinux.org/packages?package=erlang%25&repo=all&arch=x86_64).

## Usage

```
FROM msaraiva/elixir-dev

RUN apk --update add bash vim git-bash-completion && \
    rm -rf /var/cache/apk/*

RUN git clone https://github.com/elixir-lang/vim-elixir.git ~/.vim

RUN curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh -o ~/.git-prompt.sh

ADD prompt.sh /etc/profile.d/prompt.sh
ADD aliases.sh /etc/profile.d/aliases.sh
ADD gitconfig /root/.gitconfig

CMD ["/bin/bash", "-l"]
```
