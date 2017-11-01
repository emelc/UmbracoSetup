# Azure Blob storage:

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
