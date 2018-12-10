---
title: Redis-Backplane für ASP.NET Core SignalR-Skalierung
author: tdykstra
description: Erfahren Sie, wie Sie eine Redis-Rückwandplatine einrichten, um die horizontale Skalierung für eine ASP.NET Core SignalR-app zu aktivieren.
monikerRange: '>= aspnetcore-2.1'
ms.author: tdykstra
ms.custom: mvc
ms.date: 11/28/2018
uid: signalr/redis-backplane
ms.openlocfilehash: c8b09c0d482da344b54d167c0c9757167eaa6186
ms.sourcegitcommit: e9b99854b0a8021dafabee0db5e1338067f250a9
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452951"
---
# <a name="set-up-a-redis-backplane-for-aspnet-core-signalr-scale-out"></a>Richten Sie einen Redis-Backplane für ASP.NET Core SignalR-Skalierung

Durch [Andrew Stanton-Nurse](https://twitter.com/anurse), [Brady Gaster](https://twitter.com/bradygaster), und [Tom Dykstra](https://github.com/tdykstra),

In diesem Artikel wird erläutert, SignalR-spezifische Aspekte der Einrichtung einer [Redis](https://redis.io/) Server für das horizontale Skalieren einer ASP.NET Core SignalR-app verwenden.

## <a name="set-up-a-redis-backplane"></a>Richten Sie eine Redis-Rückwandplatine

* Bereitstellen eines Redis-Servers.

  Für die Produktion ist eine Redis-Rückwandplatine nur für vor-Ort-Infrastruktur empfohlen. Zur Minimierung der Wartezeit sollte sich der Redis-Server im selben Rechenzentrum wie die SignalR-app. Wenn Ihre SignalR-app in der Azure-Cloud ausgeführt wird, empfehlen wir anstelle einer Redis-Rückwandplatine Azure SignalR Service. Sie können den Azure Redis Cache-Dienst verwenden, für die Entwicklung und testumgebungen. Weitere Informationen finden Sie in den folgenden Ressourcen:

  * <xref:signalr/scale>
  * [Dokumentation zu redis](https://redis.io/)
  * [Azure Redis Cache-Dokumentation](https://docs.microsoft.com/en-us/azure/redis-cache/)

::: moniker range="= aspnetcore-2.1"

* Installieren Sie in der SignalR-app die `Microsoft.AspNetCore.SignalR.Redis` NuGet-Paket.

* In der `Startup.ConfigureServices` -Methode, rufen `AddRedis` nach `AddSignalR`:

  ```csharp
  services.AddSignalR().AddRedis("<your_Redis_connection_string>");
  ```

* Konfigurieren Sie Optionen aus, je nach Bedarf:
 
  Die meisten Optionen können festgelegt werden, in der Verbindungszeichenfolge oder in der [ConfigurationOptions](https://stackexchange.github.io/StackExchange.Redis/Configuration#configuration-options) Objekt. Optionen, die im angegebenen `ConfigurationOptions` legen Sie in der Verbindungszeichenfolge überschreiben.

  Das folgende Beispiel zeigt, wie Sie Optionen festlegen, der `ConfigurationOptions` Objekt. Dieses Beispiel fügt eine Kanalpräfix, damit mehrere Anwendungen dieselbe Redis-Instanz freigeben können wie im nächsten Schritt beschrieben.

  ```csharp
  services.AddSignalR()
    .AddRedis(connectionString, options => {
        options.Configuration.ChannelPrefix = "MyApp";
    });
  ```

  Im vorangehenden Code `options.Configuration` wird initialisiert, indem Sie den Inhalt in der Verbindungszeichenfolge angegeben wurde.

::: moniker-end

::: moniker range="> aspnetcore-2.1"

* Installieren Sie in der SignalR-app die `Microsoft.AspNetCore.SignalR.StackExchangeRedis` NuGet-Paket.

* In der `Startup.ConfigureServices` -Methode, rufen `AddStackExchangeRedis` nach `AddSignalR`:

  ```csharp
  services.AddSignalR().AddStackExchangeRedis("<your_Redis_connection_string>");
  ```

* Konfigurieren Sie Optionen aus, je nach Bedarf:
 
  Die meisten Optionen können festgelegt werden, in der Verbindungszeichenfolge oder in der [ConfigurationOptions](https://stackexchange.github.io/StackExchange.Redis/Configuration#configuration-options) Objekt. Optionen, die im angegebenen `ConfigurationOptions` legen Sie in der Verbindungszeichenfolge überschreiben.

  Das folgende Beispiel zeigt, wie Sie Optionen festlegen, der `ConfigurationOptions` Objekt. Dieses Beispiel fügt eine Kanalpräfix, damit mehrere Anwendungen dieselbe Redis-Instanz freigeben können wie im nächsten Schritt beschrieben.

  ```csharp
  services.AddSignalR()
    .AddStackExchangeRedis(connectionString, options => {
        options.Configuration.ChannelPrefix = "MyApp";
    });
  ```

  Im vorangehenden Code `options.Configuration` wird initialisiert, indem Sie den Inhalt in der Verbindungszeichenfolge angegeben wurde.

  Weitere Informationen zu Redis-Optionen finden Sie unter den [StackExchange Redis-Dokumentation](https://stackexchange.github.io/StackExchange.Redis/Configuration.html).

::: moniker-end

* Wenn Sie einen Redis-Server für mehrere SignalR-apps verwenden, verwenden Sie einen anderen Kanal-Präfix für die einzelnen SignalR-Apps.

  Festlegen von einem Kanalpräfix isoliert eine SignalR-app von anderen Benutzern, die Präfixe der anderen Kanal zu verwenden. Wenn Sie keine unterschiedliche Präfixen zuweisen, geht aus einer app an alle seine Clients gesendete Nachrichten an alle Clients für alle apps, die den Redis-Server als eine Rückwandplatine verwenden.

* Konfigurieren Sie Ihre Server Webfarm-Lastenausgleichs Software für persistente Sitzungen. Hier sind einige Beispiele der Dokumentation zur Vorgehensweise, die ein:

  * [IIS](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing)
  * [HAProxy](https://www.haproxy.com/blog/load-balancing-affinity-persistence-sticky-sessions-what-you-need-to-know/)
  * [Nginx](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#sticky)
  * [pfSense](https://www.netgate.com/docs/pfsense/loadbalancing/inbound-load-balancing.html#sticky-connections)

## <a name="redis-server-errors"></a>Redis-Server-Fehler

Wenn ein Redis-Server ausfällt, löst SignalR Ausnahmen, die angeben, dass die Nachricht übermittelt werden, wird nicht aus. Einige typische ausnahmemeldungen:

* *Fehler beim Schreiben einer Nachricht*
* *Fehler beim Aufrufen der hubmethode 'Methodenname'*
* *Fehler bei der Verbindung mit Redis*

SignalR anforderungsinhaltsdatenstrom nicht puffert, Nachrichten an diese senden, wenn der Server wieder verfügbar ist. Alle Nachrichten, die gesendet, während sich der Redis-Server ausgefallen ist, gehen verloren.

SignalR stellt automatisch wieder her, wenn der Redis-Server wieder verfügbar ist.

### <a name="custom-behavior-for-connection-failures"></a>Benutzerdefiniertes Verhalten für Verbindungsfehler

Hier ist ein Beispiel, das zeigt, wie Ereignisse durch Fehler beim Redis-Verbindung behandelt.

::: moniker range="= aspnetcore-2.1"

```csharp
services.AddSignalR()
        .AddRedis(o =>
        {
            o.ConnectionFactory = async writer =>
            {
                var config = new ConfigurationOptions
                {
                    AbortOnConnectFail = false
                };
                config.EndPoints.Add(IPAddress.Loopback, 0);
                config.SetDefaultPorts();
                var connection = await ConnectionMultiplexer.ConnectAsync(config, writer);
                connection.ConnectionFailed += (_, e) =>
                {
                    Console.WriteLine("Connection to Redis failed.");
                };

                if (!connection.IsConnected)
                {
                    Console.WriteLine("Did not connect to Redis.");
                }

                return connection;
            };
        });
```

::: moniker-end

::: moniker range="> aspnetcore-2.1"

```csharp
services.AddSignalR()
        .AddMessagePackProtocol()
        .AddStackExchangeRedis(o =>
        {
            o.ConnectionFactory = async writer =>
            {
                var config = new ConfigurationOptions
                {
                    AbortOnConnectFail = false
                };
                config.EndPoints.Add(IPAddress.Loopback, 0);
                config.SetDefaultPorts();
                var connection = await ConnectionMultiplexer.ConnectAsync(config, writer);
                connection.ConnectionFailed += (_, e) =>
                {
                    Console.WriteLine("Connection to Redis failed.");
                };

                if (!connection.IsConnected)
                {
                    Console.WriteLine("Did not connect to Redis.");
                }

                return connection;
            };
        });
```

::: moniker-end

## <a name="clustering"></a>Clusterbildung

Clustering ist eine Methode zum Erreichen der hochverfügbarkeit mit mehreren Redis-Server. Clustering wird offiziell nicht unterstützt, aber es funktioniert.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen finden Sie in den folgenden Ressourcen:

* <xref:signalr/scale>
* [Dokumentation zu redis](https://redis.io/documentation)
* [StackExchange Redis-Dokumentation](https://stackexchange.github.io/StackExchange.Redis/)
* [Azure Redis Cache-Dokumentation](https://docs.microsoft.com/en-us/azure/redis-cache/)