# Notes on Programming Language Theory

I am interested in the theory of programming languages (PLT), namely intresting type-systems.
These notes present an inclusive guide to PLT formed by my own learning.
I will go from the basic concepts to more advanced topics.

It is important to say that theses are *my* notes, and I am sharing them because I can and hope that others can make use of them.
Should I return to teaching these notes might form the basis of lecture notes on PLT, in which case these notes will get overhauled and made more robust and referenced.

## In flux, Always.

This is a work in progress, so there will be changes.


## Comment on Notation.

PLT has a rich notation formed from mathematical type-setting.
This notation is almost always presented in LaTeX.
Rather than doing everything in LaTeX (which *does* give you cool looking notes), I am taking a mechanical verification approach and using the dependently-typed language Idris2 to mechanise the concepts.

This allows me to constraint the schematic notation to an existing executable format.
I use Idris2 as I know it more than Agda, and prefer Idris2's approach to notation and Unicode.
That being said, for my own textual computer based notes I have design a set of conventions for ASCII representations of PLT concepts.
I will use these as well when relating the theoretical presentation with the Idris2 notation.

Further, building upon Idris2's literate mode (which agda also has) also allows me to use [jupyter-book](https://jupyterbook.org/intro.html) to create literate notes.
Jupyter-Book allows me to realise both HTML and PDF notes, as well as present options to look at ePub versions.


## Reading

Jeremy Siek has a great blog post [on elementary PLT notation](http://siek.blogspot.com/2012/07/crash-course-on-notation-in-programming.html).

Jeremy has since released more material from a two part seminar series originally presented at `lambda Conf` 2018.

+ Part 1: <https://youtu.be/vU3caZPtT2I>
+ Part 2: <https://youtu.be/MhuK_aepu1Y>
+ Notes: <https://t.co/nvWmlkPewl>
