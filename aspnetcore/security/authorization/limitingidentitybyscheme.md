---
title: "Autorizar com um esquema específico - ASP.NET Core"
author: rick-anderson
description: "Este artigo explica como limitar a identidade de um esquema específico ao trabalhar com vários métodos de autenticação."
keywords: "ASP.NET Core, identidade, o esquema de autenticação"
ms.author: riande
manager: wpickett
ms.date: 10/12/2017
ms.topic: article
ms.assetid: d3d6ca1b-b4b5-4bf7-898e-dcd90ec1bf8c
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/authorization/limitingidentitybyscheme
ms.openlocfilehash: 8c9d068b88263d0c06b11a6b87416fb02885c475
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/10/2017
---
# <a name="authorize-with-a-specific-scheme"></a>Autorizar com um esquema específico

Em alguns cenários, como aplicativos de página única (SPAs), é comum usar vários métodos de autenticação. Por exemplo, o aplicativo pode usar autenticação baseada em cookie para fazer logon e autenticação do portador JWT para solicitações de JavaScript. Em alguns casos, o aplicativo pode ter várias instâncias de um manipulador de autenticação. Por exemplo, dois manipuladores de cookie onde um contém uma identidade básica e um é criado quando uma autenticação multifator (MFA) foi acionada. MFA pode ser acionado porque o usuário solicitou uma operação que requer segurança adicional.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

Um esquema de autenticação é chamado quando o serviço de autenticação é configurado durante a autenticação. Por exemplo:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthentication()
        .AddCookie(options => {
            options.LoginPath = "/Account/Unauthorized/";
            options.AccessDeniedPath = "/Account/Forbidden/";
        })
        .AddJwtBearer(options => {
            options.Audience = "http://localhost:5001/";
            options.Authority = "http://localhost:5000/";
        });
```

No código anterior, foram adicionados dois manipuladores de autenticação: um para cookies e outro para transmissão.

>[!NOTE]
>Especifica o esquema padrão resulta no `HttpContext.User` propriedade sendo definida como essa identidade. Se esse comportamento não for desejado, desabilite-o invocando a forma sem parâmetros de `AddAuthentication`.

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Esquemas de autenticação são nomeados quando middlewares de autenticação são configurados durante a autenticação. Por exemplo:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    // Code omitted for brevity

    app.UseCookieAuthentication(new CookieAuthenticationOptions()
    {
        AuthenticationScheme = "Cookie",
        LoginPath = "/Account/Unauthorized/",
        AccessDeniedPath = "/Account/Forbidden/",
        AutomaticAuthenticate = false
    });
    
    app.UseJwtBearerAuthentication(new JwtBearerOptions()
    {
        AuthenticationScheme = "Bearer",
        AutomaticAuthenticate = false,
        Audience = "http://localhost:5001/",
        Authority = "http://localhost:5000/",
        RequireHttpsMetadata = false
    });
```

No código anterior, foram adicionados dois middlewares de autenticação: um para cookies e outro para transmissão.

>[!NOTE]
>Especifica o esquema padrão resulta no `HttpContext.User` propriedade sendo definida como essa identidade. Se esse comportamento não for desejado, desabilite-o definindo o `AuthenticationOptions.AutomaticAuthenticate` propriedade `false`.

---

## <a name="selecting-the-scheme-with-the-authorize-attribute"></a>Selecionar o esquema com o atributo de autorização

No ponto de autorização, o aplicativo indica o manipulador a ser usado. Selecione o manipulador com o qual o aplicativo autorizará passando uma lista delimitada por vírgulas de esquemas de autenticação para `[Authorize]`. O `[Authorize]` atributo especifica o esquema de autenticação ou esquemas de usar, independentemente se um padrão está configurado. Por exemplo:

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
[Authorize(AuthenticationSchemes = AuthSchemes)]
public class MixedController : Controller
    // Requires the following imports:
    // using Microsoft.AspNetCore.Authentication.Cookies;
    // using Microsoft.AspNetCore.Authentication.JwtBearer;
    private const string AuthSchemes =
        CookieAuthenticationDefaults.AuthenticationScheme + "," +
        JwtBearerDefaults.AuthenticationScheme;
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
[Authorize(ActiveAuthenticationSchemes = AuthSchemes)]
public class MixedController : Controller
    // Requires the following imports:
    // using Microsoft.AspNetCore.Authentication.Cookies;
    // using Microsoft.AspNetCore.Authentication.JwtBearer;
    private const string AuthSchemes =
        CookieAuthenticationDefaults.AuthenticationScheme + "," +
        JwtBearerDefaults.AuthenticationScheme;
```

---

No exemplo anterior, o cookie e o portador manipuladores execute em terá a oportunidade de criar e acrescentar uma identidade para o usuário atual. Ao especificar um único esquema, o manipulador correspondente é executado.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
[Authorize(AuthenticationSchemes = 
    JwtBearerDefaults.AuthenticationScheme)]
public class MixedController : Controller
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
[Authorize(ActiveAuthenticationSchemes = 
    JwtBearerDefaults.AuthenticationScheme)]
public class MixedController : Controller
```

---

No código acima, somente o manipulador com o esquema de "Portador" em execução. Todas as identidades baseada em cookie são ignoradas.

## <a name="selecting-the-scheme-with-policies"></a>Selecionar o esquema com as políticas

Se você preferir especificar os esquemas de desejado [política](xref:security/authorization/policies), você pode definir o `AuthenticationSchemes` ao adicionar sua política de coleta:

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("Over18", policy =>
    {
        policy.AuthenticationSchemes.Add(JwtBearerDefaults.AuthenticationScheme);
        policy.RequireAuthenticatedUser();
        policy.Requirements.Add(new MinimumAgeRequirement());
    });
});
```

No exemplo anterior, a política de "O Over18" só é executada em relação a identidade criada pelo manipulador de "Portador". Usar a política, definindo o `[Authorize]` do atributo `Policy` propriedade:

```csharp
[Authorize(Policy = "Over18")]
public class RegistrationController : Controller
```
