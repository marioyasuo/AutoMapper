Although AutoMapper covers quite a few destination member mapping scenarios, there are the 1 to 5% of destination values that need a little help in resolving.  Many times, this custom value resolution logic is domain logic that can go straight on our domain.  However, if this logic pertains only to the mapping operation, it would clutter our source types with unnecessary behavior.  In these cases, AutoMapper allows for configuring custom value resolvers for destination members.  For example, we might want to have a calculated value just during mapping:

```c#
    public class Source
    {
    	public int Value1 { get; set; }
    	public int Value2 { get; set; }
    }
    
    public class Destination
    {
    	public int Total { get; set; }
    }
```

For whatever reason, we want Total to be the sum of the source Value properties.  For some other reason, we can't or shouldn't put this logic on our Source type.  To supply a custom value resolver, we'll need to first create a type that implements IValueResolver:

```c#
    public interface IValueResolver
    {
    	ResolutionResult Resolve(ResolutionResult source);
    }
```

The ResolutionContext contains all of the contextual information for the current resolution operation, such as source type, destination type, source value and so on.  For most scenarios, we won't need this more advanced interface.  Instead, we can derive from the ValueResolver&lt;TSource, TDestination&gt; abstract class:

```c#
    public class CustomResolver : ValueResolver<Source, int>
    {
    	protected override int ResolveCore(Source source)
    	{
    		return source.Value1 + source.Value2;
    	}
    }
```

Once we have our IValueResolver implementation, we'll need to tell AutoMapper to use this custom value resolver when resolving a specific destination member.  We have several options in telling AutoMapper a custom value resolver to use, including:

* ResolveUsing&lt;TValueResolver&gt;
* ResolveUsing(typeof(CustomValueResolver))
* ResolveUsing(aValueResolverInstance)

In the below example, we'll use the first option, telling AutoMapper the custom resolver type through generics:

```c#
    Mapper.CreateMap<Source, Destination>()
    	.ForMember(dest => dest.Total, opt => opt.ResolveUsing<CustomResolver>());
    Mapper.AssertConfigurationIsValid();
    
    var source = new Source
    	{
    		Value1 = 5,
    		Value2 = 7
    	};
    
    var result = Mapper.Map<Source, Destination>(source);
    
    result.Total.ShouldEqual(12);
```

Although the destination member (Total) did not have any matching source member, specifying a custom resolver made the configuration valid, as the resolver is now responsible for supplying a value for the destination member.  
## Custom constructor methods
Because we only supplied the type of the custom resolver to AutoMapper, the mapping engine will use reflection to create an instance of the value resolver.

If we don't want AutoMapper to use reflection to create the instance, we can either supply the instance directly, or use the ConstructedBy method to supply a custom constructor method:

```c#
    Mapper.CreateMap<Source, Destination>()
    	.ForMember(dest => dest.Total, 
    		opt => opt.ResolveUsing<CustomResolver>().ConstructedBy(() => new CustomResolver())
    	);
```

AutoMapper will execute this callback function instead of using reflection during the mapping operation, helpful in scenarios where the resolver might have constructor arguments or need to be constructed by an IoC container.
## Customizing the source value supplied to the resolver
Coming soon
## Custom value resolution expressions
Coming soon