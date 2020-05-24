# Do syntax

I'm considering a few changes to do syntax, though a few might be going too far. They do seem to work well together at any rate:

## Do case

```haskell
do
    case Just x <- foo of Nothing => ...
    -- x is in scope here
    ...
    

do
    case <- foo of
        ...
```

The first example behaves like Idris's syntax: the default layout defines the "good" path, while other arms after the `of` define the "bad" path. The second example behaves like the `>>= \case` idiom.

# Guards

I want to parse guards the same as Haskell, except with a preceding `if` before the pipe. This makes guards and multiway if parse more similar, though the real reason I want to do this is I can use `|` within patterns to define multi-argument or-patterns. For instance

```haskell
notEq : Bool -> Bool -> Bool
notEq True, False | False, True = True
notEq _, _ = False
```
With Haskell's default syntax, using `|` for both or-patterns and guards causes a conflict.

I also want to parse view patterns with `<-`, and allow `if |` in patterns as well, to embed pattern guards inside nested patterns. I don't *really* want this, but changing the grammar like this makes all these features fit together in a nice way.

One downside, compare the two different ways of doing view patterns:

```haskell
example (expr => 'x if | 'y <- x) = ...
example (('x if | 'y <- x) <- foo) = ...
```
With using `<-` we need an extra parentheses, though it simplifies parsing and makes the syntax more consistent, as we have `{patt} <- {expr}` consistent wherever it occurs.

One concern about using `if` as the guard herald comparing:

```haskell
foo Just a, b = if
    | a <= b    => a
foo a, b = b

foo Just a, b if
    | a <= b = a
foo a, b = b
```

These have very different meaning, but resemble each other quite closely: the first is a multiway if (due to the first `=` and the `=>` in the guard), and the second is a guard on the function itself. Thus, it requires making two errors to mistakenly write when the other is intended, which might be acceptable, but it makes me nervous. Using `where` as the herald would remove the ambiguity.
