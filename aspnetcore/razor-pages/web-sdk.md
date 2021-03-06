---
title: ASP.NET Core-Web SDK
author: Rick-Anderson
description: Übersicht über Microsoft.NET.Sdk.Web.
ms.author: riande
ms.date: 01/25/2020
no-loc:
- Blazor
uid: razor-pages/web-sdk
ms.openlocfilehash: 6a9d531efd2188aed525c949bb124914c31119db
ms.sourcegitcommit: f7886fd2e219db9d7ce27b16c0dc5901e658d64e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2020
ms.locfileid: "78648205"
---
# <a name="aspnet-core-web-sdk"></a>ASP.NET Core-Web SDK

### <a name="overview"></a>Übersicht

`Microsoft.NET.Sdk.Web` ist ein [MSBuild-Projekt-SDK](https://docs.microsoft.com/visualstudio/msbuild/how-to-use-project-sdk) zum Erstellen von ASP.NET Core-Apps. Es ist möglich, eine ASP.NET-Core-App ohne dieses SDK zu erstellen, für das Web SDK gilt jedoch Folgendes:

* Es ist darauf ausgerichtet, ein erstklassiges Erlebnis zu bieten.
* Es ist das empfohlene Ziel für die meisten Benutzer.

Verwenden Sie das Web.SDK in einem Projekt:

  ```xml
  <Project Sdk="Microsoft.NET.Sdk.Web">
    <!-- omitted for brevity -->
  </Project>
  ```

Features, die durch die Verwendung des Web SDK aktiviert werden:

* Projekte, die auf .NET Core 3.0 oder höher ausgerichtet sind, verweisen implizit auf:

  * Das [freigegebene ASP.NET Core-Framework](xref:fundamentals/metapackage-app).
  * [Analysetools](/visualstudio/extensibility/getting-started-with-roslyn-analyzers), die für die Erstellung von ASP.NET Core-Apps entwickelt wurden.
* Das Web SDK importiert MSBuild-Ziele, die die Verwendung von Veröffentlichungsprofilen und die Veröffentlichung mit WebDeploy ermöglichen.

### <a name="properties"></a>Eigenschaften

| Eigenschaft | Beschreibung |
| -------- | ----------- |
| `DisableImplicitFrameworkReferences` | Deaktiviert den impliziten Verweis auf das freigegebene `Microsoft.AspNetCore.App`-Framework. |
| `DisableImplicitAspNetCoreAnalyzers` | Deaktiviert den impliziten Verweis auf ASP.NET Core-Analysetools. |
| `DisableImplicitComponentsAnalyzers` | Deaktiviert den impliziten Verweis auf Analysetools von Razor-Komponenten beim Erstellen von Blazor-Anwendungen (Server). |
