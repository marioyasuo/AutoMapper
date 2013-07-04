# Profile Instances
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
````csharp
Mapper.Initialize(cfg => {
  cfg.AddProfile<OrganizationProfile>();
});
