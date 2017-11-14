# Azure CDN

Setup a CDN in Azure. 
Ensure you have setup media to use blob storage.
Install the ImageProcessor BlobCache Plugin: `Install-Package ImageProcessor.Web.Plugins.AzureBlobCache` [nuget](https://www.nuget.org/packages/ImageProcessor.Web.Plugins.AzureBlobCache/)

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
