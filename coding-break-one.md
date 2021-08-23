# Coding Break One

We can realise our terms and types as algebraic data-types.
Taking our Razor with Boolean conjunction, Natural number addition and let-bindings, we can represent it as:

```idris

data Ty = NAT | BOOL

data Term = N Nat
          | B Bool
          | And Term Term
          | Add Term Term
          | Var String
          | Let String Term Term
```

and then write a type-checker with the type:

```idris
check : (context : List (String, Ty))
     -> (term    : Term)
                -> Either Error Ty
```

where `Error` is a data-type to capture errors such as type-mismatch and variable not found.

Let's see how it works:

First, if we encounter a raw value we can directly assert its type:

```idris
check _ (N _) = Just NAT
check _ (B _) = Just BOOL
```

Here we can tell from the term itself what its type will be.
For the remaining terms we will have to ask (using an application of `check`) what a term's type will be.
Take `And` and `Add`, for each operand we ask what its type is and then check to see if it is the expected type.
Checking fails otherwise.
If both types are correct we can then assert the type of each operation.

```idris
check ctxt (And x y)
  = do BOOL <- check ctxt x | type => Left (Mismatch type BOOL)
       BOOL <- check ctxt y | type => Left (Mismatch type BOOL)
       pure BOOL

check ctxt (Add x y)
  = do NAT <- check ctxt x | type => Left (Mismatch type BOOL)
       NAT <- check ctxt y | type => Left (Mismatch type BOOL)
       pure NAT
```

Similarly, checking variables requires us to ask.
Instead of asking the term, we ask the context!

```idris
check ctxt (Var x)
  = case lookup x ctxt of
      Nothing   => Left (UnknownVar x)
      Just type => Right type
```

Once known we can return the result of the asking.

The last term, `Let`, operates by asking what the type of `this` is.
It then asks, and returns, the type of the body but also extending the typing context.

```idris
check ctxt (Let x this body)
  = do typeX <- check ctxt this
       check ((x,typeX) :: ctxt) body
```
