# Basic Umbraco Setup

### 1. Create git repository 
Try and use lowercase alphanumerics and dashes only.

### 2. Add .gitignore file
[gitignore.io](https://www.gitignore.io)
Select the following types: **VisualStudio** and **Umbraco**.

### 3. Add solution file
Name: `UmbracoClient`

### 4. Add Project
- .NET Framework 4.5.2
- ASP .NET Web Application (.NET Framework)
- Template: Empty
- Name: `UmbracoClient.Web`
  
### 5. Change assembly name and default namespace. 
`Jaywing.UmbracoClient.Web`

### 6. Install nuget packages
- `Install-Package UmbracoCms` [nuget](https://www.nuget.org/packages/UmbracoCms/)
- `Install-Package UmbracoForms` [nuget](https://www.nuget.org/packages/UmbracoForms/)
    
If you have seperate projects that require Umbraco objects
- `Install-Package UmbracoCms.Core` [nuget](https://www.nuget.org/packages/UmbracoCms.Core/)
- `Install-Package UmbracoForms.Core` [nuget](https://www.nuget.org/packages/UmbracoForms.Core/)

### 7. Upgrade Microsoft.CodeAnalysis.Analyzers to version v1.1.0
This removes a warning message when building the website.

### 8. Install Autofac:
- `Install-Package Autofac` [nuget](https://www.nuget.org/packages/Autofac/)
- `Install-Package Autofac.WebApi2` [nuget](https://www.nuget.org/packages/Autofac.WebApi2/)
- `Install-Package Autofac.Mvc5` [nuget](https://www.nuget.org/packages/Autofac.Mvc5/)

### 9. Add Class UmbracoStart.cs
Add UmbracoStartup.cs into the root and use the following syntax:

~~~csharp
public class UmbracoStartup : IApplicationEventHandler
{
    public static IContainer Container;

    public void OnApplicationInitialized(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext) { }
   
    public void OnApplicationStarting(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    { 
        Container = RegisterDependencies(applicationContext);

        GlobalConfiguration.Configuration.DependencyResolver = new AutofacWebApiDependencyResolver(Container);

        DependencyResolver.SetResolver(new AutofacDependencyResolver(Container));         
    }

    public void OnApplicationStarted(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    {
#if DEBUG
        ServicePointManager.ServerCertificateValidationCallback += (o, c, ch, er) => true;
#endif      
       
    }

    private IContainer RegisterDependencies(ApplicationContext appContext)
    {
        var builder = new ContainerBuilder();
        Assembly applicationWebAssembly = GetType().Assembly;

        builder.RegisterApiControllers(typeof(UmbracoApplication).Assembly);
        builder.RegisterApiControllers(typeof(FormTreeController).Assembly);
        
        builder.RegisterApiControllers(applicationWebAssembly).PropertiesAutowired();
        builder.RegisterControllers(applicationWebAssembly);
        builder.RegisterModule<AutofacWebTypesModule>();

        builder.Register(c => UmbracoContext.Current).As<UmbracoContext>().InstancePerRequest();
        builder.Register(c => appContext).As<ApplicationContext>().SingleInstance();
        builder.Register(c => new UmbracoHelper(UmbracoContext.Current)).As<UmbracoHelper>();
        builder.Register(c => appContext.Services.TextService).As<ILocalizedTextService>();
        builder.Register(c => appContext.Services.MediaService).As<IMediaService>();
        builder.Register(c => appContext.Services.ContentService).As<IContentService>();
        builder.Register(c => appContext.Services.UserService).As<IUserService>();
        builder.Register(c => appContext.Services.SectionService).As<ISectionService>();          

        builder.RegisterAssemblyModules(applicationWebAssembly);

        return builder.Build();
    }       
}
~~~

### 10. Update web.config: 

##### AppSettings:
- umbracoReservedPaths: append to the value: `~/.well-known/,~/assets/`   
    - `~/.well-known/` is for Let's Encrypt Azure Extension. 
    - `~/assets/` is for CMS backgrounds.
- umbracoUseSSL: **true**
- umbracoDefaultUILanguage: **en-GB**
- Umbraco.ModelsBuilder.Enable: **false**

##### Update mail settings:
Update to point to your mail server. 
~~~xml
        <system.net>
            <mailSettings>
                <smtp from="noreply@jaywing.com">
                    <network host="localhost" />
                </smtp>
            </mailSettings>
        </system.net>
~~~


### 11. Create SQL database 
- Name: `Jaywing.UmbracoClient.Web`
- Within Options, set recovery model set to Simple.

### 12. Create SQL user login
For a local SQL database, you can setup using simple credentials, although upon deployment it's recommended you don't use these settings. 

- username: `UmbracoClient`
- password: `UmbracoClient`
- **uncheck** enforce password policy
- user mapping
    - map `Jaywing.UmbracoClient.Web` to user and set the default schema as 'dbo'.
    - select **db_owner** from the database role memberships.

### 14. Update SQL database user:
- database (`UmbracoClient`), security, users, `UmbracoClient`, properties
- within the 'owned schemas' page, select '**dbo_owner**'.


### 15. Hosts file
Add the following entry to your host file:
~~~js  
    127.0.0.1     umbracoclient-local.jywng.co
~~~

### 16. IIS
##### Add website
-   sitename: *umbracoclient-local.jywng.co*
-   application pool: DefaultAppPool
-   physical path: `D:\Git\Jaywing\UmbracoClient.Web`
-   binding:
    - type: https
    - ip address: all unassigned
    - post: 80
    - hostname: *umbracoclient-local.jywng.co*
    - require server name indication: check
    - SSL certificate: `*.jywng.co`

### 17. Build the solution in Visual Studio

### 18. Browse to Website
Address: *https://umbracoclient-local.jywng.co*

### 19. Umbraco Install


Step 1:
- Name: Gareth Wright
- Email: ***email address goes here***
- Password: use a complex password, as this is the main administrator for the CMS. Please make sure you store this password into the Jaywing Development KeePass database.
- ***Uncheck*** newsletter signup.
- **IMPORTANT, DO NOT CLICK INSTALL, CLICK CUSTOMISE.**

Step 2:
- Database type: Microsoft SQL Server
- Server: `(local)`
- Database: `Jaywing.UmbracoClient.Web`
- Login: `UmbracoClient`
- Password: `UmbracoClient`
- Click continue

Step 3: 
- Configure an ASP .Net Machine Key

Step 4: 
- Select "No thanks, I do not want to install a starter website".

Step 5:
- Umbraco will now setup and redirect you to the Umbraco CMS. 


### 20. Include items in project
- Umbraco
- UmbracoClient
- App_Browsers
- App_Data/Packages

### 21. Run Developer Health Checks and complete fixes:
*https://umbracoclient-local.jywng.co/umbraco#/developer*

### 22. Change Language from English (United States): 
https://umbracoclient-local.jywng.co/umbraco/#/settings
Change default language to be `English (United Kingdom)`.

https://umbracoclient-local.jywng.co/umbraco/#/users
Change administrator's Language to be `English (United Kingdom)`. 

