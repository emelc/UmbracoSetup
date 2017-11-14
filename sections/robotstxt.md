# Robots.txt

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