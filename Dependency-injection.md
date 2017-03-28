AutoMapper supports the ability to construct [[Custom Value Resolvers]] and [[Custom Type Converters]] using static service location:
```c#
    Mapper.Initialize(cfg =>
    {
        cfg.ConstructServicesUsing(ObjectFactory.GetInstance);
        
        cfg.CreateMap<Source, Destination>();
    });
```
Or dynamic service location, to be used in the case of instance-based containers (including child/nested containers):
```c#
    var mapper = new Mapper(Mapper.Configuration, childContainer.GetInstance);

    var dest = mapper.Map<Source, Destination>(new Source { Value = 15 });
```

For those using Ninject here is an example of a Ninject module for AutoMapper 
```c#
public class AutoMapperModule : NinjectModule
{
    public override void Load()
    {
        Bind<IValueResolver<SourceEntity, DestModel, bool>>().To<MyResolver>();

        var mapperConfiguration = CreateConfiguration();
        Bind<MapperConfiguration>().ToConstant(mapperConfiguration).InSingletonScope();

        // This teaches Ninject how to create automapper instances say if for instance
        // MyResolver has a constructor with a parameter that needs to be injected
        Bind<IMapper>().ToMethod(ctx => 
             new Mapper(mapperConfiguration, type => ctx.Kernel.Get(type)));
    }

    private MapperConfiguration CreateConfiguration()
    {
        var config = new MapperConfiguration(cfg =>
        {
            // Add all profiles in current assembly
            cfg.AddProfiles(GetType().Assembly);
        });

        return config;
    }
}
```

# ASP.NET Core

There is a [NuGet package](https://www.nuget.org/packages/AutoMapper.Extensions.Microsoft.DependencyInjection/) to be used with the default injection mechanism described [here](https://lostechies.com/jimmybogard/2016/07/20/integrating-automapper-with-asp-net-core-di/).