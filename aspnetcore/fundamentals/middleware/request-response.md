---
title: Anforderungs- und Antwortvorgänge in ASP.NET Core
author: jkotalik
description: Erfahren Sie, wie Sie in ASP.NET Core den Anforderungstext lesen und den Antworttext schreiben.
monikerRange: '>= aspnetcore-3.0'
ms.author: jukotali
ms.custom: mvc
ms.date: 08/29/2019
uid: fundamentals/middleware/request-response
ms.openlocfilehash: b473fa02e1d23f02bc5d2e15fa54ab7b1dbbb17c
ms.sourcegitcommit: f7886fd2e219db9d7ce27b16c0dc5901e658d64e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2020
ms.locfileid: "78650833"
---
# <a name="request-and-response-operations-in-aspnet-core"></a>Anforderungs- und Antwortvorgänge in ASP.NET Core

Von [Justin Kotalik](https://github.com/jkotalik)

In diesem Artikel wird erläutert, wie Sie den Anforderungstext lesen und den Antworttext schreiben. Möglicherweise ist Code für diese Vorgänge erforderlich, wenn Sie Middleware schreiben. Abgesehen vom Schreiben von Middleware ist benutzerdefinierter Code in der Regel nicht erforderlich, da die Vorgänge von MVC und Razor Pages behandelt werden.

Es stehen zwei Abstraktionen für die Anforderungs- und Antworttexte zur Verfügung: <xref:System.IO.Stream> und <xref:System.IO.Pipelines.Pipe>. [HttpRequest.Body](xref:Microsoft.AspNetCore.Http.HttpRequest.Body) ist ein <xref:System.IO.Stream> zum Lesen der Anforderung, und `HttpRequest.BodyReader` ist ein <xref:System.IO.Pipelines.PipeReader>. [HttpResponse.Body](xref:Microsoft.AspNetCore.Http.HttpResponse.Body) ist eine <xref:System.IO.Stream> zum Schreiben der Antwort, und `HttpResponse.BodyWriter` ist eine <xref:System.IO.Pipelines.PipeWriter>.

Anstelle von Datenströmen werden [Pipelines](/dotnet/standard/io/pipelines) empfohlen. Datenströme können bei einigen einfachen Vorgängen einfacher zu verwenden sein, aber Pipelines haben einen Leistungsvorteil und sind in den meisten Szenarien einfacher zu verwenden. ASP.NET Core beginnt, intern Pipelines anstelle von Datenströmen zu verwenden. Beispiele:

* `FormReader`
* `TextReader`
* `TextWriter`
* `HttpResponse.WriteAsync`

Datenströme werden jedoch nicht aus dem Framework entfernt. Datenströme werden weiterhin in .NET verwendet, und für viele Datenstromtypen gibt es keine entsprechenden Pipelines, z. B. `FileStreams` und `ResponseCompression`.

## <a name="stream-examples"></a>Datenstrombeispiele

Angenommen, Sie möchten eine Middleware erstellen, die den gesamten Anforderungstext als Liste von Zeichenfolgen liest, die in neue Zeilen umbrochen werden. Eine einfache Datenstromimplementierung könnte wie im folgenden Beispiel aussehen:

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringsFromStream)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

Dieser Code funktioniert, aber es gibt einige Probleme:

* Vor dem Anfügen an den `StringBuilder` wird im Beispiel eine andere Zeichenfolge erstellt (`encodedString`), die sofort verworfen wird. Weil dieser Vorgang für alle Bytes im Datenstrom erfolgt, ist das Ergebnis eine zusätzliche Speicherzuweisung in der Größe des gesamten Anforderungstexts.
* Im Beispiel wird die gesamte Zeichenfolge vor dem Umbruch in neue Zeilen gelesen. Es ist viel effizienter, das Bytearray auf neue Zeilen zu überprüfen.

Es folgt ein Beispiel, in dem einige dieser Probleme behoben werden:

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringsFromStreamMoreEfficient)]

Dieses obige Beispiel:

* Puffert nicht den gesamten Anforderungstext in einem `StringBuilder`, wenn keine Zeilenvorschubzeichen vorhanden sind.
* Ruft nicht `Split` für die Zeichenfolge auf.

Es gibt jedoch noch ein paar Probleme:

* Wenn nur wenige neue Zeilenvorschubzeichen vorliegen, wird ein großer Teil des Anforderungstexts in der Zeichenfolge gepuffert.
* Der Code erstellt weiterhin Zeichenfolgen (`remainingString`) und fügt diese zum Zeichenfolgenpuffer hinzu, was zu einer weiteren Zuteilung führt.

Diese Probleme können behoben werden, aber der Code wird immer komplizierter, und die Verbesserung ist gering. Pipelines bieten eine Möglichkeit, diese Probleme mit minimaler Codekomplexität zu lösen.

## <a name="pipelines"></a>Pipelines

Das folgende Beispiel zeigt, wie das gleiche Szenario mit einem `PipeReader` behandelt werden kann:

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringFromPipe)]

In diesem Beispiel werden viele Probleme der Datenstromimplementierungen behoben:

* Ein Zeichenfolgenpuffer ist nicht erforderlich, weil der `PipeReader` Bytes behandelt, die nicht verwendet wurden.
* Codierte Zeichenfolgen werden direkt der Liste der zurückgegebenen Zeichenfolgen hinzugefügt.
* Die Erstellung von Zeichenfolgen ist zuordnungsfrei, abgesehen von dem Speicher, den die Zeichenfolge belegt (mit Ausnahme des `ToArray()`-Aufrufs).

## <a name="adapters"></a>Adapter

Die Eigenschaften `Body` und `BodyReader/BodyWriter` sind für `HttpRequest` und `HttpResponse` verfügbar. Wenn Sie für `Body` einen anderen Datenstrom festlegen, passen neue Adapter die Typen automatisch aneinander an. Wenn Sie für `HttpRequest.Body` einen neuen Datenstrom festlegen, wird für `HttpRequest.BodyReader` automatisch ein neuer `PipeReader` festgelegt, der `HttpRequest.Body` umschließt.

## <a name="startasync"></a>StartAsync

`HttpResponse.StartAsync` wird verwendet, um anzugeben, dass Header nicht änderbar sind, und um `OnStarting`-Rückrufe auszuführen. Bei der Verwendung von Kestrel als Server garantiert der Aufruf von `StartAsync` vor der Verwendung von `PipeReader`, dass von `GetMemory` zurückgegebener Arbeitsspeicher zu internen <xref:System.IO.Pipelines.Pipe>-Pipeline von Kestrel gehört, anstatt einem externen Puffer.

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* [Einführung in System.IO.Pipelines](https://devblogs.microsoft.com/dotnet/system-io-pipelines-high-performance-io-in-net/)
* <xref:fundamentals/middleware/write>
