# Noit

<img src="Graphical Assets/textlogo2.png" height=200px>

<img src="https://img.shields.io/badge/Version-0.0-lightgray.svg"> <img src="https://img.shields.io/badge/License-_Apache_2.0-green">

**Noit** is a functional programming language with a concurrency model based on light processes and message passing. It features static typing, explicit casting, and immutability. Noit compiles to C, so it's fast and portable. Noit is designed to be easy enough to learn that you can learn it in an afternoon.

## About the language

We are still designing the language to look and feel the way we want it to. If you want to read what we've been planning, [take a look here](Ideas.md).

Noit may end up looking like this:

```haskell
fun hello_world () -> nothing
{
    print("Hello world!")
}

memoized fun factorial (n: int) -> int
{
    1 if n <= 1 else n * factorial(n - 1)
}

hello_world()
let x = 50
print($"{x}! is {factorial(x})")
```


## Learning Noit

<img src="Graphical Assets/Noit Coding.png" height=150px>

To-do, as there's no Noit to learn yet!

## Contributing to Noit

Once we finish designing the language, a note about contributing will be added here. That said, if you like what we are making here, feel free to drop an issue or something to get in touch with us!

## Where can I get more help, if I need it?

Oh, don't we all wonder the same?

## License

Noit is released under the Apache 2.0 License. All Noit graphics and the Noit mascot are released under a Creative Commons Attribution 4.0 International (CC BY 4.0) license. Copyright (c) 2019 MOX.
