# Rex.AspNet.WebApi.Extensions

## 简介
基于框架 `.NET Framework 4.7`，包含开发中最基础的扩展功能。

### 1. 异常捕获并记录日志

*[C#]*

```csharp
config.Filters.Add(new WebApiExceptionAttribute());
```

### 2. 跨域请求设置
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

### 3. 设置只返回JSON

> for [返回 JSON 的正确做法](https://www.strathweb.com/2013/06/supporting-only-json-in-asp-net-web-api-the-right-way/ "返回 JSON 的正确做法")

*[C#]*

```csharp
config.Services.Replace(typeof(IContentNegotiator), new JsonNetContentNegotiator());
```

### 4. 启用压缩请求数据 Gzip/Deflate
> for [Microsoft.AspNet.WebApi.Extensions.Compression.Server](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Extensions.Compression.Server "Microsoft.AspNet.WebApi.Extensions.Compression.Server")

*[C#]*

```csharp
config.MessageHandlers.Insert(0, new ServerCompressionHandler(new GZipCompressor(), new DeflateCompressor()));
```

### 5. 区域设置（支持无限级）

***目录结构：***

```
Controllers
└─ School
└───── Teacher
└────────── TeacherDemoController.cs
└──── Students
└────────── StudentsDemoController.cs
└──── SchoolDemoController.cs
└─ DemoController.cs
```

***API 地址：***

- /api/School/Teacher/TeacherDemo/Get
- /api/School/Students/StudentsDemo/Get
- /api/School/SchoolDemo/Get
- /api/Demo/Get

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
    routeTemplate: "api/{controller}/{action}/{id}",
    defaults: new { id = RouteParameter.Optional }
);
//放在 Register 方法最后一行
config.Services.Replace(typeof(IHttpControllerSelector), new AreaHttpControllerSelector(config));
```

### 完整实例

注册到 `App_Start\WebApiConfig.cs` 文件中

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
    // Web API 异常捕获并记录日志
    config.Filters.Add(new WebApiExceptionAttribute());

    // Web API 返回 JSON 数据
    config.Services.Replace(typeof(IContentNegotiator), new JsonNetContentNegotiator());

    // Web API 启用压缩
    config.MessageHandlers.Insert(0, new ServerCompressionHandler(new GZipCompressor(), new DeflateCompressor()));

    // Web API 跨域请求设置
    var origins = AppSettings.GetValue("origins");
    var headers = AppSettings.GetValue("headers");
    var methods = AppSettings.GetValue("methods");
    config.EnableCors(new EnableCorsAttribute(origins, headers, methods));

    // Web API 路由
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
        routeTemplate: "api/{controller}/{action}/{id}",
        defaults: new { id = RouteParameter.Optional }
    );

    //放在 Register 方法最后一行
    config.Services.Replace(typeof(IHttpControllerSelector), new AreaHttpControllerSelector(config));
}
```

### 6. 缓存设置
类似 `MVC` 的 `OutputCacheAttribute` 可以用于 Web API
> for [CacheOutput](https://www.nuget.org/packages/Strathweb.CacheOutput.WebApi2/ "CacheOutput")

*[C#]*

**`Action` 设置缓存**

```csharp
// 服务器缓存100秒，通知客户端响应有效100秒。
[CacheOutput(ClientTimeSpan = 100, ServerTimeSpan = 100)]
public IEnumerable<string> Get(){
    return new string[] { "value1", "value2" };
}

// 服务器缓存100秒，通知客户端响应有效100秒，仅为匿名用户缓存。
[CacheOutput(ClientTimeSpan = 100, ServerTimeSpan = 100, AnonymousOnly = true)]
public IEnumerable<string> Get(){
    return new string[] { "value1", "value2" };
}

// 通知客户端响应有效50秒，强制客户端重新验证。
[CacheOutput(ClientTimeSpan = 50, MustRevalidate = true)]
public string Get(int id){
    return "value";
}

// 服务器缓存50秒，在提供缓存内容时忽略 QueryString 参数。
[CacheOutput(ServerTimeSpan = 50, ExcludeQueryStringFromCacheKey = true)]
public string Get(int id){
    return "value";
}
```

**`Controller` 设置缓存**

```csharp
// 对 Controller 进行缓存设置
[CacheOutput(ClientTimeSpan = 50, ServerTimeSpan = 50)]
public class IgnoreController : ApiController{
    public string GetCached()    {
        return DateTime.Now.ToString();
    }

    // 使用 IgnoreCacheOutput 属性退出缓存
    [IgnoreCacheOutput]
    public string GetUnCached(){
        return DateTime.Now.ToString();
    }
}
```
