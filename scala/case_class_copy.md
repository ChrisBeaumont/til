## case class copy method

CaseClasses have a copy method for overriding some subset of fields

```
scala> case class Foo(a: Int, b: Int)
defined class Foo

scala> val f = Foo(1, 2)
f: Foo = Foo(1,2)

scala> f.copy(b=3)
res2: Foo = Foo(1,3)
```
