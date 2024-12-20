# OpenTelemetry in Microservices

<img src="plaatjes/open-telemetry-logo.png" width="250" align="right" alt="Logo Open Telemetry">

*[Wouter Gosseling, oktober 2024.](https://github.com/hanaim-devops/devops-blog-Woutuhr)*

In deze blogpost neem ik, samen met jou, een kijkje in de wereld van OpenTelemetry, een monitoring tool voor jouw applicatie(s). Ik bespreek de belangrijkste concepten, hoe je OpenTelemetry (in de basis) implementeert, en concludeer hoe bruikbaar OpenTelemetry is voor een Microservice architectuur.

- [OpenTelemetry in Microservices](#opentelemetry-in-microservices)
  - [The basics](#the-basics)
  - [An overview](#an-overview)
    - [Signals, a deeper dive](#signals-a-deeper-dive)
      - [Traces](#traces)
    - [Samenwerking](#samenwerking)
  - [How do they do it](#how-do-they-do-it)
    - [Setting up](#setting-up)
    - [Automatische instrumentatie toevoegen](#automatische-instrumentatie-toevoegen)
    - [Handmatige instrumentatie toevoegen](#handmatige-instrumentatie-toevoegen)
    - [Resultaat](#resultaat)
  - [Best practices](#best-practices)
  - [Conclusie](#conclusie)
  - [Bronnen](#bronnen)

## The basics

OpenTelemetry (ook bekend als OTel) verzamelt data zoals traces, logs en metrics van je applicaties en services. Deze informatie geeft inzicht in hoe je systeem presteert, waar problemen optreden, en helpt bij het opsporen van bottlenecks of fouten. Hierdoor kun je makkelijker en sneller problemen oplossen.

"A major goal of OpenTelemetry is that you can easily instrument your applications or systems, no matter their language, infrastructure, or runtime environment. The storage and visualization of telemetry is intentionally left to other tools."
(OpenTelemetry, 2024)

De bovenstaande quote laat zien dat OpenTelemetry ontworpen is om observability makkelijk en universeel toepasbaar te maken, ongeacht de gebruikte programmeertaal, infrastructuur of runtime. Dit is vooral relevant voor microservices, omdat deze vaak bestaan uit verschillende services die ook nog eens in diverse talen geschreven zijn (polyglot architectuur). OpenTelemetry biedt een standaard manier om deze polyglot omgevingen te instrumenteren, zonder gebonden te zijn aan een specifieke opslag- of visualisatietool, wat zorgt voor flexibiliteit en schaalbaarheid.

## An overview

<img src="plaatjes/opentelemetry-architecture.png" height="400" align="right" alt="Architectuur van OTel">

OpenTelemetry (verder benoemd als OTel) is een uitgebreide tool met veel geavanceerde functies. Om het eenvoudig te maken, focus ik op de belangrijkste concepten die je nodig hebt om aan de slag te gaan. Hieronder vind je een overzicht van deze concepten, samen met een korte uitleg.

- **Signals**<br>
  Een Signal is een overkoepelend concept binnen OTel welke in de basis system outputs zijn welke een activiteit binnen het systeem beschrijven. Een signal kan in meerdere vormen voorkomen, namelijk:

  - Traces
  - Metrics
  - Logs

  Het concept Signal is [onderstaand](#signals-a-deeper-dive) verder beschreven.
  
  (OpenTelemetry, 2024)

- **Context Propagation**<br>
  Binnen OpenTelemetry kunnen metadata, zoals trace-ID's, wordt doorgegeven tussen microservices tijdens een verzoek. Dit heet Context Propagation. Dit helpt om de volledige route van het verzoek te volgen over bijv. verschillende microservices, waardoor je beter kunt zien waar problemen zich voordoen en hoe de prestaties van het systeem zijn.

- **Semantic Concepts**<br>
  Dit zijn gestandaardiseerde afspraken over de naamgeving en structuur van telemetry data. Dit zorgt ervoor dat gegevens consistent opgeslagen worden. Dit maakt het makkelijker om data te analyseren en te vergelijken. Voorbeelden zijn gestandaardiseerde namen voor HTTP-methoden zoals `http.request.method` en `server.address` (OpenTelemetry, 2024).
  
- **The OpenTelemetry Collector**<br>
  Dit component verzamelt, verwerkt en exporteert data naar verschillende observability-tools. Het fungeert als een tussenstation dat gegevens uit verschillende bronnen samenbrengt, zodat je deze kunt analyseren en visualiseren. Hierdoor wordt het eenvoudiger om inzicht te krijgen in de prestaties en het gedrag van je systemen. De OpenTelemetry Collector kan draaien als een zelfstandige service, in containers, of als sidecar naast applicaties.

### Signals, a deeper dive

Zoals eerder benoemd is een Signal een overkoepelende term, binnen OTel komt een Signal voor in één van drie vormen:

- **Traces**<br>
  Dit zijn gegevens die een verzoek (door verschillende microservices) volgen. Ze geven inzicht in bijv. hoe lang elke stap duurt en waar eventuele vertragingen of fouten zich voordoen. Traces zijn [onderstaand](#traces) verder uitgelegd.
- **Metrics**<br>
  Dit zijn numerieke waarden die informatie geven over de prestaties van een systeem, zoals responstijden, CPU-gebruik en foutpercentages. Metrics het monitoren van de algehele gezondheid van de applicatie.
- **Logs**<br>
  Dit zijn registraties van gebeurtenissen die plaatsvinden binnen een systeem. Logs bieden context over wat er is gebeurd op specifieke momenten, wat helpt bij foutopsporing en analyse.

#### Traces

Bij een Signal van het type Trace bestaat deze uit een nieuw concept, namelijk een Span. Een Span vertegenwoordigt een specifieke actie of bewerking binnen een Trace, zoals een API-aanroep of een database-query. Samen vormen de spans een compleet overzicht van de flow van een verzoek door verschillende microservices.

Onderstaand is de relatie tussen een Trace en een span als tijdlijn weergegeven.

```text
––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time

 [Span A···················································]
   [Span B··········································]
      [Span D······································]
    [Span C····················································]
         [Span E·······]        [Span F··]
```

*bron: https://opentelemetry.io/docs/specs/otel/overview/* 

Belangrijk om te weten is dat een Span altijd de volgende informatie bevat:

- Een naam voor de operatie;
- Een start- en eindtijd;
- Attributes: Een lijst van sleutel-waarde paren;
- Een set van nul of meer Events als Tuple (timestamp, naam, Attributes);
- Nul of meer links naar gerelateerde Spans;
- SpanContext: de informatie welke nodig is om alles aan elkaar te koppelen.

### Samenwerking

<img src="plaatjes/concept-overview.png" height="200" align="right" alt="Concepten van OTel">

In de onderstaande afbeelding zie je een conceptueel en simpele OpenTelemetry-instantie. Hierbij zijn twee applicaties welke Signals sturen naar de OpenTelemetry Collector. Binnen deze collector staat een Enrichment Processor, deze kan bijv. Signals filteren, samenvoegen, enz. Mocht je het interessant vinden, zie dan de [docs](https://opentelemetry.io/docs/collector/) over de Collector. De Collector exporteert de data daarna weer naar een externe database. Zoals er te zien is, wordt alle data geëxporteerd naar de "Telemetry Database", in de praktijk zal dit verwisseld worden met een tool als [Grafana](https://grafana.com/) of [Jaeger](https://www.jaegertracing.io/).

## How do they do it

Naast te weten hoe OpenTelemetry (in grote lijnen) werkt, is het ook handig om een idee te hebben hoe dit er in code uit ziet. Voor dit voorbeeld maak ik gebruik van de programmeertaal .NET. Hierbij maak ik een simpele Web API welke een bestelsysteem moet voorstellen. Hierbij kan je een bestelling plaatsen naar `POST /Order`. Om het makkelijk te houden, genereert de app zelf een bestelling.

### Setting up

Begin met het opzetten van een .NET Web API en het toevoegen van de volgende NuGet packages:

- Microsoft.EntityFrameworkCore
- Microsoft.EntityFrameworkCore.InMemory

Omdat deze demo een simulatie is, maak ik gebruik van een In Memory database. Ik ga niet teveel in detail over hoe dit werkt, maar ik laat de stappen wel zien. Begin hiervoor met het Order model aan te maken:

```C#
using System.ComponentModel.DataAnnotations;

namespace OpenTelemetry.Demo.DAL;

public class Order {
  [Key]
  public int Id { get; set; }
  public Guid OrderNumber { get; set; }
}
```

Maak hierna een DbContext aan voor deze applicatie:

```C#
using Microsoft.EntityFrameworkCore;

namespace OpenTelemetry.Demo.DAL;

public class DatabaseContext(DbContextOptions<DatabaseContext> options): DbContext(options) {
  public DbSet<Order> Orders { get; set; }
}
```

En voeg deze DbContext toe aan jouw `Program.cs`:

```C#
using Microsoft.EntityFrameworkCore;

...

builder.Services.AddDbContext<DatabaseContext>(options =>{
  options.UseInMemoryDatabase("OpenTelemetryDemo");
});
```

Voeg als laatste een controller toe:

```C#
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using OpenTelemetry.Demo.DAL;

namespace OpenTelemetry.Demo.Controllers;

[ApiController]
[Route("[controller]")]
public class OrderController(DatabaseContext dbContext): ControllerBase 
{
  [HttpPost]
  public async Task<IActionResult> Post()
  {
    var orderNumber = Guid.NewGuid();
    var order = new Order
    {
      OrderNumber = orderNumber
    };
    
    dbContext.Orders.Add(order);
    await dbContext.SaveChangesAsync();

    return Created();
  }
  
  [HttpGet]
  public async Task<List<Order>> Get()
  {
    return await dbContext.Orders.ToListAsync();
  }
}
```

### Automatische instrumentatie toevoegen

Begin met het toevoegen van de OpenTelemetry packages:

- OpenTelemetry.Extensions.Hosting
- OpenTelemetry.Instrumentation.AspNetCore
- OpenTelemetry.Instrumentation.Http
- OpenTelemetry.Exporter.Console

Hierna is het e.e.a. aan setup nodig, hiervoor voeg je de het volgende aan de `Program.cs` toe:

```C#
builder.Services.AddOpenTelemetry()
    .ConfigureResource(configure => configure.AddService("OrderService"))
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddConsoleExporter()
    ).WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddConsoleExporter()
    ).WithLogging(logging => logging
        .AddConsoleExporter()
    );
```

In dit voorbeeld gebruik ik de Console Exporter (de `OpenTelemetry.Exporter.Console` package). In productie kun je naar een externe tool exporteren voor loganalyse. Alle drie de signalen zijn geïnitieerd, maar je kunt delen, zoals Metrics, weglaten als je ze niet nodig hebt.

Bij bij zowel de Tracing als de Metrics setup komen de method calls `AddAspNetCoreInstrumentation` en `AddHttpClientInstrumentation`. Deze zijn toegevoegd vanuit de eerder genoemde packages, en voegen "automatische instrumentatie" toe. Dit houdt in dat, zonder verder code aan te passen, er  gemeten, gelogd en getraced wordt binnen de applicatie.

### Handmatige instrumentatie toevoegen

In de voorbeeldapplicatie wil ik meten hoe lang het duurt om een order aan te maken. Daarom voeg ik een trace toe in .NET. We updaten hiervoor de controller met de volgende code:

```C#
using System.Diagnostics;

...

[ApiController]
[Route("[controller]")]
public class OrderController(DatabaseContext dbContext, ILogger<OrderController> logger): ControllerBase
{
    private static readonly ActivitySource ActivitySource = new(nameof(OrderController));

    [HttpPost]
    public async Task<IActionResult> Post()
    {
        using var activity = ActivitySource.StartActivity("CreateOrder", "1.0.0");
        var orderNumber = Guid.NewGuid();

        activity?.SetTag("operation.orderNumber", orderNumber);
        logger.LogInformation("Saving order with order number {orderNumber}", orderNumber);


        var order = new Order
        {
            OrderNumber = orderNumber
        };

        dbContext.Orders.Add(order);
        await dbContext.SaveChangesAsync();
        
        return Created();
    }

    ...
}

```

Verder is belangrijk dat deze ActivitySource ook wordt toegevoegd in de `Program.cs`, anders wordt deze niet in de console geëxporteerd. Don't ask how I know. Binnen de `WithTracing` method call voeg je het volgende toe:

```C#
  .AddSource(nameof(OrderController))
```

Het is belangrijk dat de naam overeenkomt met die in de controller. In dit voorbeeld gebruik ik de `nameof`-methode in .NET, die de naam en namespace van bijvoorbeeld een klasse retourneert.

Nu hoor ik je denken, ik ging toch een Trace inbouwen? Ik zie hier nergens Trace staan! Dat klopt, dit komt door de .NET-standaarden. In .NET worden `ActivitySource` en `Activity` gebruikt, begrippen die al langer bestaan en afkomstig zijn uit de `System.Diagnostics` namespace. OpenTelemetry heeft deze overgenomen. Eigenlijk is een `ActivitySource` hetzelfde als een `Tracer`, en een `Activity` staat gelijk aan een `Span`. (Thwaites, 2024)

Misschien is je ook opgevallen dat ik een Activity aanmaak evenals een Log. Dit zijn gelijk twee manieren om een stukje instrumentatie aan jouw code toe te voegen.

### Resultaat

Wanneer je de bovenstaande code geïmplementeerd hebt, krijg je, wanneer je de app runt en de POST endpoint aanroept, het onderstaande resultaat in jouw console.

```console
LogRecord.SpanId:                  36125aca871393a3
LogRecord.TraceFlags:              Recorded
LogRecord.CategoryName:            Microsoft.EntityFrameworkCore.Update
LogRecord.Severity:                Info
LogRecord.SeverityText:            Information
LogRecord.Body:                    Saved {count} entities to in-memory store.
LogRecord.Attributes (Key:Value):
    count: 1
    OriginalFormat (a.k.a Body): Saved {count} entities to in-memory store.
LogRecord.EventId:                 30100
LogRecord.EventName:               Microsoft.EntityFrameworkCore.Update.ChangesSaved

Resource associated with LogRecord:
service.name: OrderService
service.instance.id: f32347da-71e2-40b3-91df-6debe81e5ca3
telemetry.sdk.name: opentelemetry
telemetry.sdk.language: dotnet
telemetry.sdk.version: 1.9.0

Activity.TraceId:            1426c3ecd8373730fbca5c157a2158cc
Activity.SpanId:             36125aca871393a3
Activity.TraceFlags:         Recorded
Activity.ParentSpanId:       0e8e1e161f68f746
Activity.ActivitySourceName: OrderController
Activity.DisplayName:        CreateOrder
Activity.Kind:               Internal
Activity.StartTime:          2024-10-09T12:29:56.5415987Z
Activity.Duration:           00:00:02.8862412
Activity.Tags:
    operation.orderNumber: e0386547-d942-4085-89fe-cbcddd04b2a4
Resource associated with Activity:
    service.name: OrderService
    service.instance.id: f32347da-71e2-40b3-91df-6debe81e5ca3
    telemetry.sdk.name: opentelemetry
    telemetry.sdk.language: dotnet
    telemetry.sdk.version: 1.9.0

Activity.TraceId:            1426c3ecd8373730fbca5c157a2158cc
Activity.SpanId:             0e8e1e161f68f746
Activity.TraceFlags:         Recorded
Activity.ActivitySourceName: Microsoft.AspNetCore
Activity.DisplayName:        POST Order
Activity.Kind:               Server
Activity.StartTime:          2024-10-09T12:29:56.1451350Z
Activity.Duration:           00:00:03.3330143
Activity.Tags:
    server.address: localhost
    server.port: 44314
    http.request.method: POST
    url.scheme: https
    url.path: /Order
    network.protocol.version: 2
    user_agent.original: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36
    http.route: Order
    http.response.status_code: 204
Resource associated with Activity:
    service.name: OrderService
    service.instance.id: f32347da-71e2-40b3-91df-6debe81e5ca3
    telemetry.sdk.name: opentelemetry
    telemetry.sdk.language: dotnet
    telemetry.sdk.version: 1.9.0

Resource associated with Metric:
    service.name: OrderService
    service.instance.id: f32347da-71e2-40b3-91df-6debe81e5ca3
    telemetry.sdk.name: opentelemetry
    telemetry.sdk.language: dotnet
    telemetry.sdk.version: 1.9.0
```

## Best practices

- **Begin klein**<br>
  Start met het instrumenteren van één belangrijke service of functie, en breid het vervolgens uit.
- **Gebruik automatische instrumentatie**<br>
  Waar mogelijk, gebruik automatische instrumentatie om snel inzicht te krijgen zonder veel codewijzigingen.
- **Kies relevante signalen**<br>
  Verzamel alleen de signalen die echt waardevol zijn, om ruis te verminderen.
- **Voeg nuttige attributen toe**<br>
  Zorg dat je spans en metrics duidelijke attributen hebben, zoals foutmeldingen of gebruikerscontext, om debugging eenvoudiger te maken.
- **Verzamel data op één plek**<br>
  Gebruik tools zoals een collector om alle telemetry data op één plaats samen te brengen voor analyse.

(Sommerville and Simons, 2024) (ChatGPT, 2024)

## Conclusie

OpenTelemetry (OTel) is een handige tool voor inzicht in applicaties. Met centrale logging en context propagation is het goed geschikt voor een microservice-architectuur. Een centrale telemetry service kan eenvoudig worden opgenomen in het landschap. Dankzij de automatische instrumentatie is de eerste stap om OTel in te bouwen minimaal, zoals aanbevolen door best practices. Daarnaast helpt OTel om sneller problemen te vinden en de prestaties van microservices te verbeteren, wat essentieel is voor de betrouwbaarheid en schaalbaarheid van je diensten.

## Bronnen

Getting started. (2024, Juni 2). OpenTelemetry. Geraadpleegd op 8 Oktober 2024, van https://opentelemetry.io/docs/languages/net/getting-started/<br>
Overview. (n.d.). OpenTelemetry. Geraadpleegd op 8 Oktober 2024, van https://opentelemetry.io/docs/specs/otel/overview/<br>
Semantic Conventions for HTTP metrics. (n.d.). OpenTelemetry. Geraadpleegd op 8 Oktober 2024, van https://opentelemetry.io/docs/specs/semconv/http/http-metrics/<br>
Signals. (n.d.). OpenTelemetry. Geraadpleegd op 8 Oktober 2024, van https://opentelemetry.io/docs/concepts/signals/<br>
Somerville, A., & Simons, H. (21 maart 2024). OpenTelemetry best practices: A user’s guide to getting started with OpenTelemetry. Grafana Labs. Geraadpleegd op 8 Oktober 2024, van https://grafana.com/blog/2023/12/18/opentelemetry-best-practices-a-users-guide-to-getting-started-with-opentelemetry/<br>
Thwaites, M. (10 Juni 2022). What the Hell is Activity Anyway? Honeycomb. Geraadpleegd op 9 Oktober 2024, van https://www.honeycomb.io/what-is-activity-in-dotnet<br>
What is OpenTelemetry? (7 Augustus 2024). OpenTelemetry. Geraadpleegd op 8 Oktober 2024, van https://opentelemetry.io/docs/what-is-opentelemetry/<br>
ChatGPT. (n.d.). https://chatgpt.com/share/6703b59a-8b0c-8012-9e38-3001462c6a10