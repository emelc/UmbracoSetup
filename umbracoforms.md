# Umbraco Forms

## Google Recaptcha 2


![Recaptcha Not A Robot](https://github.com/Jaywing/UmbracoSetup/blob/master/images/recaptcha-not-a-robot.PNG)

Firstly you need to setup a new site on: [Google Recaptcha](https://www.google.com/recaptcha).

Include your localhost, your local domain (e.g. umbracoClient.local), *-iat.jywng.co, *-uat.jywng.co and any production URLs.

Take note of the `SiteKey` and `SecretKey`, you'll need these later for the configuration files . 

I've adapted the work found here: [UmbracoFormsReCAPTCHA](https://gitlab.com/umakers-opensource/UmbracoFormsReCAPTCHA/tree/master), and rewritten based on Jaywing's standard practices. 

Certain nTier architecture and Autofac IoC container setup has been assumed. Knowledge of UmbracoForms setup is required. 



### Business or Logic Layer
---


`~/Forms/FormsRecaptcha2Service.cs`

This service abstracts UmbracoForms objects away from the generic Recaptcha service. 

~~~csharp
public class FormsRecaptcha2Service
{
    private readonly UmbracoHelper _umbracoHelper;
    private readonly GlobalConfig _config;
    private readonly RecaptchaService _recaptchaService;

    public FormsRecaptcha2Service(UmbracoHelper umbracoHelper, GlobalConfig config, RecaptchaService recaptchaService)
    {
        _umbracoHelper = umbracoHelper;
        _config = config;
        _recaptchaService = recaptchaService;
    }

    private string RecaptchaPrivateKey => Configuration.GetSetting("RecaptchaPrivateKey");

    public string RecaptchaFieldId(Form form)
    {
        var matchingFields = form
            .AllFields
            .Where(f => f.FieldType.Name.Contains("googleRecaptcha"))
            .ToList();

        if (!matchingFields.Any()) return string.Empty;
        var reCaptchaField = matchingFields.FirstOrDefault();
        return reCaptchaField != null ? reCaptchaField.Id.ToString() : string.Empty;
    }

    public string Validate(string id)
    {
        if (id == null) return string.Empty;

        var recaptchaRequest = new RecaptchaRequest
        {
            ResponseCode = HttpContext.Current.Request[_config.Recaptcha.RecaptchaResponse],
            IPAddress = HttpContext.Current.GetIpAddress()
        };

        var response = _recaptchaService.VerifyWithService(recaptchaRequest, RecaptchaPrivateKey);

        if (response.Errors == null || !response.Errors.Any())
            return string.Empty;

        var compiledErrors = new List<string>();

        foreach (var code in response.Errors)
            compiledErrors.Add(_umbracoHelper.GetDictionaryValue("Recaptcha." + code));

        return $"{string.Join(",", compiledErrors)}";
    }
}
~~~

`~/Extensions/HttpContextExtensions.cs`

This provides the current IP Address for the request. 

~~~csharp
public static class HttpContextExtensions
{
    public static string GetIpAddress(this HttpContext httpContext)
    {
        string ip = httpContext.Request.ServerVariables["HTTP_X_FORWARDED_FOR"];
        if (string.IsNullOrEmpty(ip))
            ip = httpContext.Request.ServerVariables["REMOTE_ADDR"];
        return ip;
    }
}
~~~


`~/Forms/Recaptcha2.cs`

This model adds the `FieldType` for `Recaptcha2` to Umbraco Forms. 

~~~csharp
public class Recaptcha2 : FieldType
{
    public Recaptcha2()
    {
        Id = new Guid("9C804AA5-D7D6-42D8-B492-2E0E196331AD");
        Name = "googleRecaptcha";
        Description = "Renders Google Recaptcha 2";
        Icon = "icon-eye";
        DataType = FieldDataType.String;
        SortOrder = 100;
    }
}
~~~

`~/Forms/UmbracoFormEvents.cs`

This extends the ApplicationEventHandlers to process each validation on Umbraco Forms. 

~~~csharp
public class UmbracoFormsEvents : IApplicationEventHandler
{
    public void OnApplicationInitialized(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext) { }

    public void OnApplicationStarted(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    {           
        UmbracoFormsController.FormValidate += (s, e) =>
        {
            var formsRecaptcha2Service = DependencyResolver.Current.GetService<FormsRecaptcha2Service>();

            var id = formsRecaptcha2Service.RecaptchaFieldId(e.Form);
            var error = formsRecaptcha2Service.Validate(id);                
            if (!string.IsNullOrEmpty(error))
                ((Controller)s)?.ModelState.AddModelError(id, error);
        };
    }  

    public void OnApplicationStarting(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext) { }
}
~~~


`~/Services/Recaptcha/RecaptchaService.cs`

This is a generic RecaptchaService that connects to a configurable API endpoint and validates the user isn't a robot. 

~~~csharp
public class RecaptchaService
{
    private readonly GlobalConfig _config;

    public RecaptchaService(GlobalConfig config)
    {
        _config = config;
    }

    public RecaptchaResponseVm VerifyWithService(RecaptchaRequest request, string privateKey)
    {
        if (string.IsNullOrEmpty(privateKey))
            return new RecaptchaResponseVm() { Success = false, Errors = new string[] { "reCAPTCHA key and secret not provided." } };

        try
        {
            using (var client = new WebClient())
            {
                var reqparm = new NameValueCollection
                {
                    { "secret", privateKey },
                    { "response", request.ResponseCode },
                    { "remoteip", request.IPAddress }
                };

                byte[] responsebytes = client.UploadValues(_config.Recaptcha.ApiEndpoint, HttpMethod.Post.Method, reqparm);
                var response = JsonConvert.DeserializeObject<RecaptchaResponseVm>(Encoding.UTF8.GetString(responsebytes));

                if (!response.Success && response.Errors == null)
                    return new RecaptchaResponseVm { Success = false, Errors = new[] { "reCAPTCHA failed but no errors returned." } };

                return response;
            }
        }
        catch (Exception ex)
        {
            LogHelper.Error<RecaptchaService>("Recaptcha Verify", ex);
            return new RecaptchaResponseVm() { Success = false, Errors = new string[] { "reCAPTCHA verify service failed." } };
        }
    }
}
~~~

`~/Services/Recaptcha/RecaptchaRequest.cs`
~~~csharp
public class RecaptchaRequest
{        
    public string ResponseCode { get; set; }
    public string IPAddress { get; set; }
}
~~~

`~/Services/Recaptcha/RecaptchaResponseVm.cs`
~~~csharp
public class RecaptchaResponseVm
{
    [JsonProperty(PropertyName = "success")]
    public bool Success { get; set; }

    [JsonProperty(PropertyName = "error-codes")]
    public string[] Errors { get; set; }
}
~~~



### Config Layer
---
`~/GlobalConfig.cs`
~~~csharp
[ConfigurationProperty("recaptcha", IsRequired = true)]
public RecaptchaElement Recaptcha
{
    get { return (RecaptchaElement)base["recaptcha"]; }
}
~~~

`~/Elements/RecaptchaElement.cs`
~~~csharp 
public class RecaptchaElement : ConfigurationElement
{
    [ConfigurationProperty("recaptchaResponse", DefaultValue = "g-recaptcha-response")]
    public string RecaptchaResponse
    {
        get { return (string)this["recaptchaResponse"]; }
        set { this["recaptchaResponse"] = value; }
    }

    [ConfigurationProperty("apiEndpoint", DefaultValue = "https://www.google.com/recaptcha/api/siteverify")]
    public string ApiEndpoint
    {
        get { return (string)this["apiEndpoint"]; }
        set { this["apiEndpoint"] = value; }
    }        
}
~~~


### Website Layer
---
Autofac Register Types

`~/Modules/BusinessModule.cs`
~~~csharp
builder.Register(c =>
{
    var scope = c.Resolve<ILifetimeScope>();
    return new FormsRecaptcha2Service(
        scope.Resolve<UmbracoHelper>(),
        scope.Resolve<GlobalConfig>(),
        scope.Resolve<RecaptchaService>());
})
.AsSelf()
.InstancePerRequest();
~~~


`~/App_Plugins/UmbracoForms/Backoffice/Common/FieldTypes/recaptcha2.html`

~~~html
<h5>Google reCAPTCHA</h5>
~~~


`~/Views/Partials/Forms/Themes/default/FieldTypes/FieldType.GoogleReCaptcha.cshtml`

~~~csharp
@model Umbraco.Forms.Mvc.Models.FieldViewModel
@{
    var siteKey = Configuration.GetSetting("RecaptchaPublicKey");
    if (string.IsNullOrEmpty(siteKey))
    {
        LogHelper.Warn<UmbracoFormsEvents>("ERROR: ReCaptcha v.2 is missing the Site Key - Please update the web.config to include 'key=\"RecaptchaSiteKey\"'");
    }
    else
    {       
        if (!string.IsNullOrEmpty(siteKey))
        {
            var theme = "clean";
            var fieldSettingViewModel = Model.AdditionalSettings.FirstOrDefault(x => x.Key == "Theme");
            if (!string.IsNullOrEmpty(fieldSettingViewModel.Value))
            {
                theme = fieldSettingViewModel.Value;
            }

            <script src="https://www.google.com/recaptcha/api.js" async defer></script>
            <div class="g-recaptcha" data-sitekey="@siteKey" data-theme="@theme"></div>
        }
    }
}
~~~


### Configuration Files
---
`~/web.config`

Replace `[SiteKey]` with the SiteKey provided by Google Recaptcha.
~~~xml
<add key="RecaptchaSiteKey" value="[SiteKey]" />
~~~


`~/App_Plugins/UmbracoForms/UmbracoForms.config`

Replace `[SiteKey]` with the SiteKey provided by Google Recaptcha.
Replace `[SecretKey]` with the SecretKey provided by Google Recaptcha.
~~~xml
<!-- Recaptcha -->
<setting key="RecaptchaPublicKey" value="[SiteKey]" />
<setting key="RecaptchaPrivateKey" value="[SecretKey]" />
~~~


### uSync Dictionary Files
---
`~/uSync/data/DictionaryItem/Recaptcha.config`

This should work with uSync, allowing you to import the dictionary items required for the above services. 

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<DictionaryItem Key="Recaptcha" guid="26e9380e-c551-4016-b306-177748e95d14">
  <DictionaryItem Key="Recaptcha.invalid-input-response">
    <Value LanguageId="1" LanguageCultureAlias="en-GB"><![CDATA[Unfortunately we didn't recognise your reCAPTCHA response, Please confirm you're not a robot.]]></Value>
  </DictionaryItem>
  <DictionaryItem Key="Recaptcha.invalid-input-secret">
    <Value LanguageId="1" LanguageCultureAlias="en-GB"><![CDATA[reCAPTCHA: The secret parameter is invalid or malformed.]]></Value>
  </DictionaryItem>
  <DictionaryItem Key="Recaptcha.missing-input-response">
    <Value LanguageId="1" LanguageCultureAlias="en-GB"><![CDATA[Please confirm you're not a robot.]]></Value>
  </DictionaryItem>
  <DictionaryItem Key="Recaptcha.missing-input-secret">
    <Value LanguageId="1" LanguageCultureAlias="en-GB"><![CDATA[reCAPTCHA: The secret parameter is missing.]]></Value>
  </DictionaryItem>
  <DictionaryItem Key="Recaptcha.timeout-or-duplicate">
    <Value LanguageId="1" LanguageCultureAlias="en-GB"><![CDATA[Unfortunately your reCAPTCHA has timed out, please refresh and try again.]]></Value>
  </DictionaryItem>
  <DictionaryItem Key="Recaptcha.verification-failed">
    <Value LanguageId="1" LanguageCultureAlias="en-GB"><![CDATA[Recaptcha Verification Failed]]></Value>
  </DictionaryItem>
</DictionaryItem>
~~~

