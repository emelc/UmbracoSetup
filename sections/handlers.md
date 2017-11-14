# Handlers

## Robots.txt: 
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

## Sitemap.xml:
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
