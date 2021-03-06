# AutoMapper Self Config

| DEV |MASTER|BLEEDING|NUGET|
|-----|------|--------|-----|
|[![CI status][1]][2]|[![Release Build status][3]][4]|[![MyGet CI][5]][6]|[![NuGet CI][7]][8]|

> Updated to AutoMapper 5 in version 1.0.0

I help create mappings for AutoMapper based on interfaces.

Based on code presented on Pluralsight by @matthoneycutt. He blogs at http://trycatchfail.com/blog.

> From version 0.10.0 makes use of AutoMapper new instance based API. Use version 0.9.0 if need to to support AutoMapper 4.1.1

## Usage

The library provides 3 interfaces and some helper methods.

Install with the following nuget command:
> `Install-Package AutoMapper.SelfConfig`

### IMapFrom<T>

```csharp   
public interface IMapFrom<T>
{}
```    
This is placed on a class to mark that an AutoMapper configuration be created mapping *from* T to the current class.

### IMapTo<T>

```csharp   
public interface IMapTo<T>
{}
```
  
This is the reverse of IMapFrom. This is placed on a class to mark that an AutoMapper configuration be created mapping *to* T from the current class.

The advantage with the 2 way mapping is you can place all mappings on say your **view models** and leave your ** domain models** free of mapping clutter while still keeping them close at hand with the files you working on rather than off in a seperate mapping Profile.

### IHaveCustomMapping

```csharp
public interface IHaveCustomMappings
{
    void CreateMappings(IMapperConfigurationExpression configuration);
}
```

> UPDATE: Parameter type changed from `IConfiguration` to `IMapperConfiguration` from version 0.10.0
> UPDATE: Parameter type changed from `IConfiguration` to `IMapperConfiguration` from version 1.0.0

This is for when more complex mappings are required and allows you to directly interact with the [AutoMapper IConfiguration](https://github.com/AutoMapper/AutoMapper/wiki/Configuration).


### Example Usage

```csharp
public class Source
{
    public int Nr { get; set; }
    public string Name { get; set; }
}

public class TestEasy : IMapTo<Source>, IMapFrom<Source>
{
    public int Nr { get; set; }
    public string Name { get; set; }
}
```

Then the mappings are created as such:

```csharp
var types = new Type[] { typeof(TestEasy) };
MappingLoader.LoadStandardMappings(types);
```

This would create a 2 mappings in AutoMapper *TestEasy -> Source* and *Source -> TestEasy*.

If you need finer control of the mappings above AutoMappers defaults you can use IHaveCustomMApping interface:

```csharp
public class TestConfig : IHaveCustomMappings
{
    public int Number { get; set; }
    public string Name { get; set; }

    public void CreateMappings(IMapperConfigurationExpression configuration)
    {
        configuration.CreateMap<Source, TestConfig>().ForMember(dest => dest.Number, opt => opt.MapFrom(src => src.Nr));
        configuration.CreateMap<TestConfig, Source>().ForMember(dest => dest.Nr, opt => opt.MapFrom(src => src.Number)); ;
    }
}
```

Loading both the simple and complex maps is now as simple as:

```csharp
var types = new Type[] { typeof(TestEasy), typeof(TestConfig) };
MappingLoader.LoadAllMappings(types);
```

### Real World Example

> Update: `MappingLoader` changed to `MappingConfigFactory` at version 0.10.0

The command class below is taken from a real project and uses the [My Reflection Library](https://github.com/dburriss/PhilosophicalMonkey) and [My ASP.NET Lifecycle Middleware](https://github.com/dburriss/AspNetLifecycle) To load up all mappings at startup of a ASP.NET application.

```csharp
public class AutoMapperSetupTask : IRunAtStartup
{
    public void Execute()
    {
        var seedTypes = new Type[] { typeof(Startup) };
        var assemblies = Reflect.OnTypes.GetAssemblies(seedTypes);
        var typesInAssemblies = Reflect.OnTypes.GetAllExportedTypes(assemblies);
        AutoMapper.SelfConfig.MappingConfigFactory.LoadAllMappings(typesInAssemblies);
    }
}
```

[1]: https://ci.appveyor.com/api/projects/status/b5htqjta1pw7gb6n?svg=true
[2]: https://ci.appveyor.com/project/dburriss/automapperselfconfig
[3]: https://ci.appveyor.com/api/projects/status/b5htqjta1pw7gb6n/branch/master?svg=true
[4]: https://ci.appveyor.com/project/dburriss/automapperselfconfig/branch/master
[5]: https://img.shields.io/myget/dburriss-ci/vpre/AutoMapper.SelfConfig.svg
[6]: http://myget.org/gallery/dburriss-ci
[7]: https://img.shields.io/nuget/v/AutoMapper.SelfConfig.svg
[8]: https://www.nuget.org/packages/AutoMapper.SelfConfig/
