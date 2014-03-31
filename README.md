# **Autobahn**|Cpp

**Autobahn**|Cpp is a subproject of [Autobahn](http://autobahn.ws/) which implements the [Web Application Messaging Protocol (WAMP)](http://wamp.ws/) in C++ supporting the following application roles

 * **Caller**
 * **Callee**
 * **Publisher**
 * **Subscriber**

The API and implementation make use of modern C++ 11 and new asynchronous idioms using (upcoming) features of the standard C++ library, in particular **Futures**, [**Continuations**](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3634.pdf) and **Lambdas**. [Continuations](http://en.wikipedia.org/wiki/Continuation) are *one* way of managing control flow in an asynchronous program. Other styles include:

 * asynchronous [Callbacks](http://en.wikipedia.org/wiki/Callback_%28computer_programming%29)
 * [Coroutines](http://en.wikipedia.org/wiki/Coroutine) (`yield` or `await`)
 * Actors ([Erlang/OTP](http://www.scala-lang.org/), [Scala](http://www.scala-lang.org/)/[Akka](http://akka.io/) or [Rust](http://www.scala-lang.org/))
 * [Transactional memory](http://en.wikipedia.org/wiki/Transactional_Synchronization_Extensions)

The library is "header-only", light-weight (< 2k code lines) and **depends on** the following:

 * C++ 11 compiler
 * `boost::future`
 * `boost::any`
 * `boost::asio`

**Autobahn**|Cpp supports running WAMP `rawsocket-msgpack` over TCP(-TLS), Unix domain sockets or pipes (`stdio`).
 
> The library and example programs are tested and developed with **clang 3.4**, **libc++** and **Boost trunk/1.56** on an Ubuntu 13.10 x86-64 bit system. It also works with **gcc 4.8**, **libstdc++** and **Boost trunk/1.56**. Your mileage with other versions of the former may vary, but we accept PRs;)


## Show me some code!

Here is how programming with C++ and **Autobahn**|Cpp looks like.

**Calling a remote Procedure**

```c++
auto c1 = session.call("com.mathservice.add2", {23, 777})
.then(
   [&](future<any> f) {

      // call result received
		//
		cout << "Got RPC result " << any_cast<uint64_t> (f.get()) << endl;
	});
```

**Registering a remoted Procedure**
```c++
auto r1 = session.provide("com.myapp.cpp.square",

   [](const anyvec& args, const anymap& kwargs) {

      cerr << "Someone is calling my lambda function .." << endl;

      uint64_t x = any_cast<uint64_t> (args[0]);
      return x * x;
   }
);
```

**Publishing an Event**

```c++
session.publish("com.myapp.topic2", {23, true, string("hello")});
```

**Subscribing to a Topic**

```c++
// 4) subscribe an event handler to a topic
//
auto s1 = session.subscribe("com.myapp.topic1",
   [](const anyvec& args, const anymap& kwargs) {
      cout << "Got event: " << any_cast<uint64_t>(args[0]) << endl;
   })
.then(
   [](future<subscription> sub) {
      cout << "Subscribed with subscription ID " << sub.get().id << endl;
   });

```


## Building

> Notes:
>
> * Support for GNU g++/libstdc++ is tracked on this [issue](https://github.com/tavendo/AutobahnCpp/issues/1). Support for Window/MSVC is tracked on this [issue](https://github.com/tavendo/AutobahnCpp/issues/2)
> * While C++ 11 provides a `future` - but this does not yet support continuations. **Autobahn**|Cpp makes use of `boost::future.then` for attaching continuations to futures as outlined in the proposal [here](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3634.pdf). This feature will come to standard C++, but probably not before 2015 (see [C++ Standardisation Roadmap](http://isocpp.org/std/status))
> * Support for `when_all` and `when_any` depends on Boost 1.56 (!) or higher.


### Build tools

Install some libs and build tools:

```shell
sudo apt-get install libbz2-dev libssl-dev ruby libtool autoconf scons
```

### clang

Install [clang](http://clang.llvm.org/) and [libc++](http://libcxx.llvm.org/):

```shell
sudo apt-get install clang-3.4 libc++1 libc++-dev
```

### Boost

Get the latest Boost from [here](http://www.boost.org/). Then

```shell
cd $HOME
tar xvjf Downloads/boost_1_55_0.tar.bz2
cd boost_1_55_0/
./bootstrap.sh
./b2 toolset=clang cxxflags="-stdlib=libc++" linkflags="-stdlib=libc++" -j 4
```

Get Boost trunk by doing:

```shell
git clone --recursive git@github.com:boostorg/boost.git
```


Add the following to `$HOME/.profile`

```shell
export LD_LIBRARY_PATH=${HOME}/boost_1_55_0/stage/lib:${LD_LIBRARY_PATH}
```

### MsgPack-C

Get [MsgPack-C](https://github.com/msgpack/msgpack-c) and build with clang:

```shell
cd $HOME
git clone https://github.com/msgpack/msgpack-c.git
cd msgpack-c
./bootstrap
CXX=`which clang++` CC=`which clang` CXXFLAGS="-std=c++11 -stdlib=libc++" \
   LDFLAGS="-stdlib=libc++" ./configure --prefix=$HOME/msgpack_clang
make
make install
```

Add the following to `$HOME/.profile`

```shell
export LD_LIBRARY_PATH=${HOME}/msgpack_clang/lib:${LD_LIBRARY_PATH}
```


## Use clang
##
export CC='clang'
export CXX='clang++'

## Libaries (clang based)
##
export BOOST_ROOT=${HOME}/boost_trunk_clang
export LD_LIBRARY_PATH=${BOOST_ROOT}/stage/lib:${LD_LIBRARY_PATH}

export MSGPACK_ROOT=${HOME}/msgpack_clang
export LD_LIBRARY_PATH=${MSGPACK_ROOT}/lib:${LD_LIBRARY_PATH}


## Use GNU
##
export CC='gcc'
export CXX='g++'

## Libraries (GCC based)
##
export BOOST_ROOT=${HOME}/boost_trunk_gcc
export LD_LIBRARY_PATH=${BOOST_ROOT}/stage/lib:${LD_LIBRARY_PATH}

export MSGPACK_ROOT=${HOME}/msgpack_gcc
export LD_LIBRARY_PATH=${MSGPACK_ROOT}/lib:${LD_LIBRARY_PATH}



### **Autobahn**|Cpp

## Note: You need to either set BOOST_ROOT to the root of a stock Boost distribution
## or set BOOST_INCLUDES and BOOST_LIBS if Boost comes with your OS distro e.g. and
## needs BOOST_INCLUDES=/usr/include/boost and BOOST_LIBS=/usr/lib like Ubuntu.
##

## Note: You need to either set MSGPACK_ROOT to the root of a stock MsgPack-C distribution
## or set MSGPACK_INCLUDES and MSGPACK_LIBS if MsgPack-C comes with your OS distro e.g. and
## needs MSGPACK_INCLUDES=/usr/include/boost and MSGPACK_LIBS=/usr/lib.
##

scons -j 4



Finally, to build **Autobahn**|Cpp

```shell
source $HOME/.profile
cd $HOME
git clone git@github.com:tavendo/AutobahnCpp.git
cd AutobahnCpp
scons
```

## Building Documentation

The documentation is built using Sphinx, Doxygen and Breathe.

Doxygen takes the C++ source files and autogenerates XML files from the documented C++ source code.

Breathe takes the XML files generated by Doxygen and generates Sphinx RST files.

Sphinx takes the RST files generated plus manually written RST files and generates the final documentation in HTML format.

### Install documentation build tools

```shell
sudo apt-get install doxygen python-sphinx
sudo /usr/bin/pip install breathe
```


_gen/doxygen



## Futures

* [ASIO C++11 Examples](http://www.boost.org/doc/libs/1_55_0/doc/html/boost_asio/examples/cpp11_examples.html)
* [Using Asio with C++11](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3388.pdf)
* [C++17: I See a Monad in Your Future! ](http://bartoszmilewski.com/2014/02/26/c17-i-see-a-monad-in-your-future/)
* [Boost Thread](http://www.boost.org/doc/libs/1_55_0/doc/html/thread.html)
* [Boost Issue: when_all](https://svn.boost.org/trac/boost/ticket/7447)
* [Boost Issue. when_any](https://svn.boost.org/trac/boost/ticket/7446)
* [Boost Issue: future fires twice](https://svn.boost.org/trac/boost/ticket/9711)
* [Boost C++ 1y](http://www.boost.org/doc/libs/1_55_0/doc/html/thread/compliance.html#thread.compliance.cxx1y.async)
* [Asynchronous API in C++ and the Continuation Monad](https://www.fpcomplete.com/blog/2012/06/asynchronous-api-in-c-and-the-continuation-monad)

## Closures Cheetsheet

* `[]` Capture nothing (or, a scorched earth strategy?)
* `[&]` Capture any referenced variable by reference
* `[=]` Capture any referenced variable by making a copy
* `[=, &foo]` Capture any referenced variable by making a copy, but capture variable `foo` by reference
* `[bar]` Capture `bar` by making a copy; don't copy anything else
* `[this]` Capture the this pointer of the enclosing class
