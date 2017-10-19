AutoMapper allows you to add conditions to properties that must be met before that property will be mapped. 

This can be used in situations like the following where we are trying to map from an int to an unsigned int.
````
class Foo{
  public int baz;
}

class Bar { 
  public uint baz; 
}
````
In the following mapping the property baz will only be mapped if it is greater than or equal to 0 in the source object.
````
Mapper.Initialize(cfg => {
  cfg.CreateMap<Foo,Bar>()
    .ForMember(dest => dest.baz, opt => opt.Condition(src => (src.baz >= 0))); 
});
````

### Resolving the destination value still has to work

For each property mapping, AutoMapper attempts to resolve the destination value **before** evaluating the condition. So it needs to be able to do that without throwing an exception even if the condition will prevent the resulting value from being used.

As an example, here's sample output from [BuildExecutionPlan](https://github.com/AutoMapper/AutoMapper/wiki/Understanding-your-mapping) (processed using [ReadableExpressions](https://www.nuget.org/packages/AgileObjects.ReadableExpressions)) for a single property:

````
try
{
	var resolvedValue =
	{
		try
		{
			return // ... tries to resolve the destination value here
		}
		catch (NullReferenceException)
		{
			return null;
		}
		catch (ArgumentNullException)
		{
			return null;
		}
	};

	if (condition.Invoke(src, typeMapDestination, resolvedValue))
	{
		typeMapDestination.WorkStatus = resolvedValue;
	}
}
catch (Exception ex)
{
	throw new AutoMapperMappingException(
		"Error mapping types.",
		ex,
		AutoMapper.TypePair,
		AutoMapper.TypeMap,
		AutoMapper.PropertyMap);
};
````
The default generated code for resolving a property, if you haven't customized the mapping for that member, generally doesn't have any problems.  But if you're using custom code to map the property that will crash if the condition isn't met, the mapping will fail despite the condition.

This example code would fail:

````
public class SourceClass 
{ 
	public string Value { get; set; }
}

public class TargetClass 
{
	public int ValueLength { get; set; }
}

// ...

var source = new SourceClass { Value = null };
var target = new TargetClass;

CreateMap<SourceClass, TargetClass>()
	.ForMember(d => d.ValueLength, o => o.MapFrom(s => s.Value.Length))
	.ForAllMembers(o => o.Condition((src, dest, value) => value != null));
````
The condition prevents the Value property from being mapped onto the target, but the custom member mapping would fail before that point because it calls Value.Length, and Value is null. 

Prevent this by ensuring the custom member mapping code can complete safely regardless of conditions:

```
	.ForMember(d => d.ValueLength, o => o.MapFrom(s => s != null ? s.Value.Length : 0))
```

## Preconditions
Similarly, there is a precondition. The difference is that it runs sooner in the mapping process, before the source value is resolved (think MapFrom or ResolveUsing). So the precondition is called, then we decide which will be the source of the mapping (resolving), then the condition is called and finally the destination value is assigned. You can [see the steps](https://github.com/AutoMapper/AutoMapper/wiki/Understanding-your-mapping) yourself.