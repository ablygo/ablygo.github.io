## My developer diary

I'm using this as a developer diary for personal coding projects I'm working on. I'm currently designing a toy Haskell-like programming language, and so much of it will be my thoughts on my design. This is largely just a personal project for fun, so I'm not necessarily worrying about "would this feature improve/hamper popularity". Realistically I don't see any given programming language becoming popular as being something you should hope for. However, a big part of what I'm actually interested in is how to design a good language from a usability perspective, and so things like error messages, built-in linting, language servers, documentation, and all that good stuff are large matters of interest for me, even if they're not strictly important, given that I'm the only user. I just happen to find thinking about those things fun.

Occassionally I will make choices that are at odds with those goals however, as they're not my only interest, and as I'm the only user I don't see anything wrong with that. It's a project for fun, and I don't want to design a different language for every idea that gets into my head. But I will try to be honest when I make a decision where I think "I should do this different in a real world setting".

I'm particularly interested in syntax, which I realize is often considered a trivial concern, but I think more substantive discussion is possible than saying it's all subjective. As a rule I think you should consider the community of programmers you want to attract, and the syntax they're likely to be familiar with, and a lot of thought should go into any deviations from that.

As an example, I was considering a change from Haskell's constructor syntax, which would look like this.

```haskell
isJust Just(x) = True()
isJust Nothing() = False()
```

As a motivating example for why I'm considering this syntax, I want to remove the restrictions on capitalization being limited to type/data constructors, as not all languages have a notion of capitalization, and I feel like people tend to hack around it anyway to refer to "constructor like things", such as prisms. I would rather users be able to choose which makes the most sense for their problem, rather than having that choice made for them, and hoping that it makes their code more legible most of the time.

However, case is syntactically significant in Haskell, as in a pattern lower case variables bind free variables, while upper case pattern matches against data constructors. Likewise, when defining a function it might be that we actually aren't definining a function at all, but destructuring a value, as in `let Just x = y in ...` not defining a function named `Just`, but rather being equivalent to

```haskell
case y of
    Just x -> ...
```

So removing the syntactic meaning of capitilization requires these different cases to be disambiguated. I also wanted to experiment with lenses/prisms being built into the language (probably a bigger motivator than the capitalization issue, actually), so making them easy to use is important. One reason I feel like this syntax can be justified is due to the fact prisms can't really be partially applied. To give an example of what I mean, consider the hypothetical desugaring

```haskell
expression: Foo(x,y)   =>   review _Foo (x,y)
pattern:    Foo(x,y)   =>   preview _Foo -> Just (x,y)
```

If we see a data constructor like `Bar x y z`, we don't really know when to package up its arguments in a tuple to call with `review`, unless we do semantic analysis of the type of `Bar`. I don't like the idea of needing to do that, so using traditional prefix call notation provides an obvious syntactic queue for when the prism is ready to be (p)reviewed.

Whether due to something inherent or simply because the convention is so established, I feel like prefix notation has a sense of finality to it, compared to whitespace as function application, which doesn't (and so suits partial function application). After all, we typically expect `foo` and `foo()` to behave very different in imperative languages, one being a function vs. actually calling the function. Since constructing/deconstructing also has a sense of finality (in terms of all the arguments being applied), I feel like punning on that is appropriate.

A con of this choice is that if you do want to partially apply a constructor you need to write `\y => Foo(x,y)`, but I'm okay with losing that. Another con is that having to write `True()` seems a little odd, but I don't see it as the end of the world, even if I dislike it.

But the bigger con comes down to type constructors. I want my language to be future compatible with being dependently typed, and then there's the question is should type constructors also use this syntax. Is there a type level analogue of optics?

Compare `StateT Int (ReaderT Bool IO Text)` vs `StateT(Int(), ReaderT(Bool(), IO()), Text())`. Prisms might not have a notion of partial application, but type constructors are partially applied constantly, and so prefix notation feels just awful. I'm not even really sure if `IO` should have parentheses there, but `ReaderT` has parentheses, and its not being fully applied, so maybe `IO` should?

The issue really hinges on whether you can pattern match on type constructors. If I can't pattern match on type constructors then I like the syntax. Type constructors and functions would both use juxtaposition, and neither can be pattern matched on normally, and so that consistency can serve as an aid to users to help develop intuition about how type constructors and data constructors are different. Open type families could still use the juxtaposition syntax, to emphasize that open type families don't really behave the same way as normal functions do. The syntax relates semantically similar things together, and so I consider it a win.

But if I *can* pattern match on type constructors then I feel like it should use consistent syntax with data constructors, and given that type constructors get partially applied so much I feel like all constructors using juxtaposition wins out.

As a rule, I feel like pattern matching on type constructors feels wrong, even in dependently typed languages. And there are ways of encoding type level functions to get around that, which feels correct, but sometimes awkward and boilerplatey, and I wonder if there's a better way to do it. Overall I'm not so sure of my opinion that I want to commit to a choice right now, so for now I'm deciding to use traditional Haskell syntax, and leaving myself open to changing that in the future.

So that was a lot of words to end up back where I started, but as syntax is a big part of what interests me I feel like I should put a lot thought of into it. Also I want to keep notes written down and out of my head, thus the blog.

All of this notwithstanding the actual semantic questions of having optics be so built in to the language, due to issues like exhaustiveness checking, runtime efficiency, type inference with GADTs, type errors, and the exact optic representation.
