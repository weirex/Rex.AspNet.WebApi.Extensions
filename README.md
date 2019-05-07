# Rex.AspNet.WebApi.Extensions

## ���
���ڿ�� `.NET Framework 4.7`�����������л�������չ���ܡ�

### 1. �쳣���񲢼�¼��־

*[C#]*

```csharp
config.Filters.Add(new WebApiExceptionAttribute());
```

### 2. ������������
> for [Microsoft.AspNet.WebApi.Cors](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Cors "Microsoft.AspNet.WebApi.Cors")

*[AppSettings]*

```xml
<appSettings>
  <add key="origins" value="*" />
  <add key="headers" value="*" />
  <add key="methods" value="*" />
</appSettings>
```

*[C#]*

```csharp
var origins = AppSettings.GetValue("origins");
var headers = AppSettings.GetValue("headers");
var methods = AppSettings.GetValue("methods");

config.EnableCors(new EnableCorsAttribute(origins, headers, methods));
```

### 3. ����ֻ����JSON

> for [���� JSON ����ȷ����](https://www.strathweb.com/2013/06/supporting-only-json-in-asp-net-web-api-the-right-way/ "���� JSON ����ȷ����")

*[C#]*

```csharp
config.Services.Replace(typeof(IContentNegotiator), new JsonNetContentNegotiator());
```

### 4. ����ѹ���������� Gzip/Deflate
> for [Microsoft.AspNet.WebApi.Extensions.Compression.Server](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Extensions.Compression.Server "Microsoft.AspNet.WebApi.Extensions.Compression.Server")

*[C#]*

```csharp
config.MessageHandlers.Insert(0, new ServerCompressionHandler(new GZipCompressor(), new DeflateCompressor()));
```

### 5. �������ã�֧�����޼���

***Ŀ¼�ṹ��***

Controllers
���� School
������������ Teacher
���������������������� TeacherDemoController.cs
���������� Students
���������������������� StudentsDemoController.cs
���������� SchoolDemoController.cs
���� DemoController.cs

***API ��ַ��***

/api/School/Teacher/TeacherDemo/Get
/api/School/Students/StudentsDemo/Get
/api/School/SchoolDemo/Get
/api/Demo/Get

*[C#]*

```csharp
config.Routes.MapHttpRoute(
    name: "DefaultArea2Api",
    routeTemplate: "api/{area1}/{area2}/{controller}/{action}/{id}",
    defaults: new { id = RouteParameter.Optional }
);
config.Routes.MapHttpRoute(
    name: "DefaultArea1Api",
    routeTemplate: "api/{area}/{controller}/{action}/{id}",
    defaults: new { id = RouteParameter.Optional }
);
config.Routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);
//���� Register �������һ��
config.Services.Replace(typeof(IHttpControllerSelector), new AreaHttpControllerSelector(config));
```

### ����ʵ��

ע�ᵽ `App_Start\WebApiConfig.cs` �ļ���

*[AppSettings]*

```xml
<appSettings>
  <add key="origins" value="*" />
  <add key="headers" value="*" />
  <add key="methods" value="*" />
</appSettings>
```

*[C#]*

```csharp
public static void Register(HttpConfiguration config) {
    // Web API �쳣���񲢼�¼��־
    config.Filters.Add(new WebApiExceptionAttribute());

    // Web API ���� JSON ����
    config.Services.Replace(typeof(IContentNegotiator), new JsonNetContentNegotiator());

    // Web API ����ѹ��
    config.MessageHandlers.Insert(0, new ServerCompressionHandler(new GZipCompressor(), new DeflateCompressor()));

    // Web API ������������
    var origins = AppSettings.GetValue("origins");
    var headers = AppSettings.GetValue("headers");
    var methods = AppSettings.GetValue("methods");
    config.EnableCors(new EnableCorsAttribute(origins, headers, methods));

    // Web API ·��
    config.Routes.MapHttpRoute(
        name: "DefaultArea2Api",
        routeTemplate: "api/{area1}/{area2}/{controller}/{action}/{id}",
        defaults: new { id = RouteParameter.Optional }
    );
    config.Routes.MapHttpRoute(
        name: "DefaultArea1Api",
        routeTemplate: "api/{area}/{controller}/{action}/{id}",
        defaults: new { id = RouteParameter.Optional }
    );
    config.Routes.MapHttpRoute(
        name: "DefaultApi",
        routeTemplate: "api/{controller}/{id}",
        defaults: new { id = RouteParameter.Optional }
    );

    //���� Register �������һ��
    config.Services.Replace(typeof(IHttpControllerSelector), new AreaHttpControllerSelector(config));
}
```

### 6. ��������
���� `MVC` �� `OutputCacheAttribute` �������� Web API
> for [CacheOutput](https://www.nuget.org/packages/Strathweb.CacheOutput.WebApi2/ "CacheOutput")

*[C#]*

**`Action` ���û���**

```csharp
// ����������100�룬֪ͨ�ͻ�����Ӧ��Ч100�롣
[CacheOutput(ClientTimeSpan = 100, ServerTimeSpan = 100)]
public IEnumerable<string> Get(){
    return new string[] { "value1", "value2" };
}

// ����������100�룬֪ͨ�ͻ�����Ӧ��Ч100�룬��Ϊ�����û����档
[CacheOutput(ClientTimeSpan = 100, ServerTimeSpan = 100, AnonymousOnly = true)]
public IEnumerable<string> Get(){
    return new string[] { "value1", "value2" };
}

// ֪ͨ�ͻ�����Ӧ��Ч50�룬ǿ�ƿͻ���������֤��
[CacheOutput(ClientTimeSpan = 50, MustRevalidate = true)]
public string Get(int id){
    return "value";
}

// ����������50�룬���ṩ��������ʱ���� QueryString ������
[CacheOutput(ServerTimeSpan = 50, ExcludeQueryStringFromCacheKey = true)]
public string Get(int id){
    return "value";
}
```

**`Controller` ���û���**

```csharp
// �� Controller ���л�������
[CacheOutput(ClientTimeSpan = 50, ServerTimeSpan = 50)]
public class IgnoreController : ApiController{
    public string GetCached()    {
        return DateTime.Now.ToString();
    }

    // ʹ�� IgnoreCacheOutput �����˳�����
    [IgnoreCacheOutput]
    public string GetUnCached(){
        return DateTime.Now.ToString();
    }
}
```
