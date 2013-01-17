AutoMapper supports the ability to construct [[Custom Value Formatters]], [[Custom Value Resolvers]] and [[Custom Type Converters]] using static service location:
```c#
    Mapper.Initialize(cfg =>
    {
        cfg.ConstructServicesUsing(ObjectFactory.GetInstance);
        
        cfg.CreateMap<Source, Destination>();
    });
```
Or dynamic service location, to be used in the case of instance-based containers (including child/nested containers):
```c#
    var dest = Mapper.Map<Source, Destination>(new Source { Value = 15 },
        opt => opt.ConstructServicesUsing(childContainer.GetInstance));
```