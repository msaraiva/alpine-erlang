Erlang/Elixir on Alpine Linux
=====

Alpine Linux is a lightweight Linux distribution built around **musl** libc and **busybox**.
The main focus of this distribution is security, simplicity and resource efficiency.
All that makes Alpine Linux perfect to work as base images for linux **containers**.

### Minimal docker images

When creating a docker image, you probably want to minimize its size as much as possible. At the same time, you might want to have access to a full-featured package system with a large range of packages available. As far as I can see, Alpine Linux is the best choice for that.

### How minimal?

```
REPOSITORY                     TAG      IMAGE ID       CREATED           VIRTUAL SIZE
msaraiva/phoenix_chat_example  latest   e2748af130aa   18 minutes ago    23.53 MB
msaraiva/elixir                1.0.5    7cce889dd856   48 minutes ago    29.53 MB
msaraiva/erlang                18.0     afe36ddc5624   58 minutes ago    16.78 MB

```

### Getting started

- [Packages](#packages)
  - [What is apk?](#what-is-apk)
  - [Installing Erlang with apk](#installing-erlang)
  - [Installing Elixir with apk](#installing-elixir)
- [Docker images](#docker-images)
  - <a href="https://registry.hub.docker.com/u/msaraiva/alpine-erlang/" target="_blank">msaraiva/alpine-erlang</a>
  - <a href="https://registry.hub.docker.com/u/msaraiva/erlang/" target="_blank">msaraiva/erlang</a>
  - <a href="https://registry.hub.docker.com/u/msaraiva/alpine-elixir/" target="_blank">msaraiva/alpine-alixir</a>
  - <a href="https://registry.hub.docker.com/u/msaraiva/elixir/" target="_blank">msaraiva/elixir</a>
  - msaraiva/lfe (TODO)
- [Examples](#examples)
  - Erlang
    - Cowboy Hello World (TODO)
  - Elixir
    - [Hello world (compilation on host machine)](#hello-world-compilation-host)
    - [Hello world (compilation inside the container)](#hello-world-compilation-container)
    - [Phoenix + Elixir Release Manager (exrm)](#phoenix-exrm)
    - [Hello Phoenix](#hello-phoenix)
    - [Phoenix Chat Example](#phoenix-chat)
    - [Hello NIF](#hello-nif)
  - LFE (TODO)
- [Building packages](building-packages)
  - [Patches](#patches)
  - [Build status and test results](build-status)
- [Contributing](#contributing)
- [Credits](#credits)
- [Other Resources and News](#other-resources)


## Packages

In order to keep packages as compact as possible, Erlang libraries for Alpine Linux are split into many different packages. The full list of Erlang packages available can be found [here](http://pkgs.alpinelinux.org/packages?package=erlang%25&repo=all&arch=x86_64).


### <a name="what-is-apk"></a> What is apk?

The `apk` command is the official tool for package management on Alpine Linux. Something like `apt-get` on Ubuntu. More information about `apk` can be found [here](http://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management).

### <a name="installing-erlang"></a> Installing Erlang with apk

Create a Dockerfile

```Dockerfile
FROM msaraiva/alpine-erlang-base:3.2

RUN apk --update add erlang && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]
```

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
FROM msaraiva/alpine-erlang-base:3.2

RUN apk --update add elixir && rm -rf /var/cache/apk/*

CMD ["/bin/sh"]
```

## <a name="docker-images"></a> Docker Images
> **Note:** All base images listed here are automated builds and their Dockerfiles can be found in the [dockerfiles](https://github.com/msaraiva/docker-alpine/tree/master/dockerfiles) folder.

## <a name="examples"></a> Examples

I'll describe here some examples on how to create minimal docker images for Erlang/Elixir projects using Alpine Linux.

### <a name="hello-world-compilation-host"></a> Hello world (compilation on host machine)
A simple command line executable

- Source: <https://github.com/msaraiva/docker-alpine-examples/>
- Built with `mix escript.build`
- Compilation on host machine
- Requires Erlang 18.0 and Elixir 1.0.5 on the host machine
- **You need to compile your application on the host machine before building this image**
- Image size: **19.5MB**

Dockerfile:

```Dockerfile
FROM msaraiva/alpine-elixir-base:18.0

ADD hello /usr/local/bin/hello

ENTRYPOINT ["/usr/local/bin/hello"]
```

Building:

```Dockerfile
$ git clone https://github.com/msaraiva/docker-alpine-examples
$ cd docker-alpine-examples/hello
$ MIX_ENV=prod mix escript.build
$ docker build -t hello .
```

Run `docker images`. You should see something like:

```
REPOSITORY           TAG           IMAGE ID           CREATED             VIRTUAL SIZE
hello                latest        dd650b702665       18 seconds ago      19.5 MB
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
- Image size: **19.55MB**
ex
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
- Image size: **23.21MB**


Dockerfile:

```Dockerfile
FROM msaraiva/alpine-elixir-base:18.0

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
- Image size: **23.53MB**

Dockerfile:

```Dockerfile
FROM msaraiva/alpine-elixir-base:18.0

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
- Image size: **19.44MB**

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

## <a name="building-packages"></a> Building packages

You can see how packages are built by looking at the APKBUILD scripts:

- [Erlang 17.5](http://git.alpinelinux.org/cgit/aports/tree/testing/erlang17/APKBUILD?id=d353b4d850cc0ad9c4a99251e8d9e734080a49b5)
- [Erlang 18.0](http://git.alpinelinux.org/cgit/aports/tree/testing/erlang/APKBUILD?id=d353b4d850cc0ad9c4a99251e8d9e734080a49b5)
- [Elixir 1.0.5](http://git.alpinelinux.org/cgit/aports/tree/testing/elixir/APKBUILD?id=d353b4d850cc0ad9c4a99251e8d9e734080a49b5)

For more info, see <http://wiki.alpinelinux.org/wiki/APKBUILD_Reference>

### <a name="patches"></a> Patches

If you take a look at the APKBUILD scripts, you'll notice that some patches are applied in order to build the packages.
Some of those patches are related to musl, some to Busybox and some just split or remove stuff to make packages smaller.
 - [Patches for Erlang](http://git.alpinelinux.org/cgit/aports/tree/testing/erlang?id=d353b4d850cc0ad9c4a99251e8d9e734080a49b5)
 - [Patches for Elixir](http://git.alpinelinux.org/cgit/aports/tree/testing/elixir?id=d353b4d850cc0ad9c4a99251e8d9e734080a49b5)


### <a name="build-status"></a> Build status and test results

Erlang packages:

| Package  | Version | Build |   Smoke Tests       |    Complete test suites           | Maintainer     |   Repo  |
|----------|:-------:|:-----:|:-------------------:|:---------------------------------:|----------------|:-------:|
| Erlang17 |   17.5  |   OK  |         TODO        | [11475 passed, 108 failed][t0001] | Marlus Saraiva | testing |
| Erlang   | 18.0.1  |   OK  |         TODO        | [12036 passed,  94 failed][t0002] | Marlus Saraiva | testing |

Other packages, applications or libraries:

| Package  | Version | Build |             Tests            | Maintainer     |   Repo  |
|----------|:-------:|:-----:|:----------------------------:|----------------|:-------:|
| Elixir   |  1.0.5  |   OK  |              OK              | Marlus Saraiva | testing |
| Ejabberd |  15.04  |   OK  |             TODO             | John Regan     | testing |

[t0001]: http://alpine-erlang-tests.s3-website-us-east-1.amazonaws.com/0001/
[t0002]: http://alpine-erlang-tests.s3-website-us-east-1.amazonaws.com/0002/

## <a name="contributing"></a> Contributing

I'll definitely need help to keep this project alive, so contributions are more than welcome. There're a lot of ways to contribute:

- Compiling and testing other projects based on Erlang
- Maintaining packages or creating new ones
- Patching existing packages
- Creating or improving docker images
- Creating new examples
- Improving this page
  
Feedback is also very important. If you have something to share, fell free to open an issue.
  
## <a name="credits"></a> Credits

  - John Regan (original maintainer of the Erlang packages)
  - Peter Lemenkov (See the [patches](#patches))
  
## <a name="other-resources"></a> Other Resources and News

* <https://twitter.com/MarlusSaraiva>
* <http://www.alpinelinux.org>
* <https://registry.hub.docker.com/u/library/alpine/>
* <https://github.com/gliderlabs/docker-alpine>
