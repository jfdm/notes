# Naming things is Hard

Consider our running exemplar:

    t : TYPE ::= BOOL | NAT
    n : NAT  ::= 0,1,2,...    | (add e e)
    b : BOOL ::= True | False | (and e e)
    e : t    ::= v | let v be e in e | n | b

its realisation in Idris:

```idris
data Term = N Nat
          | B Bool
          | And Term Term
          | Add Term Term
          | Var String
          | Let String Term Term
```

and the following example term:

```idris
example : Term
example =  Let "x" (And (B True) (B False))
          (Let "x" (Add (N 1)    (N 1))
                   (Var "x"))
```

What is the value of `x`?

This question is important to know when understanding how to execute programs.
We need to be able to *substitute* bound variables with their values.
Technically, `example` is well-typed **and** well-scoped:

1. It is well-typed in that each expressions will type-check.
2. It is well-scope in that the variable does have a declaration.

Here the second declaration of `x` shadows the first one.
When executing programs, however, we need to distinguish between shadowed variables.

One way to do this is through: alpha-renaming or alpha conversion in which we must rename all variables so that they are unique.

Another way to do this is to use a nameless approach that 'names' variables based on their binding position.

For example our language now becomes:

```idris
data Term = N Nat
          | B Bool
          | And Term Term
          | Add Term Term
          | Var Nat
          | Let Nat Term Term
```

and the example:

```idris
example : Term
example =  Let 0 (And (B True) (B False))
          (Let 1 (Add (N 1)    (N 1))
                   (Var 1))
```

With this realisation, however, we still need to ensure that variables are named appropriately.
There is a better way because, as it turns out, De Bruijn indexing is a good option when representing languages in Idris as it also helps us embed the typing rules directly in our language's definition.
