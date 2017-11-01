# Plugins

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
