# Language Design Principles

- Order shouldn't matter, foo(int x, int y) -- not real syntax -- should be callable as foo(x = 4, y = 5).
- Maybe order should matter for some functions, or for callers who don't use names - if I already have x and y variables then foo(x = x, y = y) is annoying.
  - Can be solved directly for that case, where the names match, but often they are close but not matching.
- Maybe order should matter for the OO style x.foo(y) - even if the function is foo(int x, int y) we can infer that x.foo(y) means the same as foo(x, y).
- That is harder when order is present.
- Functional style encourages a different order than OO style:
  - fun map(f: T -> U, ts: List T): List U -- not real syntax
  - List<T>.map(f: T -> U): List<U> -- not real syntax
  - Allow the function to denote which parameter is 'this'
    - fun map(f: T -> U, this ts: List T): List U -- not real syntax
    - This works either as list.map(f) or map(f, list)
    - Can be done in an import or alias - import foo.map(_, this = ts)
- It should be possible to add new default parameters anywhere without breaking code.

- Maybe parenless and OO styles can coexist.
- fun foo(int x, int y): Int -- not real syntax
- (foo 5) 6 == foo(5, 6) == foo 5 6 == 5.foo(6)
- Style/formatter would prefer foo 5 6 or 5.foo(6), so when would we use foo(5, 6)?
- Maybe just to collapse foo (bar baz) to foo(bar baz), println(foo 5 6)
- How about when the outer one has multiple args - println(foo 5 6, bar 7 8) vs println (foo 5 6) (bar 7 8)

