Projection transforms a source to a destination beyond flattening the object model.  Without extra configuration, AutoMapper requires a flattened destination to match the source type's naming structure.  When you want to project source values into a destination that does not exactly match the source structure, you must specify custom member mapping definitions.  For example, we might want to turn this source structure:

    public class CalendarEvent
    {
    	public DateTime EventDate { get; set; }
    	public string Title { get; set; }
    }

Into something that works better for an input form on a web page:

    public class CalendarEventForm
    {
    	public DateTime EventDate { get; set; }
    	public int EventHour { get; set; }
    	public int EventMinute { get; set; }
    	public string Title { get; set; }
    }

Because the names of the destination properties do not exactly match up to the source property (CalendarEventForm.EventDate would need to be CalendarEventForm.EventDateDate), we need to specify custom member mappings in our type map configuration:

    // Model
    var calendarEvent = new CalendarEvent
    	{
    		EventDate = new DateTime(2008, 12, 15, 20, 30, 0),
    		Title = "Company Holiday Party"
    	};
    
    // Configure AutoMapper
    Mapper.CreateMap<CalendarEvent, CalendarEventForm>()
    	.ForMember(dest => dest.EventDate, opt => opt.MapFrom(src => src.EventDate.Date))
    	.ForMember(dest => dest.EventHour, opt => opt.MapFrom(src => src.EventDate.Hour))
    	.ForMember(dest => dest.EventMinute, opt => opt.MapFrom(src => src.EventDate.Minute));
    
    // Perform mapping
    CalendarEventForm form = Mapper.Map<CalendarEvent, CalendarEventForm>(calendarEvent);
    
    form.EventDate.ShouldEqual(new DateTime(2008, 12, 15));
    form.EventHour.ShouldEqual(20);
    form.EventMinute.ShouldEqual(30);
    form.Title.ShouldEqual("Company Holiday Party");

The each custom member configuration uses an action delegate to configure each member.  In the above example, we used the MapFrom option to perform custom source/destination member mappings.  The MapFrom method takes a lambda expression as a parameter, which then evaluated later during mapping.  The MapFrom expression can be any Func<TSource, object> lambda expression.