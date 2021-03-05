
We use programming to compute answers to questions. From the trivial 

    "What are all the numbers less than 10?"

to the more complex 
 
    "What pixels should I show on a screen to display a like?"

In programming we usually define what it is we want by describing how the answer should be computed. So we write "Sum up all the numbers from 0 to 5" to get the sum of all the numbers less than 5. In a declarative programming language, we may have written "Return the sum of all numbers from 0 to 5" instead. The difference in this case is negligible, but typically, there is additional overhead involved in describing the procedure as opposed to just the result. There exist some programming languages that try to allow the programmer to just declare what it is he wants, without also expecting him to describe how to get it (declarative, logic and constraint programming). They do so to varying degrees of success, but these languages are either limited in what they can express (domain specific) or they require certain 'hints' in their syntax to actually compute these results (cuts in Prolog, reordering SQL queries..). Hints limit the practical expressiveness of these languages, but it is perceived to be necessary, since otherwise they would not be able to compute nearly as many different concepts.

I will argue that this is an unnecessary requirement. I believe that a general purpose (programing-like) language stripped of syntax intended to allow the programmer to describe *how* something is to be computed would be extremely useful [1]. Formally, I propose a language who's sole purpose is to allow humans to *declare what conditions a result must satisfy*, with absolutely *no* regard as to *how* this result may be computed, or whether it can be computed at all. In essence, this would be a programming language like most others, with modules, variables, possibly interpreters... However, statements will not have a correspondence to any particular resolution algorithm. This allows you for example to write a declaration among the lines of "The numbers [7,4,6,1] in sorted order", without specifying any particular sorting algorithm.

This is the first benefit such a language would have. The language would allow you to formalize higher level concepts before delving into the details, in a similar fashion as with plain math. But contrary to math, you would be able to easily reference previous definitions with the module system and organize the new definitions in a similar way as a conventional codebase would be organized, with separations into files and documentation. In addition to this, the mathematical definitions could later be loaded into an interpreter and reasoned about automatically, generating proofs and/or implementations of the described phenomenon (provided such interpreters are made).

The second main benefit, which contrasts with currently available programming languages, is that definitions can be *bidirectional*. What this means is that, for example, once you define the product of two numbers as `a * b = _`, you can immediately use the same definition to describe division `a * _ = c`. Although some programming languages exist that can do this as well (Prolog), they only allow bidirectional use in a limited number of circumstances. For the current proposal, bidirectionality would be enabled for all statements. However, this does not mean that you automatically have an *implementation* that can compute these inverses. [2]

Thirdly, the language would not have the readability/efficiency trade-off all computable programming languages need to deal with, since computability is not a requirement to begin with.

[1] Note that it is still possible to describe a 'how' with 'whats' if needed. For example, you can reword the procedure "Swap A and B, then swap B and C" as "The algorithm that first Swaps A with B and then B with C". You can also constraint the algorithm in different ways. For example: "The algorithm that swaps A and C with less than 2 operations". You could in fact write a compiled program by specifying what assembly instructions it should execute. You could also describe lambda calculus expressions and how they should be resolved.

[2] I believe it has been proven that no general algorithms exist that can do this, which is why languages like Prolog only do it for some statements.

# An example: Pure Prolog

A well-known language that already incorporates some of this ideas is Pure Prolog. Pure Prolog is Prolog but without the cuts and other computational hints. It functions quite well as a math notation and available interpreters can compute some problems written in it with their depth-first algorithms (to compute all that are computable, you need the hints/cuts). However, I think a better language could be designed to serve as mathematical notation/prototype design language.

The obvious available improvements are a more modern and consistent standard library as well as a shorter notation and modern naming standards. For example, the following PProlog code:

    Man(X) :- Living(X).
    CloseTo(X,Y) :- CloseTo(Y,X)
    A(X) :- B(X), C(X).

Could lose the dashes and the brackets without loss of meaning:

    man x: living x.
    close_to x y: close_to y x.
    a x: b x, c x.

[1]

Whether this change is an improvement may be a little bit subjective as some may rather stick with what is familiar. My opinion is that if you are going to redesign a language you might as well modernize it.

A more original improvement is a standard syntax for for specifying the return value of a `predicate expression`. Prolog code is riddled with single-use variables (`X`, `Y`) because it does not know of any return values other than `true` or `false` for it's predicates (whether they were satisfied or not). The expression `Likes(X,Y)` for example evaluates to either `true` or `false`, but what you may actually be interested in is the value matching for `X`. So you could write `Likes$0(Y)` for example to specify the first argument (with `$0`, index 0) as the value of the expression. The statement `X = Likes$0(Y).` would be equivalent to `Likes(X,Y).`. If instead you are interested in the second argument, you may write `Y = Likes$1(X)`, where `$1` refers to the second argument. A more useful example is `Likes(X,Y), Likes(Y,X)`. If you are only interested in the `X` of this expression, you wouldn't need to give a name to `Y` by writing instead: `Likes$1(X) = Likes$0(X)`. Improvements to this idea may be possible, for example, the syntax can be expanded to return multiple values in a tuple-like format (`(X,Y) = Likes$0$1()`). Ideally, with this type of syntax the programmer would never be *required* to declare a variable if he does not want to do so [2].

On a different note, it is well known that Pure Prolog expressions are horn clauses. In logic, PProlog then constitutes to a conjunction of implications with negations, like `a \land (b \rightarrow \neg a) \land (x \rightarrow x)`. I think it is worth reconsidering whether this is the ideal choice of logical operators (a conjunction of implications with negations). All normal forms [3] are powerful enough to encode any logical expression [4], but some may be more intuitive than others for different applications. Allowing all (binary as in two inputs) operators, arbitrarily nested, is also an option, but some might argue that operators like `xor` or `nand` are not be useful enough to warrant built-in support (and could instead be composed of other operators). Personally I would prefer if the language does not discriminate between logical relationships and instead lets the end user decide what he uses. 

The open/closed world assumption comes into mind as well.

Also, like with some conventional programming languages, but possibly more useful for some use cases of the proposed language, the language could allow predicates to be declared but not defined. For example, allow the declaration of `Prime(X)` with the docstring `Prime(X) is true for X if X is a prime number` without specifying what this means (other than in the docstring). This is useful for TODO's (do it later) but, given the context this language is intended to be used in (defining things, not computing them) you could just not define a component of something if you don't feel like it, and use natural English to describe it. As long as others that read this know what it means and you don't intend to compute anything with it this should be fine. That being said, it is of course better if people *did* define everything in the language, for specificity's sake. [5]

<!-- There are a number of different combinations of operators that can be chosen to be able to declare any logical relations (normal forms), but it is also strictly speaking not necessary to be able to do this. You can still define anything if you allow the use of predicates that are not defined in this language. For example, the module `Int` could be defined in plain english instead, or with conventional mathematical notation. You could even have *no* operators and just rely on the second language to define everyhing. But obviously it is not my goal to create a language that will 'delegate' everything to a different language. The point is that it is not the end of the world to allow some delegation, especially in the early stages. -->

Last but not least, the language should have a good working module/package system to facilitate the re-use of code. An independent package manager could be made or, alternatively, an existing one like `cargo` or `npm` could be used.

Other, general, design considerations are:
- The programmer should never have to write code twice, he should always be able re-use definitions (other than writing down the variable/module name).
- The programmer should have the freedom to reorganize code to make it easier to read.
- Adding features/requirements should require no code reorganization.
- The behavior of modules should be protected. So it should not be possible to write code importing a module that changes the definitions of the module.
- Don't make any of the obvious mistakes, like undefined behavior.

Not all of this goals may be achievable. Although it should be easier to achieve more of them than with procedural 'cumputable' languages.

[1] There is an interesting project on github that implements a terser version of Prolog (`https://github.com/JCumin/Brachylog`).

[2] Considerable syntax extensions are needed to do this in a text-based programming language. In the extreme case, there would be no variables at all and you should still be able to define the same relationships as before. If predicates were nodes in a graph you would link them to each other via their arguments (labeling the edges appropriately). If the graph was 3 dimensional you could always draw these relationships (edges) without them ever touching each other. If the graph was 2 dimensional you could still make arbitrary connections, but the edges would sometimes intersect each other. Text, however, is 1 dimensional [6], so you can only draw edges to adjacent nodes. This means that you need one of:

- 'link' Nodes that connect 2 or more separate locations in the 1 dimensional graph (these are variables).
- 'jumping' References, so that a constraint can have an equality with not only adjacent constraints but also constraints that are '2 to the left' or '3 to the right'. The syntax for this could be instead of just `/x` to reference argument x, `$x->d` to reference that argument `x` has an equality with the constraint `d` to the right. So `Predicate(Y), Odicate(X), Nedicate(Y)` could be written as `Predicate$0->1(), Odicate(X), Nedicate$0<-1()`. I think variables are preferable, unless a syntax is found that naturally supports this as well as the other requirements (but variables should be supported anyways).

[3] https://www.cs.sfu.ca/~ggbaker/zju/math/normalform.html

[4] The baseline requirement for the language is that arbitrary data can be encoded in it and that arbitrary properties of this data can be defined.

[5] Makes me think that a way of declaring things to let others define them later might be a useful feature, like a sort of module-wide generics. So a module could define high-level stuff like equations without either defining numbers or specifying what module implements them. Then, users of the high-level module can specify a module that defines the numbers themselves if they need to.

[6] Unless meaning is given to the 'column' position of symbols. For example, the following statement cannot be expressed without variables or jumping references in 1d:

    Predicate(X,Y), Qodicate(Y,Z), Redicate(Z,X).

The best you can do with the current syntax:

    Predicate$1(X) = Qodicate$0(Z), Redicate(Z,X).

Or extending the syntax so that constraints can have a different value to the right than to the left:

    Predicate$1(X) = Qodicate$0()1\ = Redicate$0(X).

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

The difference between this language and any other language is explicitly that it is not *designed* to be computable. That being said, interpreters/compilers *could* be written for the language and they may be quite useful. However, the implementation of these tools should *not* influence the design of the language. If these tools need hints in order to compute certain results, these should be given manually or imported from a separate source. For example, you could write an algorithm in python that lists all the numbers up to x, and tell the interpreter that it can use that implementation to compute a certain part of your declaration.

# Practical applications

At the very least, designing this language is a theoretical exercise that can help to understand the design of (other) programming languages. The language can represent a model for a programming language with, in theory, near-'ideal' expressiveness, although in practice this would not be the case since the language by itself is not enough to make any computations, so additional information/code is needed to do this (the interpreters and whatever hints you need to give them).

On the more practical side, the language could be used to write formalizations that would otherwise be written in math. If the package/module ecosystem becomes developed enough, this could be significantly easier to do in this language. Other than the ease with which propper math may be written in it there is the added advantage that it may be loaded into an interpreter and reasoned about automatically, or translated to different formats. Depending on the interpreters that are written the functionality could include proving relationships between predicates and/or doing calculations.

The last feature (calculations) would be especially useful in my opinion. The general case would be that if you have written a definition of for example what prime numbers are, you could easily write a query to get the value of the tenth prime number. The interpreter would not always be able to do this automatically, in which case you would have to give it some information on how to do the computation. In the worst case, if the interpreter is not very potent, you might for example have to write an implementation of `getNthPrime` in Haskell and tell the interpreter that it can use that to compute your result, but for a better interpreter it may be sufficient to implement just `isPrime` and it would automatically calculate the result with brute force. I believe interpreters can be written that are potent enough to cover most common computations, freeing up the programmer from thinking about how a calculation can be done and allowing him to focus on describing the result instead. These 'hints' could also be distributed through a package ecosystem, allowing programmers to benefit from efficient algorithms created by someone else.

This concept of writing the specification first and the implementation later can be especially useful for hard problems where you are not even sure what result you are actually looking for. Personally, I have lost myself quite often in writing an implementation, only to realize later that the concept itself was not possible. I believe it is standard practice to have this separation in many fields, and this language would tie neatly into this process.

The language may also be useful for doing program synthesis, as a specification language. In fact the proposed language *is* a specification language.

# Appendix

While I have searched, I have not been able to find much prior work on this subject. If you do know of a similar project please contact me, even if it is only very similar. Feel free to discuss anything on this page with me. I will edit as I learn. Also share your ideas on the syntax such a language could have :)

I will try to document different syntaxes and ideas in this repository.

This idea is related to declarative, logic, and constraint programming. The syntax of the language itself could be the syntax for any of these types of languages, if it was interpreted as such and given the necessary hints. I think the language would probably classify as a constraint programming language. There is also some overlap with automated theorem provers.

Other similar concepts are data formats like RDF. The same notions could be expressed in it as with the example I discussed, however, RDF is not particularly pleasing to work with directly, since it is intended as a data format.

[Sentient](https://sentient-lang.org/) is an interesting programming language that shares some of the same goals.

Another interesting project that probably comes the closest to the ideas discussed here is the [Alan](https://alan-lang.org/the-turing-completeness-problem.html) language. It forgoes Turing-Completeness in the interest of automatic parallelization.
