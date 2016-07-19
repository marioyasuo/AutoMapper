The 4.2.0 release of AutoMapper marked the entire static configuration and mapping API as obsolete. Typical usage from 4.1 and before was:

```
Mapper.Initialize(cfg => {
    cfg.AddProfile<AppProfile>();
    cfg.CreateMap<Source, Dest>();
});

var dest = Mapper.Map<Source, Dest>(new Source());
```

In 4.2 and later, this would look like:

```
var config = new MapperConfiguration(cfg => {
    cfg.AddProfile<AppProfile>();
    cfg.CreateMap<Source, Dest>();
});

var mapper = config.CreateMapper();
var dest = mapper.Map<Source, Dest>(new Source());
```

# Preserving static feel

The `MapperConfiguration` instance can be stored statically, as well as the `IMapper` instance:

```
MvcApplication.MapperConfiguration = new MapperConfiguration(cfg => {
    cfg.AddProfile<AppProfile>();
    cfg.CreateMap<Source, Dest>();
});

MvcApplication.Mapper = MvcApplication.MapperConfiguration.CreateMapper();

public ActionResult Index(int id) {
    var product = dbContext.Products.Where(p => p.Id == id).SingleOrDefault();
    var dto = MvcApplication.Mapper.Map<Product, ProductDto>(product);
    return View(dto);
}
```

Though typically you can configure this through dependency injection instead. With StructureMap 4.0:

```
public class AutoMapperRegistry : Registry
{
    public AutoMapperRegistry()
    {
        var profiles =
            from t in typeof (AutoMapperRegistry).Assembly.GetTypes()
            where typeof (Profile).IsAssignableFrom(t)
            select (Profile)Activator.CreateInstance(t);

        var config = new MapperConfiguration(cfg =>
        {
            foreach (var profile in profiles)
            {
                cfg.AddProfile(profile);
            }
        });

        For<MapperConfiguration>().Use(config);
        For<IMapper>().Use(ctx => ctx.GetInstance<MapperConfiguration>().CreateMapper(ctx.GetInstance));
    }
}
```

Then in your code using AutoMapper, for example in a controller:

```
public class ProductsController : Controller {
    public ProductsController(IMapper mapper) {
        this.mapper = mapper;
    }
    private IMapper mapper;

    public ActionResult Index(int id) {
        var product = dbContext.Products.Where(p => p.Id == id).SingleOrDefault();
        var dto = mapper.Map<Product, ProductDto>(product);
        return View(dto);
    }    
}
```

# LINQ projections

LINQ now requires us to pass in the MapperConfiguration instance:

```
public class ProductsController : Controller {
    public ProductsController(MapperConfiguration config) {
        this.config = config;
    }
    private MapperConfiguration config;

    public ActionResult Index(int id) {
        var dto = dbContext.Products
                               .Where(p => p.Id == id)
                               .ProjectTo<ProductDto>(config)
                               .SingleOrDefault();

        return View(dto);
    }    
}
```

# Unsupported operations

One "feature" of AutoMapper allowed you to modify configuration at runtime. That caused many problems, so the new API does not allow you to do this. You'll need to move all your `Mapper.CreateMap` calls into a profile, or into the construction of the `MapperConfiguration` object.

For dynamic mapping, such as `Mapper.DynamicMap`, you can configure AutoMapper to create missing maps as needed:

```
var config = new MapperConfiguration(cfg => cfg.CreateMissingTypeMaps = true);
```

Internally this uses conventions to create maps as necessary.