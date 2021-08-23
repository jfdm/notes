# Coding Break Two

Recall our running example as coded in Idris:

```idris

data Ty = NAT | BOOL

data Term = N Nat
          | B Bool
          | And Term Term
          | Add Term Term
          | Var String
          | Let String Term Term
```

This representation requires an extrinsic type-checker, and is stringly named.

## Make it Nameless...

First we look at making the representation 'nameless'.
To do so we index the type of `Term` with a naming context that allows us to keep track of the names used.


```idris
data Term : List String -> Type where
```

Values exist in some naming context:

```idris
  N : Nat  -> Term ctxt
  B : Bool -> Term ctxt
```

So do their operations:

```idris
  And : (l,r : Term ctxt) -> Term ctxt
  Add : (l,r : Term ctxt) -> Term ctxt
```

Variables now refer to where in the naming context they where defined:

```idris
  Var : Elem name ctxt -> Term ctxt
```

This will be the most recently defined binding for that name!
Thus locally named bindings shadow their global counterparts but we can still refer to each based on their binding position.
How cool is that!

Finally, Let bindings introduce names to the naming environment.

```idris
  Let : (name : String)
     -> (this : Term          ctxt)
     -> (body : Term (name :: ctxt))
             -> Term          ctxt
```

Here we made the name introduction explicit.
We can remove this, if we are generating instances of Term from external sources.

We can then write examples as:

```idris
example : Term Nil
example =  Let "x" (And (B True) (B False))
          (Let "x" (Add (N 1)    (N 1))
                   (Var (Here))) -- points to the x with type Nat
```

## Making It Intrinsically Typed

A better representation is to also embed within the type of `Term` the typing rules.

First we define our types:

```idris
data Ty = NAT | BOOL
```

then terms:

```idris
data Term : List (String, Ty) -> Ty -> Type where
```

Again, values exist in some naming context:

```idris
  N : Nat  -> Term ctxt NAT
  B : Bool -> Term ctxt BOOL
```

and so do their operations:

```idris
  And : (l,r : Term ctxt BOOL) -> Term ctxt BOOL
  Add : (l,r : Term ctxt NAT)  -> Term ctxt NAT
```

We could even make the definition of `And` and `Add` polymorphic, but that may cause issues later on when working with these terms.

Variables now refer to where in the naming context they where defined:

```idris
  Var : Elem (name,type) ctxt -> Term ctxt type
```

This will be the most recently defined binding for that name!
Thus locally named bindings over shadow their global counterparts.
How cool is that!

Finally, Let bindings introduce names to the naming environment.

```idris
  Let : (name : String)
     -> (this : Term                 ctxt  type)
     -> (body : Term ((name,type) :: ctxt) b)
             -> Term                 ctxt  b
```


## Common

Our latest definition is a good one, but when you do not need to provide better error messages we can remove the names from the definition:

```idris
data Term : List Ty -> Ty -> Type where

  N : Nat  -> Term ctxt NAT
  B : Bool -> Term ctxt BOOL

  And : (l,r : Term ctxt BOOL) -> Term ctxt BOOL
  Add : (l,r : Term ctxt NAT)  -> Term ctxt NAT

  Var : Elem type ctxt -> Term ctxt type

  Let : (this : Term           ctxt  type)
     -> (body : Term (type :: ctxt) b)
             -> Term          ctxt  b
```
