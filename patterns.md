# Patterns

I've seen a lot of proposals for improvements for patterns in Haskell, and with each I've thought "that would be cool to have", but also that I don't really want it. I try to avoid having much complexity in patterns, and making the pattern language more dense always makes me nervous. My overall feeling is that it would be nice to have some well-thought form of pattern calculus, that allows abstraction, and all that neat stuff, and thus the draw of these proposals is based on a real desire for improvement, but that none of them actually achieve that goal, thus the nervousness about accepting those proposals. Thus, my ideas around patterns have this contradiction in mind.

It may be the case that optics are the pattern calculus I desire, though they do have a few issues. Operationally they are going to be potentially less efficient, due to having to tuple up their arguments, only to immediately pattern match on them. They also don't work with existentially types, and can throw away information (patterns allow affine traversals, while optics allow general traversals). This last issue is a big one for me.

Even if the issue with existential types could be solved, and the operational issues could be optimized away when possible, I think there are benefits to having syntax that reflects that true patterns are O(1) and at worse affine, whereas optics may both be slower, and throw away information.

## Arguments separated by commas

I plan to use arguments separated by commas on the left hand of a definition, like `'foo 'x, 'y 'z = ...`.

A plus to this is it generalizes lambda case to multiple arguments quite nicely. It also allows eliding some parentheses.

A downside of this is that the left hand side of definitions doesn't mirror when functions are used. For instance `f x y = ...` reads rather nice as a definition: whenever you see `f a b` in an expression you bind the variables to those in the left hand side of function definition, and then replace it by the right hand side. I'm not sure losing that consistency is *that* bad a loss though. It wasn't until I was pretty familiar with Haskell that I actually noticed that definitions could be read that way. Thus, the inconsistency might only be apparent to people after it's been pointed out.

Overall, while it's inconsistent, I don't think user's would find it confusing to use commas to separate arguments in definitions, but not in use. I would need to ask some other programmers to be sure.

## Or-patterns

Use `|` as a way to separate patterns, which match if either side match. Both sides would need to bind the same variables (and possibly the same type, though with type classes that wouldn't be required). In the worst case this could simply be compiled into a combinatoric expansion of patterns, which would allow them to differ in type.

`|` is the obvious choice of syntax, and I feel most Haskellers would agree, but due to using `|` for guards it presents ambiguity. In Haskell the proposal uses `;`, though I still feel the meaning is more clear with `|`. My solution is to have some herald before a guard, like `foo x if | x == 0 = True`. Thus, the `if` separates the or-patterns from the guards. The choice of `if` presents some new issues (`where` avoids them, if overloading the meaning of `where` in a way I don't like). Overall I like this idea.

With multiple arguments you would write:

```haskell
foo a, b | c, d | e, g = ...
```

This would be a two argument function with three or-patterns, so `,` binds more tightly than `|`. Thinking of `,` as a product and `|` as a sum also gives that intuition. Thus if you wanted an or-pattern to refer to a single positional argument parentheses would be required.

## View patterns

Haskell uses `->`, I would be using `=>` if I use the same syntax. Using `<-` would make the language more consistent, as it's also used in pattern guards and do-blocks. If pattern guards are allowed in nested patterns, as in [https://gitlab.haskell.org/ghc/ghc/-/wikis/view-patterns-alternative], then I feel `=>` would be better, as we would need to write `(patt | pattguard) <- expr`, compared to `expr => patt | pattguard`. I like the consistency that that proposal has, but consistency is really the only thing I like about it.

I do like the consistency of using `<-` for pattern guards, do, and view patterns, but also feel `=>` reads better even without the view pattern alternative. I'm not sure if that's just a matter of habit, or due to reading left to right (call this function on the argument first, *then* match it against this pattern. I feel like I would need to ask someone not too familiar with Haskell what they thought to evaluate them.

## Ticks

I want to avoid the need for capitilization to distinguish fresh variables vs. data constructors, and plan to use ticks to disambiguate. So `'id 'x = x` is the identity function, while `'id x = x` pattern matches its argument against the data constructor `x`, and also returns the data constructor `x`.

Though these are somewhat confusable, I'm *slightly* okay with this. The idea of requiring this is not to discontinue using capitals for constructors and lower case for other expressions/freshly bound variables, but to allow user's to break from that requirement *if* they feel it makes their code more readable. Thus `x` as a constructor name would be a bad idea for the same reason as `putStrLn` would be a bad name for a function that reformats your harddrive.

With explicit shadowing warning and a culture of qualified imports, I think this is acceptable; having an unqualified constructor named `x` in scope should be unlikely. This still makes me slightly nervious, though I really don't like needing to use semantically significant capitilazation. I'm not super sure about this.

## Eq patterns

Unticked variables in patterns could desugar to an equality constraint if the name doesn't refer to a data constructor. This feels very natural to me: matching against a data structure is a form of equality check; it just happens to be one that we may override when defining equality on a data type.

One huge caveat, this would have to be opted into explicitly. So you would write something like:

```haskell
#[EqPattern(x)]
'equals 'x x = True
'equals _ _ = False
```

Note in the example, `'x` brings `x` into scope for use in other patterns in the definition.

The reason for this requirement is both the operational concern, and also because it might be easy to cause accidentally if intending to shadow a variable (that might be caught by the type checker, but then again it might not).

I might still disallow this. Using the pattern family syntax below, I could write `‹Equals x›`, which is more explicit, but still more lightweight than `(== x) => True`. This is my overall preference.

## Pattern families

I like the Haskell pattern family proposal a lot, except that there's no syntactic way to tell when it's being used. This has some of the same issue I have with pattern syntax using optics. View patterns makes the fact that arbitrary computation is being done syntactically significant, while pattern families hide it (while possibly making the semantic meaning of the code more clear).

Likewise, there's no way to even tell if a variable is being bound. Even if we know that things are O(1), what does `foo (Foo x) = x` mean? `x` could be an expression already in scope and `Foo` a pattern family, or we could be pattern matching against a variable. I could have `foo ((Foo x)) = x`. Since we can't have expressions which return constructors, parentheses around the first argument are technically never going to occur in patterns, though it breaks the fact that `(f x) y = f x y`. I don't like this solution.

Another idea is to just have an outright syntactic marker. I was considering something like `‹pattern family› a b` (subject to bikeshedding, I don't want to need unicode). I do like this idea, as it separates the expression from the bound variables. It could also be used for pattern synonyms in general, so `'foo Foo = ` would always be an O(1) pattern match against a data constructor, while `'foo ‹Foo›` would be a pattern match against a nullary pattern synonym. This is my favorite solution, as it entirely distinguishes between pattern synonyms/families (and unifies them) and raw pattern matches, but is also lightweight.

I did also consider somethig like `?f 'x y`, which would denote that `f` is a pattern family, and then unticked variables could be determined to be expressions or data constructors depending on the definition of `f`. This would allow the pattern and expression parts of the pattern family to occur in any order, which maybe might be useful? I don't have any examples, but I'm open to the possibility that one might exist. I don't love this.

If I go this route I could still also use (almost) the same syntax for optics, with something like `‹_These›('a,'b)`. Thus, pattern families would use juxtaposition as seperators, while optics would use prefix function application syntax, which reflects that optics need to have their arguments tupled up. While I like that this is possible (and was considering this syntax initially), I feel like just having the pattern family syntax is preferable to making the pattern language even more complex. We can afterall make a pattern family for using optics, and then seeing it in use also emphasizes that optics aren't necessarily affine.

One thought, in expressions should you need to write `‹pattern family› a b`, with the same number of parameters inside the brackets and outside? The distinction only matters when it separates the expression and patterns, but in an expression context it doesn't matter. Putting patterns in a separate namespace and just writing `‹pattern_name›` in expressions (moving all expressions outside the bracket) would be more convenient for partial application, so you don't need to write `\x, y => ‹patt x› y`. On the other hand, it introduces inconsistency. What do?

If we did put it in a different namespace, we could write `‹patt_name› a b` even for pattern families which take arguments, and rely on the compiler to figure out which arguments are patterns and which are expressions (in which case it is not clear syntactically which are expressions and which are patterns, but is still clear that a pattern family is being used). I'm not sure which is the better design.

I'm leaning towards being fully explicit in pattern contexts, requiring the brackets to enclose the expression arguments, but being less explicit in expression contexts, where there is only need to know that a pattern family is being used, but not which arguments are part of the family in a pattern context. If the user wrote `‹f a› b` in an expression it would result in a compiler error and message explaining that the correct answer is `‹f› a b` in an expression.

Are functions which take patterns and return patterns a thing? We could partially apply a pattern family to return a new pattern family, up until all the expression arguments are given. It might be better to try to give them a first class type and just use the brackets for syntax, but have them all exist in the same namespace still.

```haskell
‹'View› : ('a -> 'b) -> Pattern 'r 'b -> Pattern 'r 'a
```

I also kind of wonder about patterns with side effects, which could be useful with STM, or if using pattern matching to pop elements from a splay tree. Though at that point it may just be that patterns really just are an instance of MonadPlus, where `mzero` makes any side-effects unobservable (that is, a failed pattern match shouldn't have observable side-effects). If that was the case it might not be worth it to enrich the pattern language, but simply write monadic code. Patterns that behave like regular expressions (which can match multiple results) also seem interesting, but again, MonadPlus.

### Pattern declarations

Some notes on declarations for pattern families, I find the syntax to be kind of confusing, though I don't know this idea is an improvement. We could have something like:

‹Equals› : Eq a => a -> a
‹Equals a› = ((==) a => True)
  where
    ‹Equals› : a -> a
    ‹Equals a› = a
    
The type declaration would thus show the constraints on type variables, and the number of variables within the brackets in the body would show which ones are values and which ones are patterns. Or maybe something like:

```
‹'Equals : Eq 'a => 'a› -> 'a
‹'Equals 'a› = (== a) => True
  where
    ‹'Equals : 'a› -> 'a
    ‹'Equals 'a› = a
    
‹'At : At 'm => Index 'm› -> IxValue 'm
‹'At 'k› 'v <- preview (at k) => Just v
```
I need to distinguish between constraints that are brought into scope, and constraints must already be in scope. I could have

```
‹'Pattern : BroughtIntoScope 'a› => AlreadyInScope 'b => ...
```

This reads better than Haskell's syntax, but it kind of muddies the fact that the arguments to the pattern family aren't really related to the constraints brought into scope.

# Prefix function application constructors

I was at one point planning to use `Foo(x,y)` as the syntax for data constructors, which had a few nice advantages. One, it removed the need for ticks to distinguish fresh variables vs. constructors (nullary constructors would be written `True()`, while `True` would be a new variable.

I had intended to have optics baked into the language, and it also worked nicely with those. Optics require all their arguments to be tupled up, and I felt the need to use parentheses got that intuition across. Just as `foo` and `foo()` are interpretted differently, with the latter having more of a *finality* than the former, the same intuition would apply: we can't partially apply an optic and get another optic, so we need to distinguish when all of its arguments have been applied.

Overall, I feel like prefix function application feels more intuitive when there's a sense of finality, while juxtaposition makes more sense when there's not (a function could always return another function, so there's no point syntactically when we're *done* applying it). The same is not true for data constructors. So I like that the syntax makes that distinction apparent.

Note, type constructors get partially applied a lot, as in `Free (State s) a`, so I feel like they should continue to use juxtaposition. Prefix application and partial application look visually kind of weird when used together, would it be `Free(State(s),a)`, or `Free(State(s))(a)`?

I also liked this about this syntax, as it would make the intution that type constructors and functions can't be pattern matched against (due to using juxtaposition), while data constructors can (and use prefix notation) syntactically apparent. 

However, I decided against it. One, even if we don't need ticks to denote fresh variables in patterns, we still need them in types, otherwise something like `instance Foo Bar` becomes ambiguous (is Bar a type variable or a type constructor?). Likewise, if I have ticks I can still have implicit forall, like `'id : 'a -> 'a`. So even if the prefix notation removes the need for ticks in some cases I still need them overall. Also, even if I don't hate `True()` I certainly don't like it.
    

# Restatement of current plans

```haskell
-- ticks are used to bring new names into scope, unticked variables in patterns are constructors

-- Function arguments are separated by commas, or-patterns are separated by pipes, commas bind more tightly
'notEq : Bool -> Bool -> Bool
'notEq True, False | False, True = True
'notEq _ _ = False

-- view patterns probably use =>
'foo (bar => 'x) = x

-- pattern families (and synonyms) are unified, and use some form of grouping to be used.
'foo ‹bar›, ‹qux x› 'y = y
```

I wonder if view patterns can be replaced by pattern families entirely, though complex pattern families are syntactically defined in terms of view patterns, so I'm not sure how that would work. 
