# Login Screen

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
