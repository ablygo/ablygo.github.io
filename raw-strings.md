I'm trying to make these a little more organized; right now they're pretty much stream-of-thought. The syntax I plan on using looks like the following (with example text from https://rahul-thakoor.github.io/rust-raw-string-literals/, taken from the toml crate).

```
''
| global_string = "test"
| global_integer = 5
| [server]
| ip = "127.0.0.1"
| port = 80
| [[peers]]
| ip = "127.0.0.1"
| port = 8080
| [[peers]]
| ip = "127.0.0.1"
''
```

The thing I like about this syntax is we can embed pretty much any character into the raw string, including the end of string delimiter. The reason we can do this is that each line with actual text needs to start with `|`, while the end of string delimiter must start with `''`, so we can distinguish between the two. Lines containing only whitespace, or comments are ignored, which would allow commenting individual lines of the string without it being semantically reflected in the string contents itself.

One big disadvantage is it always uses n+2 lines of source code for n lines of a string, so it's more verbose than than rust raw strings for 1 line strings. However, I think it is more readable, as reading rust raw strings potentially requires counting hashes, which feels similar to deBruijn indices for readability. It's especially better for representing raw strings itself, compare:

```
''
| ''
| | ""
| ''
''

##"#""""#"##
```

I few readability as the priority to being able to write a raw string in a single line.

A disadvantage is the syntax for the empty string, which user's could confuse:

```
-- incorrect
''
''

-- correct
''
|
''
```
However, we can simply give an error message in that case, as the syntax otherwise has no meaning. It's unlikely many people would use raw string syntax for empty strings regardless.
