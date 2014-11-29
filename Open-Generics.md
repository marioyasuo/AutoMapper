AutoMapper can support an open generic type map. Create a map for the open generic types:
```
public class Source<T> {
    public T Value { get; set; }
}

public class Destination<T> {
    public T Value { get; set; }
}

// Create the mapping
Mapper.CreateMap(typeof(Source<>), typeof(Destination<>));
```
You don't need to create maps for closed generic types. AutoMapper will apply any configuration from the open generic mapping to the closed mapping at runtime:
```
var source = new Source<int> { Value = 10 };

var dest = Mapper.Map<Source<int>, Destination<int>>(source);

dest.Value.ShouldEqual(10);
```
Because C# only allows closed generic type parameters, you have to use the System.Type version of CreateMap to create your open generic type maps. From there, you can use all of the mapping configuration available and the open generic configuration will be applied to the closed type map at runtime.
AutoMapper will skip open generic type maps during configuration validation, since you can still create closed types that don't convert, such as `Source<Foo> -> Destination<Bar>` where there is no conversion from Foo to Bar.