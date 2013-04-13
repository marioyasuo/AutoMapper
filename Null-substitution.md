Null substitution allows you to supply an alternate value for a destination member if the source value is null anywhere along the member chain.

    Mapper.CreateMap<Source, Dest>()
        .ForMember(dest => dest.Value, opt => opt.NullSubstitute("Other Value"));

    var source = new Source { Value = null };
    
    var dest = Mapper.Map<Source, Dest>(source);

    dest.Value.ShouldEqual("Other Value");

    source.Value = "Not null";

    dest = Mapper.Map<Source, Dest>(source);

    dest.Value.ShouldEqual("Not null");