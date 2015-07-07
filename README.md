Minimal Erlang/Elixir docker images with Alpine Linux
=====

Alpine Linux is a lightweight Linux distribution built around musl libc and busybox.
The main focus of this distribution is security, simplicity and resource efficiency.
All that makes Alpine Linux perfect to work as base images for linux containers.

When creating a docker image, you probably want to minimize its size as much as possible. At the same time, you might want to have access to a full-featured package system with a large range of packages available. As far as I can see, Alpine Linux is the best choice for that.

> **Notice:** In order to keep images as compact as possible, Erlang libraries for Alpine Linux are split into many different packages. To see a full list of the packages, run `docker run --rm msaraiva/alpine-erlang:3.2 apk --update search 'erlang' | sort`

I'll describe here some examples on how to create minimal docker images for Erlang/Elixir projects using Alpine Linux.

### Index

- [Installing Erlang with apk](#installing-erlang)
- [Installing Elixir with apk](#installing-elixir)
- [Hello world (compilation on host machine)](#hello-world-compilation-host)
- [Hello world (compilation inside the container)](#hello-world-compilation-container)
- [Phoenix + Elixir Release Manager (exrm)](#phoenix-exrm)
- [Hello Phoenix](#hello-phoenix)
- [Phoenix Chat Example](#phoenix-chat)
- [Hello NIF](#hello-nif)
- [Building images for development](#build-for-dev)
- [TODO](#todo)
- [Other Resources](#other-resources)



> **Note:** All base images listed here are automated builds and their Dockerfiles can be found in the [dockerfiles](https://github.com/msaraiva/docker-alpine/tree/master/dockerfiles) folder.

### <a name="installing-erlang"></a> Installing Erlang with apk

Create a Dockerfile

```Dockerfile
FROM msaraiva/alpine-erlang:3.2

RUN apk --update add erlang && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]
```

> **Note:** The `apk` command is the official tool for package management on Alpine Linux. Something like `apt-get` on Ubuntu. More information about `apk` can be found [here](http://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management).

Build the image:

```
$ docker build -t erlang .
```

Run `docker images`. You should see something like:

```
REPOSITORY           TAG           IMAGE ID           CREATED             VIRTUAL SIZE
erlang               latest        d76965a1f753       4 seconds ago       16.78 MB
```


### <a name="installing-elixir"></a> Installing Elixir with apk

```Dockerfile
FROM msaraiva/erlang:18.0.1

RUN apk --update add elixir && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]
```

### <a name="hello-world-compilation-host"></a> Hello world (compilation on host machine)
A simple command line executable

- Source: <https://github.com/msaraiva/docker-alpine-examples/>
- Built with `mix escript.build`
- Compilation on host machine
- Requires Erlang 18.0 and Elixir 1.0.5 on the host machine
- **You need to compile your application on the host machine before building this image**
- Image size: **18.83MB**

Dockerfile:

```
FROM msaraiva/erlang:18.0.1

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
hello                latest        dd650b702665       18 seconds ago      19.44 MB
```


Running:

```
$ docker run --rm hello Docker
Hello, Docker!
```


### <a name="hello-world-compilation-container"></a> Hello world (compilation inside the container)
The same simple command line executable. But:

- Compilation inside the container
- **No need to install Erlang/Elixir on the host machine**
- Image size: **19.5MB**

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

## <a name="phoenix-exrm"></a> Phoenix + Elixir Release Manager (exrm)


In order to generate releases for phoenix applications, you need to make some minimal changes in a couple of files. See [this page](http://www.phoenixframework.org/v0.13.0/docs/advanced-deployment) from the Phoenix documentation for details. If you just want to see the changes, take a look at [this commit](https://github.com/msaraiva/docker-alpine-examples/commit/16ce93a1288cd974c11c2f2923ced16993b6709f).

Another thing to pay attention to, is the architecture of the build environment. From the same Phoenix documentation:
> We need to be sure that the architectures for both our **build** and **hosting** environments are the same, e.g. 64-bit Linux -> 64-bit Linux. **If the architectures don't match, our application might not run when deployed**.

By default, `exrm` pulls the Erlang runtime system from the build environment. That means, if you generate a release, for instance, on Windows or OSX, your application will not run on Alpine Linux. There are two ways to deal with this:

1. Instruct `exrm` to exclude the Erlang runtime from the release
2. Gererate the release inside Alpine Linux

> **Note:** To instruct `exrm` to exclude the Erlang runtime from the release, we need to create a file called `rel/relx.config` with this content: `{include_erts, false}.`.
  
Let's see how we can do this.

### <a name="hello-phoenix"></a> Hello Phoenix

This is the hello phoenix application created when you run `mix phoenix.new hello_phoenix --no-brunch --no-ecto`

- Source: <https://github.com/msaraiva/docker-alpine-examples/>
- Built with `mix release`
- Compilation on host machine
- Requires Erlang 18.0 and Elixir 1.0.5 on the host machine
- **You need to generate a release on the host machine before building this image**
- Image size: **23.16MB**


Dockerfile:

```
FROM msaraiva/erlang:18.0.1

RUN apk --update add erlang-sasl && rm -rf /var/cache/apk/*

ENV APP_NAME hello_phoenix
ENV APP_VERSION "0.0.1"
ENV PORT 4000

RUN mkdir -p /$APP_NAME
ADD rel/$APP_NAME/bin /$APP_NAME/bin
ADD rel/$APP_NAME/lib /$APP_NAME/lib
ADD rel/$APP_NAME/releases/$APP_VERSION/$APP_NAME.boot    /$APP_NAME/releases/$APP_VERSION/$APP_NAME.boot
ADD rel/$APP_NAME/releases/$APP_VERSION/$APP_NAME.rel     /$APP_NAME/releases/$APP_VERSION/$APP_NAME.rel
ADD rel/$APP_NAME/releases/$APP_VERSION/$APP_NAME.script  /$APP_NAME/releases/$APP_VERSION/$APP_NAME.script
ADD rel/$APP_NAME/releases/$APP_VERSION/start.boot        /$APP_NAME/releases/$APP_VERSION/start.boot
ADD rel/$APP_NAME/releases/$APP_VERSION/sys.config        /$APP_NAME/releases/$APP_VERSION/sys.config
ADD rel/$APP_NAME/releases/$APP_VERSION/vm.args           /$APP_NAME/releases/$APP_VERSION/vm.args

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

### <a name="phoenix-chat"></a> Phoenix Chat Example

A fork from Chris McCord's [phoenix_chat_example](https://github.com/chrismccord/phoenix_chat_example).

- Source: <https://github.com/msaraiva/phoenix_chat_example>
- Built with `mix release`
- Compilation on host machine
- Requires Erlang 18.0 and Elixir 1.0.5 on the host machine
- **You need to generate a release on the host machine before building this image**
- Image size: **23.47MB**

Dockerfile:

```
FROM msaraiva/erlang:18.0.1

RUN apk --update add erlang-sasl && rm -rf /var/cache/apk/*

ENV APP_NAME chat
ENV APP_VERSION "0.0.1"
ENV PORT 4000

RUN mkdir -p /$APP_NAME
ADD rel/$APP_NAME/bin /$APP_NAME/bin
ADD rel/$APP_NAME/lib /$APP_NAME/lib
ADD rel/$APP_NAME/releases/$APP_VERSION/$APP_NAME.boot    /$APP_NAME/releases/$APP_VERSION/$APP_NAME.boot
ADD rel/$APP_NAME/releases/$APP_VERSION/$APP_NAME.rel     /$APP_NAME/releases/$APP_VERSION/$APP_NAME.rel
ADD rel/$APP_NAME/releases/$APP_VERSION/$APP_NAME.script  /$APP_NAME/releases/$APP_VERSION/$APP_NAME.script
ADD rel/$APP_NAME/releases/$APP_VERSION/start.boot        /$APP_NAME/releases/$APP_VERSION/start.boot
ADD rel/$APP_NAME/releases/$APP_VERSION/sys.config        /$APP_NAME/releases/$APP_VERSION/sys.config
ADD rel/$APP_NAME/releases/$APP_VERSION/vm.args           /$APP_NAME/releases/$APP_VERSION/vm.args

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

### <a name="hello-nif"></a> Hello NIF

A simple command line executable that calculates a dot product of two lists on a NIF

- Source: <https://github.com/msaraiva/docker-alpine-examples/>
- Built with `mix escript.build`
- NIF compiled with GCC
- Compilation inside the container
- **No need to install Erlang/Elixir on the host machine**
- Image size: **18.78MB**

Compiling:

```
$ cd hello_nif
$ docker run --rm -v $PWD:$PWD -w $PWD -e "MIX_ENV=prod" msaraiva/elixir-gcc sh -c "mix deps.get && mix escript.build"
```

Building the docker image:

```
$ docker build -t hello_nif .
```

Running:

```
$ docker run --rm hello_nif
Hello! This dot product was calculated by a NIF:
[1.0, 2.0, 3.0] x [5.0, 10.0, 20.0] = 85.0
```




## <a name="build-for-dev"></a> Building images for development

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

## <a name="todo"></a> TODO
* Erlang examples


## <a name="other-resources"></a> Other Resources

* <http://www.alpinelinux.org/about/>
* <https://github.com/gliderlabs/docker-alpine>
* <http://gliderlabs.viewdocs.io/docker-alpine>
* [@MarlusSaraiva](https://twitter.com/MarlusSaraiva) :)

