---
title: ASP.NET Core Identity、Authentication資源搜集
tags:
  - .Net Core
  - Identity
  - Authentication
categories:
  - .Net Core
date: 2017-06-07 15:47:05
---


# ASP.NET Core 的 Identity與Authentication 資源搜集

最近團隊計畫要從 ASP.NET 4.5 轉移到 ASP.NET Core，預先研究身份驗證與授權的相關文章，把讀過的資源整理起來方便之後查找。

## 官方文件 ([連結](https://docs.microsoft.com/en-us/aspnet/core/security))
在官方文件的 Security 章節裡([連結](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/))，有提到關於Authentication(驗證)與Authorization(授權)的作法，其中關於Authentication的作法大約分成下列幾種方式:
1. OSS Authentication Providers (OSS驗證)
1. 使用 AspNetCore Identity EntityFrameworkCore
1. 啟動 Facebook, Google 或其他驗證機制
1. 透過簡訊的兩步驟驗證
1. 使用Cookie Middleware而不使用ASP.NET Core Identity ([連結](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/cookie))
1. 使用Azure Active Directory (Azure AD)
1. 使用 IdentityServer4

基於上述的「5」，大約需要在 `Startup.cs` 的 `Configure` 方法中 `app.UseMVC()` 前加入下述程式:
```c#
using Microsoft.AspNetCore.Authentication.Cookies;

namespace WebApplicationBasic
{
    public class Startup
    {
      public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
      {

        app.UseCookieAuthentication(new CookieAuthenticationOptions()
            {
              AuthenticationScheme = "CoreMVCCookieMiddlewareInstance",
              // AuthenticationScheme - this is a value by which the middleware is known. This is useful when there are multiple instances of middleware and you want to limit authorization to one instance.

              LoginPath = new PathString("/Account/Unauthorized/"),
              // 當使用者未驗證時會重新轉導的路徑

              AccessDeniedPath = new PathString("/Account/Forbidden/"),
              // 當使用者沒有使用權限時的轉導路徑

              AutomaticAuthenticate = true,
              // 在每次的 Requet(請求)前運行，嚐試驗證與重畫任何序列化主體

              AutomaticChallenge = true
              // 當授權失效時，Middleware會重新轉導到LoginPath或AccessDeniedPath
            });

        app.UseMVC();
      }
    }
}

```

使用者登入時，呼叫SignInAsync 並傳入設定好的 `ClaimsPrincipal` Object
```c#
await HttpContext.Authentication.SignInAsync("MyCookieMiddlewareInstance", principal);
```
使用者登出時，呼叫SignOutAsync
```c#
await HttpContext.Authentication.SignOutAsync("MyCookieMiddlewareInstance");
```


官方文件讀到這裡，我對 `ClaimsPrincipal` 與 `ClaimsIdentity` 產生疑問，所以順著找又找到下列的說明與文章。

## ClaimsIdentity 與 ClaimsPrincipal
簡單來說 ClaimsIdentity 指的是驗證的方法，例如: 帳號密碼、手機簡訊或Windows驗證等。
ClaimsPrincipal 則是指驗證的主體，也就是請求驗證的使用者。
在 ClaimsPrincipal 中有一個屬性為`IEnumerable<ClaimsIdentity> Identities`，這表示一個使用者可以有多種的驗證方式(驗證的方法)，來向 Server 證明自己是當事人。

## 非官方文件
下面列出在閱讀官方文件後，找到的補充資料。

### 英文
1. 解釋 ClaimsPrincipal 與 ClaimsIdentity間的關聯 by Andrew Lock ([連結](https://andrewlock.net/introduction-to-authentication-with-asp-net-core/))
1. Exploring the cookie authentication middleware in ASP.NET Core. by Andrew Lock ([連結](https://andrewlock.net/exploring-the-cookieauthenticationmiddleware-in-asp-net-core/))


### 中文
這裡列出四篇「杨晓东」的文章，用比較簡單的例子來說明Identity。
作者的網站裡還有很多ASP.NET Core的文章，就不一一例舉了。
1. ASP.NET Core 之 Identity 入门（一）([連結](http://www.cnblogs.com/savorboard/p/aspnetcore-identity.html))
1. ASP.NET Core 之 Identity 入门（二）([連結](http://www.cnblogs.com/savorboard/p/aspnetcore-identity2.html))
1. ASP.NET Core 之 Identity 入门（三）([連結](http://www.cnblogs.com/savorboard/p/aspnetcore-identity3.html))
1. ASP.NET Core 中的那些认证中间件及一些重要知识点([連結](http://www.cnblogs.com/savorboard/p/aspnetcore-authentication.html))

## 小結
目前看完這些資料後，尚未實作，先釐清Identity、Authentication與Authorization三者間的差異。
之後實後再回來補充資料。