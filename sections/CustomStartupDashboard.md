# Custom Startup Dashboard

Create a new controller `~/Controllers/CMSController.cs`, this needs to use the base class of UmbracoAuthorizedController for security:

~~~csharp
    public class CMSController : UmbracoAuthorizedController
    {
        public ActionResult Welcome()
        {
            return PartialView("~/Views/Partials/CMS/Welcome.cshtml");
        }
    }
~~~

Create a new view `~/Views/Partials/CMS/Welcome.cshtml`:
~~~html
<div ng-controller="Umbraco.Dashboard.StartUpDynamicContentController as vm">
    <umb-load-indicator ng-if="vm.loading"></umb-load-indicator>
    <h3 class="bold">Welcome</h3>
    <div class="umb-healthcheck-help-text faded">
        <p>Here are some useful resources:</p>
        <ul class="nav">
            <li><a href="https://goo.gl/RGLHYi" target="_blank" class="welcome-action-link"><i class="icon-out"></i> <span>Umbraco Content Editor Manual</span></a></li>
            <li><a href="https://umbraco.tv" target="_blank" class="welcome-action-link"><i class="icon-out"></i> <span>Umbraco.TV</span></a></li>
        </ul>
    </div>
</div>
~~~

Update `~/config/dashboards.config` to include the new view:

~~~xml
<section alias="CMSSection">
  <areas>
    <area>default</area>
    <area>content</area>
  </areas>
  <tab caption="Welcome">
    <control>/umbraco/backoffice/cms/welcome</control>
  </tab>
</section>
~~~

Update the UmbracoStartup.cs to include the custom route. 

~~~csharp
public void OnApplicationStarted(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
{
#if DEBUG
    ServicePointManager.ServerCertificateValidationCallback += (o, c, ch, er) => true;
#endif

    RouteTable.Routes.MapRoute(
        name: "CMS",
        url: GlobalSettings.UmbracoMvcArea + "/backoffice/cms/{action}/{id}",                
        defaults: new
        {
            controller = "CMS",
            action = "Welcome",
            id = UrlParameter.Optional
        });

    BundleConfig.RegisterBundles(BundleTable.Bundles);
}
~~~

