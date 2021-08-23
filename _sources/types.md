# Types & Type-Checking

Types are a way to group related bits of syntax together.
Within a programming language, expressions, operations, functions, and control structures are all typed.
When we check a program can differ:

+ **static types**
  type check programs at compile time.
  You will have seen static typing if you have programmed in Java, Scala, C, C++, Swift, OCaml, or Haskell.

+ **dynamic types**
  type check programs at runtime, duck typing (a la python) is a for of dynamic typing.
  You will have seen dynamic typing if you have programmed in Python, JavaScript, and PHP.

+ **gradual typing** allows you to do both
  You will have seen gradual typing if you have programmed in TypeScript, Python 3, and Hack.

## Types & Syntax

We use types to ensure that our programs are well-structured.
For example, consider this slight variant of Hutton's Razor that supports boolean conjunction as well as addition of natural numbers:

    b ::= True | False
    n ::= 0,1,2,...
    e ::= b | n | (and e e) | (add e e)

In this language terms can be booleans or numbers.
Without types one can construct ill-typed terms, terms that do not make sense:

    (and 0 True)
    (add True 0)

We can use our abstract syntax as a concise notation to describe which types belong to which terms, and use typing rules to describe how the types are related to each other.
Ensuring that expressions are well formed.
For instance, in our exemplar grouping the boolean terms and numerical terms together.

For example, we can define some types for our exemplar and type them as:

    t : TYPE ::= BOOL | NAT
    n : NAT  ::= 0,1,2,...    | (add e e)
    b : BOOL ::= True | False | (and e e)
    e : t    ::= n | b

Here `TYPE` represents the type-of-types.
We have in-lined the typing rules for brevity, but our definition now describes well-typed terms.
These rules dictate what it means for an expression/statement to be well-formed.


Verbose definitions follow.

    ======= [ Nat ]  ======== [ BOOL ]
    n : NAT          b : BOOL

    a : BOOL           a : Nat
    b : BOOL           b : Nat
    ========= [ And ]  ========= [ Add ]
    (and a b)          (add a b)

We read typing rules as follows:

> Things above the lines are premises such that if all premises are true then the judgement (below the line) will also be true.

When given any expression/statement in our language we can use the typing rules to construct a derivation that provides proof that the expression/statement is well-typed, that is we can apply each rule and form a derivation tree.
If we cannot construct this tree then the expression is ill-typed and syntactically not valid.
Typing rules are a compile-time static check.
We can only proceed to computation/evaluation of our language iff it is well-typed.

## Typing Contexts

Expanding our example further, we can include let-bound variables:

    t : TYPE ::= BOOL | NAT
    n : NAT  ::= 0,1,2,...    | (add e e)
    b : BOOL ::= True | False | (and e e)
    e : t    ::= v | let v be e in e | n | b

But how do we type our let-expressions?

To reason about the type of variables we need a typing context to keep track of names given to variables and their type.
We first define a context `g` as:

    g : G ::= Empty | (Extend g (v:t))

Contexts are either empty of van be extended to include a name-type pair.
With these constructs we can describe type-checking more precisely as:

    g |- e : t

which in simple terms says:

> Given a typing context `g`: Does the term `e` have type `t`?

An empty context means there are no bound variables.
The typing rules for variables and let-expressions are thus:

    (v:t) in g
    ========== [ Var ]
    g |- v : t

If the variable `v` with type `t` is in the given context `g` then `v` does have type `t`.

            g        |- e : t
    (Extend g (v:t)) |- b : s
    ======================================= [ Let ]
            g        |- let v be e in b : s

In the context of `g` there is a term `e` of type `t`.
Using let we name `e` as `v` in the body `b` ensuring that the context is extended to let `b` know about `e` as `v`.

## Properties

Our typing-system, (the types, syntax, and judgements) is type-safe iff types are preserved during reduction, and that a term will reduce, eventually to a value (a irreducible term).


+ **Preservation**
  The property that types are preserved during reduction.

+ **Progress**
  Reduction of terms will either become stuck or produce a value.

Well-typed programs do not get stuck.
What these properties are, and their proof will come later.
