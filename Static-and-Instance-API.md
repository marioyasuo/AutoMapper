In 4.2.1 version of AutoMapper and later, AutoMapper provides two APIs: a static and an instance API. The static API:

```
Mapper.Initialize(cfg => {
    cfg.AddProfile<AppProfile>();
    cfg.CreateMap<Source, Dest>();
});

var dest = Mapper.Map<Source, Dest>(new Source());
```

And the instance API:

```
var config = new MapperConfiguration(cfg => {
    cfg.AddProfile<AppProfile>();
    cfg.CreateMap<Source, Dest>();
});

var mapper = config.CreateMapper();
// or
var mapper = new Mapper(config);
var dest = mapper.Map<Source, Dest>(new Source());
```

# LINQ projections

For the instance method of using AutoMapper, LINQ now requires us to pass in the MapperConfiguration instance:

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

One "feature" of AutoMapper allowed you to modify configuration at runtime. That caused many problems, so the new static API does not allow you to do this. You'll need to move all your `Mapper.CreateMap` calls into a profile, and into a `Mapper.Initailize`.

For dynamic mapping, such as `Mapper.DynamicMap`, you can configure AutoMapper to create missing maps as needed:

```
Mapper.Initialize(cfg => cfg.CreateMissingTypeMaps = true);
```

Internally this uses conventions to create maps as necessary.