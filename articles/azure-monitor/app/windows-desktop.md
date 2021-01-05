---
title: Analyse des performances et de l’utilisation pour les applications de bureau Windows
description: Analysez l’utilisation et les performances de votre application de bureau Windows avec Application Insights.
ms.topic: conceptual
ms.date: 06/11/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 1aa1e8a9e7ccbbc90a961ebf47224f59f8a9e9fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91827863"
---
# <a name="monitoring-usage-and-performance-in-classic-windows-desktop-apps"></a>Analyse des niveaux de performance et de l’utilisation dans les applications de bureau Windows Classic

Les applications hébergées en local, dans Azure, et dans d’autres clouds peuvent toutes tirer profit d’Application Insights. La seule limitation réside dans la nécessité d’[autoriser la communication](./ip-addresses.md) vers le service Application Insights. Pour analyser des applications de plateforme Windows universelle (UWP), nous vous recommandons [Visual Studio App Center](../learn/mobile-center-quickstart.md).

## <a name="to-send-telemetry-to-application-insights-from-a-classic-windows-application"></a>Pour envoyer la télémétrie à Application Insights à partir d’une application Windows Classic
1. Dans le [portail Azure](https://portal.azure.com), [créez une ressource Application Insights](./create-new-resource.md). 
2. Copiez la clé d'instrumentation.
3. Dans Visual Studio, modifiez les packages NuGet de votre projet d’application et ajoutez Microsoft.ApplicationInsights.WindowsServer. (Ou choisissez Microsoft.ApplicationInsights si vous souhaitez simplement l’API de base, sans les modules de collecte de données de télémétrie standard.)
4. Définissez la clé d’instrumentation dans votre code :
   
    `TelemetryConfiguration.Active.InstrumentationKey = "`*votre clé*`";`
   
    ou dans ApplicationInsights.config (si vous avez installé l’un des packages de télémétrie standard) :
   
    `<InstrumentationKey>`*votre clé*`</InstrumentationKey>` 
   
    Si vous utilisez ApplicationInsights.config, assurez-vous que ses propriétés dans l’Explorateur de solutions sont définies sur **Build Action = Content, Copy to Output Directory = Copy**.
5. [Utilisez l’API](./api-custom-events-metrics.md) pour envoyer les données de télémétrie.
6. Exécutez votre application et consultez la télémétrie dans la ressource que vous avez créée dans le Portail Azure.

## <a name="example-code"></a><a name="telemetry"></a>Exemple de code

```csharp
using Microsoft.ApplicationInsights;

    public partial class Form1 : Form
    {
        private TelemetryClient tc = new TelemetryClient();
        ...
        private void Form1_Load(object sender, EventArgs e)
        {
            // Alternative to setting ikey in config file:
            tc.InstrumentationKey = "key copied from portal";

            // Set session data:
            tc.Context.Session.Id = Guid.NewGuid().ToString();
            tc.Context.Device.OperatingSystem = Environment.OSVersion.ToString();

            // Log a page view:
            tc.TrackPageView("Form1");
            ...
        }

        protected override void OnFormClosing(System.Windows.Forms.FormClosingEventArgs e)
        {
            if (tc != null)
            {
                tc.Flush(); // only for desktop apps

                // Allow time for flushing:
                System.Threading.Thread.Sleep(1000);
            }
            base.OnFormClosing(e);
        }
        
        ...
```

## <a name="override-storage-of-computer-name"></a>Remplacer le stockage du nom de l’ordinateur

Par défaut, ce kit de développement logiciel (SDK) collecte et stocke le nom d’ordinateur du système émettant la télémétrie.

Le nom de l’ordinateur est utilisé par le [niveau de prix Entreprise existant (par nœud)](./pricing.md#legacy-enterprise-per-node-pricing-tier) Application Insights pour des besoins de facturation interne. Par défaut, si vous utilisez un initialiseur de télémétrie pour remplacer `telemetry.Context.Cloud.RoleInstance`, une propriété distincte `ai.internal.nodeName` sera envoyée, qui contiendra encore la valeur de nom de l’ordinateur. Cette valeur ne sera pas stockée avec vos données de télémétrie Application Insights, mais elle sera utilisée en interne au moment de l’ingestion pour permettre une compatibilité descendante avec le modèle de facturation existant basé sur les nœuds.

Si vous êtes au [niveau de prix Entreprise existant (par nœud)](./pricing.md#legacy-enterprise-per-node-pricing-tier) et que vous avez simplement besoin de remplacer le stockage du nom de l’ordinateur, utilisez un initialiseur de télémétrie :

**Écrire un initialiseur TelemetryInitializer personnalisé comme ci-dessous.**

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;

namespace CustomInitializer.Telemetry
{
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleInstance))
            {
                // Set custom role name here. Providing an empty string will result
                // in the computer name still be sent via this property.
                  telemetry.Context.Cloud.RoleInstance = "Custom RoleInstance";
            }
        }
    }
}
```

Instanciez l’initialiseur dans la méthode `Program.cs` `Main()` ci-dessous, en définissant la clé d’instrumentation :

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;

   static void Main()
        {
            TelemetryConfiguration.Active.InstrumentationKey = "{Instrumentation-key-here}";
            TelemetryConfiguration.Active.TelemetryInitializers.Add(new MyTelemetryInitializer());
            //...
        }
```

## <a name="override-transmission-of-computer-name"></a>Remplacer la transmission du nom de l’ordinateur

Si vous n’êtes pas au [niveau de prix Entreprise existant (par nœud)](./pricing.md#legacy-enterprise-per-node-pricing-tier) et que vous souhaitez empêcher complètement l’envoi de données de télémétrie contenant le nom de l’ordinateur, vous devez utiliser un processeur de télémétrie.

### <a name="telemetry-processor"></a>Processeur de télémétrie

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;


namespace WindowsFormsApp2
{
    public class CustomTelemetryProcessor : ITelemetryProcessor
    {
        private readonly ITelemetryProcessor _next;

        public CustomTelemetryProcessor(ITelemetryProcessor next)
        {
            _next = next;
        }

        public void Process(ITelemetry item)
        {
            if (item != null)
            {
                item.Context.Cloud.RoleInstance = string.Empty;
            }

            _next.Process(item);
        }
    }
}
```

Instanciez le processeur de télémétrie dans la méthode `Program.cs` `Main()` ci-dessous, en définissant la clé d’instrumentation :

```csharp
using Microsoft.ApplicationInsights.Extensibility;

namespace WindowsFormsApp2
{
    static class Program
    {
        static void Main()
        {
            TelemetryConfiguration.Active.InstrumentationKey = "{Instrumentation-key-here}";
            var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
            builder.Use((next) => new CustomTelemetryProcessor(next));
            builder.Build();
            //...
        }
    }
}

```

> [!NOTE]
> Bien qu’il soit possible techniquement d’utiliser un processeur de télémétrie comme décrit précédemment, même avec un [niveau de prix Entreprise existant (par nœud)](./pricing.md#legacy-enterprise-per-node-pricing-tier), vous vous exposez à un risque de surfacturation du fait de l’incapacité à distinguer correctement les nœuds soumis à un tarif par nœud.

## <a name="next-steps"></a>Étapes suivantes
* [Création d’un tableau de bord](./overview-dashboard.md)
* [Recherche de diagnostic](./diagnostic-search.md)
* [Exploration des mesures](../platform/metrics-charts.md)
* [Écriture de requêtes Analytics](../log-query/log-query-overview.md)

