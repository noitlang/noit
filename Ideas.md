# Noit Lang

The functional language that will eat your bugs! :butterfly:

Este documento es para que definamos qué cosas nos gustaría que tenga el lenguaje, cómo sería la syntax, etc.

## Guiding Principles

0. Happy birthday Noit!
1. Run once, run forever.
2. Functional to a reasonable level. No complicated formalisms to deal with the real world, but no mutation in regular code either.
3. Compiles to native (C is a first class target).
4. Concurrency is baked in, Erlang-like light processes.
5. Message passing is synchronization.
6. Static, strong typing.
7. No pointers. No references. Only values.
8. Sum Types
9. Errors are values. Sum types are how you distinguish them.
10. No implicit type conversions.
11. Select patterns get syntax sugar: e.g. `?` operator when the returned error type is the same for the caller and the callee.
12. Easy to read is more important than easy to parse.
13. Linked lists are fun.
14. Hashing a list is hashing the pointer to head.
15. FFI is core.
16. No number crunching. That's what FFI libs are for.
17. All numbers are big numbers. Overflow is for nerds.
18. UTF-8 is text. ASCII is for neckbeards. Custom encodings are for hipsters. Emojis are normal glyphs.
19. A FIFO is a mailbox.
20. IPC is a mailbox.
21. TCP is a mailbox.
22. Your mom is a mailbox.
23. Mailboxes handle priorities.
24. The default mailbox interface doesn't require you to specify the message priority.
25. All values are immutable.
26. Memoisation is optional and easy to use.
27. Subprocess messages might have priorities.
28. Pattern Matching for Sum Types
29. Gleam-like expression blocks `{...}`
30. Compiler enforced syntax and styling.
31. Imports are always called with the package first and the function to import next as so to allow for IDE suggestions.
32. You can declare constants that are actual constants (unlike Python).
33. The basic string unit is not the ASCII char but the grapheme.
34. Strings are UTF-8 encoded.
35. Stability and concurrency are priorities.
36. Syntax highlighting is a priority.
37. A working LSP is also a priority.
38. :red_circle::eggplant:
39. :red_circle::butterfly:

### Varias sintaxis malas

#### LDPL-like

```
DEFINE FUNCTION sum WITH PARAMETERS (INTEGER a, INTEGER b) USING MEMOIZATION DOING
   DEFINE INTEGER VARIABLE result
   IN result SOLVE EXPRESSION a + b
   RETURN VALUE result
END FUNCTION
```
Muy verbose.

#### TADs

```
type list(a) {
    generators {
        <>     :                   -> list(a)
        ++     : a x list(a)       -> list(a)
    }
    observers {
        empty? : list(a)           -> bool
        head   : list(a) s         -> a         {not empty?(s)}
        tail   : list(a) s         -> list(a)   {not empty?(s)}
    }
    operations {
        • & •  : list(a) x list(a) -> list(a)
    }
    axioms {
        empty?(<>)   => true
        empty?(a++b) => false
        head(a++b)   => a
        tail(a++b)   => b
        a & b        => if empty?(a) then b else head(a) ++ (tail(a) & b)
    }
}
```

Cero welcoming, seguro es lento y muy orientado al pattern matching (poco práctico).



## Sintaxis diseñadas mejor

Idea Lartiana 1
```fsharp=
# Comment
(* Multiline comment
with multiple lines *)

memoized fun power(a: int, b: int) -> int
{
    if b > 0
    {
        a * power(a, b - 1)
    }
    else
    {
        1
    }
}

NamedTuple Vector3
(
    x: float,
    y: float,
    z: float,  (* Notar que puedo tener trailing , *)
)

casting vector: Vector3 -> string
{
    $"({vector.x}, {vector.y}, {vector.z})"
    (* Se asume que en $ strings el casteo a print es forzado
    (aunque medio implícito), pero es el usecase y que float
    tiene definido su casteo a string*)
}

(* 
--- Vector3 multiplication ---
Por más que * ya esté definida para otros tipos, acá agrego una
definición para Vecto3 x float. La aridad a usar se define por
orden paramétrico y si no hay función matching tira error.
*)
memoized infix fun * (vector: Vector3, scalar: float) -> Vector3
{
    Vector3
    (
        x = vector.x * scalar,
        y = vector.y * scalar,
        z = vector.z * scalar,  (* Notar que puedo tener trailing , *)
    )
}

print(Vector3(0f, 10.1f, 89f) * 5f as string)

(* snake_case for variables, CAPS for constants and camelCase for
function names and PascalCase for types is enforced by the compiler. *)

NamedTuple Range
(
    min: int,
    max: int
)

fun RandomNumberWorker (mailbox_address: string) -> nothing
{
    let ranRange: Range = receive() (* ¿Qué tipo tiene esto? *)
    let random_number: int = randint(ranRange.min, ranRamge.max)
    send(mailbox_address, random_number)
}

fun ParallelFibonacci (nth_number: int) -> int
{
    summon RandomNumberWorker() as "worker1"
    summon RandomNumberWorker() as "worker2"
    summon RandomNumberWorker() as "worker3"
    receive() + receive() + receive()
}

Enum ResultValues
(
    OK,
    ERR
)

NamedTuple Result<a>
(
    type: ResultValue,
    value: a,
    error: string
)

postfix fun ? (result: Result(a)) -> bool
{
    result.type is ResultValues.OK
}

(* Worker that prints the ellapsed time without a message *)
fun EllapsedTimeCounter (ellapsed_time: float) -> float
{
    let message: Result<string> = check_inbox() (* Non blocking receive *)
    if message?
    {
        print($"Time since last message: {ellapsed_time}")
        if message.value == "exit"
        {
            ellapsed_time
        }
        else
        {
            EllapsedTimeCounter(0)
        }
    }
    else
    {
        EllapsedTimeCounter(ellapsed_time + 1f) (* Ejemplo *)
    }
}

from noit.time import wait

summon EllapsedTimeCounter(0) as "timecounter"
wait(10f) (* Wait 10 seconds *)
send("timecounter", "exit")

```

La sintaxis lartiana 2 fue lo mismo pero en vez de {} usaba : como en Python y quedó horrible.

Otro ejemplo que quiero ver es un echoserver (inventandome cositas) con la sintaxis lartiana 1:
```fsharp=
from noit.tcpSocket import create_server, Socket, SocketServer


fun on_server_connect (client_socket: Socket) -> nothing
{
    (* Imagino que on_connect spawnea un worker con esta función *)
    let client_message: Result<string> = client_socket.read_until("\n")
    (* Eso es como un TCP read pero bufferea hasta enconrar el string
    y ahí hace el punto de corte y te pasa el string con eso incluído *)
    if client_message?
    {
        client_socket.send($"You sent me '{client_message.value}'")
        self(client_socket) (* Llamado reflexivo [1] *)
    }
    else
    {
        (* La constante NAME se define en cada worker *)
        print($"Error on worker {NAME}: {client_message.error}")
    }
}

let server: SocketServer = create_server("localhost", 6648, on_server_connect)
(* 6648 = NOIT *)
server.listen()

```
[1] Me gusta mucho más `self` o algo del estilo que lo que suele usarse que es `rec` (de _recurse_).

El tipo `SocketServer` se puede definir por ejemplo como:
```fsharp=

fun socket_pass(socket: Socket) -> nothing
{
    (* eat moth & rest *)
}

NamedTuple SocketServer
(
    ip: int = "0.0.0.0", (* Estos son default values si algun valor no se especifica al crear el tipo *)
    port: int = 1234,
    on_connect: function[Socket -> nothing] = socket_pass, (* No me decido para la sintaxis de templating *)
    on_disconnect: function[Socket -> nothing] = socket_pass 
)
```

Fizzbuzz!!

```fsharp=
fun fizzbuzz(n: int, max: int) -> nothing
{
    if n % 3 == 0 and n % 5 == 0
    {
        print("Fizzbuzz!")
    }
    else if n % 3 == 0
    {
        print("Fizz!")
    }
    else if n % 5 == 0
    {
        print("Buzz!")
    }
    else
    {
        print(n as string)
    }
    if n < max
    {
        fizzbuzz(n + 1, max)
    }
}

fizzbuzz(0, 500)
```

Disan Count!!

```fsharp=
fun disancount(n: int, max: int) -> nothing
{
    if n % 2 == 0
    {
        print($"{n} is even!")
    }
    if n < max
    {
        disancount(n + 1, max)
    }
}

disancount(0, 500)
```

Factorial

```fsharp=
fun factorial(n: int) -> int
{
    if n <= 1
    {
        1
    }
    else
    {
        n * self(n - 1)
    }
}

let x: int = 50

print($"{x}! = {factorial(x)}")
```

Fibonacci

```fsharp=
memoized fun fibonacci(n: int) -> int
{
    if n <= 2
    {
        1
    }
    else
    {
        fibonacci(n - 1) + fibonacci(n - 2)
    }
}

let x: int = 50

print($"Fibonacci number #{x} = {fibonacci(x)}")
```
