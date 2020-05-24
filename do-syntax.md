# Do syntax

I'm considering a few changes to do syntax, though a few might be going too far. They do seem to work well together at any rate:

### Do case

```haskell
do
    case Just x <- foo of
        Nothing => ...
    -- x is in scope here
    ...
```

The first example behaves like Idris's syntax: the default layout defines the "good" path, while other arms after the `of` define the "bad" path.

I also considered `case <- foo of` to behave like the `>>= \case` idiom, but I've decided against it, as the !-sigil syntax allows you to already write it as `case !(foo) of`. 

### Type signatures

I'm considering disallowing `:` as type annotations in expressions, similar to in Idris, in which case I could write

```haskell
do
    'x : IO Int
    'x <- pure (pure 2)
```

to provide type signatures in do-blocks, similar to as in let blocks. I also considered allowing `=` without `let` in do-blocks, in which case the type signatures could apply there as well. `let` would only be required for recursive bindings.

This does make the syntax for records and blocks very similar though. 1-records and blocks with one element can be distinguished due to requiring 1-records to have a comma (similar to 1-tuples); similarly blocks with type signatures or assignments would require at least 3 elements (the signature, the binding, and another line since the last line can't end with a binding (at which case they can also be disambiguated by using semicolons rather than commas). However, they still do look very similar. Consider

```haskell
{ 'x : Int , } -- record type
{ 'x = 2 , }   -- record value
{ 'x : Int ; 'x = 2 ; pure x } -- block
```

# Declarations

### Guards and pattern guards

I plan to require `where` to precede the `|` to use guards. This has two benefits:

1. `|` can also be used for or-patterns, since the separating `where` separates the or-patterns from the pattern guards, as in
    
    ```haskell
    'notEq True, False | False, True = True
    'notEq _ _ = False
    ```
    
    This can be parsed unambiguously, since we know the `|` doesn't indicate a guard due to the lack of `where`. Note that `,` binds more tightly than `|`, similar to how products bind more tightly than sums.
    
2. Pattern guards can be nested in patterns, as in `'foo ('x where | x == True)`.

I considered using `if` instead of `where`, which flows together with multiway if pretty well. However, it makes it look too similar, as in:

```haskell
'isZero 'x = if
    | x == 0 => True
'isZero _ = False

'isZero 'x if
    | x == 0 = True
'isZero _ = False
```

I remember being confused that the first `=` gets dropped when using guards when learning Haskell, and I've seen other people make the same mistake. I think this is a hard mistake to make accidentally with this syntax (you need to both mistakenly add the `=`, but also use `=>` in the multiway if for it to parse correctly). However, I still feel like making them visually more distinct is more preferable, so my preference is to use `where` for guards.

One other note, I could drop the `|` entirely in guards, by using `where` in its place. This looks better in composite pattern guards, as there will never be multiple arms, but in normal pattern guards looks more wordy. I could make it not required in composite pattern guards, though I don't like the inconsistency.

One comment on making the pattern sublanguage richer, I don't really feel super motivated to improve it. On some level I like the idea, but it feels like something I would almost never use, at the cost of extra syntax and complexity. Every proposal I've seen for changes in Haskell I think "that would be cool, but also I don't really want it". I feel like the consistency in the syntax I'm proposing for Syrup *just* manages to achieve an acceptable power:weight ratio, as the consistency makes it's meaning easy enough to understand once you understand guards. But in the long run I feel like proposing a richer pattern language needs quite a bit stronger theoretical basis.
