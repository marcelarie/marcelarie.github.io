---
title: 'Learn Async/Await in Rust creating a Web Crawler'
date: 2021-09-29T19:18:38+02:00
draft: true
---

### What is async-await ?

If you just started programming async-await may feel a strange concept. For now
you write code in a file and it runs in order, that is it right? But the web
doesn't work like that, all the communication that we do today its asynchronous.

So for example this will be synchronous code:

```rust
fn synchronous_hello() {

    println!("hello!");

}


fn main() {

    let hello = synchronous_hello(); // "hello!" gets printed

}
```

```rust
use futures::executor::block_on; // we talk about this later :)


async fn asynchronous_hello() {

    println!("hello!");

}


fn main() {

    let hello_for_later = asynchronous_hello(); // Nothing gets printed

        // ... ton of more code that can be run here

    block_on(hello_for_later); // "hello!" gets printed

}
```

So with that you are set for the rest of the post, async-await makes it easy to
write asynchronous code. Rust it's not a mature language so for some time didn't
have the async-await syntax.

But now we got it, and we gonna learn together, myself included, how it works!

### What I need to make my rust code asynchronous ?

To create asynchronous functions you just need to add the `async` keyword before the function.

```rust
async fn return_ten() -> u64 { 10 }
```

To run that code you will need an executor or the await keyword. For that we will use the futures library. That comes with multiple abstractions to work with async-await. Here you have an example of both methods:

```toml
[dependencies]

futures = "0.3"
```

```rust
use futures::executor::block_on;


async fn return_ten() -> u64 { 10 }


async fn await_for_ten_or_return_zero() -> u64 {

    if return_ten().await == 10 {

        10

    } else {

        0 //

    }

}


fn main() {

   let ten = block_on(await_for_ten());

   println!("Finally {} !", ten) // "Finally 10 !" is printed

}
```

We are using the `block_on` executor on the main function because the await keyword
can only be used inside async functions. `await_for_ten_or_return_zero` needs,
like the name says, to execute return_ten first to compare it. So we use await
to get the result from the asynchronous call.
We can use the `block_on` executor wherever we want.

There are other executors that let us do more stuff than blocking the current
thread until the `futures` completes. For example `futures::join` will get multiple
futures, run them concurrently and return a tuple with the results in order:

```rust
use futures::join;


async fn a_and_b() {        // join! just works inside async functions

    let a = async { 'a' };

    let b = async { 'b' };

    join!(a, b); // ('a', 'b')

}
```

One thing to mentoin is that in Rust the async-await implementation it's lazy, 
that means that the code will not run on the background and wait for the await, 
but rather run at the moment the await call asks for it. This design follows the 
Rust philoshopy having a better performance and error handeling.

#### So that's it ?

Well, that can be it with the basics but you have a lot more options to work 
asynchronously in Rust. You have multiple frameworks/runtimes, none of them are 
standard:


1. [ Tokio ](https://github.com/tokio-rs/tokio): A popular async ecosystem with HTTP, gRPC, and tracing frameworks.
2. [ async-std ](https://github.com/async-rs/async-std): A crate that provides asynchronous counterparts to standard library components.
3. [ smol ](https://github.com/smol-rs/smol): A small, simplified async runtime. Provides the Async trait that can be used to wrap structs like UnixStream or TcpListener.

Each one of them have different approaches, I will choose tokio because it's the
more one that looks more popular and standard.

### Crawler boilerplatte

To create a basic crawler we need the following:

- Make HTTP calls
- Parse HTML code
- Async calls system

We will be using [ reqwest ](https://github.com/seanmonstar/reqwest) for the HTTP calls,
[ html5ever ](https://github.com/servo/html5ever) for the parsing and [ tokio ](https://github.com/tokio-rs/tokio)
for the async calls.

```toml
[dependencies]

reqwest = { version = "0.11" }

html5ever = "*"

tokio = { version = "1", features = ["full"] }
```


