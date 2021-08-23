# Syntax

A programming language is a series of instructions that when executed perform a computation.
We use syntax to describe these instructions.

A programming language will have two kinds of syntax.
We will motivate syntax by considering a variant of Hutton's Razor (a language with *just* natural numbers and their addition) extended with multiplication.


## Concrete Syntax

This is the syntax that you will write your programs in, and how you structure your programs can be open to interpretation and parsing.
It can be inherently ambiguous.
For example:

    1 + 2 * 3 + 3

What is the order of execution of these expressions?

## Abstract Syntax

When reasoning about programs we want our syntax to be unambiguous.
Abstract syntax, which is often seen as **the** Abstract Syntax Tree, is the unambiguous form of the language.
For example:

    (add 1 (multiply 2 (add 3 3)))

What is the order of execution of these expressions?

## Describing

We can describe our syntax, concrete \& abstract using eBNF like grammars.
While concrete syntax *is* described using eBNF, abstract syntax takes a different form.

Compare:

    <EXPR>    ::= <OPERAND> <op> <OPERAND> | '(' <EXPR> ')'
    <OP>      ::= '+' | '*'
    <OPERAND> ::= <NAT> | <EXPR>
    <NAT>     ::= [0-9]+

with:

    n ::= 0,1,2,...
    e ::= n | (add e e) | (multiply e e)

We refer to individual pieces of syntax as *terms* with expressions being terms that describe computation and statements the flow of computation.
