I want to allow conditional inclusion of a package, so I could write things like:

```haskell
module 'FooBar where
    #[cfg(package=lens)]
    instance Ixed Foo where
```

this would only be compiled if a package depending `FooBar`  also dependend on lens. This would be a way of getting around the diamond problem for orphan instances. Neither `FooBar` needs to force a dependency on `lens` on its users, nor does `lens` have to force a dependency on `FooBar`.

It would still require a "weak" dependency of `FooBar` on `lens`, with the rule that no weak dependencies are allowed to form cycles. In this case, that would mean `lens` would not be able to depend on `FooBar`, even conditionally.

### Infix operators

I want to allow custom infix operators, but with an intransitive preorder, which also causes an issue with orphans, so this would help for that as well. One problem with this still is it could require a O(n^2) number of precedence relationships, though possibly that would be acceptable, as avoiding having too many precedence relationships that user's need to learn is kind of the point.

Fixities could potentially be made richer, such as having an open total order *or* closed preorders, allowing both, or having fixity sets, to reduce the O(n^2) while still having a way to abstract over the types of relationships (multiplications bind more tightly than additions), but as a first approximation I feel like making things a preorder is sufficient. Without some strong theoretical/practical backing for how a fixity system should be designed, I don't really want to experiment.
