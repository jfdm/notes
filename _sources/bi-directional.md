# Bi-Directional Typing

Recall the type-checker from Coding Break 1.
It has the signature:

```idris
check : (context : List (String, Ty))
     -> (term    : Term)
                -> Either Error Ty
```

where `Error` is a data-type to capture errors such as type-mismatch and variable not found.

The problem is that the function `check` does not look and feel like what a type-checker should do!
It doesn't check if `term` has a type, rather it constructs the type for `term`, if it can do so.

Here `check` is not a _checking_ function it's a type forming one!

If we look at how types flow in our implementation of `check` from Coding Break 1: you will notice to types of flows:

1. atomic terms present their types when asked; and
2. other terms will do a mixture of:
  1. asking what a sub term's type is; and
  2. asserting what the expected type of a sub-term is.

If you have programmed in Haskell, you will have seen this notion of asking and then asserting from Haskell's ability to perform type inference.
In fact the idea of asking and asserting types from terms has been formalised wonderfully as Bi-Directional Type Theory, and captures nicely the communication between types and terms during the type-checking process.

## Bi-Directional type theory

In bi-directional type-checking types for terms are either:

+ **Synthesised**---constructed from the terms; or
+ **Checked**---checked against the given term.

Below we give the bi-directional typing rules for our language.
We differ from standard notation and use `checks` and `synths` to denote what happens.

First, primitive values:

    ================= [ Nat ]  ================== [ Bool ]
    g |- n checks NAT          g |- b checks BOOL

then `And` and `Add`:

    g |- x checks BOOL                  g |- x checks NAT
    g |- y checks BOOL                  g |- y checks NAT
    ========================== [ And ]  ========================== [ Add ]
    g |- (And x y) checks BOOL          g |- (Add x y) checks BOOL

then Variables:

    (x : t) in g
    =============== [ Var ]
    g |- x synths t

and lastly let-expressions:

            g        |- e synths s
    (Extend g (x:s)) |- b synths t
    =============================== [ Let ]
    g |- (let x be e in b) synths t
