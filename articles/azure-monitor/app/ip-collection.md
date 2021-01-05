---
title: Collection d’adresses IP dans Azure Application Insights | Microsoft Docs
description: Compréhension de la gestion des adresses IP et de la géolocalisation avec Azure Application Insights
ms.topic: conceptual
ms.date: 09/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: 6131105ef78a8559b0fb95043a87e562e887ebfd
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91333305"
---
# <a name="geolocation-and-ip-address-handling"></a>Gestion de la géolocalisation et des adresses IP

Cet article explique le fonctionnement de la recherche de géolocalisation et de la gestion des adresses IP dans Application Insights, ainsi que la façon de modifier le comportement par défaut.

## <a name="default-behavior"></a>Comportement par défaut

Par défaut, les adresses IP sont collectées temporairement, mais elles ne sont pas stockées dans Application Insights. Le processus de base est le suivant :

Lorsque la télémétrie est envoyée à Azure, l’adresse IP est utilisée pour effectuer une recherche de géolocalisation à l’aide de [GeoLite2 à partir de MaxMind](https://dev.maxmind.com/geoip/geoip2/geolite2/). Les résultats de cette recherche sont utilisés pour remplir les champs `client_City`, `client_StateOrProvince` et `client_CountryOrRegion`. L’adresse est alors ignorée et `0.0.0.0` est écrit dans le champ `client_IP`.

* Télémétrie de navigateur : Nous collectons temporairement l’adresse IP de l’expéditeur. L’adresse IP est calculée par le point de terminaison d’ingestion.
* Télémétrie de serveur : Le module de télémétrie d’Application Insights collecte temporairement l’adresse IP du client. L’adresse IP n’est pas collectée localement lorsque l’en-tête `X-Forwarded-For` est défini.

Ce comportement est conçu afin d’éviter la collecte inutile de données personnelles. Dans la mesure du possible, nous vous recommandons d’éviter la collecte des données personnelles. 

## <a name="overriding-default-behavior"></a>Remplacement du comportement par défaut

Bien que le comportement par défaut consiste à ne pas collecter d’adresses IP, nous offrons toujours la possibilité de remplacer ce comportement. Toutefois, nous vous recommandons de vérifier que la collecte n’enfreint pas les exigences en matière de conformité ou les réglementations locales. 

Pour en savoir plus sur la gestion des données personnelles dans Application Insights, consultez les [conseils relatifs aux données personnelles](../platform/personal-data-mgmt.md).

## <a name="storing-ip-address-data"></a>Stockage de données d’adresses IP

Pour activer la collecte et le stockage d’adresses IP, la propriété `DisableIpMasking` du composant Application Insights doit être définie sur `true`. Cette propriété peut être définie par l’intermédiaire de modèles Resource Manager ou en appelant l’API REST. 

### <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager

```json
{
       "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/microsoft.insights/components/<resource-name>",
       "name": "<resource-name>",
       "type": "microsoft.insights/components",
       "location": "westcentralus",
       "tags": {
              
       },
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "Flow_Type": "Redfield",
              "Request_Source": "IbizaAIExtension",
              // ...
              "DisableIpMasking": true
       }
}
```

### <a name="portal"></a>Portail 

Si vous devez uniquement modifier le comportement d’une seule ressource Application Insights, utilisez le portail Azure. 

1. Accédez à votre ressource Application Insights > **Automation** > **Exporter le modèle** 

2. Sélectionnez **Déployer**

    ![Bouton avec le mot « Déployer » encadré en rouge](media/ip-collection/deploy.png)

3. Sélectionnez **Modifier un modèle**.

    ![Bouton avec le mot « Modifier » encadré en rouge](media/ip-collection/edit-template.png)

4. Apportez les modifications suivantes au fichier JSON de votre ressource, puis sélectionnez **Enregistrer** :

    ![Screenshot adds a comma after "IbizaAIExtension" and add a new line below with "DisableIpMasking": true](media/ip-collection/save.png)

    > [!WARNING]
    > Si vous voyez une erreur indiquant : **_Le groupe de ressources se trouve dans un emplacement qui n’est pas pris en charge par une ou plusieurs ressources dans le modèle. Choisissez un autre groupe de ressources._** Sélectionnez temporairement un autre groupe de ressources dans la liste déroulante, puis sélectionnez à nouveau votre groupe de ressources d’origine pour résoudre l’erreur.

5. Sélectionnez **J’accepte** > **Achat**. 

    ![Case à cocher avec les mots « J’accepte les termes et conditions mentionnés ci-dessus », encadrés en rouge au-dessus d’un bouton avec le mot « Acheter » encadré en rouge.](media/ip-collection/purchase.png)

    Dans ce cas, rien de nouveau n’est réellement acheté. Nous mettons uniquement à jour la configuration de la ressource Application Insights existante.

6. Une fois le déploiement terminé, les nouvelles données de télémétrie sont enregistrées.

    Si vous sélectionnez et modifiez à nouveau le modèle, vous verrez uniquement le modèle par défaut sans la propriété nouvellement ajoutée. Si vous ne voyez pas les données d’adresse IP et que vous souhaitez confirmer que `"DisableIpMasking": true` est défini, exécutez la commande PowerShell suivante : 
    
    ```powershell
    # Replace `Fabrikam-dev` with the appropriate resource and resource group name.
    # If you aren't using the cloud shell you will need to connect to your Azure account
    # Connect-AzAccount 
    $AppInsights = Get-AzResource -Name 'Fabrikam-dev' -ResourceType 'microsoft.insights/components' -ResourceGroupName 'Fabrikam-dev'
    $AppInsights.Properties
    ```
    
    Une liste de propriétés est retournée en tant que résultat. L’une des propriétés doit être lue `DisableIpMasking: true`. Si vous exécutez PowerShell avant de déployer la nouvelle propriété avec Azure Resource Manager, la propriété n’existera pas.

### <a name="rest-api"></a>API REST

La charge utile de l’[API REST](/rest/api/azure/) pour apporter les mêmes modifications est la suivante :

```
PATCH https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/microsoft.insights/components/<resource-name>?api-version=2018-05-01-preview HTTP/1.1
Host: management.azure.com
Authorization: AUTH_TOKEN
Content-Type: application/json
Content-Length: 54

{
       "location": "<resource location>",
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "DisableIpMasking": true
       }
}
```

## <a name="telemetry-initializer"></a>Initialiseur de télémétrie

Si vous avez besoin d’une alternative plus flexible que `DisableIpMasking`, vous pouvez utiliser un [initialiseur de télémétrie](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) pour copier tout ou partie de l’adresse IP dans un champ personnalisé. 

# <a name="net"></a>[.NET](#tab/net)

### <a name="aspnet--aspnet-core"></a>ASP.NET/ASP.NET Core

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;

namespace MyWebApp
{
    public class CloneIPAddress : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            ISupportProperties propTelemetry = telemetry as ISupportProperties;

            if (propTelemetry !=null && !propTelemetry.Properties.ContainsKey("client-ip"))
            {
                string clientIPValue = telemetry.Context.Location.Ip;
                propTelemetry.Properties.Add("client-ip", clientIPValue);
            }
        }
    } 
}
```

> [!NOTE]
> Si vous ne parvenez pas à accéder à `ISupportProperties`, assurez-vous d’exécuter la dernière version stable du kit de développement logiciel (SDK) Application Insights. Les `ISupportProperties` sont prévues pour des valeurs de cardinalité hautes, alors que les `GlobalProperties` sont plus adaptées à des valeurs de cardinalité basses telles que le nom de région, le nom de l’environnement, etc. 

### <a name="enable-telemetry-initializer-for-aspnet"></a>Activer l’initialiseur de télémétrie pour. ASP.NET

```csharp
using Microsoft.ApplicationInsights.Extensibility;


namespace MyWebApp
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
              //Enable your telemetry initializer:
              TelemetryConfiguration.Active.TelemetryInitializers.Add(new CloneIPAddress());
        }
    }
}

```

### <a name="enable-telemetry-initializer-for-aspnet-core"></a>Activez l’initialiseur de télémétrie pour. ASP.NET Core

Vous pouvez créer votre initialiseur de télémétrie de la même façon pour ASP.NET Core que pour ASP.NET, mais pour activer l’initialiseur, utilisez l’exemple suivant comme référence :

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, CloneIPAddress>();
}
```
# <a name="nodejs"></a>[Node.JS](#tab/nodejs)

### <a name="nodejs"></a>Node.js

```javascript
appInsights.defaultClient.addTelemetryProcessor((envelope) => {
    const baseData = envelope.data.baseData;
    if (appInsights.Contracts.domainSupportsProperties(baseData)) {
        const ipAddress = envelope.tags[appInsights.defaultClient.context.keys.locationIp];
        if (ipAddress) {
            baseData.properties["client-ip"] = ipAddress;
        }
    }
});
```
# <a name="client-side-javascript"></a>[JavaScript côté client](#tab/javascript)

### <a name="client-side-javascript"></a>JavaScript côté client

Contrairement aux Kits de développement logiciel (SDK) côté serveur, le SDK JavaScript côté client ne calcule pas l’adresse IP. Par défaut, le calcul d’adresse IP pour la télémétrie côté client se produit au point de terminaison d’ingestion dans Azure. 

Si vous souhaitez calculer l’adresse IP directement côté client, vous devez ajouter votre propre logique personnalisée et utiliser le résultat pour définir la balise `ai.location.ip`. Lorsque `ai.location.ip` est définie, le calcul d’adresse IP n’est pas effectué par le point de terminaison d’ingestion et l’adresse IP fournie est utilisée pour la recherche de géolocalisation. Dans ce scénario, l’adresse IP sera toujours mise à zéro par défaut. 

Pour conserver l’intégralité de l’adresse IP calculée à partir de votre logique personnalisée, vous pouvez utiliser un initialiseur de télémétrie qui copie les données d’adresse IP que vous avez fournies dans `ai.location.ip` vers un champ personnalisé distinct. Mais là encore, contrairement aux Kits de développement logiciel (SDK) côté serveur, sans compter sur les bibliothèques tierces ou sur votre propre logique de collection personnalisée, le Kit de développement logiciel (SDK) côté client ne calculera pas l’adresse IP pour vous.    


```javascript
appInsights.addTelemetryInitializer((item) => {
    const ipAddress = item.tags && item.tags["ai.location.ip"];
    if (ipAddress) {
        item.baseData.properties = {
            ...item.baseData.properties,
            "client-ip": ipAddress
        };
    }
});

```  

Si vos données côté client traversent un proxy avant d’être transférées vers le point de terminaison d’ingestion, le calcul d’adresse IP peut afficher l’adresse IP du proxy et non celle du client. 

---

### <a name="view-the-results-of-your-telemetry-initializer"></a>Afficher les résultats de votre initialiseur de télémétrie

Si vous envoyez un nouveau trafic vers votre site et patientez quelques minutes, vous pouvez ensuite exécuter une requête pour confirmer que la collecte fonctionne :

```kusto
requests
| where timestamp > ago(1h) 
| project appName, operation_Name, url, resultCode, client_IP, customDimensions.["client-ip"]
```

Les adresses IP nouvellement collectées apparaîtront dans la colonne `customDimensions_client-ip`. Les quatre octets de la colonne `client-ip` par défaut auront toujours la valeur zéro. 

Si vous testez localhost et que la valeur de `customDimensions_client-ip` est `::1`, cette valeur est le comportement attendu. `::1` représente l’adresse de bouclage dans IPv6. Elle est équivalente à `127.0.01` dans IPv4.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur la [collecte de données personnelles](../platform/personal-data-mgmt.md) dans Application Insights.

* En savoir plus sur le fonctionnement de la [collecte d’adresses IP](https://apmtips.com/posts/2016-07-05-client-ip-address/) dans Application Insights. (Cet article est un billet de blog externe plus ancien écrit par l’un de nos ingénieurs. Il a été écrit avant le comportement par défaut actuel où l’adresse IP est enregistrée en tant que `0.0.0.0`, mais il examine plus en détail les mécanismes de `ClientIpHeaderTelemetryInitializer` intégré.)
