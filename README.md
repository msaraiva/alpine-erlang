Minimal Erlang/Elixir docker images with Alpine Linux
=====

Alpine Linux is a lightweight Linux distribution built around musl libc and busybox.
The main focus of this distribution is security, simplicity and resource efficiency.
All that makes Alpine Linux perfect to work as base images for linux containers.

When creating a docker image, you probably want to minimize its size as much as possible. At the same time, you might want to have access to a full-featured package system with a large range of packages available. As far as I can see, Alpine Linux is the best choice for that.

> **Notice:** In order to keep images as compact as possible, Erlang libraries for Alpine Linux are split into many different packages. To see a full list of the packages, run `docker run --rm msaraiva/alpine:3.1 apk --update search 'erlang' | sort`



## Examples

I'll describe here some examples on how to create minimal docker images for Elixir projects using Alpine Linux.

> **Note:** All base images listed here are automated builds and their Dockerfiles can be found in the [dockerfiles](https://github.com/msaraiva/docker-alpine/tree/master/dockerfiles) folder.

### Hello world (compilation on host machine)
A simple command line executable

- Source: <https://github.com/msaraiva/docker-alpine-examples/>
- Built with `mix escript.build`
- Compilation on host machine
- Requires Erlang 17.5 and Elixir 1.0.4 on the host machine
- **You need to compile your application on the host machine before building this image**
- Image size: **18.83Mb**

Dockerfile:

```
FROM msaraiva/erlang:17.5

ADD hello /usr/local/bin/hello

ENTRYPOINT ["/usr/local/bin/hello"]
```

Building:

```
$ git clone https://github.com/msaraiva/docker-alpine-examples
$ cd docker-alpine-examples/hello
$ MIX_ENV=prod mix escript.build
$ docker build -t hello .
```

Run `docker images`. You should see something like:

```
REPOSITORY           TAG           IMAGE ID           CREATED             VIRTUAL SIZE
hello                latest        dd650b702665       18 seconds ago      18.83 MB
```


Running:

```
$ docker run --rm hello Docker
Hello, Docker!
```

If you want to manually install the erlang packages, you can replace the content of your `Dockerfile` with this:

```
FROM alpine:3.1

RUN echo 'http://dl-4.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories

RUN apk --update add erlang erlang-crypto erlang-syntax-tools && rm -rf /var/cache/apk/*

ADD hello /usr/local/bin/hello

ENTRYPOINT ["/usr/local/bin/hello"]

```

> **Note:** The `apk` command is the official tool for package management on Alpine Linux. Something like `apt-get` on Ubuntu. More information about `apk` can be found [here](http://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management).


### Hello world (compilation inside the container)
The same simple command line executable. But:

- Compilation inside the container
- **No need to install Erlang/Elixir on the host machine**
- Image size: **18.89Mb**

For this example, we'll be using [msaraiva/mix-escript-build](https://github.com/msaraiva/docker-alpine/blob/master/dockerfiles/mix-escript-build/Dockerfile/) as base image. This image uses the `ONBUILD` docker instruction to add project files required for compilation, just before the build. For more information about the `ONBUILD` instruction, check out the [Dockerfile Reference](https://docs.docker.com/reference/builder/#onbuild).

Dockerfile:

```
FROM msaraiva/mix-escript-build
```

Building:

```
$ docker build -t hello -f Dockerfile_compiling .
```

> **Notice:** You'll need docker 1.5 or higher in order to use the `-f` option. In case you cannot update the docker version, just rename `Dockerfile_compiling` to `Dockerfile` and run the docker build command as before.


Running:

```
$ docker run --rm hello Docker
Hello, Docker!
```

##Phoenix + Elixir Release Manager (exrm)


In order to generate releases for phoenix applications, you need to make some minimal changes in a couple of files. See [this page](http://www.phoenixframework.org/v0.13.0/docs/advanced-deployment) from the Phoenix documentation for details. If you just want to see the changes, take a look at [this commit](https://github.com/msaraiva/docker-alpine-examples/commit/16ce93a1288cd974c11c2f2923ced16993b6709f).

Another thing to pay attention to, is the architecture of the build environment. From the same Phoenix documentation:
> We need to be sure that the architectures for both our **build** and **hosting** environments are the same, e.g. 64-bit Linux -> 64-bit Linux. **If the architectures don't match, our application might not run when deployed**.

By default, `exrm` pulls the Erlang runtime system from the build environment. That means, if you generate a release, for instance, on Windows or OSX, your application will not run on Alpine Linux. There are two ways to deal with this:

1. Instruct `exrm` to exclude the Erlang runtime from the release
2. Gererate the release inside Alpine Linux

> **Note:** To instruct `exrm` to exclude the Erlang runtime from the release, we need to create a file called `rel/relx.config` with this content: `{include_erts, false}.`.
  
Let's see how we can do this.

### Hello Phoenix

This is the hello phoenix application created when you run `mix phoenix.new hello_phoenix --no-brunch --no-ecto`

- Source: <https://github.com/msaraiva/docker-alpine-examples/>
- Built with `mix release`
- Compilation on host machine
- Requires Erlang 17.5 and Elixir 1.0.4 on the host machine
- **You need to generate a release on the host machine before building this image**
- Image size: **22.55Mb**


Dockerfile:

```
FROM msaraiva/erlang:17.5

RUN apk --update add erlang-sasl && rm -rf /var/cache/apk/*

ENV APP_NAME hello_phoenix
ENV PORT 4000

RUN mkdir -p /$APP_NAME
ADD rel/$APP_NAME/bin /$APP_NAME/bin
ADD rel/$APP_NAME/lib /$APP_NAME/lib
ADD rel/$APP_NAME/releases /$APP_NAME/releases

EXPOSE $PORT

CMD trap exit TERM; /$APP_NAME/bin/$APP_NAME foreground & wait
```

Building:

```
$ cd hello_phoenix
$ mix deps.get
$ MIX_ENV=prod mix release
$ docker build -t hello_phoenix .
```

Running:

```
$ docker run --rm -p 4000:4000 hello_phoenix
```

### Phoenix Chat Example

A fork from Chris McCord's [phoenix_chat_example](https://github.com/chrismccord/phoenix_chat_example).

- Source: <https://github.com/msaraiva/phoenix_chat_example>
- Built with `mix release`
- Compilation on host machine
- Requires Erlang 17.5 and Elixir 1.0.4 on the host machine
- **You need to generate a release on the host machine before building this image**
- Image size: **22.86Mb**

Dockerfile:

```
FROM msaraiva/erlang:17.5

RUN apk --update add erlang-sasl && rm -rf /var/cache/apk/*

ENV APP_NAME chat
ENV PORT 4000

RUN mkdir -p /$APP_NAME
ADD rel/$APP_NAME/bin /$APP_NAME/bin
ADD rel/$APP_NAME/lib /$APP_NAME/lib
ADD rel/$APP_NAME/releases /$APP_NAME/releases

EXPOSE $PORT

CMD trap exit TERM; /$APP_NAME/bin/$APP_NAME foreground & wait
```


Building:

```
$ git clone https://github.com/msaraiva/phoenix_chat_example
$ cd phoenix_chat_example
$ mix deps.get
$ MIX_ENV=prod mix release
$ docker build -t msaraiva/phoenix_chat_example .
```

Running:

```
$ docker run --rm -p 4000:4000 msaraiva/phoenix_chat_example
```


## Building images for development

In case you want to create images for development, you'll need to install git and wget. This way, mix can properly download and compile all dependencies. Check out the Dockerfile from [msaraiva/elixir](https://registry.hub.docker.com/u/msaraiva/elixir/):

```
FROM msaraiva/erlang:17.5
MAINTAINER Marlus Saraiva <marlus.saraiva@gmail.com>

RUN apk --update add elixir wget git && rm -rf /var/cache/apk/*

RUN mix local.hex --force && \
    mix local.rebar --force

CMD ["/bin/sh"]
```

The Dockerfile above creates an image of **47.54MB**.

Running:

```
$ docker run --rm -it msaraiva/elixir
$ git clone https://github.com/msaraiva/docker-alpine-examples
$ cd docker-alpine-examples/hello
$ mix deps.get
$ mix escript.build
$ ./hello Docker
Hello, Docker!
```

You can use the above image as base image for your own Dockerfile. Add more packages so you can customize your environment as much as you need. Here is an example:

Dockerfile from <https://github.com/msaraiva/docker-alpine-examples/blob/master/elixir-dev/>:

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

## TODO
* More examples?


## Other Resources

* <https://github.com/gliderlabs/docker-alpine>
* <http://www.alpinelinux.org/about/>
* <http://gliderlabs.viewdocs.io/docker-alpine>

