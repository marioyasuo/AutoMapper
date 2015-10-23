When using an ORM such as NHibernate or Entity Framework with AutoMapper's standard `Mapper.Map` functions, you may notice that the ORM will query all the fields of all the objects within a graph when AutoMapper is attempting to map the results to a destination type.

If your ORM exposes `IQueryable`s, you can use AutoMapper's QueryableExtensions helper methods to address this key pain.

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
                 .ProjectTo<OrderLineDTO>().ToList();
      }
    }

The `.ProjectTo<OrderLineDTO>()` will tell AutoMapper's mapping engine to emit a `select` clause to the IQueryable that will inform entity framework that it only needs to query the Name column of the Item table, same as if you manually projected your `IQueryable` to an `OrderLineDTO` with a `Select` clause. 

Note that for this feature to work, all type conversions must be explicitly handled in your Mapping. For example, you can not rely on the `ToString()` override of the `Item` class to inform entity framework to only select from the `Name` column, and any data type changes, such as `Double` to `Decimal` must be explicitly handled as well.

### Preventing lazy loading/SELECT N+1 problems

Because the LINQ projection built by AutoMapper is translated directly to a SQL query by the query provider, the mapping occurs at the SQL/ADO.NET level, and not touching your entities. All data is eagerly fetched and loaded into your DTOs.

Nested collections use a Select to project child DTOs:
```
from i in db.Instructors
orderby i.LastName
select new InstructorIndexData.InstructorModel
{
    ID = i.ID,
    FirstMidName = i.FirstMidName,
    LastName = i.LastName,
    HireDate = i.HireDate,
    OfficeAssignmentLocation = i.OfficeAssignment.Location,
    Courses = i.Courses.Select(c => new InstructorIndexData.InstructorCourseModel
    {
        CourseID = c.CourseID,
        CourseTitle = c.Title
    }).ToList()
};
```
This map through AutoMapper will result in a SELECT N+1 problem, as each child `Course` will be queried one at a time, unless specified through your ORM to eagerly fetch. With LINQ projection, no special configuration or specification is needed with your ORM. The ORM uses the LINQ projection to build the exact SQL query needed.

### Custom projection

In the case where members names don't line up, or you want to create calculated property, you can use MapFrom (and not ResolveUsing) to supply a custom expression for a destination member:

    Mapper.CreateMap<Customer, CustomerDto>()
        .ForMember(d => d.FullName, opt => opt.MapFrom(c => c.FirstName + " " + c.LastName))
        .ForMember(d => d.TotalContacts, opt => opt.MapFrom(c => c.Contacts.Count()));

AutoMapper passes the supplied expression with the built projection. As long as your query provider can interpret the supplied expression, everything will be passed down all the way to the database.

If the expression is rejected from your query provider (Entity Framework, NHibernate, etc.), you might need to tweak your expression until you find one that is accepted.

### Custom Type Conversion

Occasionally, you need to completely replace a type conversion from a source to a destination type. In normal runtime mapping, this is accomplished via the ConvertUsing method. To perform the analog in LINQ projection, use the ProjectUsing method:
```
Mapper.CreateMap<Source, Dest>().ProjectUsing(src => new Dest { Value = 10 });
```
`ProjectUsing` is slightly more limited than `ConvertUsing` as only what is allowed in an Expression and the underlying LINQ provider will work.

### Custom destination type constructors

If your destination type has a custom constructor but you don't want to override the entire mapping, use the ConstructProjectionUsing method:
```c#
Mapper.CreateMap<Source, Dest>()
    .ConstructProjectionUsing(src => new Dest(src.Value + 10));
```
AutoMapper will automatically match up destination constructor parameters to source members based on matching names, so only use this method if AutoMapper can't match up the destination constructor properly, or if you need extra customization during construction.

### String conversion

AutoMapper will automatically add `ToString()` when the destination member type is a string and the source member type is not.

```c#
public class Order {
    public OrderTypeEnum OrderType { get; set; }
}
public class OrderDto {
    public string OrderType { get; set; }
} 
var orders = dbContext.Orders.ProjectTo<OrderDto>().ToList();
orders[0].OrderType.ShouldEqual("Online");
```

### Explicit expansion

In some scenarios, such as OData, a generic DTO is returned through an IQueryable controller action. Without explicit instructions, AutoMapper will expand all members in the result. To control which members are expanded during projection, pass in the members you want to explicitly expand:
```c#
dbContext.Orders.ProjectTo<OrderDto>(
    parameters = null,
    dest => dest.Customer,
    dest => dest.LineItems);
// or string-based
dbContext.Orders.ProjectTo<OrderDto>()
    parameters = null,
    "Customer",
    "LineItems");
```

### Aggregations

LINQ can support aggregate queries, and AutoMapper supports LINQ extension methods. In the custom projection example, if we renamed the `TotalContacts` property to `ContactsCount`, AutoMapper would match to the `Count()` extension method and the LINQ provider would translate the count into a correlated subquery to aggregate child records.

AutoMapper can also support complex aggregations and nested restrictions, if the LINQ provider supports it:
```
Mapper.CreateMap<Course, CourseModel>()
    .ForMember(m => m.EnrollmentsStartingWithA,
          opt => opt.MapFrom(c => c.Enrollments.Where(e => e.Student.LastName.StartsWith("A")).Count()));
```
This query returns the total number of students, for each course, whose last name starts with the letter 'A'.

### Parameterization
Occasionally, projections need runtime parameters for their values. Consider a projection that needs to pull in the current username as part of its data. Instead of using post-mapping code, we can parameterize our MapFrom configuration:
```
string currentUserName = null;
Mapper.CreateMap<Course, CourseModel>()
    .ForMember(m => m.CurrentUserName, opt => opt.MapFrom(src => currentUserName));
```
When we project, we'll substitute our parameter at runtime:
```
dbContext.Courses.ProjectTo<CourseModel>(new { currentUserName = Request.User.Name });
```
This works by capturing the name of the closure's field name in the original expression, then using an anonymous object/dictionary to apply the value to the parameter value before the query is sent to the query provider.
### Supported mapping options
Not all mapping options can be supported, as the expression generated must be interpreted by a LINQ provider. Only what is supported by LINQ providers is supported by AutoMapper:
* MapFrom
* Ignore
* UseValue
* NullSubstitute

Not supported:
* Condition
* DoNotUseDestinationValue
* SetMappingOrder
* UseDestinationValue
* ResolveUsing
* Before/AfterMap
* Custom resolvers
* Custom type converters
* **Any calculated property on your domain object**

Additionally, recursive or self-referencing destination types are not supported as LINQ providers do not support this. Typically hierarchical relational data models require common table expressions (CTEs) to correctly resolve a recursive join.