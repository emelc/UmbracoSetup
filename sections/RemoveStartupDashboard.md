# Remove Startup dashboard

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
