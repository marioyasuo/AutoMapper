Automapper supports translating Expressions from one object to another.
This is done by substituting the properties from the source class to what they map to in the destination class.

Given the example classes:
```
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

public class OrderLineDTO
{
  public int Id { get; set; }
  public int OrderId { get; set; }
  public string Item { get; set; }
  public decimal Quantity { get; set; }
}

Mapper.Initialize(cfg => 
{
  Mapper.CreateMap<OrderLine, OrderLineDTO>()
    .ForMember(dto => dto.Item, conf => conf.MapFrom(ol => ol.Item.Name);
});
```
When mapping from DTO Expression
```
Expression<Func<OrderLineDTO, bool>> dtoExpression = dto=> dto.Item.StartsWith("A");
var expression = Mapper.Map<Func<Expression<OrderLine, bool>>>(dtoExpression);
```
Expression will bet translated to `ol => ol.Item.Name.StartsWith("A")`

Automapper knows `dto.Item` is mapped to `ol.Item.Name` so it substituted it for the expression.

Expression translation can work on expressions of collections as well.
```
Expression<Func<IQueryable<OrderLineDTO>,IQueryable<OrderLineDTO>>> dtoExpression = dtos => dtos.Where(dto => dto.Quantity > 5).OrderBy(dto => dto.Quantity);
var expression = Mapper.Map<Expression<Func<IQueryable<OrderLine>,IQueryable<OrderLine>>>(dtoExpression);
```
Resulting in `ols => ols.Where(ol => ol.Quantity > 5).OrderBy(ol => ol.Quantity)`

### Supported Mapping options

Much like how Queryable Extensions can only support certain things that the LINQ providers support, expression translation follows the same rules as what it can and can't support.

# UseAsDataSource
Mapping expressions to one another is a tedious and produces long ugly code.

`UseAsDataSource().For<DTO>()` makes this translation clean by not having to explicitly map expressions.
It also calls `Project().To<TDO>()` for you as well, where applicable.

Using EntityFramework as an example

`dataContext.OrderLines.UseAsDataSource().For<OrderLineDTO>().Where(dto => dto.Name.StartsWith("A"))`

Does the equivalent of 

`dataContext.OrderLines.Where(ol => ol.Item.Name.StartsWith("A")).Project().To<OrderLineDTO>()`

### When Project().To() is not called
Expression Translation work for all kinds of functions, including `Select` calls.  If `Select` is used after `UseAsDataSource()` and changes return type, then `Project().To<>()` won't be called and value with be returned instead using `Mapper.Map`.

Example:

`dataContext.OrderLines.UseAsDataSource().For<OrderLineDTO>().Select(dto => dto.Name)`

Does the equivalent of 

`dataContext.OrderLines.Select(ol => ol.Item.Name)`
