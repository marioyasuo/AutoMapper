Profile Instances
Can be used to organize AutoMapper Configuration
````c#
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
````v#
Mapper.Initialize(cfg => {
  cfg.AddProfile<OrganizationProfile>();
});
