## Overview

[![Join the chat at https://gitter.im/scala-native/scala-native](https://badges.gitter.im/scala-native/scala-native.svg)](https://gitter.im/scala-native/scala-native?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status](https://travis-ci.org/scala-native/scala-native-example.svg?branch=master)](https://travis-ci.org/scala-native/scala-native-example)

This project is an example of how you can employ [Scala Native] on your next project.

For an introduction for the completely newbie on the subject, please find the FAQ down below.


## For the impatient

This is what you will be doing, in a nutshell:

* installation of LLVM and Clang (if needed!)
* installation of libgc-dev
* run *this* application and see the output generated

#### Installing LLVM and CLang

**NOTE:** If you already have LLVM >= 3.7 and Clang >= 3.7 installed in your system, you probably can skip to the next section.

You can install LLVM and Clang from their APT repository:
* [Installation script for Debian Jessie]

In case the LLVM APT repository is down, please install from pre-built binaries:

* Under section "Download LLVM 3.8.0", look for section [Pre-Build Binaries]

**This is an example on how you can install the pre-built binaries:**

<pre>
    # This is an example on how you can install the pre-built binaries
    $ mkdir -p $HOME/Downloads && cd $HOME/Downloads
    $ wget http://llvm.org/releases/3.8.0/clang+llvm-3.8.0-x86_64-linux-gnu-debian8.tar.xz
    $ mkdir -p $HOME/tools/developer && cd $HOME/tools/developer
    $ tar xpf $HOME/Downloads/clang+llvm-3.8.0-x86_64-linux-gnu-debian8.tar.xz
    $ export PATH=$HOME/tools/developer/clang+llvm-3.8.0-x86_64-linux-gnu-debian8/bin:$PATH
    $ hash -r
</pre>


#### Installing libgc-dev
```bash
    $ sudo apt-get install libgc-dev -y --force-yes
```

#### Run *this* application and see the output generated

Run the example application
```bash
    $ sbt run
```

Install xdg-open, which will help you open the generated image, like shown below:
```bash
    $ sudo apt-get install xdg-utils
    $ xdg-open image0.ppm
```

![Ray Tracing Demo](/image0.png)

## Troubleshooting

Some problems observed and well known solutions:

* In case the build fails, and you are using ``sbt-coursier`` plugin, try to remove its cache at ``$HOME/.coursier`` or try to remove the plugin from the SBT configuration.


## For advanced users only

If your application needs features only available in most recent Scala Native sources, not yet
available in public Maven repositories, you will probably find the instructions below useful.

#### Building and installing Scala Native

Downloading...
```bash
    $ mkdir -p $HOME/workspace
    $ cd $HOME/workspace
    $ git clone https://github.com/scala-native/scala-native.git
    $ cd $HOME/workspace/scala-native
```

Make sure you disable generation of documentation in ``build.sbt``:
```scala
lazy val baseSettings = Seq(
  organization := "org.scala-native",
  version      := nativeVersion,
  sources in doc in Compile := List(), // doc generation currently broken
  scalafmtConfig := Some(file(".scalafmt"))
)
```

Clean everything involving scala-native under your Ivy repository. Then proceed with the build:
```bash
    $ find $HOME/.ivy2 -type d -name '*scala-native*' | xargs rm -r -f
    $ sbt clean nscplugin/publishLocal nativelib/publishLocal publishLocal
```


## FAQ

#### What Scala is?

[Scala] is a functional programming language originally based on the JVM (Java Virtual Machine), which means that the compiler generates _bytecode_ to be executed by the JVM, instead of generating _native code_.

#### What Scala Native is?

Scala Native is pretty much Scala (as explained above), but able to generate _native code_ to be executed by the _target platform_, in other words: executed directly on the "bare metal", instead of being executed by the JVM.

#### Which target platforms are supported?

Short answer: OSX and Linux.

Long answer: Scala Native employs something called [LLVM], which does the job of generating the binary code for a given platform. So, the answer is, in principle: all platforms supported by LLVM. In reality, supporting a platform involves other aspects too, not simply delegating the task to LLVM. So, stay tuned to [Scala Native] in this regard.

#### Can I write code in Scala Native which is able to run in a browser?

You can use [ScalaJS] for that. ScalaJS is pretty much Scala (as explained above), but able to generate JavaScript, instead of generating _bytecode_ for the JVM. Using ScalaJS (and not Scala Native!), you can write code in Scala and generate JavaScript which a browser is able to understand.

#### Scala Native employs AOT and not JIT... OK, what does it mean?

The Scala Native compiler generates _native code_ for the _target platform_ and optimizes the code _ahead of time_ (AOT) of the execution of your program.

In contrast, Scala generates _bytecode_ for a _virtual machine_ and optimizes the code _just in time_ (JIT) with the execution of your program. In other words, the JIT compiler transforms _bytecode_ onto _native code_ behind the scenes, whilst your program is executing.

#### Should I use Scala with JIT or Scala Native with AOT?

It depends. If you are interested in binary portability across platforms, the answer is Scala with JIT. If you need fast startup times, or get rid of the JVM, or integrating system languages... the answer is Scala Native. If you are interested on writing code to run in a browser, the answer is ScalaJS.

#### What a system language is?

It basically means that you are able to write system software, such as native libraries and other executable binaries targetting specific platforms. In other words, a system language generates _native code_, able to run straight onto the "bare metal". You can even write an operating system with a system language or a combination of system languages.

#### Can I link Scala Native code with binaries from other languages?

At the time of this writing, Scala Native allows linking with C. Work is in progress in order to allow other programming languages. There are annotations which are intended to inform the linker about the foreign language symbol to be associated to variables and functions of your Scala Native code.

#### Does Scala Native with AOT performs better than Scala with JIT?

In regards to startup times, the answer is clearly _yes!_. In regards to other aspects, it's a bit early to tell.

The subjects "performance" and "benchmarks" involve several aspects which may behave in different ways, depending on the way these aspects are put together.

#### So, I do not need the JVM anymore, right?

Hum.... not really. You still need [SBT] in order to build Scala Native and _this_ project. And SBT needs the JVM. But you don't need the JVM on the target platform.

#### Will I be free of the JVM someday?

When [SBT] gets compiled with Scala Native. This is possibly a difficult task, which will take a while to happen or even eventually never happens.

A much more promissing alternative is another build tool called [CBT] being compiled with Scala Native. Since CBT has a much simpler code base than SBT, chances are that porting CBT to Scala Native is a relatively simple task. Since CBT is a new project, there's no ecosystem around it, which means that there's no commitment with past or risk of breaking plugins, for example.

#### Do I need LLVM for running a Scala Native program?

No. In the tutorial above you ran ``sbt run``, which creates an executable and immediately launches it. But you could have employed the command ``sbt nativeLink`` instead, which just creates the executable, without running it, as shown below:
```bash
    $ sbt nativeLink
    $ ls target/scala-2.11/scala-native-example-out
    $ target/scala-2.11/scala-native-example-out
    $ xdg-open image0.ppm
```
This way, you can deploy only the executable to the target platform, and nothing else.

## Advance stuff

#### Linking 3rd party libraries

Scala native lets you to lib Scala code with other 3rd party native libraries (i.e. C libraries).

Example shows how to link own simple arithmetic library.

Assuming that you have library `arith.c` :

```c
int addition(int a,int b)
{
int result;
result = a + b;
return result;
}
```

you need to compile it with command:

`gcc -c addition.c`

this will create `addition.o` file.

Then you need to create `*.a` file with :

`ar cr libarith.a addition.o`

which will create your `libarith.a` file.

Now in `scala-native-example` project, add this line in `build.sbt`

`nativeClangOptions ++= Seq("-L/path_to_your_library")` which is folder where your  `libarith.a` file is (-L parameter is important).

After that add link in your *.scala file :

```scala
package demo

import scalanative.native._, stdlib._, stdio._

object Main {
  def main(args: Array[String]): Unit = {

    fprintf(stderr, c"Hello world!\n   %d", addition.addition(1,3))  // our function call

  }
}

@extern
@link("arith")
object addition {
  def addition(a: CInt, b: CInt): CInt = extern
}

```

you can read more about C libraries here: http://www.thegeekstuff.com/2010/08/ar-command-examples/



[LLVM]: http://llvm.org
[Scala]: http://scala-lang.org
[ScalaJS]: http://scala-js.org
[Scala Native]: http://scala-native.org
[SBT]: http://scala-sbt.org
[CBT]: https://github.com/cvogt/cbt
[installation script for Debian Jessie]: https://gist.github.com/frgomes/daa33b2f7a6489196a95
[Installing CMake from sources]: http://gist.github.com/frgomes/d803ddaa3b124bb5abf32ba6cad8f3b8
[Installing LLVM and Clang from sources]: http://gist.github.com/frgomes/bb7426e70d118d06d7d30338cbdb1fe8
[Pre-Build Binaries]: http://llvm.org/releases/download.html#3.8.0
