# Executing Programs Operationally

Operational semantics describe how we evaluate our programs.
This describes how we can *reduce*/evaluate our language expressions and statements to a single value.
There are generally two common styles of operational semantics: Big-Step, and Small-Step.
There are more formal names given but we generally refer to the styles using these names.

So alongside our type-system will be a series of reduction rules that describe how terms can be reduced to a *value*.

+ **Values** are a subset of terms we consider to be irreducible.

We describe these reductions either in small-steps or big-steps.

+ **Small Step** For each term we describe how we reduce each sub-term (or term as a whole) to get towards a value.

+ **Big Step** For each term we describe how we get down to the value directly.


For this section we will consider our exemplar, extended with conditional expressions:

    t : TYPE ::= BOOL | NAT
    n : NAT  ::= 0,1,2,...    | (add e e)
    b : BOOL ::= True | False | (and e e)
    e : t    ::= x | n | b
               | let x be e in e
               | if e then e else e

For this language our values will be:

    n ::= 0,1,2...
    b ::= True | False
    v ::= n | b

## Big Step

Big-Step semantics are concerned with what the final result is; we can skip description of intermediate computations.
We denote the transition using a fat down arrow, but for ASCII we use a fat right arrow.
Here are the Big-Step semantics for our language.

    ====== [ Nat ]
    n => n

    ====== [ Bool ]
    b => b

    a => a'
    b => b'
    ==================== [ Add ]
    (add a b) => a' + b'

    a => a'
    b => b'
    ===================== [ And ]
    (and a b) => a' /\ b'

    c => True
    t => t'
    ======================== [ If-True ]
    if c then t else f => t'

    c => False
    f => f'
    ======================== [ If-False ]
    if c then t else f => f'

    e => e'
    replace x with e' in b => b'
    ============================ [ Let ]
    let x be e in b => b'

Here we use *real* operations to show how an expression is reduced using *real* operations.
The rules for `IF-True` and `IF-False` describe the branching that occurs with use of conditional statements.

## Small Step

Small-Step semantics are concerned with how we get to the final result; we cannot skip intermediate computations.

    ====== [ Nat ]
    n ~> n

    ====== [ Bool ]
    b ~> b

    a ~> n
    ====================== [ Add-E ]
    (add a b) ~> (add n b)

    b ~> n'
    =================== [ Add-B ]
    (add n b) => n + n'

    a ~> n
    ====================== [ And-E ]
    (and a b) ~> (and n b)

    b ~> n'
    ==================== [ And-B ]
    (and n b) => n /\ n'

    c ~> b
    ======================================== [ If-E ]
    if c then t else f ~> if b then r else f

    c ~> True
    ======================== [ If-True-B ]
    if c then t else f ~> t

    c ~> False
    ================== [ If-False-B ]
    if c then t else f ~> f

    e ~> e'
    =================================== [ Let-E]
    let x be e in b ~> let x be e' in b

    replace x with v in b ~> b'
    =========================== [ Let-B ]
    let x be v in b ~> b'

Again we use *real* operations to show how an expression is reduced using *real* operations.
We denote the transition using a right squiggly arrow, and multiple reductions with `~>*`.
With these semantics, however, we must describe how we reduce each sub-term in the language.
Rules that reduce *just* a sub-term are called eta-reductions (rules named x-E), and rules that collapse a term beta-reductions (rules named x-B).
Notice that for each beta-rule to be applied we must have already moved from left-to-right across the term.
