
Some of this is repetition, I just want all my ideas on patterns in one place:

## Pattern families

Syntax subject to bikeshedding, but something like: `‹F a b› 'c 'd`, as syntax for a pattern family that takes two arguments, and binds two variables. This makes the separation between arguments and pattern variables syntactically distinct (which Haskell's proposal doesn't), and also makes the fact that non-O(1) code may be being executed in the pattern match syntactically distinct. Pattern synonyms would simply be pattern families with 0 arguments.

## With-patterns

Allow syntax like:

```
'f 'x 'y with g x y of
    G 'z = ...
```

Which would behave like the following in Haskell:

```
f x y = case g x y of
    G z -> ...
```

Except that if the case is partial it falls through to the next pattern match in `f`'s definition. PatternGuards in Haskell can be used to accomplish this particular case, but only if the `g x y` pattern match has a single branch (multiple branches would require duplicating the PatternGuard in multiple guards, leading to repetition of code and potentially duplicating the call to `g`. This also allows ViewPatterns to be done like:

```
'f 'x 'y with g x of
    F 'z => ...
```

instead of `'f (g => 'z) 'y`, though with different operational characteristics (if patterns are matched left to right).

## Guards

Guards support optional `then` or `else` clauses like:

```
f x y if
    | g x = ...
    , h x else = ... 
    , h x then | i x = ...
               | j x = ...
    , otherwise then = ...
```

This would require `if |`, `then |`, and `else |` to be layout heralds, with the layout starting at the `|`, and each line needing to begin with a comma or pipe. I don't love using layout for this, but I don't hate it either. It does present a need for a special case if we want to be able to write

```
f x y if
    | g x
    , h x
    = body
```

Though requiring a `then = ` for each body that evaluates on a successful condition would make that less likely to be written, and would remove some redundency that I don't like, as in statements like

```
f x y if
    | g x then = body
    , h x = body
    | i x then = body
    | h x = body
```

I could make a `then` only be allowed when it's followed by another guard, but I do want `else =` to be allowed, so making `then =` mandatory makes it consistent with `else =`, and removes the ambiguity of allowing both `cond = body` and `cond then = body`.

A line like `cond else |` would evaluate the right guard if `cond` fails, or evaluate the following comma if it succeeds, while a line like `cond then |` would do the opposite. Pattern guards could also be supported, though I'm not sure if they should use `let`, so they would look more like Rust's if-let, which I think is an easier syntax to understand that pattern guards are.

With-patterns could be nested in guards as well, like

```
'f 'x if
    | g x then =
    | otherwise with h x of
        Just 'y = 
        Nothing y =
```

If a `with` is followed by a guard preceded by a comma and the `with` fails to match we would backtrack and try the comma. So in the following two examples

```
'f 'x if
    | h x
    , with g x of
        Just 'y = ...
    , otherwise = ...
```

The otherwise branch will only happen if `h x` succeeds, but the `with` fails. If we change it to:

```
'f 'x if
    | h x
    , with g x of
        Just 'y = ...
    | otherwise = ...
```

The otherwise branch will be followed if either `h x` or the `with` block fail. Thus, `with` behaves like `then` in a guard.

One issue I'm not sure of with `with` in guards is what to do if the next guard is preceded by a comma. If the `with` fails to match we could backtrack and try the comma

### Pattern guards

Pattern guards would also support `then` and `else`, with them defining which direction we take if the guard succeeds or fail, similar to regular guards. I'm not sure if I want to have a `with` version of pattern guards, which would go down on success or right to try other patterns on failure. It could look something like this:

```
'f 'x if
    | Just 'y <- g x with
        Nothing = ..
    , h y then = ...
```

I might want similar syntax for bindings in do-notation and let-notation, for both `else` and `with`. In this form `else` is really just a `with` where the pattern we're matching against is `_`.

## Ticks

I'm planning on using prefix ticks to be used when binding new variables in patterns. This technically leaves unticked variables to infer equality constraints if the variable in scope isn't a constructor, which feels almost valid (as comparing against a constructor is already a form of equality constraint in the first place). There are two reasons why I don't want to do that:

1. An equality comparison can result in arbitrary computation, which is now syntactically hidden, unlike if we have something like `f <Equals x>`.

2. "Bad" equality instances (like with NaN) could give weird behaviour. What if we write:

```
data Number = NaN | Just Double

let 'NaN = NaN
    'f : Number -> Bool
    'f NaN = True
    'f _ = False
in f NaN
```

Because the `NaN` constructor is shadowed, this would result in False, as shadowing of the variable will change the pattern match from a structural pattern match to an equality constraint. I feel like the appropriate solution is to force user's to be explicit with a pattern family, guard, with pattern, or view pattern, if they want an equality constraint.
