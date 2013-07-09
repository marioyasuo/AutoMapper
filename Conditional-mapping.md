AutoMapper allows you to add conditions to properties that must be met before that property will be mapped. 
This can be used in situations like the following where we are trying to map from an int to an unsigned int.
````
class Foo{
  int baz;
}

class Bar { 
  uint baz; 
}

Mapper.CreateMap<Foo,Bar>()
  .ForMember(dest => dest.baz, opt => opt.Condition(src => (src.baz >= 0)); 

````