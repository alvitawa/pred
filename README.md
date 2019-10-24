
We use programming to compute answers to questions. From the trivial 

    "What are all the numbers less than 10?"

to the more complex 
 
    "What pixels should I show on a screen to display a like?"

In programming we usually define what it is we want by describing how the answer should be computed. So we write "Sum up all the numbers from 0 to 5" to get the sum of all the numbers less than 5. In a declarative programming language, we may have written "Return the sum of all numbers from 0 to 5" instead. The difference in this case is negligible, but typically, there is aditional overhead involved in describing the procedure as opposed to just the result. There exist some programming languages that try to allow the programmer to just declare what it is he wants, without also expecting him to describe how to get it (declarative, logic and constraint programming). They do so to varying degrees of success, but these languages are either limited in what they can express (domain specific) or they require certain 'hints' in their syntax to actually compute these results (cuts in Prolog, reordering SQL queries..). Hints limit the practical expressiveness of these languages, but it is percieved to be necessary, since otherwise they would not be able to compute nearly as many different concepts.

I will argue that this is an unecessary requirement. I believe that a general purpose (programing-like) language stripped of syntax intended to allow the programmer to describe *how* something is to be computed would be extremely useful [1]. Formally, I propose a language who's sole purpose is to allow humans to *declare what conditions a result must satisfy*, with absolutely *no* regard as to *how* this result may be computed, or whether it can be computed at all. In essence, this would be a programming language like most others, with modules, variables, possibly interpreters... However, statements will not have a correspondence to any particular resolution algorithm. This allows you for example to write a declaration among the lines of "The numbers [7,4,6,1] in sorted order", without specifiying any particular sorting algorithm.

This is the first benefit such a language would have. The language would allow you to formalize higher level concepts before delving into the details, in a similar fashion as with plain math. But contrary to math, you would be able to easily reference previous definitions with the module system and organize the new definitions in a similar way as a conventional codebase would be organized, with separations into files and documentation. In addition to this, the mathematical definitions could later be loaded into an interpreter and reasoned about automatically, generating proofs and/or implementations of the described phenomenon (provided such interpreters are made).

The second main benefit, wich contrasts with currently available programming languages, is that definitions can be *bidirectional*. What this means is that, for example, once you define the product of two numbers as `a * b = _`, you can immediately use the same defintion to describe division `a * _ = c`. Although some programming languages exist that can do this as well (like Prolog), they only allow bidirectional use in a limited number of circumstances. For the current proposal, bidirectionality would be enabled for all statements. However, this does not mean that you automatically have an *implementation* that can compute these inverses. [2]

[1] Note that it is still possible to describe a 'how' with 'whats' if needed. For example, you can reword the procedure "Swap A and B, then swap B and C" as "The algorithm that first Swaps A with B and then B with C". You can also constraint the algorithm in different ways. For example: "The algorithm that swaps A and C with less than 2 operations". You could in fact write a compiled program by specifying what assembly instructions it should execute. You could also describe lambda calculus expressions and how they should be resolved.

[2] I believe it has been proven that no general algorithms exist that can do this, which is why languages like Prolog only do it for some statements.

# An example

To see what such a language might look like, I will define a syntax and explore what can be expressed in it, as well as how the previously used terminology translates into it. This syntax should *not* be seen as *the* syntax of the language. I hope there are (wildly) different syntaxes out there that can be compared against each other to see which is best.

This syntax is similar to that of prolog on first sight. It consists of expressions and constraints, which declare relationships between the expressions. First I will only consider expressions that are just variables (`x`, `y`, ...). There is no syntax sugar whatsoever.

Constraints are the application of a predicate to some expressions. For example you can take the predicate `Int.LessThan` that takes 2 arguments and apply it to the expressions `a` and `b`. This would look like this `Int.LessThan(a, b)` and it would be saying that `a` is less than `b`. It effectively 'constrains' what values a and b can have.

    Int.Is(x), Int.Is(y), Int.Is(z), Int.Sum(z, y, x),

The above code says that `x`, `y`, and `z` are integers and that `x + y = z` (the first argument to Sum is the result of the sum). So this declaration also specifies that `z - y = x` and `z - x = y`.

You could expand it with the following constraints:

    Int.IsTwo(x), Int.IsThree(y).

And then you can conclude that `z` is five. That is, that `Int.IsFive(z)` is true (again, no syntax sugar).

`x`, `y` and `z` are variables. `Int.Is`, `Int.Sum`, `Int.IsTwo` and `Int.IsThree` are constraints. `Int` is a module. I will specify its contents later.

Putting these two declarations together into one, you may get this:

    Int.Is(x), Int.Is(y), Int.Is(z), Int.Sum(z, y, x),

    Int.IsTwo(x), Int.IsThree(y),

    z.


Statements are separated by commas and if the last statement of the declaration is an expression (variable), this expression specifies the result of the declaration. I suppose the last statement could also have a syntax like `return z` or `result z`, but there is no objective need for a keyword in this case.

(A 'statement' is either an expression or a constraint. Newlines and spaces are ignored. There must be a dot after the last statement.)

The previous declaration has the same result as the following:

    Int.IsFive(n), n.

Because `2 + 3 = 5`.

Let me point out that there is no 'true' way to define the value of an expression. The constraint `Int.IsFive(z).` specifies that `z` is five, just as well as `Int.Is(x), Int.Is(y), Int.Is(z), Int.Sum(z, y, x), Int.IsTwo(x), Int.IsThree(y).`. It is just that one format may be more convenient than the other, or it may emphasize different properties to the reader. In other words, things are defined relative to other things.

This is the module `Int` (Peano numbers):

    Is(zero), Zero(zero).
    Is(n) -> Is(s), Successor(s, n).

    IsOne(n1), Successor(n1, zero), Zero(zero),
    IsTwo(n2), Successor(n2, n1),
    IsThree(n3), Successor(n3, n2),
    IsFour(n4), Successor(n4, n3),
    IsFive(n5), Successor(n5, n4).

    Sum(a, a, 0), Is(a). 
    Sum(a, b, c), Is(a), Is(b), Is(c) -> Successor(sc, c), Successor(sa, a), Sum(sa, b, sc).

This module has several declarations. None of them have a result. Variables are scoped into the declarations. This means that the `n` used to define the successor is not the same `n` as the one to define sums, because they are in different declarations (ending with a dot). This is also why `Zero(zero)` is used when defining `IsOne` instead of using `zero` directly (declarations could also have been reorganized to not need to do this). 

The arrow (`->`) is an implication. In this case it says that if n is an integer, there is an integer s that is the successor of n. I gave it a lower precedence than the comma. Operators like implication are necessary to be able to define more complex concepts in the language itself, just the commas is not sufficient. Commas are basically 'AND' operators, everything separated by commas must all be true together. 

There are a number of different combinations of operators that can be chosen to be able to declare any logical relations (normal forms), but it is also strictly speaking not necessary to be able to do this. You can still define anything if you allow the use of predicates that are not defined in this language. For example, the module `Int` could be defined in plain english instead, or with conventional mathematical notation. You could even have *no* operators and just rely on the second language to define everyhing. But obviously it is not my goal to create a language that will 'delegate' everything to a different language. The point is that it is not the end of the world to allow some delegation, especially in the early stages.

Okay, back to discussing the syntax itself.

The discussed syntax is somewhat cumbersome for 3 reasons:
- There are no global variables, so identity constraints are needed (you need to write `Int.Zero(zero), zero.` to get zero).
- The programmer is always forced to create a variables when invoking constraints.
- The module prefix is always required (`Int.`)

These can be adressed as follows:
- Syntax sugar, you can have `Int.Zero(zero), zero.` be equivalent to just `Int.Zero`. For natural numbers and strings, literals can be added as wel.
- Introduce the subscript `/x` that can be appended to constraints so that the statement becomes an expression returning the `x`'th argument. So predicate `Sum` that takes three arguments can be invoked with the expression `Sum/0(x, y)`, which would represent the sum of x and y, and `Sum/1(z, y)` would be `z - y`. I think it looks ugly and could be confusing, but other than that it is very practical.
- `use Int`, `import Int`, `alias Int`,... etc but I don't think this is a very big issue

Other than that, the usual syntax sugars for assignment, multiplication, summation ... etc can be implemented.

Technically, one could simply use an existing programming language like python and forget the order of execution, but then you wouldn't be taking full advantage of not having to be computable.

Other design considerations are:
- The programmer should ideally never be required to create a variable and give it a name unless he wants to.
    Going back to the subscript `/x`, it could be exanded to multiple arguments, `/x/y`. So you can rewrite `Zeta(X), Gamma(X, Y, X).` as `Gamma/0/2(Y) = Zeta/0().` (With the addition of syntax for equality). More cases are supported. `Zeta(X), Beta(X,X), Gamma(X, Y).` can become `Zeta/0() = Beta/0/1() = Gamma/0(Y).` (An equality expression must equal it's result then). Another, uglier example `Zeta(X), Gamma(X, Y), Beta(Y), Alpha(Y).` -> `Zeta/0() = Gamma/0(Beta/0() = Alpha/0())`.[1]
- The programmer should never have to write code twice, he should always be able re-use (other than writing down the variable/module name).
- The programmer should have the freedom to reorganize code to make it easier to read.
- Adding features/requirements should require no code reorganization.
- The behaviour of modules should be protected. So it should not be possible to write code importing a module that breaks the module.

Not all of this goals may be achievable. Although it should be easier to achieve more of them than with procedural languages.

[1] Considerable syntax extensions are needed to do this in a text-based programming language. In the extreme case, there would be no variables at all and you should still be able to define the same relationships as before. If predicates were nodes in a graph you would link them to each other via their arguments (labeling the edges appropiately). If the graph was 3 dimensional you could always draw these relationships (edges) without them ever touching each other. If the graph was 2 dimensional you could still make arbitrary connections, but the edges would sometimes intersect each other. Text, however, is 1 dimensional [2], so you can only draw edges to adjacent nodes. This means that you need one of:

- 'link' Nodes that connect 2 or more separate locations in the 1 dimensional graph (these are variables).
- 'jumping' References, so that a constraint can have an equality with not only adjacent constraints but also constraints that are '2 to the left' or '3 to the right'. The syntax for this could be instead of just `/x` to reference argument x, `\x->d` to reference that argument `x` has an equality with the constraint `d` to the right. So `Predicate(Y), Odicate(X), Nedicate(Y)` could be written as `Predicate\(0->1)(), Odicate(X), Nedicate\(0<-1)()`. I think variables are prefferable, unless a syntax is found that naturally supports this as well as the other requirements (but variables should be supported anyways).

[2] Unless meaning is given to the 'column' position of symbols. For example, the following statement cannot be expressed without variables or jumping references in 1d:

    Predicate(X,Y), Qodicate(Y,Z), Redicate(Z,X).

The best you can do with the current syntax:

    Predicate/1(X) = Qodicate/0(Z), Redicate(Z,X).

Or extending the syntax so that constraints can have a different value to the right than to the left:

    Predicate/1(X) = Qodicate/0()1\ = Redicate\0(X).

In 2d, however (excuse my ascii art skills):

        Predicate()--| 0
             |1       \
             |         \
             |0         \
        Qodicate()      |
             |1         /
             |         /
             |0       /
        Redicate()---| 1

(every outgoing edge is labeled with an argument index)

The 2d example is still text, but now the horizontal position of symbols has significance. This is not a feasible syntax for a programming language, but I wonder what could be gained from programming in these graphs with the proper visual tools.

# Computing

The difference between this language and any other language is explicitly that it is not *designed* to be computable. That being said, interpreters/compilers *could* be written for the language and they may be quite usefull. However, the implementation of these tools should *not* influence the design of the language. If these tools need hints in order to compute certain results, these should be given manually or imported from a separate source. For example, you could write an algorithm in python that lists all the numbers up to x, and tell the interpreter that it can use that implementation to compute a certain part of your declaration.

# Practical applications

At the very least this is a theoretical exercise that can help to understand the design of (other) programming languages. It can represent a model for a language with, in theory, near-'ideal' expressiveness, although in practice this would not be the case since the language by itself is not enough to compute results, so additional information/code is needed to do this.

On the more practical side, the language could be used to write formalizations that would otherwise be written in math. If the package/module ecosystem becomes developed enough, this could be significantly easier to do in this language. Other than the ease with which propper math can be written in it (which is admittedly hard to predict right now) there is the added advantage that it may be loaded into an interpreter and reasoned about automatically or translated to different formats. Depending on the interpreters that are written the functionality could include proving relationships between predicates and/or doing calculations.

The last feature (calculations) would be especially usefull in my opinion. The general case would be that if you have written a definition of for example what prime numbers are, you could easily write a query that to get the value of the tenth prime number. The interpreter would not always be able to do this automatically, in which case you would have to give it some information on how to do the computation. In the worst case, if the interpreter is not very potent, you might for example have to write an implementation of `getNthPrime` in haskell and tell the interpreter that it can use that to compute your result, but for a better interpreter it may be sufficient to implement just `isPrime` and it would automatically calculate the result with brute force. I believe interpreters can be written that are potent enough to cover most common computations, freeing up the programmer from thinking about how a calculation can be done and allowing him to focus on describing the result instead. These 'hints' could also be distributed through a package ecosystem, allowing programmers to benefit from efficient algorithms created by someone else. I will publish a proposal for such an interpreter at a later date.

This concept of writing the specification first and the impementation later can be especially usefull for hard problems where you are not even sure what result you are actually looking for. Personally, I have lost myself quite often in writing an implementation, only to realize later that the concept itself was not possible. I believe it is standard practice to have this separation in many fields, and this language would tie neatly into this process.

There is more that can theoretically be done with this language (and would be usefull to do), this includes specifying algorithms and compiling them. I jus t wanted to mention these, but I don't think they are worth discussing at this point. I think a programming language should open itself up to be used in creative ways, but it is important not to try to do to much too soon. 

# Appendix

While I have searched, I have not been able to find much prior work on this subject. If you do know of a similar project please contact me, even if it is only very similar. Feel free to discuss anything on this page with me. I will edit as I learn. Also share your ideas on the syntax such a language could have :)

I will try to document different syntaxes and ideas in this repository.

This idea is related to declarative, logic, and constraint programming. The syntax of the language itself could be the syntax for any of these types of languages, if it was interpreted as such and given the necessary hints. I think the language would probably classify as a constraint programming language. There is also some overlap with automated theorem provers.

Other similar concepts are data formats like RDF. The same notions could be expressed in it as with the example I discussed, however, RDF is not particularly pleasing to work with directly, since it is intended as a data format.