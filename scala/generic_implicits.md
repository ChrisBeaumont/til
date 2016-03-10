## Functions on generic types with implicits

Spray-json looks for an implicit `JsonWriter[T]` when calling
`toJson` on an instance of type `T`. For example consider the following code:


```
import spray.json._

trait Base
case class Foo(x: Int) extends Base
object JsonSerializers extends DefaultJsonProtocol {
  implicit val FooFormat = jsonFormat1(Foo)
}

def go(f: Foo) = {
  import JsonSerializers._
  f.toJson
}
go(Foo(1))   # res0: spray.json.JsValue = {"x":1}
```

Here `FooFormat` is the required implicit value that scala needed.

The following code doesn't compile:


```
def go(b: Base) = {
  import JsonSerializers._
  b.toJson
}
go(Foo(1))
```

```
Cannot find JsonWriter or JsonFormat type class for A$A114.this.Base
```

The compiler demands a JsonWriter for the generic `Base`, type, even
though it is abstract and the only concrete subtype has such a value.

We can resolve this by making a generic function and "explicitly"
providing the implicit value:

```
def generic_go[T <: Base](t: T)(implicit fmt: JsonWriter[T]) =
  t.toJson

import JsonSerializers._
generic_go(Foo(1))
```

The main difference is that, in the first function, the `JsonWriter` is
resolved to a reference when the function is called (and no such `JsonWriter[Base]` exists). In the second function,
the `JsonWriter` and `T` are resolved separately at each `generic_go` call
site.

`generic_go` can be further compacted by using a context bound:

```
def generic_go2[T <: Base : JsonWriter](t: T) =
  t.toJson

import JsonSerializers._
generic_go(Foo(1))
```

This means the same thing as defining the `implicit` parameter explicitly. If
you needed to access the implicit value within the function, you need to use `implicitly`:

```
def generic_go3[T <: Base : JsonWriter](t: T) = {
  implicitly[JsonWriter[T]] write t
}
```



