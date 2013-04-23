When using an ORM such as NHibernate or Entity Framework with Automapper's standard `Mapper.Map` functions, you may notice that the ORM will query all the fields of all the objects within a graph when Automapper is attempting to map the results to a destination type.

If your ORM exposes `IQueryable`s, you can use Automapper's QueryableExtensions helper methods to address this key pain.

Using Entity Framework for an example, say that you have an entity `OrderLine` with a relationship with an entity `Item`. If you want to map this to an `OrderLineDTO` with the `Item`'s `Name` property, the standard `Mapper.Map` call will result in Entity Framework querying the entire `OrderLine` and `Item` table. 

Use this approach instead.

Given the following entities:

    public class OrderLine
    {
      public int Id { get; set; }
      public int OrderId { get; set; }
      public Item Item { get; set; }
      public decimal Quantity { get; set; }
    }

    public class Item
    {
      public int Id { get; set; }
      public string Name { get; set; }
    }

And the following DTO:

    public class OrderLineDTO
    {
      public int Id { get; set; }
      public int OrderId { get; set; }
      public string Item { get; set; }
      public decimal Quantity { get; set; }
    }

You can use the Queryable Extensions like so:

    public List<OrderLineDTO> GetLinesForOrder(int orderId)
    {
      Mapper.CreateMap<OrderLine, OrderLineDTO>()
        .ForMember(dto => dto.Item, conf => conf.MapFrom(ol => ol.Item.Name);
    
      using (var context = new orderEntities())
      {
        return context.OrderLines.Where(ol => ol.OrderId == orderId)
                 .Project().To<OrderLineDTO>().ToList();
      }
    }

The `.Project().To<OrderLineDTO>()` will tell AutoMapper's mapping engine to emit a `select` clause to the IQueryable that will inform entity framework that it only needs to query the Name column of the Item table, same as if you manually projected your `IQueryable` to an `OrderLineDTO` with a `Select` clause. 

Note that for this feature to work, all type conversions must be explicitly handled in your Mapping. For example, you can not rely on the `ToString()` override of the `Item` class to inform entity framework to only select from the `Name` column, and any data type changes, such as `Double` to `Decimal` must be explicitly handled as well. 