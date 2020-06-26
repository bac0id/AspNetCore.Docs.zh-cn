---
title: 使用 Azure Active Directory B2C 保护 ASP.NET Core Blazor WebAssembly 独立应用
author: guardrex
description: ''
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/security/webassembly/standalone-with-azure-active-directory-b2c
ms.openlocfilehash: 2229483acc8c3a3fd6e77848e307253b7b317365
ms.sourcegitcommit: 490434a700ba8c5ed24d849bd99d8489858538e3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85103215"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-standalone-app-with-azure-active-directory-b2c"></a>使用 Azure Active Directory B2C 保护 ASP.NET Core Blazor WebAssembly 独立应用

作者：[Javier Calvarro Nelson](https://github.com/javiercn) 和 [Luke Latham](https://github.com/guardrex)

若要创建使用 [Azure Active Directory (AAD) B2C](/azure/active-directory-b2c/overview) 进行身份验证的 Blazor WebAssembly 独立应用，请执行以下操作：

请按照以下主题中的指南操作，以在 Azure 门户中创建租户并注册 Web 应用：

[创建 AAD B2C 租户](/azure/active-directory-b2c/tutorial-create-tenant)

记录以下信息：

* AAD B2C 实例（例如 `https://contoso.b2clogin.com/`，其中包含尾部斜杠）。
* AAD B2C 租户域（例如 `contoso.onmicrosoft.com`）。

请按照[教程：在 Azure Active Directory B2C 中注册应用程序](/azure/active-directory-b2c/tutorial-register-applications)中的指南操作，再次为客户端应用注册 AAD 应用，然后执行以下操作**：

1. 在“Azure Active Directory” > “应用注册”中，选择“新建注册”**** **** ****。
1. 提供应用的名称（例如 Blazor 独立 AAD B2C）**** ****。
1. 对于“支持的帐户类型”，请选择多租户选项****：任何组织目录或任何标识提供者中的帐户。**** 用于使用 Azure AD B2C 对用户进行身份验证。
1. 将“重定向 URI”下拉列表设置为“Web”，并提供以下重定向 URI：`https://localhost:{PORT}/authentication/login-callback`**** ****。 在 Kestrel 上运行的应用的默认端口为 5001。 如果应用在不同的 Kestrel 端口上运行，请使用应用的端口。 对于 IIS Express，可以在“调试”面板的应用属性中找到该应用随机生成的端口****。 由于此时应用不存在，并且 IIS Express 端口未知，因此请在创建应用后返回到此步骤，然后更新重定向 URI。 本主题后面部分会显示一个注解，以提醒 IIS Express 用户更新重定向 URI。
1. 确认已启用“权限” > “授予对 openid 和 offline_access 权限的管理员同意”**** ****。
1. 选择“注册”****。

记录应用程序 ID（客户端 ID）（例如 `11111111-1111-1111-1111-111111111111`）。

在“身份验证” > “平台配置” > “Web”**** **** **** 中，执行以下操作：

1. 确认存在 `https://localhost:{PORT}/authentication/login-callback` 的重定向 URI****。
1. 对于“隐式授权”，选中“访问令牌”和“ID 令牌”的复选框**** **** ****。
1. 此体验可接受应用的其余默认值。
1. 选择“保存”按钮****。

在“主页” > “Azure AD B2C” > “用户流”**** **** **** 中：

[创建注册和登录用户流](/azure/active-directory-b2c/tutorial-create-user-flows)

至少选择“应用程序声明” > “显示名称”用户属性以填充 `LoginDisplay` 组件 (Shared/LoginDisplay.razor) 中的 `context.User.Identity.Name`**** ******。

记录为应用创建的注册和登录用户流名称（例如 `B2C_1_signupsignin`）。

将以下命令中的占位符替换为前面记录的信息，然后在命令行界面中执行该命令：

```dotnetcli
dotnet new blazorwasm -au IndividualB2C --aad-b2c-instance "{AAD B2C INSTANCE}" --client-id "{CLIENT ID}" --domain "{TENANT DOMAIN}" -ssp "{SIGN UP OR SIGN IN POLICY}"
```

要指定输出位置（如果它不存在，则创建一个项目文件夹），请在命令中包含带有路径（例如 `-o BlazorSample`）的输出选项。 该文件夹名称还会成为项目名称的一部分。

> [!NOTE]
> 在 Azure 门户中，使用默认设置为在 Kestrel 服务器上运行的应用的端口 5001 配置应用的“身份验证” > “平台配置” > “Web” > “重定向 URI”**** **** **** ****。
>
> 如果应用是在随机 IIS Express 端口上运行的，则可以在“调试”面板的应用属性中找到该应用的端口****。
>
> 如果端口之前未使用应用的已知端口进行配置，请返回到 Azure 门户中应用的注册，并使用正确的端口更新重定向 URI。

创建应用后，应该能够：

* 使用 AAD 用户帐户登录到应用。
* 请求 Microsoft API 的访问令牌。 有关详情，请参阅：
  * [访问令牌作用域](#access-token-scopes)
  * [快速入门：配置应用程序来公开 Web API](/azure/active-directory/develop/quickstart-configure-app-expose-web-apis)。

## <a name="authentication-package"></a>身份验证包

创建应用以使用个人 B2C 帐户 (`IndividualB2C`) 时，应用会自动接收 [Microsoft 身份验证库](/azure/active-directory/develop/msal-overview) ([Microsoft.Authentication.WebAssembly.Msal](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/)) 的包引用。 此包提供了一组基元，可帮助应用验证用户身份并获取令牌以调用受保护的 API。

如果向应用添加身份验证，请手动将包添加到应用的项目文件中：

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
  Version="3.2.0" />
```

[Microsoft.Authentication.WebAssembly.Msal](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/) 包将 [Microsoft.AspNetCore.Components.WebAssembly.Authentication](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) 包间接添加到应用中。

## <a name="authentication-service-support"></a>身份验证服务支持

使用由 [Microsoft.Authentication.WebAssembly.Msal](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/) 包提供的 <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> 扩展方法在服务容器中注册对用户进行身份验证的支持。 此方法设置应用与 Identity 提供者 (IP) 交互所需的所有服务。

Program.cs**:

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAdB2C", options.ProviderOptions.Authentication);
});
```

<xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> 方法接受回叫，以配置验证应用所需的参数。 注册应用时，可以从 AAD 配置中获取配置应用所需的值。

配置由 wwwroot/appsettings.json 文件提供**：

```json
{
  "AzureAdB2C": {
    "Authority": "{AAD B2C INSTANCE}{DOMAIN}/{SIGN UP OR SIGN IN POLICY}",
    "ClientId": "{CLIENT ID}",
    "ValidateAuthority": false
  }
}
```

示例：

```json
{
  "AzureAdB2C": {
    "Authority": "https://contoso.b2clogin.com/contoso.onmicrosoft.com/B2C_1_signupsignin1",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
    "ValidateAuthority": false
  }
}
```

## <a name="access-token-scopes"></a>访问令牌作用域

Blazor WebAssembly 模板不会自动将应用配置为请求安全 API 的访问令牌。 要将访问令牌作为登录流程的一部分进行预配，请将作用域添加到 <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions> 的默认访问令牌作用域中：

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

[!INCLUDE[](~/includes/blazor-security/azure-scope.md)]

有关详细信息，请参阅“其他方案”一文的以下部分**：

* [请求其他访问令牌](xref:blazor/security/webassembly/additional-scenarios#request-additional-access-tokens)
* [将令牌附加到传出请求](xref:blazor/security/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)

## <a name="imports-file"></a>导入文件

[!INCLUDE[](~/includes/blazor-security/imports-file-standalone.md)]

## <a name="index-page"></a>索引页

[!INCLUDE[](~/includes/blazor-security/index-page-msal.md)]

## <a name="app-component"></a>应用组件

[!INCLUDE[](~/includes/blazor-security/app-component.md)]

## <a name="redirecttologin-component"></a>RedirectToLogin 组件

[!INCLUDE[](~/includes/blazor-security/redirecttologin-component.md)]

## <a name="logindisplay-component"></a>LoginDisplay 组件

[!INCLUDE[](~/includes/blazor-security/logindisplay-component.md)]

## <a name="authentication-component"></a>身份验证组件

[!INCLUDE[](~/includes/blazor-security/authentication-component.md)]

[!INCLUDE[](~/includes/blazor-security/wasm-aad-b2c-userflows.md)]

[!INCLUDE[](~/includes/blazor-security/troubleshoot.md)]

## <a name="additional-resources"></a>其他资源

* <xref:blazor/security/webassembly/additional-scenarios>
* [使用安全默认客户端的应用中未经身份验证或未经授权的 Web API 请求](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:security/authentication/azure-ad-b2c>
* [教程：创建 Azure Active Directory B2C 租户](/azure/active-directory-b2c/tutorial-create-tenant)
* [Microsoft 标识平台文档](/azure/active-directory/develop/)