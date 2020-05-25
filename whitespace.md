# Some thoughts on whitespace I'm unsure about

I'm not really sure about any of these ideas. I like that they free up the grammar a bit for more creativity, but I feel they could be contentious enough that I'm not sure they give enough to be worth the cost.

I plan on having a syntactic distinction between 0 or 1+ whitespace characters, mainly for sigils, but I thought about `f x` and `f(x)` being distinct as well. Having a distinction between tight or loosely bound operators does seem helpful, and I don't think it's hard to visually parse, but I'm not sure. `1+3` would no longer have the obvious meaning; I think it really depends on how often other programmer's actually write code with tight operators.

# Other forms of indentation sensitivity.

I think I can free up `|` in the expression grammar if I use it with layout in guards like:

```
f x if
  | b <- d
  , c
  = ...
  | otherwise
  = ...
```

Essentially any expression would need to be further indented than the pipe, so pipes which occur in expression position would no longer be ambiguous. Though it's common enough to write guards with the `=` on the same line as the guard, so I might need a special case in the parser to recognize the `=` both aligned or unaligned. Commas aren't really necessary here either, we could use normal layout rules in a pattern guard.

Not sure if this is worth it.

# A parsing problem

I wanted to parse Syrup into a token tree before parsing it into the proper AST, though parsing indentation in Haskell has some tricky cases. As an example of a difficult parse

```
[do
  case () of
    _ | a , b -> c, d]
```

Here the indented block of the `do` ends at the second block, because it is delimited, in a similar way to how `[do x, y]` doesn't extend the intended block past the comma. We want to parse things like `do [x, y]` correctly, so we have to have the grammar nest lists and layout correctly, but because commas are also allowed in guards it also needs to nest guards. I was also planning to allow commas left of the guard, for multi-lambda case, so it would also need to understand patterns. Functional dependencies also use commas.

Likewise, layout can also be terminated by `then` or `else`, since `if_then_else_` is basically parsed like a delimiter. So parsing layout requires understanding a significant amount of the syntax of the full grammar, in which case separating the TokenTree parser from the AST parser doesn't seem worth it. These are only issues if they're kept separate; they're rather trivial if combined.

Another solution is to simply allow layout terminated by commas, so `(do x, y)` would parse as `(do {x, y})`, or even disallow layout terminated by anything but a proper dedent, in which case it would parse as `(do {x, y)}`. In practice, pretty much all the code I write fits this rule (I think the parens case is common in Haskell code though), but I *really* don't like the idea of making this a requirement.

A big thing motivating me to do this is that if I can parse layout without other syntax being able to terminate it early then I can support syncronization points in the parser more easily, e.g. if a parse error is encountered jump ahead to the next layout block. If a layout block can be terminated early you might jump over where it ended. I could also support arbitrary nested grammars, using a dedent as the end-of-grammar token. These would be really nice things to have, but I'm not sure they're worth it.
