---
layout: drafts
title: ASP.NET Core 錯誤處理(Error Handling)
tags:
  - .Net Core
  - Error Handle
date: 2017-06-07 18:45:34
---


# 幾種錯誤處理類型
1. [開發者的例外頁面](#開發者的例外頁面) - ([連結](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#the-developer-exception-page))
1. [設定自訂的例外處理頁面](#設定自訂的例外處理頁面)  - [連結](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#configuring-a-custom-exception-handling-page))
1. [設定Http狀態碼頁面](#設定Http狀態碼頁面) - ([連結](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#configuring-status-code-pages))
1. [例外處理程式](#例外處理程式) - ([連結](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#exception-handling-code))
1. [伺服器例外處理](#伺服器例外處理) - ([連結](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#server-exception-handling))
1. [啟動時的例外處理](#啟動時的例外處理) - ([連結](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#startup-exception-handling))
1. [MVC的錯誤處理](#MVC的錯誤處理) - ([連結](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#aspnet-mvc-error-handling))


下面只列出我這次的研究目標。
裡面所提到的設定，都是在`Starup`類別的`Configure`方法中，於`app.UseMvc()`之前，加入相關的程式碼作設定，除有例外，將額外作說明。

## 開發者的例外頁面
讓開發人員於開發環境中，可以很快的檢視例外的細節。

1. 安裝 `Microsoft.AspNetCore.Diagnostics` NuGet套件
2. 在Configure中，加入相關的設定
```c#
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    // 設定在開發環境時，會顯示開發者的例外頁面
}

```
在本機可以在基本網址上加上Query String `?throw=true` 預覽開發者的例外頁面。


## 設定自訂的例外處理頁面
設定在Production中，要顯示的錯誤畫面。

```c#
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
    // 設定在非開發環境時，會顯示的例外頁面
}
```

在MVC中，不要設定特定的Http Method的Attributes(例: HttpGet, HttpPost)於錯誤處理的Method上，因為這可能會使不相符的Request無法到達錯誤處理的Method。

```c#
//[HttpGet] 不要指定特定的Http Method，避免其他種類的Request的Error無法到這個方法
[Route("/Error")]
public IActionResult Index()
{
    // Handle error here
}
```

## 設定Http狀態碼頁面
以預設來說，你的應用沒有提供Http 404(Not Found)或500(Internal Server Error)的錯誤頁面。你可以在`Configure`加入 `StatusCodePagesMiddleware` 來處理:
```c#
app.UseStatusCodePages();
```
藉由這個Middleware，預設會顯示純文字的錯誤Http代碼畫面。
它還提供了幾種不同的擴展方法來顯示format String

```c#
// Lambda expression
app.UseStatusCodePages(async context =>
{
    context.HttpContext.Response.ContentType = "text/plain";
    await context.HttpContext.Response.WriteAsync(
        "Status code page, status code: " + 
        context.HttpContext.Response.StatusCode);
});
```

```c#
// content type
app.UseStatusCodePages("text/plain", "Status code page, status code: {0}");
```

它也提供了Redirect方法:
```c#
app.UseStatusCodePagesWithRedirects("/error/{0}");
// sends a 302 status code to the client,

app.UseStatusCodePagesWithReExecute("/error/{0}");
// returns the original status code to the client but also executes the handler for the redirect URL.
```

如果你需要關閉特定Requer的Status code面頁，你也可以這樣寫:
```c#
var statusCodePagesFeature = context.Features.Get<IStatusCodePagesFeature>();
if (statusCodePagesFeature != null)
{
  statusCodePagesFeature.Enabled = false;
}
```
