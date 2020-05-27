
I plan to allow the following

```
do
   Just 'x <- foo of
       Nothing => badPath
   goodPath
```

as sugar for the following:

```
do
    case !(foo) of
        Just 'x => goodPath
        Nothing => badPath
```

I may also allow `{patt} = {expr}` as statements in do-blocks, with a similar rule, except for the lack of the `!`, though that rules out `=` in expressions. Idris uses `=` as a (built-in?) type constructor for equality proofs for comparison.

The purpose of this syntax is to separate put the bad path up front, so it can be dealt with quickly, while avoiding the need to continuously indent the good path. This is useful for large do-blocks with lots of case statements, where most of the logic consists of error handling, but which also has a single thread of logic throughout which we hope to focus most of our attention on. Thus, all the error handling code becomes further indented, while the good path does not.

# Side-benefit

As I was considering letting `=` be used without `let` in do blocks this has an unexpected side-benefit: we can write code like `{ x = x + 1 ; ... }`, and using a similar desugaring to `<-` gives an obvious meaning to the assignment as being non-recursive. `Let` blocks in do-statements would still be allowed (and needed) with semantics for recursive bindings. There's occasionally need for non-recursive let, and even without monadic code `{ x = x + 1 ; x }` would be desugared as a pure expression, due to lacking any `<-` and only having a single terminating expression. So while this syntax is not intended to provide non-recursive let, it gives an unambiguous meaning for what `x = x + 1` should mean in a statement, without needing to introduce even more syntax for non-recursive bindings. It's meaning falls naturally from the meaning for desugaring bindings to case statements.

As a special case, (syntactic) function definitions could be recursive, so `{ f x = f x }` would work, while `{f = \x -> f x}` would not. I'm leaning towards that being disabled without an annotation, if not disallowing syntactic function definitions outright, as desugaring into a case statement doesn't actually work (so the desugaring is more magic). Cases like the following seem problematic:

```
do
    hasJust : Maybe a -> Bool
    hasJust (Just _) = True
    hasJust Nothing = False
```

Viewed with the same logic as assigning other values, this would actually create a partial function and then immediately shadow it with another partial function, so I feel if I allow this syntax it would give a custom error message for syntactic function definitions, informing the user that this isn't allowed, and to use a let.

Non-syntactic function definitions `f = \x -> ...` would still be allowed, as there is no ambiguity, though the fact that they are non-recursive would be odd. However, if someone writes a recursive function and it's not already bound they'll receive an error, and if it is already bound (and thus is shadowing) they will receive a warning, so maybe that's okay.

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

Idris's syntax presents a bit of an ambiguity with guards, as you can write `f x | x = do ... | ...`, and it might not be clear if the pipe is part of the guard or case statement. For example, considering:



I'm not sure that this is technically ambiguous, since the second pipe follows a binding, and so can't be a guard, since do blocks can't end with a binding, and so it can't be over. However, I feel like a forgiving parser should *syntactically* allow do blocks which end on a binding, to improve parse errors when user's make a mistake. Thus, even if not ambiguous it's awkward to parse correctly, while also parse the error "correctly", and may also be hard for a human to parse.

Making guards layout sensitive would fully remove the ambiguity, but I don't know that I want to do that. I do prefer that solution aesthetically, but I prefer using `of` for the consistency. It removes the potential issues with guards, and it's also more consistent in that `of` precedes the arms of a case statement, rather than pipes sometimes being a guard and sometimes being a case statement, and sometimes a case statement not needing them.

Note, case statements also have the issue with forgiving parsers presenting an "ambiguity". Consider `case do x <- y of`. I'm more certain that this isn't actually ambiguous, but it's still tricky to not introduce ambiguity in the forgiving parser. 

There is a discussion to implement something similar in Haskell [https://github.com/Ericson2314/ghc-proposals/blob/case-bind/proposals/0000-case-bind.rst]

Though it doesn't address the actual issue of visually distinguishing between the good/bad path, since this proposal does discuss non-recursive let I actually prefer

```haskell
let #![NonRecursiveLet]
    x = ...
in
```


## A most vexing parse

How do we parse the following: `case do x <- y of`. The meaning is obvious for a non-forgiving parser; a case statement requires an expression between `case_of`, and `do x <- y` isn't a valid expression, because the last line of a do can't be a binding (either with `<-`, or because we now allow it, `=`). However, a forgiving parser likely wants to recognize those cases as syntactically valid to improve error messages, at which point the parse becomes ambiguous. Except it's not truely ambiguous, since the "successful" parse is only successful from the point of including invalid parses in the AST.

I'd like to avoid simply parsing it whichever way, and then fixing the ambiguity after the fact, but I'm not sure how to go about that. The basic rule would be that a case statement can't contain a do statement which ends with a binding, unless the `of` is less-indented than the `do`-blocks alignment. Encoding that in the parser seems super awkward.

Idris's syntax also has this issue when combined with guards, for instance:

```
case () of
    | x => do x <- x | x => x
```

That could also be fixed with layout, by making guards have to be aligned, and the expressions they are guarding to be further indented. That seems like an easier thing to implement though. Hmm...
