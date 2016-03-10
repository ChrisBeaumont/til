### The Pimp My Library Pattern

The "Pimp My Library" pattern is a brogrammish label for how libraries
can extend external types with additional features. Example:


```
import spray.json._
val x = 5.toJson  // spray.json.JsValue = 5
```

How did spray manage to add a `toJson` method to `Int`?

One approach (the only approach? not sure) is to create a "Pimped"
class with the extra methods:

```
class PimpedInt(value: Int) {
    def jsonify = JsValue(value)
}
class PimpedInt(value: Int) {
  def jsonify: JsValue = JsNumber(value)
}
new PimpedInt(5).jsonify  // JsValue(5)
```

Then, define an implicit converter function:

```
implicit def pimp(value: Int): PimpedInt = new PimpedInt(value)
5.jsonify
```

The compiler figures out that `jsonify` isn't valid on Int, but is valid
on PimpedInt and there's an implicit converter in scope. So it performs the
equivalent of

```
pimp(5).jsonify
```

`implicit def` only seems to work "1 level deep" -- that is, we can't define

```
implicit def asint(value: Double) = value.toInt
```

and expect the compiler to turn

```
(3.0).jsonify
```

into

```
pimp(asint(3.0)).jsonify
```
