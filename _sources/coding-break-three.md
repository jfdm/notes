<!-- idris
import Data.List.Elem
import Data.Fuel

%default total

infixr 6 +=

(+=) : List a -> a -> List a
(+=) xs x = x :: xs

Contains : List a -> a -> Type
Contains xs x = Elem x xs

namespace Renaming
  public export
  weaken : (func : Contains old type
                -> Contains new type)
        -> (Contains (old += type') type
         -> Contains (new += type') type)

  weaken func Here = Here
  weaken func (There rest) = There (func rest)

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

namespace Substitution

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

  namespace Single
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

-->

# Coding Break Three

Recall our running example as coded in Idris:

```idris
data Ty = BOOL | NAT

data Term : List Ty -> Ty -> Type where
  Var : Elem type ctxt
     -> Term ctxt type

  B : Bool -> Term ctxt BOOL
  N : Nat  -> Term ctxt NAT

  Add : (l,r : Term ctxt NAT)
            -> Term ctxt NAT

  And : (l,r : Term ctxt BOOL)
            -> Term ctxt BOOL

  Let : {typeT,typeB : Ty}
     -> (this : Term ctxt typeT)
     -> (body : Term (typeT :: ctxt) typeB)
             -> Term ctxt typeB
```

which is renamed as:

```idris
Rename Ty Term where
  var = Var

  rename f (Var x)
    = Var (f x)
  rename f (B x)
    = B x
  rename f (N k)
    = N k
  rename f (Add l r)
    = Add (rename f l) (rename f r)

  rename f (And l r)
    = And (rename f l) (rename f r)

  rename f (Let this body)
    = Let (rename f this) (rename (weaken f) body)

```

with substitution defined as:

```idris
Substitute Ty Term where

  subst f (Var x)
    = f x
  subst f (B x)
    = B x
  subst f (N k)
    = N k
  subst f (Add l r)
    = Add (subst f l) (subst f r)
  subst f (And l r)
    = And (subst f l) (subst f r)
  subst f (Let this body)
    = Let (subst f this) (subst (weakens f) body)
```

## Values
```idris
namespace Value
```
We can define our terms values as:


```idris
  public export
  data Value : (term : Term ctxt type) -> Type where
    N : Value (N n)
    B : Value (B b)
```

## Reductions

The reductions are defined as:

```idris
public export
data Reduce : (this, that : Term ctxt type)
                         -> Type
  where
    -- Add
    SimplifyAddL : Reduce this that
              -> Reduce (Add this right) (Add that right)

    SimplifyAddR : Value left
                -> Reduce this that
                -> Reduce (Add left this) (Add left that)

    ReduceAdd : Reduce (Add (N a) (N b))
                       (N (a + b))

    -- And
    SimplifyAndL : Reduce this that
              -> Reduce (And this right) (And that right)

    SimplifyAndR : Value left
                -> Reduce this that
                -> Reduce (And left this) (And left that)

    ReduceAnd : Reduce (And (B a) (B b))
                       (B (a && b))

    -- Binders
    SimplifyLet : Reduce this that
               -> Reduce (Let this body)
                         (Let that body)

    ReduceLet : Value this
             -> Reduce (Let this body)
                       (Single.subst this body)
```

## Progress

```idris
namespace Progress
```

These reduction rules describe a single reduction, we still need to show what happens when a reduction is applied.
We do so with `Progress` that captures the result of applying a single reduction step applied to a term.

```idris
  public export
  data Progress : (term : Term Nil type)
                       -> Type
    where
```

Progress (i.e. reduction) will stop if a value is returned.

```idris
      Stop : (value : Value term)
                   -> Progress term
```

Otherwise progress has been made through application of a reduction rule.

```idris
      Step : {this,that : Term Nil type}
          -> (step : Reduce this that)
                  -> Progress this
```

with this definition of progress, we can 'prove' that each reduction step is used as follows:
We do so, however, on closed terms.

```idris
  export
  progress : (term : Term Nil type)
                  -> Progress term
```

In a closed term, variables do not exist.

```idris
  progress (Var _) impossible
```

When values are found we stop.

```idris
  progress (B x) = Stop B
  progress (N k) = Stop N
```

Otherwise we need to evaluate each operand to a value, and then reduce the conjunction or addition.

```idris
  progress (Add l r) with (progress l)
    progress (Add (N n) r) | (Stop N) with (progress r)
      progress (Add (N n) (N m)) | (Stop N) | (Stop N)
        = Step ReduceAdd
      progress (Add (N n) r) | (Stop N) | (Step step)
        = Step (SimplifyAddR N step)
    progress (Add l r) | (Step step)
      = Step (SimplifyAddL step)

  progress (And l r) with (progress l)
    progress (And (B n) r) | (Stop B) with (progress r)
      progress (And (B n) (B m)) | (Stop B) | (Stop B)
        = Step ReduceAnd
      progress (And (B n) r) | (Stop B) | (Step step)
        = Step (SimplifyAndR B step)
    progress (And l r) | (Step step)
      = Step (SimplifyAndL step)
```

Finally, as we are operating on bound terms these bound terms need to reduce...

```idris
  progress (Let this body) with (progress this)
    progress (Let this body) | (Stop value)
      = Step (ReduceLet value)
    progress (Let this body) | (Step step)
      = Step (SimplifyLet step)
```

## Evaluation
```idris
namespace Evaluation
```

`Progress` talks about reduction of a single term.
We still need to describe how we chain these reduction steps together i.e. `~>*`.

### Chaining reductions

We do so using `Reduces`:

```idris
  public export
  data Reduces : (this,that : Term ctxt type)
                           -> Type
    where
```

`Refl` captures that progress has stopped, the term can no longer be reduced.

```idris
      Refl : Reduces this this
```

`Trans` captures the sequencing of reductions (`~>*`).

```idris

      Trans : Reduce  this that
           -> Reduces      that end
           -> Reduces this      end
```

### Are we finished?

We need to combine `Reduces` with another structure called `Finished` that captures the idea that computations will either reach a value, or carry on becomes stuck.


```idris
  public export
  data Finished : (term : Term ctxt type)
                       -> Type
    where
```

Reductions stop when either a value is found,

```idris
      IsValue : Value term
             -> Finished term
```

or there is not enough fuel to carry on:

```idris
      NoFuel : Finished term
```

### Evaluation

`Evaluation` captures the idea that computations can potentially carry on forever until a result is found.

```idris
  public export
  data Evaluation : (term : Term Nil type)
                         -> Type
    where
      Eval : {this, that : Term Nil type}
          -> (steps  : Inf (Reduces this that))
          -> (result : Finished that)
                    -> Evaluation this
```

To realise this, we need to have a source of `Fuel`.
Predefined in the Idris `contrib`, `Fuel` is an infinite source of computation we can used to reason about potentially infinite processes.
The Idris book describes this in a bit more detail.

```idris
  public export
  evaluate : (fuel : Fuel)
          -> (term : Term Nil type)
                  -> Evaluation term
```

Evaluation stops when there is no more fuel.

```idris
  evaluate Dry term = Eval Refl NoFuel
```

But doesn't if there is fuel, and progress can be made.

```idris
  evaluate (More fuel) term with (progress term)
```

If we have reached a value, then stop.

```idris
    evaluate (More fuel) term | (Stop value)
      = Eval Refl (IsValue value)
```

otherwise try and evaluate further.

```idris

    evaluate (More fuel) term | (Step step {that}) with (evaluate fuel that)
```

and chain the reductions together.

```idris
      evaluate (More fuel) term | (Step step {that = that}) | (Eval steps result)
        = Eval (Trans step steps) result

```


## Running Things
```idris
namespace Run
```

With evaluation covered, we can bring it all together to describe how we run our programs.

That is, how we take a closed term, and reduce it to a value.
The function `run` will either return the value, or nothing if it runs out of fuel.

```idris
  export covering
  run : {type : Ty}
     -> (this : Term Nil type)
             -> Maybe (that : Term Nil type ** Reduces this that)
```

Using the infinite source of fuel, run the computaton.

```idris
  run this with (evaluate forever this)
```

We found a value, now return it with the steps used to get it.

```idris
    run this | (Eval steps (IsValue {term} x))
      = Just (term ** steps)
```

If there is no fuel, then no result.

```idris
    run this | (Eval steps NoFuel)
      = Nothing
```
