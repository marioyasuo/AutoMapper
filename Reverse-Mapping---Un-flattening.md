Starting with 6.1.0, AutoMapper now supports richer reverse mapping support. Given our entities:

```c#
public class Order {
  public decimal Total { get; set; }
  public Customer Customer { get; set; } 
}
public class Customer {
  public string Name { get; set; }
}
```

We can flatten this into a DTO:

```c#
public class OrderDto {
  public decimal Total { get; set; }
  public string CustomerName { get; set; }
}
```

We can map both directions, including unflattening:

```c#
Mapper.Initialize(cfg => {
  cfg.CreateMap<Order, OrderDto>()
     .ReverseMap();
});
```

By calling `ReverseMap`, AutoMapper creates a reverse mapping configuration that includes unflattening:

```c#

```
