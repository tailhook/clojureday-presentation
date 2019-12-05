:title: Typing is not just OOP
:css: my.css
:css: highlight.css

.. role:: tag
   :class: tag


Typing is not just OOP
======================

----

Types:
======

* Correctness
* Semver

-----

Types:
======

* One-time use
* (Im)mutability
* State Machines
* Efficiency
* Errors

----

.. code-block:: rust

    fn next_token(s: Stream)
        -> Result<Token, TokenError>
    {
        unimplemented!();
    }

:tag:`result`
:tag:`errors`

----

.. code-block:: rust

    pub enum Result<T, E> {
        Ok(T),
        Err(E),
    }
    return Ok(Token { ... })
    return Err(TokenError::new())

:tag:`enums`
:tag:`generics`

----

.. code-block:: rust

    enum TokenError {
        Io(io::Error),
        UnexpectedCharacter,
        IntError { cause: ParseIntError,
                   position: Pos },
    }

:tag:`enums`
:tag:`errors`
:tag:`error-hierarchy`

----

.. code-block:: rust

    match err {
        Io(io) => connection.close(),
        UnexpectedCharacter
        => println!("bad char"),
    }

:tag:`match`
:tag:`non-exhaustive`

----

.. class:: small

.. code-block:: rust

    error[E0004]: non-exhaustive patterns: `IntError { .. }` not covered
      --> src/main.rs:14:11
       |
    6  | / enum TokenError {
    7  | |     Io(io::Error),
    8  | |     UnexpectedCharacter,
    9  | |     IntError { cause: ParseIntError,
       | |     -------- not covered
    10 | |                position: Pos },
    11 | | }
       | |_- `TokenError` defined here
    ...
    14 |       match err {
       |             ^^^ pattern `IntError { .. }` not covered
       |
       = help: ensure that all possible cases are being handled,
               possibly by adding wildcards or more match arms

:tag:`rustc-errors`

----

.. code-block:: rust

    match err {
        Io(io) => connection.close(),
        UnexpectedCharacter
        => println!("bad char"),
        IntError { cause, .. }
        => println!("bad int: {}", cause),
    }

:tag:`match`
:tag:`structural-match`

----

.. code-block:: rust

    #[non_exhaustive]
    enum TokenError {
        Io(io::Error),
        UnexpectedCharacter,
        IntError { cause: ParseIntError,
                   position: Pos },
    }

:tag:`forward-compatibility`
:tag:`semver`

----

.. code-block:: rust

    match err {
        Io(io) => connection.close(),
        _ => println!("error: {}", err),
    }

:tag:`non-exhaustive`
:tag:`catch-all`

----

.. code-block:: rust

    enum ParserError {
        Io(io::Error),
        Lexer(Box<dyn Error>),
        Parser(Box<dyn Error>),
    }

:tag:`errors`
:tag:`dynamic`
:tag:`exhaustive`
:tag:`boxing`
:tag:`efficiency`

----

.. code-block:: rust

    pub struct ParserError {
        internal: InternalError,
    }
    enum InternalError {
        Io(...),
        Lexer(...),
    }

:tag:`visibility`
:tag:`forward-compatibility`
:tag:`semver`

----

.. code-block:: rust

    pub struct ParserError(InternalError);
    pub(crate) enum InternalError {
        Io(...),
        Lexer(...),
    }


:tag:`new-type`
:tag:`visibility`

----

.. code-block:: rust

    pub struct UpstreamName(Arc<str>);
    pub struct DownstreamName(Arc<str>);
    struct Server {
        up: Map<UpstreamName, Upstream>,
        down: Map<DownstreamName, Downstream>,
    }

:tag:`new-type`
:tag:`arc`
:tag:`interning`
:tag:`validation`

----

.. code-block:: rust

    pub struct Percentile(u16);
    impl TryFrom {...}
    impl Into {...}
    fn get(perc: Percentile) -> Value {
        return self.percentiles[perc.0];
    }

:tag:`new-type`
:tag:`validation`
:tag:`performance`
:tag:`ints`
:tag:`conversion`

----

.. code-block:: rust

    for (&key, _) in &map {
        if key.start_with("x") {
            map.remove(&key);
        }
    }

:tag:`borrow-checker`
:tag:`ownership`

----

.. class:: small
.. code-block:: rust

    error[E0502]: cannot borrow `m` as mutable because it is also borrowed as immutable
     --> src/main.rs:7:13
      |
    5 |     for (&key, _) in &map {
      |                       ---
      |                       |
      |                       immutable borrow occurs here
      |                       immutable borrow later used here
    6 |         if key.starts_with("x") {
    7 |             map.remove(&key);
      |             ^^^^^^^^^^^^^^^^ mutable borrow occurs here

:tag:`borrow-checker`
:tag:`rustc-error`

----

.. code-block:: rust

    for (&key, _) in &map {
        if key.start_with("x") {
            map.remove(&key);
            break;
        }
    }

:tag:`borrow-checker`
:tag:`ownership`
:tag:`nll`

----

.. code-block:: rust

    let x = HashMap::new();
    x.insert("key1", v1);
    x.insert("key2", v2);
    let shared = Arc::new(x);
    thread1_channel.send(x.clone());
    thread2_channel.send(x);
