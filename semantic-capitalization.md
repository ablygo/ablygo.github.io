
So I'm using ticks as an alternative to semantic capitalization for variable bindings, so I can write code like:

```
'id : 'a -> 'a
'id 'x = a
```

Which variables are constructors vs. fresh variables then becomes unambiguous (though I'm using conceals and syntax highlighting to replace the ticks with color.

I'm interested in using ticks for other cases that follow this same logic, though I'm not sure how good of an idea that would be, as some of the other uses seem mutually exclusive. For instance:

1. Use ticked variables in expressions to refer to variables bound by record puns. For instance, we could write:

```
'foo {..} = 'x
```

To indicate that `x` is newly introduced, just as we use ticks in patterns to show that a variable is being newly introduced. This may conflict with using ticks in type signatures, as we could write:

```
'id : 'a -> case foo of {..} -> 'x
```

in which case it's not clear whether the `x` is a newly introduced type variable or a record pun. I really like this idea, possibly enough to use it but make explicit forall mandatory. Another option is to depend on context, if the expression is the right of a `:` then it an implicit new variable, but otherwise it is a pun. I don't hate this, but I can't say I love it either.

2. Use ticked variables for cheaper lambdas.

I already use `{ ... }` to denote the scope of hoists in expressions, so punning on it indicating scope, I could write code like `{ foo '2 '1 }` as sugar for `\x1 x2 => foo x2 x1`. I don't love this idea, but it's an idea.
