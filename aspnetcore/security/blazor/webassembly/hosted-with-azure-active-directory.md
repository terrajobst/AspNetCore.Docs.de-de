---
title: Sichern einer Blazor ASP.NET gehosteten Core WebAssembly-App mit Azure Active Directory
author: guardrex
description: ''
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 04/08/2020
no-loc:
- Blazor
- SignalR
uid: security/blazor/webassembly/hosted-with-azure-active-directory
ms.openlocfilehash: 8fec9f585f42469665cf29069674a199e1626629
ms.sourcegitcommit: f0aeeab6ab6e09db713bb9b7862c45f4d447771b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/08/2020
ms.locfileid: "80977131"
---
# <a name="secure-an-aspnet-core-opno-locblazor-webassembly-hosted-app-with-azure-active-directory"></a>Sichern einer Blazor ASP.NET gehosteten Core WebAssembly-App mit Azure Active Directory

Von [Javier Calvarro Nelson](https://github.com/javiercn) und Luke [Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

[!INCLUDE[](~/includes/blazorwasm-3.2-template-article-notice.md)]



In diesem Artikel wird beschrieben, wie Sie eine [ Blazor gehostete WebAssembly-App](xref:blazor/hosting-models#blazor-webassembly) erstellen, die [Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) für die Authentifizierung verwendet.

## <a name="register-apps-in-aad-b2c-and-create-solution"></a>Registrieren von Apps in AAD B2C und Erstellen einer Lösung

### <a name="create-a-tenant"></a>Erstellen eines Mandanten

Befolgen Sie die Anleitung unter [Schnellstart: Richten Sie einen Mandanten ein,](/azure/active-directory/develop/quickstart-create-new-tenant) um einen Mandanten in AAD zu erstellen.

### <a name="register-a-server-api-app"></a>Registrieren einer Server-API-App

Befolgen Sie die Anleitung unter [Schnellstart: Registrieren Sie eine Anwendung mit der Microsoft-Identitätsplattform](/azure/active-directory/develop/quickstart-register-app) und nachfolgenden Azure AAD-Themen, um eine AAD-App für die *Server-API-App* im Registrierungsbereich der Azure Active > **Directory-App** des Azure-Portals zu registrieren: **Azure Active Directory**

1. Wählen Sie **Neue Registrierung** aus.
1. Geben Sie einen **Namen** für die App an (z. B. ** Blazor Server AAD**).
1. Wählen Sie einen **unterstützten Kontotyp aus.** Für diese Erfahrung können Sie **nur Konten in diesem Organisationsverzeichnis** (Einzelmandant) auswählen.
1. Die *Server-API-App* benötigt in diesem Szenario keinen **Umleitungs-URI,** sodass die Dropdown-Liste auf **Web** festgelegt ist, und geben Sie keinen Umleitungs-URI ein.
1. Deaktivieren Sie das Kontrollkästchen **Permissions** > **Grant admin concent zum öffnen und offline_access Berechtigungen.**
1. Wählen Sie **Registrieren**.

Entfernen Sie in **API-Berechtigungen**die **Berechtigung Microsoft Graph** > **User.Read,** da die App keinen Anmelden oder keinen Profilzugriff erfordert.

In **Expose eine API**:

1. Wählen Sie **Bereich hinzufügen**.
1. Wählen Sie **Speichern und fortfahren** aus.
1. Geben Sie einen **Bereichsnamen** an (z. B. `API.Access`).
1. Geben Sie einen **Administrator-Zustimmungsanzeigenamen** an (z. B. `Access API`).
1. Geben Sie eine **Administrator-Zustimmungsbeschreibung** an (z. B. `Allows the app to access server app API endpoints.`).
1. Bestätigen Sie, dass der **Status** auf **Aktiviert**festgelegt ist.
1. Wählen Sie **Bereich hinzufügen** aus.

Zeichnen Sie die folgenden Informationen auf:

* *Server-API-App* Anwendungs-ID (Client-ID) `11111111-1111-1111-1111-111111111111`(z. B. )
* App-ID-URI (z. B. `https://contoso.onmicrosoft.com/11111111-1111-1111-1111-111111111111`, `api://11111111-1111-1111-1111-111111111111`oder der von Ihnen angegebene benutzerdefinierte Wert)
* Verzeichnis-ID (Tenant ID) `222222222-2222-2222-2222-222222222222`(z. B. )
* AAD-Mandantendomäne (z. B. `contoso.onmicrosoft.com`)
* Standardbereich (z. `API.Access`B. )

### <a name="register-a-client-app"></a>Registrieren einer Client-App

Befolgen Sie die Anleitung unter [Schnellstart: Registrieren Sie eine Anwendung mit der Microsoft-Identitätsplattform](/azure/active-directory/develop/quickstart-register-app) und nachfolgenden Azure AAD-Themen, um eine AAD-App für die *Client-App* im Registrierungsbereich der Azure Active > **Directory-App** im Azure-Portal zu registrieren: **Azure Active Directory**

1. Wählen Sie **Neue Registrierung** aus.
1. Geben Sie einen **Namen** für die App an (z. B. ** Blazor Client AAD**).
1. Wählen Sie einen **unterstützten Kontotyp aus.** Für diese Erfahrung können Sie **nur Konten in diesem Organisationsverzeichnis** (Einzelmandant) auswählen.
1. Lassen Sie die Droptauchliste **"Umleitungs-URI"** auf **Web**festgelegt, und stellen Sie einen Umleitungs-URI von `https://localhost:5001/authentication/login-callback`bereit.
1. Deaktivieren Sie das Kontrollkästchen **Permissions** > **Grant admin concent zum öffnen und offline_access Berechtigungen.**
1. Wählen Sie **Registrieren**.

In > **Authentifizierungsplattform-Konfigurationen** > **Web**: **Authentication**

1. Bestätigen Sie, dass `https://localhost:5001/authentication/login-callback` der **Umleitungs-URI** von vorhanden ist.
1. Aktivieren Sie für **Implizite Erteilung**die Kontrollkästchen für **Access-Token** und **ID-Token**.
1. Die verbleibenden Standardeinstellungen für die App sind für diese Erfahrung akzeptabel.
1. Wählen Sie die Schaltfläche **Speichern** aus.

In **API-Berechtigungen**:

1. Vergewissern Sie sich, dass die App über die **Berechtigung Microsoft Graph** > **User.Read** verfügt.
1. Wählen Sie **Eine Berechtigung hinzufügen,** gefolgt von **Meine APIs**.
1. Wählen Sie die *Server-API-App* aus der Spalte **Name** (z. B. ** Blazor Server AAD**) aus.
1. Öffnen Sie die **API-Liste.**
1. Aktivieren sie den Zugriff auf `API.Access`die API (z. B. ).
1. Wählen Sie **Berechtigungen hinzufügen** aus.
1. Wählen Sie die Schaltfläche **Admin-Inhalt für "TENANT NAME"** gewähren aus. Klicken Sie auf **Ja**, um zu bestätigen.

Zeichnen Sie die *Client-App-Anwendungs-ID* `33333333-3333-3333-3333-333333333333`(Client-ID) auf (z. B. ).

### <a name="create-the-app"></a>Erstellen der App

Ersetzen Sie die Platzhalter im folgenden Befehl durch die zuvor aufgezeichneten Informationen, und führen Sie den Befehl in einer Befehlsshell aus:

```dotnetcli
dotnet new blazorwasm -au SingleOrg --api-client-id "{SERVER API APP CLIENT ID}" --app-id-uri "{SERVER API APP ID URI}" --client-id "{CLIENT APP CLIENT ID}" --default-scope "{DEFAULT SCOPE}" --domain "{DOMAIN}" -ho --tenant-id "{TENANT ID}"
```

Um den Ausgabespeicherort anzugeben, der einen Projektordner erstellt, falls dieser nicht vorhanden ist, fügen `-o BlazorSample`Sie die Ausgabeoption in den Befehl mit einem Pfad ein (z. B. ). Der Ordnername wird auch Teil des Projektnamens.

> [!NOTE]
> Übergeben Sie den App-ID-URI an die `app-id-uri` Option, beachten Sie jedoch, dass möglicherweise eine Konfigurationsänderung in der Client-App erforderlich ist, die im Abschnitt [Access-Tokenbereiche](#access-token-scopes) beschrieben wird.

## <a name="server-app-configuration"></a>Server-App-Konfiguration

*Dieser Abschnitt bezieht sich auf die **Server-App** der Lösung.*

### <a name="authentication-package"></a>Authentifizierungspaket

Die Unterstützung für die Authentifizierung und Autorisierung von Aufrufen von `Microsoft.AspNetCore.Authentication.AzureAD.UI`ASP.NET Core Web APIs wird von der bereitgestellt:

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.AzureAD.UI" 
    Version="3.1.0" />
```

### <a name="authentication-service-support"></a>Unterstützung des Authentifizierungsdienstes

Die `AddAuthentication` Methode richtet Authentifizierungsdienste in der App ein und konfiguriert den JWT Bearer-Handler als Standardauthentifizierungsmethode. Die `AddAzureADBearer` Methode richtet die spezifischen Parameter im JWT Bearer-Handler ein, die zum Überprüfen von Token erforderlich sind, die von Azure Active Directory emittiert werden:

```csharp
services.AddAuthentication(AzureADDefaults.BearerAuthenticationScheme)
    .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
```

`UseAuthentication`und `UseAuthorization` stellen Sie sicher, dass:

* Die App versucht, Token für eingehende Anforderungen zu analysieren und zu validieren.
* Jede Anforderung, die versucht, ohne ordnungsgemäße Anmeldeinformationen auf eine geschützte Ressource zuzugreifen, schlägt fehl.

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### <a name="useridentityname"></a>User.Identity.Name

Standardmäßig füllt die Server-App-API `User.Identity.Name` den Wert `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` aus dem Anspruchstyp (z. `2d64b3da-d9d5-42c6-9352-53d8df33d770@contoso.onmicrosoft.com`B. ).

Um die App so zu `name` konfigurieren, dass sie den Wert vom Anspruchstyp empfängt, konfigurieren Sie die [TokenValidationParameters.NameClaimType](xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType) der <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> in: `Startup.ConfigureServices`

```csharp
services.Configure<JwtBearerOptions>(
    AzureADDefaults.JwtBearerAuthenticationScheme, options =>
    {
        options.TokenValidationParameters.NameClaimType = "name";
    });
```

### <a name="app-settings"></a>App-Einstellungen

Die Datei *appsettings.json* enthält die Optionen zum Konfigurieren des JWT-Trägerhandlers, der zum Überprüfen von Zugriffstoken verwendet wird.

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "{DOMAIN}",
    "TenantId": "{TENANT ID}",
    "ClientId": "{API CLIENT ID}",
  }
}
```

### <a name="weatherforecast-controller"></a>WeatherForecast-Controller

Der WeatherForecast-Controller (*Controllers/WeatherForecastController.cs*) macht `[Authorize]` eine geschützte API mit dem attribut verfügbar, das auf den Controller angewendet wird. Es ist **wichtig** zu verstehen, dass:

* Das `[Authorize]` Attribut in diesem API-Controller ist das einzige, was diese API vor unbefugtem Zugriff schützt.
* Das `[Authorize]` in der Blazor WebAssembly-App verwendete Attribut dient nur als Hinweis für die App, dass der Benutzer autorisiert werden soll, damit die App ordnungsgemäß funktioniert.

```csharp
[Authorize]
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        ...
    }
}
```

## <a name="client-app-configuration"></a>Client-App-Konfiguration

*Dieser Abschnitt bezieht sich auf die **Client-App** der Lösung.*

### <a name="authentication-package"></a>Authentifizierungspaket

Wenn eine App für die Verwendung`SingleOrg`von Arbeits- oder Schulkonten ( erstellt`Microsoft.Authentication.WebAssembly.Msal`wird, erhält die App automatisch eine Paketreferenz für die [Microsoft-Authentifizierungsbibliothek](/azure/active-directory/develop/msal-overview) ( ). Das Paket stellt eine Reihe von Primitiven bereit, die der App helfen, Benutzer zu authentifizieren und Token zum Aufrufen geschützter APIs zu erhalten.

Wenn Sie einer App eine Authentifizierung hinzufügen, fügen Sie das Paket manuell zur Projektdatei der App hinzu:

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
    Version="{VERSION}" />
```

Ersetzen `{VERSION}` Sie im vorherigen Paketverweis `Microsoft.AspNetCore.Blazor.Templates` durch die <xref:blazor/get-started> im Artikel gezeigte Version des Pakets.

Das `Microsoft.Authentication.WebAssembly.Msal` Paket fügt das `Microsoft.AspNetCore.Components.WebAssembly.Authentication` Paket transitiv zur App hinzu.

### <a name="authentication-service-support"></a>Unterstützung des Authentifizierungsdienstes

Unterstützung für die Authentifizierung von Benutzern wird `AddMsalAuthentication` im Dienstcontainer `Microsoft.Authentication.WebAssembly.Msal` mit der vom Paket bereitgestellten Erweiterungsmethode registriert. Diese Methode richtet alle Dienste ein, die für die Interaktion der App mit dem Identitätsanbieter (IDENTITY Provider, IP) erforderlich sind.

*Program.cs*:

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    var authentication = options.ProviderOptions.Authentication;
    authentication.Authority = "https://login.microsoftonline.com/{TENANT ID}";
    authentication.ClientId = "{CLIENT ID}";
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

Die `AddMsalAuthentication` Methode akzeptiert einen Rückruf, um die Parameter zu konfigurieren, die zum Authentifizieren einer App erforderlich sind. Die zum Konfigurieren der App erforderlichen Werte können bei der Registrierung der App über die Azure Portal AAD-Konfiguration abgerufen werden.

### <a name="access-token-scopes"></a>Zugriffstokenbereiche

Die Standardzugriffstokenbereiche stellen die Liste der Zugriffstokenbereiche dar, die:

* Standardmäßig in der Anmeldeanforderung enthalten.
* Wird verwendet, um ein Zugriffstoken unmittelbar nach der Authentifizierung zu bereitstellen.

Alle Bereiche müssen gemäß Azure Active Directory-Regeln derselben App angehören. Bei Bedarf können zusätzliche Bereiche für zusätzliche API-Apps hinzugefügt werden:

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

> [!NOTE]
> Wenn das Azure-Portal einen Bereichs-URI bereitstellt und **die App eine nicht behandelte Ausnahme auslöst,** wenn sie eine *401 nicht autorisierte* Antwort von der API empfängt, versuchen Sie, einen Bereichs-URI zu verwenden, der das Schema und den Host nicht enthält. Das Azure-Portal kann z. B. eines der folgenden Bereichs-URI-Formate bereitstellen:
>
> * `https://{ORGANIZATION}.onmicrosoft.com/{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}`
> * `api://{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}`
>
> Geben Sie den Bereichs-URI ohne Schema und Host:
>
> ```csharp
> options.ProviderOptions.DefaultAccessTokenScopes.Add(
>     "{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}");
> ```

Weitere Informationen finden Sie unter <xref:security/blazor/webassembly/additional-scenarios#request-additional-access-tokens>.

### <a name="imports-file"></a>Imports-Datei

[!INCLUDE[](~/includes/blazor-security/imports-file-hosted.md)]

### <a name="index-page"></a>Indexseite

[!INCLUDE[](~/includes/blazor-security/index-page-msal.md)]

### <a name="app-component"></a>App-Komponente

[!INCLUDE[](~/includes/blazor-security/app-component.md)]

### <a name="redirecttologin-component"></a>RedirectToLogin-Komponente

[!INCLUDE[](~/includes/blazor-security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>LoginDisplay-Komponente

[!INCLUDE[](~/includes/blazor-security/logindisplay-component.md)]

### <a name="authentication-component"></a>Authentifizierungskomponente

[!INCLUDE[](~/includes/blazor-security/authentication-component.md)]

### <a name="fetchdata-component"></a>FetchData-Komponente

[!INCLUDE[](~/includes/blazor-security/fetchdata-component.md)]

## <a name="run-the-app"></a>Ausführen der App

Führen Sie die App aus dem Serverprojekt aus. Wenn Sie Visual Studio verwenden, wählen Sie das Serverprojekt im **Projektmappen-Explorer** aus, und wählen Sie die Schaltfläche **Ausführen** in der Symbolleiste aus, oder starten Sie die App im **Menü Debug.**

<!-- HOLD
[!INCLUDE[](~/includes/blazor-security/usermanager-signinmanager.md)]
-->

[!INCLUDE[](~/includes/blazor-security/troubleshoot.md)]

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* [Zusätzliche Zugriffstoken anfordern](xref:security/blazor/webassembly/additional-scenarios#request-additional-access-tokens)
* <xref:security/authentication/azure-active-directory/index>
* [Dokumentation zur Microsoft Identity Platform](/azure/active-directory/develop/)
