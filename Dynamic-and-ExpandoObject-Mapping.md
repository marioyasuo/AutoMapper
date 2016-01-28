AutoMapper can map to/from dynamic objects without any explicit configuration:

```cs
public class Foo {
    public int Bar { get; set; }
    public int Baz { get; set; }
}
dynamic foo = new MyDynamicObject();
foo.Bar = 5;
foo.Baz = 6;

var config = new MapperConfiguration(cfg => { });
var mapper = config.CreateMapper();

var result = mapper.Map<Foo>(foo);
result.Bar.ShouldEqual(5);
result.Baz.ShouldEqual(6);

dynamic foo2 = mapper.Map<MyDynamicObject>(result);
foo2.Bar.ShouldEqual(5);
foo2.Baz.ShouldEqual(6);
```

Similarly you can map straight from dictionaries to objects, AutoMapper will line up the keys with property names.
