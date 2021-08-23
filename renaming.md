# ReNaming things is Harder

Consider our running exemplar:

    t : TYPE ::= BOOL | NAT
    n : NAT  ::= 0,1,2,...    | (add e e)
    b : BOOL ::= True | False | (and e e)
    e : t    ::= v | let v be e in e | n | b

its realisation in Idris:

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

and the following example term:

```idris
example : Term Nil NAT
example =  Let (And (B True) (B False))
          (Let (Add (N 1)    (N 1))
               (Var (Here)))
```

When we execute `example`, we need to *substitute* all variables with their definitions.

We can formally describe substitution as a series of rules that allow us to interate over our expressions/statements and swap variables for terms.
Note how they form a recursive call.
We will use these rules to help us describe how we can transform our program instance during its evaluation.

For De Bruijn indexed terms this means we need to rename our variables as we reduce the context size.

## Renaming

```idris
namespace Renaming
```

```idris
  public export
  weaken : (func : Contains old type
                -> Contains new type)
        -> (Contains (old += type') type
         -> Contains (new += type') type)

  weaken func Here = Here
  weaken func (There rest) = There (func rest)
```

```idris
  public export
  interface Rename (type : Type) (term : List type -> type -> Type) | term where
    rename : {old, new : List type}
          -> (f : {ty : type} -> Contains old ty
                              -> Contains new ty)
          -> ({ty : type} -> term old ty
                          -> term new ty)

    var : {ty   : type}
       -> {ctxt : List type}
               -> Elem ty ctxt
               -> term ctxt ty


    weakens : {old, new : List type}
           -> (f : {ty  : type}
                       -> Contains old ty
                       -> term     new ty)
           -> ({ty,type' : type}
                  -> Contains (old += type') ty
                  -> term     (new += type') ty)
    weakens f Here = var Here
    weakens f (There rest) = rename There (f rest)
```

## Substitution

```idris
namespace Substitution
```

Substitution requires that we replace all occurrences of a variable `x` with its bound term `t` in some term `b`
There are several notations for substitution in PLT such as:

    b[x/t]

and

    b[x |-> t]

For my notes I prefer a more textual notation:

    replace x with t in b

### General

```idris
  namespace General
    public export
    interface Rename type term
           => Substitute (type : Type) (term : List type -> type -> Type) | term where

      subst : {old, new : List type}
           -> (f : {ty  : type}
                       -> Contains old ty
                       -> term     new ty)
           -> ({ty : type}
                  -> term old ty
                  -> term new ty)

```

### Single
```idris
  namespace Single
```

```idris
    public export
    apply : {type : Type}
         -> {term : List type -> type -> Type}
         -> Rename type term
         => {ctxt   : List type}
         -> {typeA  : type}
         -> {typeB  : type}
         -> (this   : term      ctxt    typeB)
         -> (idx    : Contains (ctxt += typeB) typeA)
                   -> term      ctxt           typeA
    apply this Here = this
    apply this (There rest) = var rest

    public export
    subst : {type : Type}
         -> {term : List type -> type -> Type}
         -> Rename type term
         => Substitute type term
         => {ctxt          : List type}
         -> {typeA         : type}
         -> {typeB         : type}
         -> (this          : term  ctxt           typeB)
         -> (inThis        : term (ctxt += typeB) typeA)
                          -> term  ctxt           typeA
    subst {ctxt} {typeA} {typeB} this inThis
      = subst (apply this) inThis
```
