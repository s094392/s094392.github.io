---
title: Lambda Calculus
date: 2019-10-08 21:11:15
tags: [Math]
categories: [Notes]
---
This is a note and simple tutorial of lambda-calculus. I studied lambda-calculus for my final project in the Department of Computer Science.

## The Untyped $\lambda$-calculus
$\lambda$-calculus is equivalent to Turing machines in computational power. 

### Conversion rules
The $\lambda$-calculus provides several conversion rules for transforming one $\lambda$-calculus into an equivalent one. The conversion rules are defined as follows:
* $\beta$-conversion:
    $$(\lambda x.e)e' \Leftrightarrow e[e'/x]$$
    Just subsitute $x$ with $e'$ in $e$ like normal mathematic functions.
    
* $\alpha$-conversion:
    $$\lambda x.e \Leftrightarrow \lambda y.e[y/x] \quad y \notin 
    FV(e)$$
    
* $\eta$-conversion:
    $$\lambda x.(e \space x) \Leftrightarrow e \quad x \notin FV(e)$$
    Just eliminate a redundant $\lambda$-abstraction
    
* $\delta$-conversion:
    This rule define conversion of built-in constrants and functions, for example,
    $$(times \space 3 \space 4) \Leftrightarrow 12$$

We divide the set of $\lambda$-terms into *$\alpha$-equivalence classes*; this means that any two $\lambda$-terms that can be transfromed into one another via $\alpha$-conversion are in the same equivalence class.
### Reduction rules
The conversion rules express that two terms are equivalent, reduction rules are used to evaluate a term. There two reduction rules.
* $\beta$-reduction:
    $$(\lambda x.e)e' \Rightarrow e[e'/x]$$
    
* $\delta$-reduction:
    $$(times \space 3 \space 4) \Rightarrow 12$$

### Evaluation strategies
Serveral different evaluation strategies for the $\lambda$-calculus which have been studied over the years by programming language designers and theorists.

For example, consider the term:
$$(\lambda x.x) \space ((\lambda x.x) \space (\lambda z . (\lambda x.x)z))$$

And we have a definition of *identity function*, which just do nothing but return its argument:
$$id = \lambda x.x$$

Thus our target can be rewrite to a more readable form:
$$id \space (id \space (\lambda z.id \space z))$$

* Full $\beta$-reduction
Under this strategy, we might choose, for example, bo begin with the innermost redex(reducible expression), then do the one in the middle, then the outermost:
    $$\begin{equation}
    \quad id \space (id \space (\lambda z.\underline{id \space z})) \\
    \rightarrow id \space \underline{(id \space (\lambda z.z)}) \\
    \rightarrow \underline{(id \space (\lambda z.z)} \\
    \rightarrow \lambda z.z
    \end{equation}$$
    
* Normal order strategy
Under this strategy, the leftmost, outermost redex is always reduced first:
    $$\begin{equation}
    \quad \underline{id \space (id \space (\lambda z.id \space z))} \\
    \rightarrow \underline{id \space (\lambda z.id \space z)} \\
    \rightarrow \lambda z.\underline{id \space z} \\
    \rightarrow \lambda z.z
    \end{equation}$$

* Call-by-name
The *call-by-name strategy* is yet more restrictive, allowing no reductions inside abstractions.
    $$\begin{equation}
    \quad \underline{id \space (id \space (\lambda z.id \space z))} \\
    \rightarrow \underline{id \space (\lambda z.id \space z)} \\
    \rightarrow \lambda z.id \space z
    \end{equation}$$

* Call-by-value
This strategy is like to evaluate the parameters first.
    $$\begin{equation}
    \quad id \space \underline{(id \space (\lambda z.id \space z))} \\
    \rightarrow \underline{id \space (\lambda z.id \space z)} \\
    \rightarrow \lambda z.id \space z
    \end{equation}$$

### Programming in the $\lambda$-calculus
* Church Booleans
    Boolean values is a language feature tahat can easily be encoded in the $\lambda$-calculus. Define the terms `tru` and `fls` as follows:
    $$\begin{split}
    tru &= \lambda t. \lambda f. t; \\
    fls &= \lambda t. \lambda f. f;
    \end{split}$$
    
    According to the definition, the function `tru` accepts two arguments and returns the first one while `fls` returns the second one.
    Now, we can define some boolean operatos like logical conjunction as functions:
    $$\begin{split}
    and &= \lambda b. \lambda c. b \space c \space fls; \\
    or &= ... \\
    not &= ...
    \end{split}$$
    
* Church Numerals
    We define the *Church numerals* $c_0, c_1, c_2, etc.$ as follows:
    $$\begin{split}
    c_0 &= \lambda s. \lambda z.z; \\
    c_1 &= \lambda s. \lambda z. s \space z; \\
    c_2 &= \lambda s. \lambda z. s \space (s \space z); 
    \end{split}$$
    
    Each number $n$ is represented by a combinator $c_n$ that takes two argument, $s$ and $z$(for "successor" and "zero"). Then, we can define the successor function on Church numerals as follows:
    $$succ = \lambda n. \lambda s. \lambda z. s \space (n \space s \space z);$$
    So,
    $$\begin{split}
    succ \space c_1 &= \lambda n. \lambda s. \lambda z. s \space (n \space s \space z) \space c_1 \\
    &= \lambda s. \lambda z. s \space ((\lambda s. \lambda z. s \space z) \space s \space z) \\
    &= \lambda s. \lambda z. s \space (s \space z) \\
    &= c_2
    \end{split}$$
    
* Enrich the calculus
* Resursion

## The Simply Typed $\lambda$-Calculus
Typed $\lambda$-calculi are like untyped one, except that every bound identifier is given a type, which means each variables and functions are typed. The typed $\lambda$-calculus introduce the notion of *type correctness* of a term, which one would like to check before trying to evaluate the term.

### Introduction
We are going to consider the *successor function* as an example. A successor is a function that you put n than you got n + 1 as result, here is a sample practice in c language:
```cpp=
int succ(int n) {
    return n + 1;
}
```
In Typed $\lambda$-calculus we define the successor function as:
$$succ = \lambda n:int.n+1$$

We can see that the variable n is given a type $int$.

Now, we assume the addition operator + is typed as:
$$+: int \times int \rightarrow int$$

With the two definitions above, we could obtain the type:
$$succ: int \rightarrow int$$

Where $\rightarrow$ is the function type constructor and $\times$ the tuple type constructor used for multiple function arguments. We could then define:
$$twice = \lambda f: int \rightarrow int.\lambda x: int.f(f \space x)$$

and the term:
$$(twice \space succ) \space 4$$

would be *type-correct* and result in 6. On the other hand,
$$twice \space 7$$

would not be *type-correct*, since the type of 7 is $int$ rather than $$int \rightarrow int$$
