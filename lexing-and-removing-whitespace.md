# Lexing and even very whitespace sensitivity

So I'm considering the question of whether symbols should be allowed in character names in my language, as well as the use of sigils, and it's resulted in an interesting question to ask about my grammar (and grammars in general), is it a bad thing if the need for whitespace is context sensitive. I was trying to write a datatype for a lossless syntax tree, and wanted it to be correct by construction, so that printing and then parsing it would always result in the same tree, but my desire for sigils caused a weird question for how to represent the tree (and thus grammar, as my LST is in fact derived directly from the grammar).

As an example to show the issue, consider the string `a + !b`. With whitespace it reads as the token stream `[Name "a", Operator "+", Prefix "!", Name "b"]`, but without it reads as `[Name "a", Operator "+!", Name "b"]`. So to represent this correctly in the LST we need to add whitespace tokens to any `Infix` constructors, to show that it is mandatory. I don't have a huge problem with this, though I feel like it might be more contentious than I want it to be, which makes me nervous.

The big question comes down to "are there any constructors in the AST where whitespace is only mandatory *sometimes*?" As an example, suppose we write `let f x = !y`. That's fine, as is `let f x = y` but `let f x =!y` is not. If we print it and then lex it we get `[Let, Name "f", Name "x", Sigil "=!", Name "y"]`.

So if the LST follows the grammar exactly do I need different nodes for ". Of course, the LST doesn't have to follow the grammar exactly, and we could just look ahead to .

Suppose we drop the requirement of the LST being an LST, and only be an AST. Here's a similar question. Can we print out the AST as a string with minimal whitespace, without needing to look at child nodes to determine whether there should be whitespace? So if we have a constructor like `AppE [AST]`, we print it out with aIt almost feels like printing stops being context free otherwise.

If we have a constructor like `AppE : List2 AST -> AST`, and want to print it out with minimal whitespace, we need to look into each list element to see if it needs whitespace. After all `f x` requires it, but `f(x)` does not. Making whitespace mandatory anywhere it might be required fixes that. What constitutes *where* is kind of contextual: if a language represents field access with a different node in the AST than infix operators, then it can make the distinction without cheating, since printing a `FieldAccess` node without whitespace is always safe.

The reason the AST representation gets kind of muddied with the grammar here is due to the way I'm writing my LST. I'm using generics to derive a parser and pretty printer from my grammar, as well as derive an arbitrary instance for my grammar. However, even if I store whitespace in the grammar there's the question of is it even necessary. If whitespace is optional in the `AppE` constructor, an arbitrary tree will get printed out as `fx`. If it's not mandatory we can no longer represent the AST for `f(x)`. So by making it mandatory, we now have `AppE : SepBy2 WhiteSpace LST -> LST`, which is safe to randomly generate, describes how to parse it, and how to print it.

If we want to allow both we need to have separate nodes in our AST for function application of names and function application of composites. Alternatively, I represent prefix notation differently than juxtaposition, so `f g(x)` would actually get parsed as `AppE [Name "f", WhiteSpace 1, Prefix "g" (Paren (Name "x"))]`.

I think that's the better representation anyway: `g(x)` is *tighter*, and so has a different meaning (which could still be function application, but not what I'm considering using it for).

But I'm concerned how far I have to go for this safety. I can write `let f x = !getLine`, but not `let f x =!getLine`, so I need to make whitespace mandatory after the equals. Again, I like this, but I feel like it would be contentious, which makes me not like it (not that I have a problem with people disagreeing, but I feel from a usability standpoint considering it is important). I'm not actually sure how far I need to apply this, and that's the big thing that makes me nervous

I guess the question is, is that a bad choice? I actually kind of like it, but I feel like anything to do with whitespace is controversial. People will want to 
```
