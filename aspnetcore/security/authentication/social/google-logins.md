---
title: Google externe Anmeldung Setup in ASP.NET Core
author: rick-anderson
description: Dieses Tutorial veranschaulicht die Integration von Google-Konto der Benutzerauthentifizierung in eine vorhandene ASP.NET Core-app.
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 03/19/2020
uid: security/authentication/google-logins
ms.openlocfilehash: a114d23c25201c9fe31ad0397efaf99fe98a312a
ms.sourcegitcommit: 9b6e7f421c243963d5e419bdcfc5c4bde71499aa
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/21/2020
ms.locfileid: "79989772"
---
# <a name="google-external-login-setup-in-aspnet-core"></a>Google externe Anmeldung Setup in ASP.NET Core

Von [Valeriy Novytskyy](https://github.com/01binary) und [Rick Anderson](https://twitter.com/RickAndMSFT)

In diesem Tutorial wird gezeigt, wie Sie es Benutzern mithilfe des auf der [vorherigen Seite](xref:security/authentication/social/index)erstellten Projekts ASP.net Core 3,0 ermöglichen, sich mit Ihrem Google-Konto anzumelden.

## <a name="create-a-google-api-console-project-and-client-id"></a>Erstellen Sie ein Google API-Konsolen Projekt und eine Client-ID.

* Installieren Sie [Microsoft. aspnetcore. Authentication. Google](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Google).
* Navigieren Sie zu [integrieren der Google-Anmeldung in Ihre Web-App](https://developers.google.com/identity/sign-in/web/devconsole-project) , und wählen Sie **Projekt konfigurieren**aus.
* Wählen Sie im Dialogfeld **Konfigurieren des OAuth-Clients** den **Webserver**aus.
* Legen Sie im Textfeld **autorisierte Umleitungs-URIs** den Umleitungs-URI fest. Beispiel: `https://localhost:44312/signin-google`
* Speichern Sie die **Client-ID** und den **geheimen Client**Schlüssel.
* Wenn Sie den Standort bereitstellen, registrieren Sie die neue öffentliche URL in der **Google-Konsole**.

## <a name="store-the-google-client-id-and-secret"></a>Speichern der Google-Client-ID und des geheimen Schlüssels

Speichern Sie sensible Einstellungen wie die Google-Client-ID und die geheimen Werte mit [Secret Manager](xref:security/app-secrets). Führen Sie für dieses Beispiel die folgenden Schritte aus:

1. Initialisieren Sie das Projekt für die geheime Speicherung gemäß den Anweisungen unter [Aktivieren der geheimen Speicherung](xref:security/app-secrets#enable-secret-storage).
1. Speichern Sie die sensiblen Einstellungen im lokalen Geheimnis Speicher mit den geheimen Schlüsseln `Authentication:Google:ClientId` und `Authentication:Google:ClientSecret`:

    ```dotnetcli
    dotnet user-secrets set "Authentication:Google:ClientId" "<client-id>"
    dotnet user-secrets set "Authentication:Google:ClientSecret" "<client-secret>"
    ```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

Sie können Ihre API-Anmelde Informationen und die Verwendung in der [API-Konsole](https://console.developers.google.com/apis/dashboard)verwalten.

## <a name="configure-google-authentication"></a>Konfigurieren von Google-Authentifizierung

Fügen Sie den Google-Dienst zu `Startup.ConfigureServices`hinzu:

[!code-csharp[](~/security/authentication/social/social-code/3.x/StartupGoogle3x.cs?highlight=11-19)]

[!INCLUDE [default settings configuration](includes/default-settings2-2.md)]

## <a name="sign-in-with-google"></a>Mit Google anmelden

* Führen Sie die APP aus, und klicken Sie auf **Anmelden**. Eine Option zum Anmelden mit Google wird angezeigt.
* Klicken Sie auf die **Google** -Schaltfläche, die zur Authentifizierung an Google umgeleitet wird.
* Nachdem Sie Ihre Google-Anmelde Informationen eingegeben haben, werden Sie an die Website zurückgeleitet.

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

Weitere Informationen zu den von der Google-Authentifizierung unterstützten Konfigurationsoptionen finden Sie in der <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>-API-Referenz. Dies kann verwendet werden, um verschiedene Informationen über den Benutzer anzufordern.

## <a name="change-the-default-callback-uri"></a>Ändern des Standard-Rückruf-URI

Der URI-Segment `/signin-google` als den standardrückruf des Google-Authentifizierungsanbieter festgelegt ist. Sie können den standardrückruf-URI ändern, während der Konfiguration die Middleware für die Google-Authentifizierung über die geerbte [RemoteAuthenticationOptions.CallbackPath](/dotnet/api/microsoft.aspnetcore.authentication.remoteauthenticationoptions.callbackpath) Eigenschaft der [GoogleOptions](/dotnet/api/microsoft.aspnetcore.authentication.google.googleoptions) Klasse.

## <a name="troubleshooting"></a>Problembehandlung

* Wenn die Anmeldung nicht funktioniert und Sie keine Fehler erhalten, wechseln Sie in den Entwicklungsmodus, um das Problem zu vereinfachen.
* Wenn die Identität nicht durch Aufrufen von `services.AddIdentity` in `ConfigureServices`konfiguriert ist, wird versucht, *die Ergebnisse in argumumtexception zu authentifizieren: die Option "signinscheme" muss angegeben werden*. Die Projektvorlage, die in diesem Tutorial verwendete wird sichergestellt, dass dies geschehen ist.
* Wenn die Standortdatenbank nicht erstellt wurde, indem die ursprüngliche Migration anwenden, erhalten Sie *Fehler bei ein Datenbankvorgang beim Verarbeiten der Anforderung* Fehler. Wählen Sie **Migrationen anwenden** aus, um die Datenbank zu erstellen, und aktualisieren Sie die Seite, um den Fehler fortzusetzen.

## <a name="next-steps"></a>Nächste Schritte

* In diesem Artikel wurde gezeigt, wie Sie mit Google authentifiziert werden können. Führen Sie einen ähnlichen Ansatz für die Authentifizierung mit anderen Anbietern aufgeführt, auf die [Vorgängerseite](xref:security/authentication/social/index).
* Nachdem Sie die app in Azure veröffentlicht haben, setzen Sie die `ClientSecret` in der Google-API-Konsole zurück.
* Legen Sie die `Authentication:Google:ClientId` und `Authentication:Google:ClientSecret` Anwendungseinstellungen im Azure-Portal. Das Konfigurationssystem ist zum Lesen von Schlüsseln aus Umgebungsvariablen eingerichtet.
