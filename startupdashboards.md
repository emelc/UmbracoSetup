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
