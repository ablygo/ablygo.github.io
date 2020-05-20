## Syntax that maybe I will use for my language

This is an overview of syntax that I like, and a non-exhaustive collection of my thoughts on each. Some of it is
subject to being able to get semantics right to unify various concepts (such as having a single syntax for records and
tuples being conditional on having a single type that unifies both concepts).

### Tuple syntax

I'm considering using `[a,b]` for tuple and list syntax, though I worry about type inference. I do need a 1-tuple, and `(a,)`
would cause me to lose tuple sections. Granted, I don't love sections. But I also would like a separate syntax for tuple types,
and even if I allow `(a,)` for a 1-tuple type list syntax for expressions 

Idris does use round brackets for both type and value types, so that still is an option, but I also worry about ambiguity.
Though I could also use `'(a, b)` for types and `(a, b)` for values.

Subject to such a thing actually working, I would be interested in tuple syntax being dependent by default, so you could write
`'(a : Int, f a)` for a dependent pair, `'(0 a : Type, f a)` for existential types, and `'(a : List Bool, 0 _ : null a)` for
liquid types.

Semantically I don't know how well that would work, and operationally I'm not sure how code generation would work
for tuples which have some of their fields erased. For instance, can we generate code for `snd : (a, b) -> b` if `a` has
multiplicity 0 that doesn't still represent a pair as two pointers?

But being able to have existentials, tuples, and liquid types, all without needing separate syntax for each, would be nice,
as I don't want the grammar to be too crowded, and each are potentially useful concepts to explore. They would also help with
prisms for GADTs. For instance, if we want to create a GADT like:

```haskell
data Exists : Type where
    Exists : a -> Exists
_Exists : Prism' Exists (0 a : Type, a)

data Exists' : (Type -> Constraint) -> Type where
    Exists' : c a => a -> a -> Exists' c
_Exists : Prism' (Exists' c) (0 a : Type, c a, a, a)
```

I don't know how I like the second example, as the dictionary should really be implicit. I would prefer the type to be
`exists c. c a => (a, a)`. But it does make it very clear how such a type is represented, and avoids needing to introduce more
syntax. Unicode exists could be used as sugar if I really wanted to, maybe with `∃(a,a)` as sugar for the value level pair.

If `∃` is also usable at the type level that would also make `∃a` confusing, as I feel like it should be the start of a type,
but it also looks like the value level 1-tuple. It's not truly ambiguous: we would have `∃(a,) : ∃a. '(a,)`. But I feel like it
would be confusing. I would rather the value level construction use a different symbol, though I'm not sure what reads well.
I could use `some (a,) : ∃a. '(a,)`? I dunno. I don't like it (also I don't actually plan on requiring unicode). Or I could do
`(a,) : ∃a. '(a,)`, though I worry that having every tuple.

Though I'm not sure if the fact that the sugar looks like a pair, but the desugared form is actually a 4-tuple, is confusing.

One thing I dislike about the unsugared syntax is parsing the infix operators. `(a : Int, b : F a, G a b)` isn't too hard to
understand, given that the rules for pairs are pretty well established, but I find it hard to parse at a glance. I would really
like the syntax for dependent sums mirroring dependent products, so I could do:

```haskell
dependent tuple/sum: (a : Int) ** (b : F a) ** G a b
dependent product:   (a : Int) -> (b : F a) -> G a b
```

However, unlike `->` being right associative, `**` being sugar for dependent tuples is deeply magical. We wouldn't actually be
parsing it as a series of pairs, but as a single flat pair. Not only that, but we would still need syntax for 1-tuples and
0-tuples. So while it parses nicer at a glance, those reasons make me *really* dislike the idea.

To summarize, if I were to do all of that I would have `(a : Type, b : F a, G a b)` being a flat dependent pair, `exists a.
Show a => (a, a)` for a tuple with some erased/constraint fields`, `∃(a,a)` for the value,  and
`(a : Int) ** (b : F a) ** G a b`
for sugar for the dependent pair again. That seems like possibly more syntax than I'm happy with for all slightly different
takes on the same concept.

I absolutely want tuples to support dependent fields by default, and I like the idea of having sugar for the special case of
existential types. For values, I'm leaning towards simply having to write the true tuple, so I'm settling on the following
syntax

```haskell
(Int, Show, 1, 2) : exists c. (Show c) => '(c, c)
-- desugars to
(Int, Show, 1, 2) : '(0 c : Type, Show c, c, c)
```

The value construction is more explicit than I'd like, but it does make things somewhat more clear. I'm still not sure what I
want. The type level construction also has issues now that I think about it. I would expect `exists c. Show c => c` to be a
1-tuple and not in a tuple at the same time. People would likely forget to write `exists c. Show c => (c,)` all the time, as
everything after an exists is always a tuple, so the meaning is the same. Or both could have the same meaning, but I don't
like that either...

Also note, the tick on the type could be dropped when a field is given a name, as in that case it's not ambiguous, but keeping
things consistent might be better.

### Record syntax

I want to have anonymous records, and mostly plan on the obvious syntax `{x = 1, y = 2}` and `'{x : Int, y : Double}`, though
I have a few thoughts on the matter. I have seen semantics for row types that allow duplicate fields, and if we have that we
can unify tuples and rows, by having `{x, y}` be sugar for `{"" = x, "" = y}`. Essentially, have a sentinel value,
representing a positional field. This would solve the issue with having 1-tuples, as `(x)` and `{x}` become distinct, but it
causes other issues.

1. Is `{x}` a pun (e.g. sugar for `{x = x}`, or a positional field? Do we want to disallow puns? They are helpful.
2. I also want to use `{x}` as a scope for !-idioms, so I can write `{putLine !getLine}` and have it be sugar for

```haskell
do
    tmp <- getLine
    putLine tmp
```

Now it's triply ambiguous. If there's a semicolon or comma we can disambiguate (blocks use semicolon, records use comma), and
0-tuples are also unambiguous (empty blocks aren't valid), but I still don't like the issue.

Another issue is the question "should tuples and records be unified". I don't know how well the duplicate field rows work in
practice, and if I want tuples to allow dependent types then I also need dependent rows to work, and I don't know if they are
well-defined. Field names in records are typically expected to be unordered, but dependency can cause an order to be required.

Also, the syntax for tuples using round brackets and records using curly brackets mirrors math. Value tuples are written as
flat cartesian products, and are ordered, and records use curly brackets like sets, which are unordered.

So I already don't know whether the semantics should be unified, and unifying the syntax also causes problems with both puns
and !-blocks, so for the moment I'm keeping them separate. If the former question was answered I might consider unifying the
syntax, but simply choosing something different than curly brackets, but there's still the issue of puns. I could use a sigil?

## !-idioms

I want to be able to write `{putLine !getLine}` as I mentioned above, but even ignoring the issue of clashes with other
syntaxes it introduces a few problems. I intend it to behave like do-blocks, so semicolons and binds would also work. Still
what if someone writes `if x then putLine !getLine else pure ()`. One solution to this problem is to have ad-hoc rules where
the scope behaves differently, so it wouldn't lift over `if`, a binding (whether in a lambda or case) or any value declaration,
though I frequently see objections to this approach, as it is very ad-hoc, and I agree. My solution is to have the same rules,
but instead generate a warning, showing the user the desugaring, and presenting the option of disabling the warning locally for
that declaration if they wanted to. So they could write

```haskell
#[allow_hoist]
f x = do 
    if x then putLine !getLine else pure ()
```
Which would make it desugar according to that rule, rather than doing so automatically. (When I say desugar according to that
rule it would still use the naive algorithm of hoist to the nearest line in the do/!-block, not hoist to the nearest line or
binder or equals or then/else. I would not want the meaning to change depending on whether the warning is disabled, only whether
the warning is ignored).

I think the strength of this approach
isn't just that it allows people to use it how they like, but specifically because every new programmar is likely to make this
mistake, and then see an error messaging seeing how it might not be what they want. This gives them the opportunity to use
their knowledge from other languages to adapt to monadic code more quickly, but also to learn how it differs more quickly as
well.

I think an issue with using `=<<` or do-blocks explicitly is that the extra noise you introduce to do something as simple as
`f !a b !c d` makes understanding a simple concept much more difficult, and also makes intent harder to discern even when you
do discern the idiom (if we are binding variable names explicitly are they being used more than once?). By having an error
message that's easy to encounter they get the best of both worlds: legibility, and a learning moment when the compiler explains
where their intuition breaks.

One question is abstraction. If I define my own `ifThenElse` function will I lose the warning. My solution to that is to allow
an annotation (subject to bike shedding)
```haskell
#[dontHoist(t,f)]
ifThenElse : Bool -> (t : a) -> (f : a) -> a
ifThenElse b t f = ...
```
So that any function can result in that error message. This isn't perfect; I can easily imagine monadic functions which
shouldn't be hoisted over depending on the monad in question, but the idea isn't for it to abstract over every form of control
flow possible. Rather it's just to ensure that new users are likely to learn the precise meaning of the desugaring more
quickly, so that they develop an intuition for how it actually desugars, at which point the warning isn't really necessary
except as a double safety check.

One other note, it would be possible syntactically to write `{\x => !x}`, as the inner `x` gets bound before it actually
is passed into the function. You could interpret it as 

```haskell
do
    y <- x
    \x => y
```

While that's the only meaning I can give to such a statement, I feel like even with the "don't hoist over bindings" warning
disabled it's better to reject that with a specific error message to that case. Otherwise, if the user wrote `{\x => !y}`,
that would be allowed (modulo the warning message) as

```haskell
do
    y' <- y
    \x => y'
```

I frequently have also wanted syntax for idiom brackets, though we can technically do that like `{pure (f !x !y)}`. However,
I think that extra noise detracts from the important information, so I'm proposing doing the same with `{{f !x !y}}`. Since
`{x}` => `x`, there isn't any unique meaning to ascribe to nesting extra parentheses, so using it to mean something different
seems acceptable to me. Thus we could write `{{x}}` for `pure x`, `{{f !x}}` for `fmap f x`, and so on.

Another advantage of
having separate syntax for pure-terminated blocks is we no longer need the hack for ApplicativeDo needing to recognize
`pure (...)` and `pure $ ...`, as the pure is implicit. Though possibly it would still be necessary if someone wrote out a
do-block explicitly; I'm not sure if that should be read intentionally or not.

I'm aware that idiom brackets are a bit different (not requiring the bangs), but I feel like explicit is better than implicit
in this case, as it allows a mix of effectful and pure arguments nicely, and intuition about the `{putLine !getLine}`
transfers to the pure-terminated case.

One last point behind this syntax is it's not intended to be used to compress as much information into a single line as
possible; rather it's to be able to express code in whichever way feels more readable for the idea being expressed. I have
similar feelings about lambda sugar: it isn't strictly necessary, and I think it's often more preferable to give a function
a name (and explicit type), but sometimes it is more readable to have a shorthand for the common form of "bind a variable and
use it immediately".

Last comment, if I do use curly braces for this and records then there's the question of how to interpret `{x}`. Since I'm
likely to end up needing `(x,)` for 1-tuples I might do the same for 1-records; they're likely not going to be used frequently
anyway. But it is a little bit ugly. You could infer that a record is intended if they write `{x = y}`, but I've also seen
desire for `=` in do-blocks. Then you could also write `{x : IO Int}`, but that could also be ambiguous (record type or type
signature in a do-block?). I've also also seen desire for `x : IO Int` as being a type signature for a following line in a
do-block, though I'm considering disallowing `:` in expression contexts, so that removes one source of ambiguity; it would
have to be a type signature or record type, but couldn't be "execute x, which has type `IO Int`.

1. What about `{x = y}`. I've seen desire for equals as assignment in do-blocks without `let`. This isn't actually ambiguous
though, as the ambiguity goes away when we have a do/record with multiple entries, as a comma tells us we're in a record, and
a semicolon tells us we're in a do-block, and binding variables in a single entry do-block doesn't make sense. However, that
isn't the same thing as this being a good idea.

2. What about `{x : IO Int}`? I've seen desire for type signatures above the binder in do-blocks, which already would be
ambiguous, as `:` is also allowed in expressions. However, Idris doesn't allow `:` in expressions, so that's one solution.
It would also remove the ambiguity for the same rason as point 1: a do-block with a type variable would need to have a
following binder, like:

```haskell
do
    x : IO Int
    x <- pure 2
    ...
```

So `{x : IO Int}` has to be a record, since a type signature in a !-block has to be followed by a semicolon, then the bound
variable. However, that doesn't mean it's unambiguous to the coder.

Another solution (if still committing to the `:` as type declaration in do-block idea), is to simply disallow the same in
!-blocks, similar to the fact that guards can't be used in lambdas. The syntax is intended for code that is small; if someone
needs that much logic they should use the full do-block, and not the sugar.

A solution to all of these would be to need a comma following 1-records, similar to how a 1-tuple might need a comma, so we
would write `{x : IO Int,}`. This solves all the issues, and if I'm doing the same in 1-tuples that would make it easier to
learn that it applies to records as well; knowledge transferring in syntax I think is important. My biggest concern here is
that 1-records are likely to not get used much, so the fact that they use the same syntax might not be learned very quickly.

It's ugly, but we don't see it much so it's not a problem, but we don't see it much so we don't learn the syntax. I think the
type-checker would need to understand if the sugar was used, so it can give an informative error message if the user confuses
them.

My biggest issue with my other solutions is that while there are ways to resolve the ambiguity,
I feel like programmer's are still likely to not realize it's ambiguous, in which case the fact that the compiler thinks it's
not ambiguous isn't really relevent.


Another solution is to need a comma following records, so for the record we would write

My big concern is there are lots of ways to interpret brackets here, and I want to avoid needing ad-hoc rules, so I need to
make a firm decision at some point.





















