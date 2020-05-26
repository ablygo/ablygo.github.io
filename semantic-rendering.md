
I kind of feel like a lot of the issues that come up in deciding a grammar are purely artifacts of the lack of semantic editors, but they don't hide the fact that deciding on a grammar is still important (even with semantic editors). I think the true problem that is at issue is deciding how code should be rendered, not how it should be parsed; these problems just happen to coincide with text based editors. We want to optimize for readability, not writeability.

I don't think waiting for semantic editors is super realistic, but with ViM I've been able to use conceals to hide certain parts of code, and then give things like color/italicization semantic meaning, which seems to be working as a middle ground. This could for instance allow one to design a language with no reserved words, as the read-optimized grammar could bold keywords, while the write-optimized grammar could use a sigil.


## What would it take to make this actually defensible?

I don't think this removes the need to still carefully consider both the write/read grammar. Color blindness needs to be considered, as does the fact user's might have preferences for light/dark themes. It's probably a good idea to keep the two grammars closely related as well; at the extreme one could be writing LaTeX, but even with an editor capable of customizing how code is rendered to that degree it's probably a poor choice. People may need to read code written out by hand, or on another person's monitor, or github/blog posts, so the write-optimized grammar should still be reasonably readable.

I think for this to actually be acceptable it would also need support in a large percentage of major coding editors, so that limits you to the insersection of those editor's capabilities. I don't think "you need to learn this editor to write code in this language" is ever acceptable if you're arguing your language is being designed with usability in mind.

Still, I think it's more realistic than just hoping for a semantic editors. I intend to play with the idea a bit to see how it allows a little more freedom in my grammar, though if a large user base was an actual possibility I'd probably have to abandon it.
