using AutoMapper;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace UnitTestProject1
{
    [TestClass]
    public class AutomapperTests
    {
        public class Person
        {
            public int Code { get; set; }
            public string FirstName { get; set; }
            public string LastName { get; set; }
        }

        public class PersonDTO
        {
            public int Code { get; set; }
            public string FirstName { get; set; }
            public string LastName { get; set; }
        }

        [TestMethod]
        public void TestNullIgnore()
        {
            var config = new MapperConfiguration(cfg =>
            {
                cfg.CreateMap<PersonDTO, Person>()
                    .ForAllMembers(opt => opt.Condition((src, p, srvVal, destVal) => srvVal != null));

            });

            config.AssertConfigurationIsValid();

            var mapper = config.CreateMapper();
            
            var sourcePerson = new Person
            {
                FirstName = "Bill",
                LastName = "Gates",
                Code = 10
            };
            var destinationPersonDto = new PersonDTO
            {
                FirstName = null,
                LastName = "Smith",
                Code = 10
            };

            mapper.Map(destinationPersonDto, sourcePerson);

            Assert.AreNotEqual(sourcePerson.FirstName,null);
            Assert.AreEqual(sourcePerson.LastName, "Smith");
        }
    }
}
