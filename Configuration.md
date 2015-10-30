## Initialization

Initialization is the preferred mode of configuring AutoMapper, and should be done once per AppDomain:

```csharp
Mapper.Initialize(cfg => {
    cfg.CreateMap<Foo, Bar>();
    cfg.AddProfile<FooProfile>();
});
```

Mapping configuration is static and should not change/be modified.

## Profile Instances
can be used to organize AutoMapper Configuration
````csharp
public class OrganizationProfile : Profile 
{
  protected override void Configure() 
  {
    //Put CreateMap... Etc.. here
  }
}
````

Initialize the profile like so 
```` csharp
Mapper.Initialize(cfg => {
  cfg.AddProfile<OrganizationProfile>();
});
````

## Naming Conventions
You can set the source and destination naming conventions
````csharp
Mapper.Initialize(cfg => {
  cfg.SourceMemberNamingConvention = new LowerUnderscoreNamingConvention();
  cfg.DestinationMemberNamingConvention = new PascalCaseNamingConvention();
});
````
This will map the following properties to each other: 
`  property_name -> PropertyName `

You can also set this at a per profile level 
````csharp
public class OrganizationProfile : Profile 
{
  protected override void Configure() 
  {
    //Put your Mapper.CreateMap... Etc.. here
    SourceMemberNamingConvention = new LowerUnderscoreNamingConvention();
    DestinationMemberNamingConvention = new PascalCaseNamingConvention();
  }
}
````

## Replacing characters
You can also replace individual characters or entire words in source members during member name matching:
```c#
public class Source
{
    public int Value { get; set; }
    public int Ävíator { get; set; }
    public int SubAirlinaFlight { get; set; }
}
public class Destination
{
    public int Value { get; set; }
    public int Aviator { get; set; }
    public int SubAirlineFlight { get; set; }
}
```
We want to replace the individual characters, and perhaps translate a word:
```c#
Mapper.Initialize(c =>
{
    c.ReplaceMemberName("Ä", "A");
    c.ReplaceMemberName("í", "i");
    c.ReplaceMemberName("Airlina", "Airline");
});
```
## Recognizing pre/postfixes

Sometimes your source/destination properties will have common pre/postfixes that cause you to have to do a bunch of custom member mappings because the names don't match up. To address this, you can recognize pre/postixes:

```c#
public class Source {
    public int frmValue { get; set; }
    public int frmValue2 { get; set; }
}
public class Dest {
    public int Value { get; set; }
    public int Value2 { get; set; }
}
Mapper.Initialize(cfg => {
    cfg.RecognizePrefix("frm");
    cfg.CreateMap<Source, Dest>();
});
Mapper.AssertConfigurationIsValid();
```

By default AutoMapper recognizes the prefix "Get", if you need to clear the prefix:

```c#
Mapper.Initialize(cfg => {
    cfg.ClearPrefixes();
    cfg.RecognizePrefixes("tmp");
});
```

## Global property/field filtering

By default, AutoMapper tries to map every public property/field. You can filter out properties/fields with the property/field filters:

```c#
Mapper.Initialize(cfg =>
{
	// don't map any fields
	cfg.ShouldMapField = fi => false;

	// map properties with a public or private getter
	cfg.ShouldMapProperty = pi =>
		pi.GetMethod != null && (pi.GetMethod.IsPublic || pi.GetMethod.IsPrivate);
});
```

## Configuring visibility

By default, AutoMapper only recognizes public members. It can map to private setters, but will skip internal/private methods and properties if the entire property is private/internal. To instruct AutoMapper to recognize members with other visibilities, override the default filters ShouldMapField and/or ShouldMapProperty :
```c#
Mapper.Initialize(cfg =>
{
    // map properties with public or internal getters
    cfg.ShouldMapProperty = p => p.GetMethod.IsPublic || p.GetMethod.IsAssembly;
    cfg.CreateMap<Source, Destination>();
});
```
Map configurations will now recognize internal/private members.