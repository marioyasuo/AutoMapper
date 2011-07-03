# Getting Started
## What is AutoMapper?

AutoMapper is an object-object mapper.  Object-object mapping works by transforming an input object of one type into an output object of a different type.  What makes AutoMapper interesting is that it provides some interesting conventions to take the dirty work out of figuring out how to map type A to type B.  As long as type B follows AutoMapper's established convention, almost zero configuration is needed to map two types.

## Why use AutoMapper?

Mapping code is boring.  Testing mapping code is even more boring.  AutoMapper provides simple configuration of types, as well as simple testing of mappings.  The real question may be "why use object-object mapping?"  Mapping can occur in many places in an application, but mostly in the boundaries between layers, such as between the UI/Domain layers, or Service/Domain layers.  Concerns of one layer often conflict with concerns in another, so object-object mapping leads to segregated models, where concerns for each layer can affect only types in that layer.

## How do I use AutoMapper?

First, you need both a source and destination type to work with.  The destination type's design can be influenced by the layer in which it lives, but AutoMapper works best as long as the names of the members match up to the source type's members.  If you have a source member called "FirstName", this will automatically be mapped to a destination member with the name "FirstName".  AutoMapper also supports [[Flattening]], which can get rid of all those pesky null reference exceptions you might encounter along the way.

Once you have your types, and a reference to AutoMapper, you can create a map for the two types.

    Mapper.CreateMap<Order, OrderDto>();

The type on the left is the source type, and the type on the right is the destination type.  To perform a mapping, use the Map method.

    OrderDto dto = Mapper.Map<Order, OrderDto>(order);

AutoMapper also has non-generic versions of these methods, for those cases where you might not know the type at compile time.

## Where do I configure AutoMapper?

If you're using the static Mapper method, configuration only needs to happen once per AppDomain.  That means the best place to put the configuration code is in application startup, such as the Global.asax file for ASP.NET applications.  Typically, the configuration bootstrapper class is in its own class, and this bootstrapper class is called from the startup method.

## How do I test my mappings?

To test your mappings, you need to create a test that does two things:
* Call your bootstrapper class to create all the mappings
* Call Mapper.AssertConfigurationIsValid

Here's an example:

    AutoMapperConfiguration.Configure();
    
    Mapper.AssertConfigurationIsValid();


## Can I see a demo?

Check out the [dnrTV episode on AutoMapper](http://www.dnrtv.com/default.aspx?showNum=155).