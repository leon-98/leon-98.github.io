layout	title
home
Notes on Programming Paradigms
Levels of Abstraction
Understand the motivation for different programming paradigms: to abstract machine operation into human understandable and composable programs
Understand the difference between syntax, the textual symbols and grammatical rules of a program, and semantics, the meaning of what is computed
Understand that there are different models of computation upon which different programming languages are based, including machine models such as Turing Machines and von Neumann architecture, and the Lambda Calculus based on mathematical functions.
Compare the dominant paradigms of Functional and Objected-Oriented Programming in terms of how they abstract data and behaviour.
Introduction to JavaScript
Understand and use basic JavaScript coding concepts and features
Understand the difference between mutable and immutable (const) variables
Explain the relationship between javascript functions and objects
Understand that the scope of variables is limited to delineated code blocks and within functions
Understand that a closure captures variables referenced within its scope
Create and apply anonymous functions to fluent style code
Compare arrow functions and regular function syntax
Explain JavaScript’s prototype mechanism for creating classes from functions
Create ES6 style classes with constructors and getters
Compare object oriented polymorphism to dependency injection through functions
Functional JavaScript
Create programs in JavaScript in a functional style
Understand the definitions of Function Purity and Referential Transparency
Explain the role of pure functional programming style in managing side effects
See how pure functions can be used to model sophisticated computation
Introduction to TypeScript
Create programs in TypeScript using types to ensure correctness at compile time.
Explain how TypeScript features like Interfaces, Union Types and Optional Properties allow us to model types in common JavaScript coding patterns with precision.
Explain how Generics allow us to create type safe but general and reusable code.
Compare and contrast strongly, dynamically and gradually typed languages.
Describe how compilers that support type inference assist us in writing type safe code.
Lazy Evaluation
Understand how functions can be used to delay evaluation of code until the result is actually required
Understand how this lazy evaluation allows for the definition of infinite sequences
Code and use lazily evaluated infinite number sequences
Functional Reactive Programming
Understand that the Functional Reactive Programming Observable construct is just another container of elements, but whose "push-based" architecture allows them to be used to capture asynchronous behaviour
Understand that Observables provide the benefits of functional programming: composability, reusability.
See that Observables structure complex stateful programs in a more linear and understandable way that maps more easily to the underlying state machine.
Use Observables to create simple UI programs in-place of asynchronous event handling.
Observable Asteroids
A fully worked example of FRP using rx.js Observables to create an in-browser version of a classic arcade game.

Combining Higher Order Functions
Understand that Higher-Order Functions are those which take other functions as input parameters or which return functions
Understand that curried functions support partial application and therefore creation of functions that are partially specified for reuse scenarios.
Understand that a Combinator is a higher-order function that uses only function application and earlier defined combinators to define a result from its arguments
Use simple Combinator functions to manipulate and compose other functions
Lambda Calculus
Understand that the lambda calculus provides a complete model of computation
Relate the lambda calculus to functional programming
Apply conversion and reduction rules to simplify lambda expressions
From JavaScript to Haskell (via PureScript)
Compare a lambda-calculus inspired Haskell-like language (PureScript) with the functional programming concepts explored earlier in JavaScript
Understand how tail call optimisation is applied in languages which support it
Creating and Running Haskell Programs
Use the GHCi REPL to test Haskell programs and expressions
Compare the syntax of Haskell programs to Functional-style code in JavaScript
Understand that by default Haskell uses a lazy evaluation strategy
Create and use Haskell lists and tuples
Create Haskell functions using pattern-matching, guards, and local definitions using where and let clauses
Data Types and Type Classes
Define data structures using Haskell's Algebraic Data Types and use pattern matching to define functions that handle each of the possible instances
Use the alternate record syntax to define data structures with named fields
Understand that Haskell type classes are similar to TypeScript interfaces in providing a definition for the set of functions that must be available for instances of those type classes and that typeclasses can extend upon one another to create rich hierarchies
Understand that the Maybe type provides an elegant way to handle partial functions
Functor and Applicative
Understand how eta-conversion, operator sectioning and compose, together provide the ability to transform code to achieve a composable point free form and use this technique to refactor code.
Understand that in Haskell the ability to map over container structures is generalised into the Functor typeclass, such that any type that is an instance of Functor has the fmap or (<$>) operation.
Understand that the Applicative Typeclass extends Functor such that containers of functions may be applied (using the (<*>) operator) to containers of values.
Understand that Functor and Applicative allow powerful composable types through exploring a simple applicative functor for parsing.
Foldable and Traversible
Understand that the "reduce" function we met for arrays and other data structures in JavaScript is referred to as "folding" in Haskell and there are two variants foldl and foldr for left and right folds respectively
Understand that the Monoid typeclass for things that have a predefined rule for aggregation (concatenation), making containers of Monoid values trivial to fold
Understand that Foldable generalises containers that may be folded (or reduced) into values
Understand that Traversable generalises containers over which we can traverse applying a function with an Applicative effect
Monad
Understand that Monad extends Functor and Applicative to provide a bind (>>=) operation which allows us to sequence effectful operations such that their effects are flattened or joined into a single effect.
Understand the operation of the monadic bind and join functions in the Maybe, IO, List and Function instances of Monad.
Be able to refactor monadic binds using do notation.
Loop with Monadic effects.
Parser Combinators
Understand that a parser is a program which extracts information from a structured text file
Apply what we have learned about Haskell typeclasses and other functional programming concepts to create solutions to real-world problems
In particular, we learn to use Parser combinators and see how they are put together
