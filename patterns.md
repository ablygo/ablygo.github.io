# Patterns

I've seen a lot of proposals for improvements for patterns in Haskell, and with each I've thought "that would be cool to have", but also that I don't really want it. I try to avoid having much complexity in patterns, and making the pattern language more dense always makes me nervous. My overall feeling is that it would be nice to have some well-thought form of pattern calculus, that allows abstraction, and all that neat stuff, and thus the draw of these proposals is based on a real desire for improvement, but that none of them actually achieve that goal, thus the nervousness about accepting those proposals. Thus, my ideas around patterns have this contradiction in mind.

It may be the case that optics are the pattern calculus I desire, though they do have a few issues. Operationally they are going to be potentially less efficient, due to having to tuple up their arguments, only to immediately pattern match on them. They also don't work with existentially types, and can throw away information (patterns allow affine traversals, while optics allow general traversals). This last issue is a big one for me.

Even if the issue with existential types could be solved, and the operational issues could be optimized away when possible, I think there are benefits to having syntax that reflects that true patterns are O(1) and at worse affine, whereas optics may both be slower, and throw away information. 

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

I want to avoid the need for capitilization to distinguish fresh variables vs. data constructors, and plan to use ticks to disambiguate. So `'id 'x = x` is the identity function, while `'id x = x` pattern matches its argument against the data constructor `x`.

The tick on the `id` is for consistency, anywhere a name is introduced it is ticked. I don't love needing to do this, but there are a few different places where using ticks this way is helpful, and thus I feel the consistency is worth it. It actually helps for knowing whether we are pattern matching against the `id` constructor or defining a function as well, as pattern matching is allowed in declarations.

This also could let me use unticked variables in patterns as Eq constraints if the variable isn't a data constructor, though this also has the issue of hiding operational characteristics behind syntax. Semantically though, using unticked variables for both eq constraints and data constructors is really compelling to me. I might allow it, but with an annotation at the use site, so people would have to write:

```haskell
#[EqPattern]
'f x = ...
```

For it to be accepted, even if it would parse without the annotation.

## Pattern families

I like the Haskell pattern family proposal a lot, except that there's no syntactic way to tell when it's being used. This has some of the same issue I have with pattern syntax using optics. View patterns makes the fact that arbitrary computation is being done syntactically significant, while pattern families hide it (while possibly making the semantic meaning of the code more clear).

Likewise, there's no way to even tell if a variable is being bound. Even if we know that things are O(1), what does `foo (Foo x) = x` mean? `x` could be an expression already in scope and `Foo` a pattern family, or we could be pattern matching against a variable. I could have `foo ((Foo x)) = x`. Since we can't have expressions which return constructors, parentheses around the first argument are technically never going to occur in patterns, though it breaks the fact that `(f x) y = `f x y`. I don't like this solution.

Another idea is to just have an outright syntactic marker. I was considering something like `‹pattern family› a b` (subject to bikeshedding, I don't want to need unicode). I do like this idea, as it separates the expression from the bound variables. It could also be used for pattern synonyms in general, so `'foo Foo = ` would always be an O(1) pattern match against a data constructor, while `'foo ‹Foo›` would be a pattern match against a nullary pattern synonym. This is my favorite solution, as it entirely distinguishes between pattern synonyms/families (and unifies them) and raw pattern matches, but is also lightweight.

I did also consider somethig like `?f 'x y`, which would denote that `f` is a pattern family, and then unticked variables could be determined to be expressions or data constructors depending on the definition of `f`. This would allow the pattern and expression parts of the pattern family to occur in any order, which maybe might be useful? I don't have any examples, but I'm open to the possibility that one might exist. I don't love this.

If I go this route I could still also use (almost) the same syntax for optics, with something like `‹_These›('a,'b)`. Thus, pattern families would use juxtaposition as seperators, while optics would use prefix function application syntax, which reflects that optics need to have their arguments tupled up. While I like that this is possible (and was considering this syntax initially), I feel like just having the pattern family syntax is preferable to making the pattern language even more complex. We can afterall make a pattern family for using optics, and then seeing it in use also emphasizes that optics aren't necessarily affine.
