Building Erlang/OTP against musl libc
=====

I'll be using Alpine Lixux here to describe how to build Erlang against musl libc. In case you want to use any other distribution, make sure you find and install all the corresponding dependencies.

### Using Docker
I'll also be using docker to get started with a clean linux environment. In case you already have Alpine Linux installed, you can skip this part.

```
docker run -it alpine:3.2 /bin/sh
```

### Installing dependencies

```
apk update
apk add git autoconf build-base perl-dev zlib-dev ncurses-dev openssl-dev
```

For building Erlang/OTP with jinterface (optional):

```
apk add openjdk7
export PATH="/usr/lib/jvm/java-1.7-openjdk/bin:$PATH"
```

For building Erlang/OTP with odbc (optional):

```
apk add unixodbc-dev
```
### Cloning the OTP repo and choosing a version

```
git clone https://github.com/erlang/otp.git
cd otp
git checkout OTP-18.0.2
```

### Setting environment variables

```
export ERL_TOP=$PWD
export PATH=$ERL_TOP/bin:$PATH
export CPPFLAGS="-D_BSD_SOURCE $CPPFLAGS"
```

### Applying patches
This is probably the most important part of this tutorial. In order to make your life easier, I placed all the patches you need in the patches folder.

> **Notice:** When building the official Erlang packages for Alpine Linux, other patches are also applied. Some of them are related to Busybox and some just split or remove stuff to make packages smaller. For this tutorial I'm only applying those necessary to have a successful build. In case you want apply any of the other patches, you can download them from: <http://git.alpinelinux.org/cgit/aports/tree/main/erlang>



```
PATH_TO_PATCHES="https://raw.githubusercontent.com/msaraiva/alpine-erlang/master/building_against_musl/patches/"
curl $PATH_TO_PATCHES/remove-private-unit32.patch | patch -p1
curl $PATH_TO_PATCHES/replace_glibc_check.patch | patch -p1
curl $PATH_TO_PATCHES/hipe_x86_signal-fix.patch | patch -p1
```

### Generate the `configure` file

```
./otp_build autoconf
```

### Configure and build

```
./configure
make
```

### Try it out!

```
$ERL_TOP/bin/erl
```

You should now see something like:

```
Erlang/OTP 18 [erts-7.0.2] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V7.0.2  (abort with ^G)
1>
```

### Installing

```
make install
```

That's all!

In case you have any question, feel free to ping me on [twitter](https://twitter.com/MarlusSaraiva).
