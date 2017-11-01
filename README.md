# Jaywing Umbraco Setup
> This is a guide to help setup Umbraco in the right way, using UmbracoClient as an example. Please replace names, etc where required. 


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
- `Install-Package UmbracoCms`
- `Install-Package UmbracoForms` *(if applicable)*
    
If you have seperate projects that require Umbraco objects
- `Install-Package UmbracoCms.Core`
- `Install-Package UmbracoForms.Core`

### 7. Upgrade Microsoft.CodeAnalysis.Analyzers to version v1.1.0
This removes a warning message when building the website.

### 8. Install Autofac:
- `Install-Package Autofac`
- `Install-Package Autofac.WebApi2`
- `Install-Package Autofac.Mvc5`

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
        RouteTable.Routes.MapMvcAttributeRoutes();
        BundleConfig.RegisterBundles(BundleTable.Bundles);
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
- umbracoReservedPaths: append to the value: `,~/Jaywing/,~/.well-known/,~/assets/`
    - `~/Jaywing/` is for dashboards, replace Jaywing with the client name. 
    - `~/.well-known/` is for Let's Encrypt Azure Extension. 
    - `~/assets/` is for CMS backgrounds.
- umbracoUseSSL: **true**
- umbracoDefaultUILanguage: **en**
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



## Additional setups

### Azure Blob storage:

- Setup Blob storage in Azure
- `Install-Package UmbracoFileSystemProviders.Azure -Pre`
- Include `~/media/web.config` to source control
- Update web.config:

~~~xml
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
      <system.webServer>
        <handlers>
          <clear />
          <add name="StaticFileHandler" path="*" verb="*" preCondition="integratedMode" type="System.Web.StaticFileHandler"/>
          <add name="StaticFile" path="*" verb="*" modules="StaticFileModule,DefaultDocumentModule,DirectoryListingModule" resourceType="Either"
            requireAccess="Read"/>
        </handlers>
      </system.webServer>
    </configuration> 
~~~

- update `~/config/FileSystemProviders.config` and replace XXXXXXX with the appropriate details from Azure. 
~~~xml
    <?xml version="1.0"?>
    <FileSystemProviders>

      <!-- Media -->
      <Provider alias="media" type="Our.Umbraco.FileSystemProviders.Azure.AzureBlobFileSystem, Our.Umbraco.FileSystemProviders.Azure">
        <Parameters>
          <add key="containerName" value="media-dev"/>
          <add key="rootUrl" value="https://XXXXXXX.blob.core.windows.net/"/>
          <add key="connectionString" value="DefaultEndpointsProtocol=https;AccountName=XXXXXXX;AccountKey=XXXXXXX;EndpointSuffix=core.windows.net"/>
          <add key="maxDays" value="365"/>
          <add key="useDefaultRoute" value="true"/>
          <add key="usePrivateContainer" value="false"/>
        </Parameters>
      </Provider>

      <!--Umbraco Forms-->
      <Provider alias="forms" type="Our.Umbraco.FileSystemProviders.Azure.AzureBlobFileSystem, Our.Umbraco.FileSystemProviders.Azure">
        <Parameters>
          <add key="containerName" value="forms-data-dev"/>
          <add key="rootUrl" value="https://XXXXXXX.blob.core.windows.net/"/>
          <add key="connectionString" value="DefaultEndpointsProtocol=https;AccountName=XXXXXXX;AccountKey=XXXXXXX;EndpointSuffix=core.windows.net"/>
          <add key="maxDays" value="365"/>
          <add key="useDefaultRoute" value="true"/>
          <add key="usePrivateContainer" value="false"/>
        </Parameters>
      </Provider>
    
    </FileSystemProviders>
~~~

- [Microsoft Azure Storage Explorer](https://azure.microsoft.com/en-gb/features/storage-explorer) to browse and upload content to Azure blob storage.


### uSync:
`Install-Package uSync`
- In developer section of Umbraco, click on the uSync backoffice and select config data. set to Manual Sync. 
- Restart application. 


### Remove Startup dashboard:
Open the following config file `~/config/dashboard.config`

~~~xml
 <section alias="StartupDashboardSection">
    <access>
      <deny>translator</deny>
    </access>
    <areas>
      <area>content</area>
    </areas>
    <tab caption="Get Started">
      <access>
        <grant>admin</grant>
      </access>
      <control showOnce="true" addPanel="true" panelCaption="">
        views/dashboard/default/startupdashboardintro.html
      </control>
    </tab>
  </section>
  ~~~
Remove the tag "Get Started".

~~~xml
 <section alias="StartupDashboardSection">
    <access>
      <deny>translator</deny>
    </access>
    <areas>
      <area>content</area>
    </areas>
  </section>
  ~~~
### Custom Startup Dashboard:
- Ensure you have updated the web.config to include `~/Jaywing/` as a umbracoReservedPaths.
- Ensure you have attribute routing ***RouteTable.Routes.MapMvcAttributeRoutes();*** in your `UmbracoStartup.cs`. 

Create a new controller `~/Controllers/JaywingController.cs`:

~~~csharp
    [RoutePrefix("jaywing")]
    public class JaywingController : SurfaceController
    {
        public JaywingController()
        {
        }

        [Route("welcome")]
        public ActionResult Welcome()
        {
            return PartialView("Welcome"); 
        }
    }
~~~

Create a new view `~/Views/Jaywing/Welcome.cshtml`:
~~~html
<style type="text/css">
    .welcome-action-link {
        color: #1DC5C5;
    }

    .welcome-heading {
        border-bottom: 1px solid #f8f8f8;
        margin-bottom: 30px;
    }
</style>
<div class="umb-dashboard-grid" ng-controller="Umbraco.Dashboard.StartUpDynamicContentController as vm">
    <umb-load-indicator ng-if="vm.loading"></umb-load-indicator>
    <div class="span12 welcome-heading">
        <h2>Welcome to the CMS</h2>
    </div>
    <div class="span12">
        <h3>Useful resources</h3>
        <ul class="nav">
            <li><a href="https://goo.gl/RGLHYi" target="_blank" class="welcome-action-link"><i class="icon-out"></i> <span>Umbraco Content Editor Manual</span></a></li>
        </ul>
    </div>
</div>
~~~

Update `~/config/dashboards.config` to include the new view:

~~~xml
 <section alias="JaywingSection">
    <areas>
      <area>default</area>
      <area>content</area>
    </areas>
    <tab caption="Welcome">
      <control>/jaywing/welcome</control>
    </tab>
  </section>
~~~

### Customise the login screen image:
Use an image with **width: 1200px** and **height: 900px.**
Upload the appropriate image to `~/assets/cms-background.jpg`.
Update the following field in this file: `~/config/umbracoSettings.config`
~~~xml
   <loginBackgroundImage>/assets/cms-background.jpg</loginBackgroundImage>
~~~
### Customise the login screen welcome text:
Update the appropriate language file: `~/config/lang/en-GB.user.xml`

~~~xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<language>
  <area alias="login">
    <key alias="greeting0">Welcome to the CMS</key>
    <key alias="greeting1">Welcome to the CMS</key>
    <key alias="greeting2">Welcome to the CMS</key>
    <key alias="greeting3">Welcome to the CMS</key>
    <key alias="greeting4">Welcome to the CMS</key>
    <key alias="greeting5">Welcome to the CMS</key>
    <key alias="greeting6">Welcome to the CMS</key>
  </area>
</language>
~~~


### Ditto:
`Install-Package Our.Umbraco.Ditto`
Utilise the following processors: https://github.com/Jaywing/Jaywing.Ditto
Only use `DittoView` and `DittoController` to map to viewmodels. 

### Azure CDN:
Setup a CDN in Azure. 
Ensure you have setup media to use blob storage.
Install the ImageProcessor BlobCache Plugin: `Install-Package ImageProcessor.Web.Plugins.AzureBlobCache`
Update the following file: `~/config/imageprocessor/cache.config`  and replace XXXXXXX with the appropriate details from Azure. 

~~~xml
<caching currentCache="AzureBlobCache">
  <caches>
    <cache name="AzureBlobCache" type="ImageProcessor.Web.Plugins.AzureBlobCache.AzureBlobCache, ImageProcessor.Web.Plugins.AzureBlobCache" maxDays="365"
           browserMaxDays="7" folderDepth="6" >
      <settings>
        <setting key="CachedStorageAccount" value="DefaultEndpointsProtocol=https;AccountName=XXXXXXX;AccountKey=XXXXXXX;EndpointSuffix=core.windows.net" />
        <setting key="CachedBlobContainer" value="media-cache-dev" />
        <setting key="UseCachedContainerInUrl" value="true" />
        <setting key="CachedCDNRoot" value="https://XXXXXXX.azureedge.net/" />
        <setting key="CachedCDNTimeout" value="1000" />
        <setting key="SourceStorageAccount" value="DefaultEndpointsProtocol=https;AccountName=XXXXXXX;AccountKey=XXXXXXX;EndpointSuffix=core.windows.net" />
        <setting key="SourceBlobContainer" value="media-dev" />
        <setting key="StreamCachedImage" value="false" />
      </settings>
    </cache>
  </caches>
</caching>
~~~

Update the following file: `~/config/imageprocessor/security.config`  and replace XXXXXXX with the appropriate details from Azure. 

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<security>
  <services>
    <service prefix="media/" name="CloudImageService" type="ImageProcessor.Web.Services.CloudImageService, ImageProcessor.Web">
      <settings>
        <setting key="Container" value="media-dev" />
        <setting key="MaxBytes" value="31457280" />
        <setting key="Timeout" value="30000" />
        <setting key="Host" value="https://XXXXXXX.blob.core.windows.net/" />
      </settings>
    </service>
    <service name="LocalFileImageService" type="ImageProcessor.Web.Services.LocalFileImageService, ImageProcessor.Web" />

    <service prefix="remote.axd" name="RemoteImageService" type="ImageProcessor.Web.Services.RemoteImageService, ImageProcessor.Web">
      <settings>
        <setting key="MaxBytes" value="31457280" />
        <setting key="Timeout" value="3000" />
        <setting key="Protocol" value="https" />
      </settings>
      <whitelist>
        <add url="https://XXXXXXX.azureedge.net"/>
      </whitelist>
    </service>
  </services>
</security>
~~~

### Robots.txt: 
The following will update the sitemap URL dynamically based on your domain. 
~~~csharp
namespace Jaywing.Handlers 
{
    public class RobotsTxt : IHttpHandler
    {
        public bool IsReusable => true;
    
        public void ProcessRequest(HttpContext context)
        {
            context.Response.ContentType = "text/plain";
            string path = HttpContext.Current.Server.MapPath(VirtualPathUtility.ToAbsolute("~/robots.txt"));
            if (File.Exists(path))
            {
                StreamReader streamReader = File.OpenText(path);
                string text = streamReader.ReadToEnd();
                string site = HttpContext.Current.Request.ServerVariables["HTTP_HOST"] + HttpContext.Current.Request.FilePath.Replace("/robots.txt", "");
    
                context.Response.Write(text.Replace("{HTTP_HOST}", site));
                streamReader.Close();
                streamReader.Dispose();
            }
            else
                context.Response.Write(string.Empty);
        }
    }
}
~~~

Create a `~/robots.txt` with the following contents:

~~~
User-Agent: *
Sitemap: https://{HTTP_HOST}/sitemap.xml
~~~

Add the following line in your web.config under the `<system.webServer>` section. 
~~~xml
  <add name="RobotsTxt" verb="*" path="robots.txt" type="Jaywing.Handlers.RobotsTxt" />
~~~

### Sitemap.xml:
The following handler will traverse your website and produce an XML sitemap. 
~~~csharp
namespace Jaywing.Handlers 
{
  public class XmlSitemap : IHttpHandler
    {
        public bool IsReusable => false;

        /* Generates an XML Sitemap for Umbraco using LinqToXml */
        private static readonly XNamespace xmlns = "http://www.sitemaps.org/schemas/sitemap/0.9";

        private readonly Func<IPublishedContent, bool> _contentIsVisibleAndNotData = x => x.IsVisible();

        public void ProcessRequest(HttpContext context)
        {
            // Set correct headers from XML
            context.Response.ContentType = "text/xml";
            context.Response.Charset = "utf-8";

            // Get the absolute base URL for this website 
            Uri url = HttpContext.Current.Request.Url;
            string baseUrl = $"{url.Scheme}://{url.Host}{(url.IsDefaultPort ? string.Empty : ":" + url.Port)}";

            // Create a new XDocument using namespace and add root element
            XDocument doc = new XDocument(new XDeclaration("1.0", "utf-8", "yes"));
            XElement urlset = new XElement(xmlns + "urlset");

            // Get the root no
            IPublishedContent homepage = GetHomepage(HttpContext.Current.Request.RequestContext, UmbracoContext.Current);

            var homepageChildren = homepage?
                .Children(_contentIsVisibleAndNotData);
               
            // Iterate all nodes in site and add them to document
            RecurseNodes(urlset, homepageChildren, baseUrl);
            doc.Add(urlset);

            // Write XML document to response stream
            context.Response.Write(doc.Declaration + "\n");    
            context.Response.Write(doc.ToString());
        } 

        protected virtual IPublishedContent GetHomepage(RequestContext requestContext, UmbracoContext umbracoContext)
        {
            var helper = new UmbracoHelper(umbracoContext);

            IPublishedContent global = helper.TypedContentAtRoot().FirstOrDefault();
            Uri requestUri = requestContext.HttpContext.Request.Url;

            var homepages = global?.Children(x => x.DocumentTypeAlias == "homepage");

            if (homepages == null) return null; 

            foreach (IPublishedContent homepage in homepages)
            {
                Uri siteUri;
                if (Uri.TryCreate(helper.NiceUrlWithDomain(homepage.Id), UriKind.Absolute, out siteUri) && DomainMatch(requestUri, siteUri))
                    return homepage;
            }  
            return null;
        }

        private bool DomainMatch(Uri requestUri, Uri siteUri)
        {
            return siteUri.Host.Equals(requestUri.Host, StringComparison.InvariantCultureIgnoreCase) && 
                requestUri.LocalPath.StartsWith(siteUri.LocalPath, true, CultureInfo.InvariantCulture);
        }

        // Method to recurse all nodes and create each element
        private static void RecurseNodes(XElement urlset, IEnumerable<IPublishedContent> nodes, string baseUrl)
        {
            if (nodes == null) return;
            foreach (IPublishedContent n in nodes)
            {
                if (n.TemplateId > 0 && n.IsVisible())
                {
                    string url = n.UrlAbsolute();

                    if (!url.StartsWith("http"))
                        url = baseUrl + url;

                    // Create the XML node
                    XElement urlNode = new XElement(xmlns + "url",
                        new XElement(xmlns + "loc", url),
                        new XElement(xmlns + "lastmod", n.UpdateDate.ToUniversalTime()));
                    urlset.Add(urlNode);
                }

                // Check if the node has any child nodes and, if it has, recurse them
                if (n.Children() != null)
                    RecurseNodes(urlset, n.Children(), baseUrl);
            }
        }
    }
}
~~~

Add the following line in your web.config under the `<system.webServer>` section. 
~~~xml
  <add name="XmlSitemap" verb="*" path="sitemap.ashx" type="Jaywing.Handlers.XmlSitemap" />
~~~

[Install IIS Rewrite 2.0](https://www.microsoft.com/en-us/download/details.aspx?id=47337)

Add the following rules section in your web.config under the `<system.webServer>` section. If you already have a rules section, then just add the individual rule. 
~~~xml
<rewrite>
      <rules>
        <rule name="SiteMap" patternSyntax="Wildcard" stopProcessing="true">
          <match url="sitemap.xml" />
          <action type="Rewrite" url="sitemap.ashx" appendQueryString="true" />
        </rule>
      </rules>
    </rewrite>
~~~

### LastChanceContentFinder: 
The following handler will provide an appropriate 404 page for the site.
Ensure you use the documenttypealias of `fNF404` for this to work, or update accordingly. 
~~~csharp
 public class LastChanceContentFinder : IContentFinder
    {
        public bool TryFindContent(PublishedContentRequest contentRequest)
        {
            if (!contentRequest.Is404) return contentRequest.PublishedContent != null;
            if (!contentRequest.HasDomain) return false;

            var rootContentId = contentRequest.UmbracoDomain.RootContentId;
            if (rootContentId == null) return false;

            IPublishedContent rootContent = contentRequest.RoutingContext.UmbracoContext.ContentCache.GetById(rootContentId.Value);

            contentRequest.SetResponseStatus(404, "404 Page Not Found");
            contentRequest.PublishedContent = GetNotFoundNode(rootContent);

            return contentRequest.PublishedContent != null;
        }

        IPublishedContent GetNotFoundNode(IPublishedContent node)
        {
            return node != null && node.Children.Any() ? node.Children().FirstOrDefault(x => x.DocumentTypeAlias == "fNF404") : null;
        }
    }
~~~

Update the `~/UmbracoStartup.cs` to include the ContentLastChanceFinder. 

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
       
       ContentLastChanceFinderResolver.Current.SetFinder(DependencyResolver.Current.GetService<LastChanceContentFinder>());
    }

    public void OnApplicationStarted(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    {
#if DEBUG
        ServicePointManager.ServerCertificateValidationCallback += (o, c, ch, er) => true;
#endif
        RouteTable.Routes.MapMvcAttributeRoutes();
        BundleConfig.RegisterBundles(BundleTable.Bundles);
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
    
        builder.Register(c => new LastChanceContentFinder()).As<LastChanceContentFinder>().InstancePerRequest();
        
        builder.RegisterAssemblyModules(applicationWebAssembly);

        return builder.Build();
    }       
}  
            
~~~

### CmsEnvironmentIndicator:
`Install-Package Our.Umbraco.CmsEnvironmentIndicator`
Update the javascript file: `~/App_Plugins/CmsEnvironmentIndicator/js/cms-environment-indicator.js` to be more specific:
~~~js
	var config = [
        { pattern: "^.*-local.*\.jywng\.co$|^.*\.local.*$", color: "1d991d" }, // green
        { pattern: "^.*-iat.*\.jywng\.co$", color: "1d1d99" }, // blue
        { pattern: "^.*-uat.*\.jywng\.co$|^.staging.*$", color: "991d99" }, // purple
 		{ pattern: "^.*$", color: "991d1d" } //red
	];
~~~
