# Startup Dashboards

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

Create a new controller `~/Controllers/JaywingController.cs`, this needs to use the base class of UmbracoAuthorizedController for security:

~~~csharp
    public class JaywingController : UmbracoAuthorizedController
    {
        public ActionResult Welcome()
        {
            return PartialView("~/Views/Partials/Jaywing/Welcome.cshtml");
        }
    }
~~~

Create a new view `~/Views/Partials/Jaywing/Welcome.cshtml`:
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
 <section alias="JaywingSection">
    <areas>
      <area>default</area>
      <area>content</area>
    </areas>
    <tab caption="Welcome">
      <control>/umbraco/backoffice/jaywing/welcome</control>
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
                name: "Jaywing",
                url: GlobalSettings.UmbracoMvcArea + "/backoffice/jaywing/{action}/{id}",                
                defaults: new
                {
                    controller = "Jaywing",
                    action = "Welcome",
                    id = UrlParameter.Optional
                });

            BundleConfig.RegisterBundles(BundleTable.Bundles);
        }
~~~
