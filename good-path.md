
I plan to allow the following sugar

```
do                               do
   Just 'x <- foo of                 case !(foo) of
       Nothing => badPath                Just 'x => goodPath
   goodPath                              Nothing => badPath
```

I may also allowas sugar for the following:

```
do                               do
    Just 'x = foo of                 case foo of
        Nothing => badPath               Just 'x => goodPath
    goodPath                             Nothing => badPath
```

Though that rules out `=` in expressions. Idris uses `=` as a (built-in?) type constructor for equality proofs for comparison.

The purpose of this syntax is to separate put the bad path up front, so it can be dealt with quickly, while avoiding the need to continuously indent the good path. This is useful for large do-blocks with lots of case statements, where much of the logic consists of pattern matching and dealing with occasional error cases, while trying to focus most of our attention on a single branch.

# Side-benefit

As I was considering letting `=` be used without `let` in do blocks this has an unexpected side-benefit: we can write code like `{ x = x + 1 ; ... }`, and using a similar desugaring to `<-` gives an obvious meaning to the assignment as being non-recursive. Since occasionally you want non-recursive let, that coming naturally out of this syntax is nice to have. `Let` blocks in do-statements would still be allowed (and needed) with semantics for recursive bindings.

As a special case, (syntactic) function definitions could be recursive, so `{ f x = f x }` would work, while `{f = \x -> f x}` would not. I'm leaning towards that being disabled without an annotation, if not disallowing syntactic function definitions outright, as desugaring into a case statement doesn't actually work (so the desugaring is more magic). Cases like the following seem problematic:

```
do
    'hasJust : Maybe 'a -> Bool
    'hasJust (Just _) = True
    'hasJust Nothing = False
```

Viewed with the same logic as assigning other values, this would actually create a partial function and then immediately shadow it with another partial function, so I feel if I allow this syntax it would give a custom error message for syntactic function definitions, informing the user that this isn't allowed, and to use a let.

## Ambiguities with = and : in do-blocks

Though not technically ambiguous, records end up looking extremely similar to blocks. We can distinguish them based on the need for commas vs. semicolons: there are no blocks with 0 statements, and records with 1 type have a leading comma. However, the similarity is enough for me to consider a different delimiter for blocks. `[| |]` is my favorite choice, with ViM conceals to make it a single character, though I would hate to miss out on punning on curly braces for imperative programmers who are seeing the syntax for the first time. Without ViM conceals it's also not great, but it is workable. I'm likely to change from curly braces to this syntax to remove needing to rely on "technically not ambiguous" as a defense.

Another ambiguity is with `:`, which could be providing the type of the following assignment, or the type of the preceding expression. Idris disallows `:` in expressions, which resolves the ambiguity. The fact that I require newly introduced variables to have a `'` sigil also removes the ambiguity even if I do disallow it, so I could allow the (technically) unambiguous syntax if I really wanted.

# Comparison of other syntaxes

I believe Idris has a syntax like

```
do
    Just x <- foo | Nothing => ...1
    ...2
```

There is a discussion to implement something similar in Haskell [https://github.com/Ericson2314/ghc-proposals/blob/case-bind/proposals/0000-case-bind.rst]

Though it doesn't address the actual issue of visually distinguishing between the good/bad path, since this proposal does discuss non-recursive let I actually prefer

```haskell
let #![NonRecursiveLet]
    x = ...
in
```

## A most vexing parse

How do we parse the following: `case do x <- y of`. The meaning is obvious for a non-forgiving parser; a case statement requires an expression between `case_of`, and `do x <- y` isn't a valid expression, because the last line of a do can't be a binding (either with `<-`, or because we now allow it, `=`). However, a forgiving parser likely wants to recognize those cases as syntactically valid to improve error messages, at which point the parse becomes ambiguous. Except it's not truely ambiguous, since the "successful" parse is only successful from the point of including invalid parses in the AST.

Using the longest parse fixes this, as then the do block will consume as much input as possible, but representing that in my AST could be awkward, as I want to have the guarantee that no two distinct ASTs will serialize to the same string (this isn't an issue if I don't have that requirement, but I want to keep my parser and printer in sync with this constraint.

I'd like to avoid simply parsing it whichever way, and then fixing the ambiguity after the fact, but I'm not sure how to go about that. The basic rule would be that a case statement can't contain a do statement which ends with a binding, unless the `of` is less-indented than the `do`-blocks alignment. Encoding that in the parser seems super awkward.

Idris's syntax also has this issue when combined with guards, for instance:

```
case () of
    _ | x => do x <- x | x => x
```

That could also be fixed with layout, by making guards have to be aligned, and the expressions they are guarding to be further indented. That seems like an easier thing to implement though. Hmm... The GHC proposal does avoid the issue. For the time being I'm not making a choice.

# The meat of the thing

This doesn't have anything specific to do with do notation, or even monads. We could have

```
case Just 'x = foo of
    Nothing => badPath
-- the goodPath doesnt have any indentation sensitivity, except for being less indented than the bad
goodPath
```

And recover the monadic version with `!` and a custom rule for layout to allow for that indentation. Analysis of the syntax should be done with that in mind.
