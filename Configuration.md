## Profile Instances
can be used to organize AutoMapper Configuration
````csharp
public class OrganizationProfile : Profile 
{
  protected override void Configure() 
  {
    //Put your Mapper.CreateMap... Etc.. here
  }

  public override string ProfileName  
  { 
    get { return this.GetType().Name; } 
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