Hand-rolled mapping code, though tedious, has the advantage of being testable.  One of the inspirations behind AutoMapper was to eliminate not just the custom mapping code, but eliminate the need for manual testing.  Because the mapping from source to destination is convention-based, you will still need to test your configuration.

AutoMapper provides configuration testing in the form of the AssertConfigurationIsValid method.  Suppose we have slightly misconfigured our source and destination types:
```csharp
    public class Source
    {
    	public int SomeValue { get; set; }
    }
    
    public class Destination
    {
    	public int SomeValuefff { get; set; }
    }
```
In the Destination type, we probably fat-fingered the destination property.  Other typical issues are source member renames.  To test our configuration, we simply create a unit test that sets up the configuration and executes the AssertConfigurationIsValid method:
```csharp
    Mapper.Initialize(cfg => 
      cfg.CreateMap<Source, Destination>());
    
    Mapper.AssertConfigurationIsValid();
```
Executing this code produces an AutoMapperConfigurationException, with a descriptive message.  AutoMapper checks to make sure that *every single* Destination type member has a corresponding type member on the source type.
# Overriding configuration errors
To fix a configuration error (besides renaming the source/destination members), you have three choices for providing an alternate configuration:

* Custom value resolver
* [[Projection]]
* Use the Ignore() option

With the third option, we have a member on the destination type that we will fill with alternative means, and not through the Map operation.
```csharp
    Mapper.Initialize(cfg => 
      cfg.CreateMap<Source, Destination>()
    	.ForMember(dest => dest.SomeValuefff, opt => opt.Ignore())
    );
```