## Dynamic/Expando Objects

AutoMapper can map to/from dynamic objects without any configuration:

```cs
public class Foo {
    public int Bar { get; set; }
    public int Baz { get; set; }
}
dynamic foo = new MyDynamicObject();
foo.Bar = 5;
foo.Baz = 6;

var result = Mapper.Map<Foo>(foo);
result.Bar.ShouldEqual(5);
result.Baz.ShouldEqual(6);

dynamic foo2 = Mapper.Map<MyDynamicObject>(result);
foo2.Bar.ShouldEqual(5);
foo2.Baz.ShouldEqual(6);
```
