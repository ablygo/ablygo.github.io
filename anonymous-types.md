
I want to have good support for anonymous structural types, but don't have any strong opinions as to syntax. I feel like it's a somewhat minor issue in the end. I would like to support both positional/named products/sums at the value/type level.

I'm considering using `G(x) : {| F a b , G b |}` for named sums, though allowing them to be polyvariadic may be tricky. My worry is if we need named sums to have known size, I would need to store a pointer to an array of its elements, which are also going to be boxed, and thus we need two pointers to access an element. However, we could just have a pointer to a single element, and store a tuple if we need more, which would be faster for the specific case of only needing one element.

That said, I'm not sure we need them to have known size. Polymorphic functions already take boxed arguments, and the type system/pattern matching should guarantee we don't index an invalid argument. But I'm not confident about it.

For positional sums I might use `0(x) : (| a , b |)`. This syntax does make the issues with trying to have polyvariadic constructors more present. Is `(| f a |)` a 1-sum with two arguments `f` and `a`, or one argument `f a`. It might be better. If 1 of the types in the sum is empty do we write `(| a , , b |)`? Making everything consistently take 1 argument means fewer weird rules to learn.

It might be worthwhile to come up with a syntax that nicely allows named and unnamed constructors and positional/named arguments.

# If positional types are like lists, and named are like maps, what about sets

I might also want to support a sum type that behaves as a "set of types", but feel like how that should work seems less obvious. The issue comes down to sets like `forall a b. {Maybe a, Maybe b}`, which could have 1 or 2 elements depending on whether `a ~ b`. Whether this is even well-defined when dealing with polymorphic types doesn't seem clear to me.

It would also suggest there should be a product "set of types", behaving like an unordered tuple.
