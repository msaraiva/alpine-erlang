Elixir on Alpine Linux
=====
From the official [website](http://elixir-lang.org/):
> Elixir is a dynamic, functional language designed for building scalable and maintainable applications.

Image size: **47.6 MB**

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

> **Notice:** In order to keep images as compact as possible, Erlang libraries for Alpine Linux are split into many different packages. To see a full list of the packages, run `docker run --rm msaraiva/alpine:3.1 apk --update search 'erlang' | sort`

## Usage

```
FROM msaraiva/elixir

RUN apk --update add bash vim git-bash-completion && \
    rm -rf /var/cache/apk/*

RUN git clone https://github.com/elixir-lang/vim-elixir.git ~/.vim

RUN curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh -o ~/.git-prompt.sh

ADD prompt.sh /etc/profile.d/prompt.sh
ADD aliases.sh /etc/profile.d/aliases.sh
ADD gitconfig /root/.gitconfig

CMD ["/bin/bash", "-l"]
```

