---
title: "Hide a configuration section behind an interface"
datePublished: Mon Jan 16 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iubzf00xsyes1adx1463a
slug: hide-a-configuration-section-behind-an-interface
tags: csharp, dotnet, dotnetcore

---


In my previous project, the team used custom configuration sections for different parts of the application. Because each class had access to all information, I added an interface for each service with just the information that service would need.

For example, there was a class that contained all configuration properties. That class was then passed to all code that needed to access external services. Each class had access to all configuration options, even options for other services and could update those values. I added an interface per external service. The configuration class then implemented those interfaces to expose only the information the service needed.

This approach has the advantage that you only see the properties and functions you need in that particular bit of code. You can also set one custom setting and return it in different interfaces. When you change the setting, it's passed along to each interface. This does mean a bit of duplication because for each interface there will be a property that will have to return the setting.

In this example I will implement a custom configuration setting with two properties and one custom configuration element. For more information about creating a custom configuration element, check the [Microsoft documentation](https://msdn.microsoft.com/en-us/library/2tw134k3.aspx). All of the code in this article can be found in [this GitHub repository](https://github.com/KenBonny/CustomConfigurationSection).

## Creating the custom section

Creating a custom configuration section is not difficult, just inherit from `System.Configuration.ConfigurationSection` and add some properties with the attribute `ConfigurationProperty`. It is possible to do it like this and then hide that class behind an implementation of the interface (which I'll create later). Let's take the not so conventional way and do it a little compacter.

```
class SpecificConfiguration : ConfigurationSection
{
  private const string Size = "size";
  private const string Active = "active";
  private const string Path = "path";
  protected override ConfigurationPropertyCollection Properties
  {
    get
    {
      var active = new ConfigurationProperty(Active, typeof(bool), true);
      var size = new ConfigurationProperty(Size, typeof(int), 0, ConfigurationPropertyOptions.IsRequired);
      var path = new ConfigurationProperty(Path, typeof(PathElement), null, ConfigurationPropertyOptions.IsRequired);
      return new ConfigurationPropertyCollection { active, size, path };
    }
  }
}
class PathElement : ConfigurationElement
{
  private const string LocalPath = "localPath";
  [ConfigurationProperty(LocalPath, IsRequired = true, DefaultValue = "No path found")]
  public string Path
  {
    get
    {
      return (string)this[LocalPath];
    }
  }
}
```

If I would create a property with the attribute `ConfigurationProperty`, at runtime all properties with that attribute would be added to the protected `Properties` property. Here, I manually add them to the `ConfigurationPropertyCollection` by overriding the `Properties` property. This has as a drawback that no properties with the `ConfigurationProperty` attribute will be recognised. So it's either all via attributes or all via the override of the `Properties` property. I also define constant `strings` to refer to the names of the settings, this will be easier later on when I want to access the settings. It also allows me to have auto-completion and I avoid a lot of spelling mistakes.

My configuration file would look like:

```
<configuration>
  <configSection>
    Msection name="mySettings" type="CustomConfigurationSection.Configuration.SpecificConfiguration, CustomConfigurationSection" />
  </configSections>
  <mySettings size="5">
    <path localPath="C:\\" >
  </mySettings>
</configuration>
```

## Hiding it behind an interface

At this point, I would have to pass an instance of the `SpecificConfiguration` to every place I want to use it. That's a big object with a lot of extra properties and functions I don't need. To alleviate that, I create an interface that will contain all the properties and functions I expect to have access to.

```
interface ISettings
{
  int Size { get; set; }
  bool Active { get; }
  string GetPath();
}
```

When I implement this interface on the `SpecificConfiguration`, I can implement this the way I like.

```
class SpecificConfiguration : ConfigurationSection, ISettings
{
  // see previous chapter for ConfigurationSection code
  bool ISettings.Active { get { return (bool)this[Active]; } }
  int ISettings.Size
  {
    get
    {
      return (int)this[Size];
    }
    set
    {
      this[Size] = value;
    }
  }
  string ISettings.GetPath()
  {
    var pathElement = (PathElement)this[Path];
    return pathElement.Path;
  }
}
```

With these interface properties and functions I can represent the configuration values as the library or application expects it.

For example, all the external services from my previous project required our application id in the URL that called the service: https://servicelocation/appid/. Our application id remained the same over different environments (test, acceptance, production) but the base URL of each service changed according to the environment. In the configuration, multiple base URLs were referring to the specific service and one application id. Each external service was encapsulated in a class and I injected an interface with the complete URL to call. In the interface implementation I concatenated the correct base URL with the application id. The application just saw the correct URL to each service. Both the URL and application id could be updated separately in the configuration.

## Combining it with an IoC container

This little pattern also works nicely with IoC containers as I can inject the interface as any other interface and register an instance of the custom configuration to be used. [Autofac](https://autofac.org/apidoc/html/23FE467C.htm), [LightInject](http://www.lightinject.net/#values), [Castle Windsors Instance](https://docs.particular.net/samples/containers/castle/), [StructureMap UseInstance](https://structuremap.github.io/registration/registry-dsl/) are just a few of the examples. I'm sure there is a way to register an instance with the IoC framework of your choice.

Why do I need to register an instance? Because I can't do `new SpecificConfiguration()` and have it load the configuration file. I need to load it through the `<a href="https://msdn.microsoft.com/en-us/library/system.configuration.configurationmanager(v=vs.110).aspx" target="_blank">ConfigurationManager</a>` to actually get the configuration values. Now I could do this where I need the instance, but I like to make my objects aware of their creation. I define a static function on the `SpecificConfiguration` that will create an instance and return it as the correct interface.

```
class SpecificConfiguration : ConfigurationSection, ISettings
{
  public static ISettings GetSettings(string sectionName)
  {
    return (SpecificConfiguration)ConfigurationManager.GetSection(sectionName);
  }
}
```

This is a very simple example, because it misses validation such as whether the `sectionName` parameter contains an actual section name or not. I can't statically define the section name here because the section tag in the configuration will define what name it will have.

```
var settings = SpecificConfiguration.GetSettings("mySettings");
```

With this I can hide my complete configuration inside the `SpecificConfiguration` class and expose only the data I need through properties and functions defined on the `ISettings` interface.
