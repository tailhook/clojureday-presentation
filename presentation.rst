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

    fn parse(s: Stream)
        -> Result<Ast, ParseError>
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
    return Ok(Ast { ... })
    return Err(ParseError::new())

:tag:`enums`
:tag:`generics`

----

.. code-block:: rust

    enum ParseError {
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
    6  | / enum ParseError {
    7  | |     Io(io::Error),
    8  | |     UnexpectedCharacter,
    9  | |     IntError { cause: ParseIntError,
       | |     -------- not covered
    10 | |                position: Pos },
    11 | | }
       | |_- `ParseError` defined here
    ...
    14 |       match err {
       |             ^^^ pattern `IntError { .. }` not covered
       |
       = help: ensure that all possible cases are being handled,
               possibly by adding wildcards or more match arms

:tag:`rustc-error`

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
:tag:`unused-fields`

----

.. code-block:: rust

    let (new_state, action) = match (msg, state) {
        (Ping, Leader { .. })       =>  // re-elect
        (Ping, _)                   =>  // follow
        (Pong, me @ Leader { .. })  =>  // count
        (Pong, _)                   =>  // re-elect
        (Vote(id), Starting { .. }) =>  // vote
        (Vote(id),
         Electing {epoch, mut votes_for_me,
                   deadline, needed_votes}) =>  // check
        (Vote(_), me @ Voted { .. })
        | (Vote(_), me @ Leader { .. })
        | (Vote(_), me @ Follower { .. }) =>  // discard
    };

----

.. code-block:: rust

    let Config {
        host: a_host, port: a_port,
        listen_queue: _,
    } = a;
    let Config {
        host: b_host, port: b_port, ..
    } = b;
    return a_host == b_host && a_port == b_port;


:tag:`unpacking`
:tag:`exhaustiveness`
:tag:`pattern-matching`

----

.. code-block:: rust

    #[non_exhaustive]
    enum ParseError {
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

    enum ParseError {
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

    #[derive(Clone)]
    pub struct Percentile(u16);
    impl TryFrom for Percentile {...}
    impl Into for Percentile {...}
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

    pub struct Token { level: u8, slot: u32 }
    fn create_timeout() -> Token { }
    fn clear_timeout(tok: Token) { }

:tag:`new-type`
:tag:`token-pattern`
:tag:`single-use`
:tag:`linear-types`

----

.. code-block:: rust

    let (tx, rx) = channel::oneshot();
    thread::spawn(move || {
        rx.send(1);
    });

:tag:`single-use`
:tag:`linear-types`
:tag:`move`
:tag:`threading`

----

.. class:: small
.. code-block:: rust

    error[E0382]: use of moved value: `tx`
      --> src/main.rs:23:9
       |
    22 |         tx.send(1);
       |         -- value moved here
    23 |         tx.send(2);
       |         ^^ value used here after move
       |
       = note: move occurs because `tx` has type `Sender`,
               which does not implement the `Copy` trait

:tag:`linear-types`
:tag:`move`
:tag:`rustc-error`

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

    error[E0502]: cannot borrow `m` as mutable because \
                  it is also borrowed as immutable
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
    thread1_channel.send(shared.clone());
    thread2_channel.send(shared.clone());

:tag:`mutability`
:tag:`shared-state`

----

.. code-block:: rust

    let shared = Arc::new(Mutex::new(x));
    thread::spawn(move || {
        let x: MutexGuard<HashMap<_, _>>;
        x = shared.lock()
        x.insert("x");
    })

:tag:`mutability`
:tag:`shared-mutable-state`
:tag:`raii`
:tag:`lifetime`

----

.. code-block:: rust

    let gil = Python::acquire_gil();
    let py = gil.python();
    let dic1 = PyDict::new(py);
    dic1.set_item(py,
        "key1", PyInt::new(py, 1000))?;

:tag:`python`
:tag:`token-pattern`
:tag:`lifetime`

----

.. code-block:: rust

    let a = HashMap::new();
    a.insert("x", 1);
    let b = HashMap::new();
    b.insert("key1", a);  # a is moved
    # a.insert()  -- is error
    b.get_mut("key1").insert("y", 2);

:tag:`recursive-mutability`

----

.. code-block:: rust

    let a = TcpStream::connect("localhost:1234");
    a.write(b"test");
    a.read(&mut buf);

:tag:`sharing`
:tag:`sockets`

----

.. code-block:: rust

    let mut a = BufStream::new(
        TcpStream::connect("localhost:1234"));
    a.write(text);
    a.write(b"\n");
    a.read_line(&mut buf);

:tag:`sharing`
:tag:`buffering`
:tag:`mutation`

----

.. code-block:: rust

    let a = BufStream::new(
        TcpSocket::connect("localhost:1234"));
    let (mut tx, mut rx) = a.split();
    thread::spawn(move || {
        # rx.write()  -- no such method
        rx.read_line(&mut buf);
    })
    tx.write(text);
    tx.write(b"\n");

:tag:`sharing`
:tag:`buffering`
:tag:`mutation`

----

Inherent mutability
===================

* tuple vs list
* volatile-mutable
* hashable?
* performance

----

Rust-style mutability
=====================

* user decides
* initial prefill is fast
* ``map1[map2] = val`` is fine

----

Shared Mutable State
====================

* Rust fixes shared
* Clojure fixes mutable

----

Conclusion
==========

* Perfect API ← Unbreakable invariants
* Be creative!
