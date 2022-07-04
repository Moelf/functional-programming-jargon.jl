# Functional Programming Jargon in Julia

Functional programming (FP) provides many advantages, and its popularity has been increasing as a result. However, each programming paradigm comes with its own unique jargon and FP is no exception. By providing a glossary, we hope to make learning FP easier.

This is a fork of [Functional Programming Jargon](https://github.com/jmesyou/functional-programming-jargon). Examples are presented in Julia[julialang.org].

This document is WIP and pull requests are welcome!

__Translations__
* [Portuguese](https://github.com/alexmoreno/jargoes-programacao-funcional)
* [Spanish](https://github.com/idcmardelplata/functional-programming-jargon/tree/master)
* [Chinese](https://github.com/shfshanyue/fp-jargon-zh)
* [Bahasa Indonesia](https://github.com/wisn/jargon-pemrograman-fungsional)
* [Scala World](https://github.com/ikhoon/functional-programming-jargon.scala)
* [Korean](https://github.com/sphilee/functional-programming-jargon)

__Table of Contents__
<!-- RM(noparent,notop) -->

* [Arity](#arity)
* [Higher-Order Functions (HOF)](#higher-order-functions-hof)
* [Closure](#closure)
* [Partial Application](#partial-application)
* [Currying](#currying)
* [Auto Currying](#auto-currying)
* [Function Composition](#function-composition)
* [Continuation](#continuation)
* [Purity](#purity)
* [Side effects](#side-effects)
* [Idempotent](#idempotent)
* [Point-Free Style](#point-free-style)
* [Predicate](#predicate)
* [Contracts](#contracts)
* [Category](#category)
* [Value](#value)
* [Constant](#constant)
* [Functor](#functor)
* [Pointed Functor](#pointed-functor)
* [Lift](#lift)
* [Referential Transparency](#referential-transparency)
* [Equational Reasoning](#equational-reasoning)
* [Lambda](#lambda)
* [Lambda Calculus](#lambda-calculus)
* [Lazy evaluation](#lazy-evaluation)
* [Monoid](#monoid)
* [Monad](#monad)
* [Comonad](#comonad)
* [Applicative Functor](#applicative-functor)
* [Morphism](#morphism)
  * [Endomorphism](#endomorphism)
  * [Isomorphism](#isomorphism)
  * [Homomorphism](#homomorphism)
  * [Catamorphism](#catamorphism)
  * [Anamorphism](#anamorphism)
  * [Hylomorphism](#hylomorphism)
  * [Paramorphism](#paramorphism)
  * [Apomorphism](#apomorphism)
* [Setoid](#setoid)
* [Semigroup](#semigroup)
* [Foldable](#foldable)
* [Lens](#lens)
* [Type Signatures](#type-signatures)
* [Algebraic data type](#algebraic-data-type)
  * [Sum type](#sum-type)
  * [Product type](#product-type)
* [Option](#option)
* [Function](#function)

<!-- /RM -->

## Arity

The number of arguments a function takes. From words like unary, binary, ternary, etc. This word has the distinction of being composed of two suffixes, "-ary" and "-ity." Addition, for example, takes two arguments, and so it is defined as a binary function or a function with an arity of two. Such a function may sometimes be called "dyadic" by people who prefer Greek roots to Latin. Likewise, a function that takes a variable number of arguments is called "variadic," whereas a binary function must be given two and only two arguments, currying and partial application notwithstanding (see below).

```julia
julia> myadd(x,y) = x + y

julia> arity = first(methods(myadd)).nargs - 1
2

# The arity of `myadd` is 2
```

## Higher-Order Functions (HOF)

A function which takes a function as an argument and/or returns a function.

```julia
julia> filter(iseven, 1:5)
2-element Vector{Int64}:
 2
 4

julia> sum(abs, [-1, -2, 3])
6
```

## Closure

A closure is a way of accessing a variable outside its scope.
Formally, a closure is a technique for implementing lexically scoped named binding. It is a way of storing a function with an environment.

A closure is a scope which captures local variables of a function for access even after the execution has moved out of the block in which it is defined.
ie. they allow referencing a scope after the block in which the variables were declared has finished executing.


```julia
# this function returns an anonymous function that **closes over** `init`
julia> add_to(init) = x -> x + init

julia> add_to_five = add_to(5)

julia> add_to_five(3)
8
```

Anonymous function Vs Closure: A anonymous is essentially a function that is defined inline rather than the standard method of declaring functions. Lambdas can frequently be passed around as objects.

A closure is a function that encloses its surrounding state by referencing fields external to its body. The enclosed state remains across invocations of the closure.

__Further reading/Sources__
* [Lambda Vs Closure](http://stackoverflow.com/questions/220658/what-is-the-difference-between-a-closure-and-a-lambda)
* [JavaScript Closures highly voted discussion](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work)

## Partial Application

Partially applying a function means creating a new function by pre-filling some of the arguments to the original function.

```julia
julia> add3(a, b, c) = a + b + c
add3 (generic function with 1 method)

julia> partial(f, args...) = x -> f(args..., x)

julia> five_plus = partial(add3, 2, 3)

julia> five_plus(4)
9

#special case of 2-arguments original function
julia> six_plus = Base.Fix1(+, 6)
(::Base.Fix1{typeof(+), Int64}) (generic function with 1 method)

julia> six_plus(7)
13
```

Partial application helps create simpler functions from more complex ones by baking in data when you have it. [Curried](#currying) functions are automatically partially applied.


## Currying

The process of converting a function that takes multiple arguments into a function that takes them one at a time.

Each time the function is called it only accepts one argument and returns a function that takes one argument until all arguments are passed.

```julia
julia> curried_add(a) = b -> a + b

julia> curried_add(40)(2)
42

julia> add2 = curried_add(2)

julia> add2(10)
12
```

## Auto Currying
Transforming a function that takes multiple arguments into one that if given less than its correct number of arguments returns a function that takes the rest. When the function gets the correct number of arguments it is then evaluated.

The [Curries.jl](https://github.com/MasonProtter/Currier.jl) package has an currying decorator which works this way.

```python
julia> using Currier

julia> @curried add3(x, y, z) = x + y + z

julia> add3(1,2,3)
6

julia> add3(1,2)(3)
6

julia> add3(1)(2)(3)
6
```

__Further reading__
* [Favoring Curry](http://fr.umio.us/favoring-curry/)
* [Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)

## Function Composition

The act of putting two functions together to form a third function where the output of one function is the input of the other.

```julia
# `∘` can be typed with \circ<tab>
julia> floor_and_string = string ∘ floor

julia> floor_and_string(121.212121)
"121.0"
```

## Continuation

At any given point in a program, the part of the code that's yet to be executed is known as a continuation.

```julia
julia> print_as_string(num) = println("Given $num")

julia> function add_one_and_continue(num, cc)
           result = num + 1
           cc(result)
       end

julia> add_one_and_continue(2, print_as_string)
Given 3
```

## Purity

A function is pure if the return value is only determined by its
input values, and does not produce side effects. Be very careful when
using [@pure](https://discourse.julialang.org/t/pure-macro/3871) in Julia.

```julia
julia> greet(name) = "Hi, $name"

julia> greet("Brianne")
"Hi, Brianne"
```

As opposed to each of the following:

```julia
julia> name = "Brianne"
"Brianne"

julia> greet() = "Hi, $name"
greet (generic function with 2 methods)

julia> greet()
"Hi, Brianne"
```

The above example's output is based on data stored outside of the function...

```julia
julia> greeting = nothing

julia> function greet(name)
         global greeting
         greeting = "Hi, $name"
         return nothing
       end

julia> greet("Brianne")

julia> greeting
"Hi, Brianne"
```

... and this one modifies state outside of the function.

## Side effects

A function or expression is said to have a side effect if apart from returning a value, it interacts with (reads from or writes to) external mutable state.

```julia
println("IO is a side effect!")
```

## Idempotent

A function is idempotent if reapplying it to its result does not produce a different result.

```julia
identity(identity(10))
```

```julia
# this relys on sort algorithm being stable
sort(sort(sort([2, 1])))
```

## Point-Free Style

Writing functions where the definition does not explicitly identify the arguments used. This style usually requires [currying](#currying) or other [Higher-Order functions](#higher-order-functions-hof). A.K.A Tacit programming.

```julia
# Given
julia> mymap = fn -> vec -> map(fn, vec)
julia> myadd = a -> b -> a + b

# Then
# Not points-free - `numbers` is an explicit argument
julia> increment_all = numbers -> mymap(myadd(1))(numbers)

julia> increment_all([4,7,8])
3-element Vector{Int64}:
 5
 8
 9

# Points-free - The list is an implicit argument
julia> increment_all2 = mymap(myadd(1))

julia> increment_all([4,7,8])
3-element Vector{Int64}:
 5
 8
 9
```

`increment_all` identifies and uses the parameter `numbers`, so it is not points-free.  `increment_all2` is written just by combining functions and values, making no mention of its arguments.  It __is__ points-free.

Points-free function definitions look just like normal assignments without `function ...` or `blah -> `.

## Predicate
A predicate is a function that returns true or false for a given value. A common use of a predicate is as the callback for array filter.

```julia
julia> filter(>(2), [1, 2, 3, 4])
2-element Vector{Int64}:
 3
 4
```

## Contracts

A contract specifies the obligations and guarantees of the behavior from a function or expression at runtime. This acts as a set of rules that are expected from the input and output of a function or expression, and errors are generally reported whenever a contract is violated.

```julia
julia> add1(num::Integer) = num + 1

julia> add1(num) = throw(ArgumentError("Contract violated: expected `num::Integer`"))

julia> add1(2)
3

julia> add1("blah")
ERROR: ArgumentError: Contract violated: expected `num::Integer`
```

## Category

A category in category theory is a collection of objects and morphisms between them. In programming, typically types
act as the objects and functions as morphisms.

To be a valid category 3 rules must be met:

1. There must be an identity morphism that maps an object to itself.
    Where `a` is an object in some category,
    there must be a function from `a -> a`.
2. Morphisms must compose.
    Where `a`, `b`, and `c` are objects in some category,
    and `f` is a morphism from `a -> b`, and `g` is a morphism from `b -> c`;
    `g(f(x))` must be equivalent to `(g • f)(x)`.
3. Composition must be associative
    `f • (g • h)` is the same as `(f • g) • h`

Since these rules govern composition at very abstract level, category theory is great at uncovering new ways of composing things.

__Further reading__

* [Category Theory for Programmers](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)

## Value

Anything that can be assigned to a variable.

```julia
('John', 30)
(name = 'Person', age = 30)
5
a -> a^2
[1.0]
missing
```

## Constant

A variable that cannot be reassigned once defined.

```julia
const five = 5
```

Constants are [referentially transparent](#referential-transparency). That is, they can be replaced with the values that they represent without affecting the result.

## Functor

An object that implements a `map` function which, while running over each value in the object to produce a new object, adheres to two rules:

__Preserves identity__
```
object.map(x => x) ≍ object
```

__Composable__

```
object.map(compose(f, g)) ≍ object.map(g).map(f)
```

(`f`, `g` are arbitrary functions)

In Julia `map` takes the first argument as the function:

```julia
julia> map(identity, [1,2,3])
3-element Vector{Int64}:
 1
 2
 3

julia> map(identity, (1,2,3))
(1, 2, 3)
```

and

```julia
julia> f(x) = x+1

julia> g(x) = x*2

julia> map(f, map(g, [1,2,3]))
3-element Vector{Int64}:
 3
 5
 7

julia> map(f∘g, [1,2,3])
3-element Vector{Int64}:
 3
 5
 7
```

## Pointed Functor
An object with an `of` function that puts _any_ number of values into it. The following property
`of(f(x)) == of(x).map(f)` must also hold for any pointed functor. [Better explaination](https://stackoverflow.com/questions/39179830/how-to-use-pointed-functor-properly)

```julia 
# here we think of `Vector` and `Base.map` together as a functor:

julia> of(t::AbstractVector{T}, vals...) where T = T[vals...]

julia> of([], 3)
1-element Vector{Any}:
 3

julia> of(Int[], 3)
1-element Vector{Int64}:
 3

julia> of(Float64[], 3)
1-element Vector{Float64}:
 3.0
```

## Lift

Lifting is when you take a value and put it into an object like a [functor](#pointed-functor). If you lift a function into an [Applicative Functor](#applicative-functor) then you can make it work on values that are also in that functor.

Some implementations have a function called `lift`, or `liftA2` to make it easier to run functions on functors.

```julia
julia> liftA2(fn) = (a, b) -> fn.(a, b) # cheating, rely on broadcast to figure out how to apply `fn`
liftA2 (generic function with 1 method)

julia> mult(a, b) = a * b
mult (generic function with 1 method)

julia> lifted_mult = liftA2(mult) # this function now works on functors (arrays)
#5 (generic function with 1 method)

julia> lifted_mult([1, 2], [3]) # [3, 6]
2-element Vector{Int64}:
 3
 6

julia> lifted_mult([1, 2], 3)
2-element Vector{Int64}:
 3
 6

julia> lifted_mult([1, 2], [3,4])
2-element Vector{Int64}:
 3
 8
```

## Referential Transparency

An expression that can be replaced with its value without changing the
behavior of the program is said to be referentially transparent.

```python
greet() = "Hello World!"
```

Any invocation of `greet()` can be replaced with `Hello World!` hence greet is
referentially transparent. In Julia this is trivially true for any `immutable` object.

##  Equational Reasoning

When an application is composed of expressions and devoid of side effects, truths about the system can be derived from the parts.

## Lambda

An anonymous function that can be treated like a value.

```julia 
julia> f(a) = a + 1
f (generic function with 1 method)

julia> a -> a + 1
#3 (generic function with 1 method)
```
Lambdas are often passed as arguments to Higher-Order functions.

```julia
julia> filter(x -> x^2 < 10, 1:5)
3-element Vector{Int64}:
 1
 2
 3
```

You can assign a lambda to a variable.
```julia
julia> addone = a -> a + 1
#1 (generic function with 1 method)
```

## Lambda Calculus
A branch of mathematics that uses functions to create a [universal model of computation](https://en.wikipedia.org/wiki/Lambda_calculus).

## Lazy evaluation

Lazy evaluation is a call-by-need evaluation mechanism that delays the evaluation of an expression until its value is needed. In functional languages, this allows for structures like infinite lists, which would not normally be available in an imperative language where the sequencing of commands is significant.

```julia
julia> struct myrand end

julia> Base.iterate(::myrand, state=nothing) = rand()

julia> first(myrand())
0.1381937787185451
```

## Monoid

An object with a function that "combines" that object with another of the same type.

One simple monoid is the addition of numbers:

```julia
julia> 1 + 1
2
```
In this case number is the object and `+` is the function.

An "identity" value must also exist that when combined with a value doesn't change it.

The additive identity value for addition is `0`, can also be obtained by `zero()`:
```julia
julia> 1 + zero(1)
1
```

It's also required that the grouping of operations will not affect the result (associativity):
```julia
julia> 1 + (2 + 3) == (1 + 2) + 3
true
```

List concatenation also forms a monoid:
```julia
julia> vcat([1, 2], [3, 4])
4-element Vector{Int64}:
 1
 2
 3
 4
```

The identity value is empty Int array `Int[]` (can be obtained by `empty()`)
```julia
julia> vcat([1, 2], empty([1,2]))
2-element Vector{Int64}:
 1
 2
```

If identity and compose functions are provided, functions themselves form a monoid:

```julia
# `foo` is a 1-argument function
foo∘identity ≍ identity∘foo ≍ foo
```

## Monad

A monad is an object with [`of`](#pointed-functor) and `chain` functions. `chain` is like [`map`](#functor) except it un-nests the resulting nested object.

```julia
julia> of(t::AbstractVector{T}, vals...) where T = T[vals...]

julia> chain(fn, v) = mapreduce(fn, vcat, v)

julia> chain(split, of([], "cat dot", "fish bird"))
4-element Vector{SubString{String}}:
 "cat"
 "dot"
 "fish"
 "bird"

# Contrast to map
julia> map(split, of([], "cat dot", "fish bird"))
2-element Vector{Vector{SubString{String}}}:
 ["cat", "dot"]
 ["fish", "bird"]
```

`of` is also known as `return` in other functional languages.
`chain` is also known as `flatmap` and `bind` in other languages.

## Comonad

An object that has `extract` and `extend` functions. `Ref` is a natural condidate:

```julia
julia> extract(x::Ref) = x[]

julia> extend(x::T, f) where T<:Ref = T(f(x[]))

julia> extract(Ref(1.0))
1.0
```

Extend runs a function on the comonad. The function should return the same type as the comonad.
```julia
julia> extend(Ref(1.0), x->3) # notice the type is preserved correctly
Base.RefValue{Float64}(3.0)
```

## Applicative Functor

An applicative functor is an object with an `ap` function. `ap` applies a function in the object to a value in another object of the same type.

```julia
julia> ap(fn, val) = fn(val)
ap (generic function with 1 method)

julia> ap(fns::Vector, vals) = map(ap, fns, vals)
ap (generic function with 2 methods)

# Example usage
julia> ap([a->a+1], [1])
1-element Vector{Int64}:
 2
```

This is useful if you have two objects and you want to apply a binary function to their contents.

```python
# Arrays that you want to combine
arg1 = [1, 3]
arg2 = [4, 5]

# combining function - must be curried for this to work
julia> add(x) = y -> x + y

julia> partially_applied_adds = map(add, arg1)
```

This gives you an array of functions that you can call `ap` on to get the result:

```julia
julia> ap(partially_applied_adds, arg2)
2-element Vector{Int64}:
 5
 8
```

## Morphism

A transformation function.

### Endomorphism

A function where the input type is the same as the output.

```julia
# String -> String
julia> uppercase("Aasd")
"AASD"

# Number -> Number
julia> decrement(num::T)::T = num-1
```

### Isomorphism

A pair of transformations between 2 types of objects that is structural in nature and no data is lost.

For example, 2D coordinates could be stored as an array `[2,3]` or object `{x: 2, y: 3}`.

```julia
julia> coords_to_pair(coords) = (coords.x, coords.y)

julia> pair_to_coords(pair) = (x=pair[1], y=pair[2])

julia> pair_to_coords(coords_to_pair((x=1, y=2)))
(x = 1, y = 2)
```

### Homomorphism

A homomorphism is just a structure preserving map. Weaker than isomorphism in that it doesn't require `bijectivity`.
In fact, a functor is just a homomorphism between categories as it preserves the original category's structure under the mapping.

Julia's `map` satisfy this (one-to-one/injective) unless the function is not "pure":
```julia
map(uppercase, ["oreos"]) == [uppercase("oreos")]
```

### Catamorphism

A `reduce_right` function that applies a function against an accumulator and each value of the array (from right-to-left) to reduce it to a single value.

```julia
julia> sum_by_reduce(vec) = foldr(+, vec)

julia> sum_by_reduce([1,2,3])
6
```

### Anamorphism

An `unfold` function. An `unfold` is the opposite of `fold` (`reduce`). It generates a list from a single value.

```julia
julia> function unfold(f, seed)
         function go(f, seed, acc)
           res = f(seed)
           return isnothing(res) ? acc : go(f, res[2], push!(acc, res[1]))
           end
         return go(f, seed, [])
         end
unfold (generic function with 1 method)

julia> countDown(n) = unfold(n->ifelse(n<=0, nothing, (n, n-1)), 3)
countDown (generic function with 1 method)

julia> countDown(3)
3-element Vector{Any}:
 3
 2
 1
```

### Hylomorphism

The combination of anamorphism and catamorphism.

### Paramorphism

A function just like `reduce_right`. However, there's a difference:

In paramorphism, your reducer's arguments are the current value, the reduction of all previous values, and the list of values that formed that reduction.

```julia

julia> function para(reducer, accumulator, elements)
         isempty(elements) && return accumulator
         head, tail... = elements
         return reducer(head, tail, para(reducer, accumulator, tail))
       end

julia> suffixes(lst) = para((x, xs, sxs) -> [[xs]; sxs], [], lst)

julia> suffixes([1,2,3,4,5])
5-element Vector{Any}:
 [2, 3, 4, 5]
 [3, 4, 5]
 [4, 5]
 [5]
 Int64[]
```

The third parameter in the reducer (in the above example, `[[xs]; sxs]`) is kind of like having a history of what got you to your current acc value.

### Apomorphism

it's the opposite of paramorphism, just as anamorphism is the opposite of catamorphism. Whereas with paramorphism, you combine with access to the accumulator and what has been accumulated, apomorphism lets you `unfold` with the potential to return early.

## Setoid

An object that has an `equals` function which can be used to compare other objects of the same type.

In `Julia`, just use `isequal(obj1, obj2)`

## Semigroup

An object that has a `concat` function that combines it with another object of the same type.

```julia
# column-major
julia> vcat([1], [2])
2-element Vector{Int64}:
 1
 2
```

## Foldable

An object that has a `reduce` function that applies a function against an accumulator and each element in the array (from left to right) to reduce it to a single value.

```python
julia> sum_by_reduce(vec) = reduce(+, vec)

julia> sum_by_reduce([1,2,3])
6
```

## Lens ##
A lens is a structure (often an object or function) that pairs a getter and a non-mutating setter for some other data
structure.

A popular implementation is [Accessors.jl](https://github.com/JuliaObjects/Accessors.jl)

```julia
julia> using Accessors

# this is immutable
julia> struct T
            a
            b
        end

julia> obj = T("AA", "BB");

julia> obj.a = "aa"
ERROR: setfield!: immutable struct of type T cannot be changed

julia> lens = @optic _.a
(@optic _.a)

julia> lens(obj)
"AA"

julia> set(obj, lens, 2)
T(2, "BB")

julia> obj # the object was not mutated, instead an updated copy was created
T("AA", "BB")

julia> modify(lowercase, obj, lens)
T("aa", "BB")
```

## Type Signatures

Often functions in JavaScript will include comments that indicate the types of their arguments and return values.

There's quite a bit of variance across the community but they often follow the following patterns:

```julia
# add :: int -> int -> int
myadd1 = (x::Int -> y::Int -> x + y)::Int

# increment :: int -> int
increment(x::Int)::Int = x + 1
```

Due to JIT, it's better to NOT over-specificy types, because it only reduces the genrality of your function
without any speed gain, also see [Multiple Dispatch](https://docs.julialang.org/en/v1/manual/methods/#Parametric-Methods) for
more idiomatic way to write parametric methods in Julia.

__Further reading__
* [Ramda's type signatures](https://github.com/ramda/ramda/wiki/Type-Signatures)
* [Mostly Adequate Guide](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch7.html#whats-your-type)
* [What is Hindley-Milner?](http://stackoverflow.com/a/399392/22425) on Stack Overflow

## Algebraic data type
A composite type made from putting other types together. Two common classes of algebraic types are [sum](#sum-type) and [product](#product-type).

### Sum type
A Sum type is the combination of two types together into another one. It is called sum because the number of possible values in the result type is the sum of the input types.

In Julia, it's almost the same as [Type Unions](https://docs.julialang.org/en/v1/manual/types/#Type-Unions), albeit some implementation details
and lack of pattern matching, but because Julia is a dynamic language (i.e. behavior always correct when you take a value out of a union typed
container), the line is blurred.

```python
julia> a = Union{Float64, Int}[1,2,3]
3-element Vector{Union{Float64, Int64}}:
 1
 2
 3

julia> typeof.(a)
3-element Vector{DataType}:
 Int64
 Int64
 Int64

julia> a = Union{Float64, Int}[1,2,3.0]
3-element Vector{Union{Float64, Int64}}:
 1
 2
 3.0

julia> typeof.(a)
3-element Vector{DataType}:
 Int64
 Int64
 Float64
```

Sum types are sometimes called union types, discriminated unions, or tagged unions.

See an implementation [SumTypes.jl](https://github.com/MasonProtter/SumTypes.jl).

### Product type

A **product** type combines types together in a way you're probably more familiar with:

```julia
point(p) = (x=p[1], y=p[2])
```
It's called a product because the total possible values of the data structure is the product of the different values. Many languages have a tuple type which is the simplest formulation of a product type.

See also [Set theory](https://en.wikipedia.org/wiki/Set_theory).

## Option
Option is a [sum type](#sum-type) with two cases often called `Some` and `None`.

Option is useful for composing functions that might not return a value.

```julia
help?> Some
search: Some something @something @showtime VersionNumber test_colorscheme DimensionMismatch sortperm

  Some{T}


  A wrapper type used in `Union{Some{T}, Nothing}` to distinguish between the absence of a value (`nothing`)
  and the presence of a `nothing` value (i.e. `Some(nothing)`).

  Use `something` to access the value wrapped by a Some object.
```
`Option` is also known as `Maybe`. `Some` is sometimes called `Just`. `None` is sometimes called `Nothing` (as in Julia).

## Function
A **function** `f :: A => B` is an expression - often called arrow or lambda expression - with **exactly one (immutable)** parameter of type `A` and **exactly one** return value of type `B`. That value depends entirely on the argument, making functions context-independant, or [referentially transparent](#referential-transparency). What is implied here is that a function must not produce any hidden [side effects](#side-effects) - a function is always [pure](#purity), by definition. These properties make functions pleasant to work with: they are entirely deterministic and therefore predictable. Functions enable working with code as data, abstracting over behaviour:

```julia
julia> times2(n::Number) = (n * 2)::Number
times2 (generic function with 1 method)

julia> map(times2, [1, 2, 3])
3-element Vector{Int64}:
 2
 4
 6
```
