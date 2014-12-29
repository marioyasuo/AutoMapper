# Mapping Inheritance
AutoMapper 1.1 had a method called .Include<> when creating your maps which allowed AutoMapper to automatically select the most derived mapping for a class.

Take:
```c#
    public class Order { }
    public class OnlineOrder : Order { }
    public class MailOrder : Order { }

    public class OrderDto { }
    public class OnlineOrderDto : OrderDto { }
    public class MailOrderDto : OrderDto { }

    Mapper.CreateMap<Order, OrderDto>()
          .Include<OnlineOrder, OnlineOrderDto>()
          .Include<MailOrder, MailOrderDto>();
    Mapper.CreateMap<OnlineOrder, OnlineOrderDto>();
    Mapper.CreateMap<MailOrder, MailOrderDto>();

    // Perform Mapping
    var order = new OnlineOrder();
    var mapped = Mapper.Map(order, order.GetType(), typeof(OrderDto));
    Assert.IsType<OnlineOrderDto>(mapped);
```
You will notice that because the mapped object is a OnlineOrder, AutoMapper has seen you have a more specific mapping for OnlineOrder than OrderDto, and automatically chosen that.
The shortcoming of AutoMapper 1.1 was that any mapping configuration you made on the base mapping had to be recreated on each of the child mappings. This ends up with duplicated configuration and lots of extra mapping code.
```c#
    Mapper.CreateMap<Order, OrderDto>()
          .Include<OnlineOrder, OnlineOrderDto>()
          .Include<MailOrder, MailOrderDto>()
          .ForMember(o=>o.Id, m=>m.MapFrom(s=>s.OrderId));
    Mapper.CreateMap<OnlineOrder, OnlineOrderDto>()
          .ForMember(o=>o.Id, m=>m.MapFrom(s=>s.OrderId));
    Mapper.CreateMap<MailOrder, MailOrderDto>()
          .ForMember(o=>o.Id, m=>m.MapFrom(s=>s.OrderId));
```
# Mapping Configuration Inheritance in AutoMapper 2.0
In AutoMapper 2.0, this becomes:
```c#
    Mapper.CreateMap<Order, OrderDto>()
          .Include<OnlineOrder, OnlineOrderDto>()
          .Include<MailOrder, MailOrderDto>()
          .ForMember(o=>o.Id, m=>m.MapFrom(s=>s.OrderId));
    Mapper.CreateMap<OnlineOrder, OnlineOrderDto>();
    Mapper.CreateMap<MailOrder, MailOrderDto>();
```
Because we have defined the mapping for the base class, we no longer have to define it in the child mappings.

# Specifying inheritance in derived classes
Instead of configuring inheritance from the base class, you can specify inheritance from the derived classes:
```c#
Mapper.CreateMap<Order, OrderDto>()
    .ForMember(o => o.Id, m => m.MapFrom(s => s.OrderId));
Mapper.CreateMap<OnlineOrder, OnlineOrderDto>()
    .IncludeBase<Order, OrderDto>();
Mapper.CreateMap<MailOrder, MailOrderDto>()
    .IncludeBase<Order, OrderDto>();
```
## Inheritance Mapping Priorities
This introduces additional complexity because there are multiple ways a property can be mapped. The priority of these sources are as follows

 - Explicit Mapping (using .MapFrom())
 - Inherited Explicit Mapping
 - Ignore Property Mapping
 - Convention Mapping (Properties that are matched via convention)

To demonstrate this, lets modify our classes shown above
```c#
    //Domain Objects
    public class Order { }
    public class OnlineOrder : Order 
    { 
        public string Referrer { get; set; }
    }
    public class MailOrder : Order { }

    //Dtos
    public class OrderDto
    {
        public string Referrer { get; set; }
    }

    //Mappings
    Mapper.CreateMap<Order, OrderDto>()
          .Include<OnlineOrder, OrderDto>()
          .Include<MailOrder, OrderDto>()
          .ForMember(o=>o.Referrer, m=>m.Ignore());
    Mapper.CreateMap<OnlineOrder, OrderDto>();
    Mapper.CreateMap<MailOrder, OrderDto>();

    // Perform Mapping
    var order = new OnlineOrder { Referrer = "google" };
    var mapped = Mapper.Map(order, order.GetType(), typeof(OrderDto));
    Assert.Equals("google", mapped.Referrer);
```
Notice that in our mapping configuration, we have ignored Referred (because it doesn't exist in the order base class), but convention has a higher priority than Ignored properties in the base class mappings, so the property still gets mapped.

Overall this feature should make using AutoMapper with classes that leverage inheritance feel more natural.