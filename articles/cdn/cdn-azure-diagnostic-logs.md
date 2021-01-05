---
title: Journaux de diagnostic
titleSuffix: Azure Content Delivery Network
description: Découvrez comment utiliser les journaux de diagnostic Azure pour enregistrer des analyses de base, ce qui vous permet d’exporter des métriques d’utilisation à partir de votre point de terminaison Azure Content Delivery Network.
services: cdn
author: asudbring
manager: KumudD
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/15/2020
ms.author: allensu
ms.openlocfilehash: fdb609a243656e2c75159cd2d4e70e2f965ae896
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352114"
---
# <a name="diagnostic-logs---azure-content-delivery-network"></a>Journaux de diagnostic – Azure Content Delivery Network

Les journaux de diagnostic Azure permettent d’afficher l’analytique principale et de l’enregistrer dans une ou plusieurs destinations, dont les suivantes :

* Compte de Stockage Azure
* Espace de travail Log Analytics
* Azure Event Hubs

Cette fonctionnalité est disponible sur les points de terminaison de CDN à tous les niveaux tarifaires. 

Les journaux de diagnostic vous permettent d’exporter des métriques d’utilisation de base de votre point de terminaison CDN vers différents types de sources afin de pouvoir les utiliser de façon personnalisée. Voici les types d’exportation de données que vous pouvez effectuer :

* Exporter des données vers un stockage d’objets blob, exporter au format CSV et générer des graphiques dans Excel.
* Exporter des données vers des hubs d’événements et mettre les données en corrélation avec d’autres services Azure.
* Exporter des données vers des journaux Azure Monitor et afficher les données dans votre propre espace de travail Log Analytics

Un profil Azure CDN est nécessaire pour les étapes suivantes. Consultez [Créer un point de terminaison et un profil Azure CDN](cdn-create-new-endpoint.md) avant de continuer.

## <a name="enable-logging-with-the-azure-portal"></a>Activation de la journalisation avec le portail Azure

En suivant ces étapes, vous activez la journalisation pour votre point de terminaison Azure CDN :

1. Connectez-vous au [portail Azure](https://portal.azure.com). 

2. Sur le portail Azure, accédez à **Toutes les ressources** -> **votre-profil-cdn**

2. Sélectionnez le point de terminaison CDN pour lequel vous souhaitez activer les journaux de diagnostic :

    :::image type="content" source="./media/cdn-diagnostics-log/02_browse-to-diagnostics-logs.png" alt-text="Sélectionnez le point de terminaison CDN." border="true":::

3. Dans la section **Supervision**, sélectionnez **Journaux de diagnostic** :

    :::image type="content" source="./media/cdn-diagnostics-log/03_diagnostics-logs-options.png" alt-text="Sélectionner les journaux de diagnostics." border="true":::

### <a name="enable-logging-with-azure-storage"></a>Activation de la journalisation avec Stockage Azure

Pour utiliser un compte de stockage afin de stocker les journaux d’activité, effectuez les étapes suivantes :

 >[!NOTE] 
 >Un compte de stockage est nécessaire pour effectuer ces étapes. Consultez : **[Créer un compte Stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal&toc=%2fazure%2fstorage%2fblobs%2ftoc.json)** pour plus d’informations.
    
1. Pour **Nom des paramètres de diagnostic**, entrez un nom pour les paramètres du journal de diagnostic.
 
2. Sélectionnez **Archiver dans un compte de stockage**, puis **CoreAnalytics**. 

3. Dans le champ **Rétention (jours)** , choisissez le nombre de jours de rétention. Si la valeur zéro est appliquée à la rétention, les journaux d’activité sont stockés pour une durée indéfinie. 

4. Sélectionnez l’abonnement et le compte de stockage pour les journaux.

    :::image type="content" source="./media/cdn-diagnostics-log/04_diagnostics-logs-storage.png" alt-text="Journaux de diagnostic – Stockage." border="true":::

3. Sélectionnez **Enregistrer**.

### <a name="send-to-log-analytics"></a>Envoyer à Log Analytics

Pour utiliser Log Analytics pour les journaux, suivez ces étapes :

>[!NOTE] 
>Un espace de travail Log Analytics est nécessaire pour effectuer ces étapes. Consultez : **[Créer un espace de travail Log Analytics sur le portail Azure](../azure-monitor/learn/quick-create-workspace.md)** pour plus d’informations.
    
1. Pour **Nom des paramètres de diagnostic**, entrez un nom pour les paramètres du journal de diagnostic.

2. Sélectionnez **Envoyer à Log Analytics**, puis **CoreAnalytics**. 

3. Sélectionnez l’abonnement et l’espace de travail Log Analytics pour les journaux.

   :::image type="content" source="./media/cdn-diagnostics-log/05-la-workspace.png" alt-text="Journaux de diagnostic – Log Analytics." border="true":::

4. Sélectionnez **Enregistrer**.

### <a name="stream-to-an-event-hub"></a>Diffuser vers un hub d’événements

Pour utiliser un hub d’événements pour les journaux, suivez ces étapes :

>[!NOTE] 
>Un hub d’événements est nécessaire pour effectuer ces étapes. Consultez : **[Démarrage rapide : Créer un hub d’événements avec le portail Azure](../event-hubs/event-hubs-create.md)** pour plus d’informations.
    
1. Pour **Nom des paramètres de diagnostic**, entrez un nom pour les paramètres du journal de diagnostic.

2. Sélectionnez **Diffuser vers Event Hub**, puis **CoreAnalytics**. 

3. Sélectionnez l’abonnement et l’espace de noms Event Hub pour les journaux.

   :::image type="content" source="./media/cdn-diagnostics-log/06-eventhub-namespace.png" alt-text="Journaux de diagnostic – Hub d’événements." border="true":::

4. Sélectionnez **Enregistrer**.


## <a name="enable-logging-with-powershell"></a>Activer la journalisation avec PowerShell

L’exemple suivant montre comment activer les journaux de diagnostic via les applets de commande Azure PowerShell.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="enable-diagnostic-logs-in-a-storage-account"></a>Activer les journaux de diagnostic dans un compte de stockage

1. Connectez-vous à Azure PowerShell :

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```

2. Pour activer les journaux de diagnostic dans un compte de stockage, entrez ces commandes. Remplacez les variables par vos valeurs :

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $storageacct = <your-storage-account-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $storage = Get-AzStorageAccount -ResourceGroupName $rsg -Name $storageacct

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -StorageAccountId $storage.id -Enabled $true -Categories CoreAnalytics
    ```

### <a name="enable-diagnostics-logs-for-log-analytics-workspace"></a>Activer les journaux de diagnostic pour l’espace de travail Log Analytics

1. Connectez-vous à Azure PowerShell :

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Pour activer les journaux de diagnostic pour un espace de travail Log Analytics, entrez ces commandes. Remplacez les variables par vos valeurs :

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $workspacename = <your-log-analytics-workspace-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $rsg -Name $workspacename

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -WorkspaceId $workspace.ResourceId -Enabled $true -Categories CoreAnalytics
    ```
### <a name="enable-diagnostics-logs-for-event-hub-namespace"></a>Activer les journaux de diagnostic pour l’espace de noms Event Hub

1. Connectez-vous à Azure PowerShell :

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Pour activer les journaux de diagnostic pour un espace de travail Log Analytics, entrez ces commandes. Remplacez les variables par vos valeurs :

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $eventhubname = <your-event-hub-namespace-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -EventHubName $eventhubname -Enabled $true -Categories CoreAnalytics
    ```

## <a name="consuming-diagnostics-logs-from-azure-storage"></a>Utilisation des journaux de diagnostic à partir de Stockage Azure
Cette section décrit le schéma de l’analytique principale CDN et l’organisation dans un compte de stockage Azure, et fournit un exemple de code pour télécharger les journaux dans un fichier CSV.

### <a name="using-microsoft-azure-storage-explorer"></a>Utilisation de l’explorateur de stockage Microsoft Azure
Pour télécharger l’outil, consultez [Explorateur de stockage Azure](https://storageexplorer.com/). Après le téléchargement et l’installation du logiciel, configurez-le pour utiliser le même compte de stockage Azure que celui qui a été configuré en tant que destination pour les journaux de diagnostic CDN.

1.  Ouvrez **l’explorateur de stockage Microsoft Azure**
2.  Localiser le compte de stockage
3.  Développez le nœud **Conteneurs d’objets Blob** sous ce compte de stockage.
4.  Sélectionnez le conteneur nommé *journaux-insights-coreanalytics*.
5.  Les résultats s’affichent dans le volet droit, en commençant par le premier niveau, soir *resourceId=* . Continuez à sélectionner chaque niveau jusqu’à atteindre le fichier *PT1H.json*. Pour obtenir une explication sur le chemin, consultez [Format du chemin des objets blob](cdn-azure-diagnostic-logs.md#blob-path-format).
6.  Chaque fichier d’objet blob *PT1H.json* représente les journaux d’activité d’analyse pendant une heure pour un point de terminaison CDN spécifique ou son domaine personnalisé.
7.  Le schéma du contenu de ce fichier JSON est décrit dans la section Schéma des journaux d’activité Core Analytics.


#### <a name="blob-path-format"></a>Format du chemin des objets blob

Les journaux d’activité Core Analytics sont générés toutes les heures et les données sont collectées et stockées dans un objet blob Azure unique sous forme de charge utile JSON. L’outil Explorateur de stockage interprète '/' comme un séparateur de répertoire et affiche la hiérarchie. Le chemin de l’objet blob Azure apparaît comme s’il existait une structure hiérarchique et représente le nom de l’objet blob. Ce nom d’objet blob suit la convention d’affectation de noms suivante :    

```resourceId=/SUBSCRIPTIONS/{Subscription Id}/RESOURCEGROUPS/{Resource Group Name}/PROVIDERS/MICROSOFT.CDN/PROFILES/{Profile Name}/ENDPOINTS/{Endpoint Name}/ y={Year}/m={Month}/d={Day}/h={Hour}/m={Minutes}/PT1H.json```

**Description des champs :**

|Valeur|Description|
|-------|---------|
|Identifiant d’abonnement    |ID de l’abonnement Azure au format Guid.|
|Nom du groupe de ressources |Nom du groupe de ressources auquel les ressources CDN appartiennent.|
|Nom de profil |Nom du profil CDN|
|Nom du point de terminaison |Nom du point de terminaison CDN|
|Year|  Représentation à quatre chiffres de l’année, par exemple, 2017|
|Month| Représentation à deux chiffres du mois de l’année. 01 = Janvier ... 12 = Décembre|
|jour|   Représentation à deux chiffres du jour du mois|
|PT1H.json| Fichier JSON où sont effectivement stockées les données d’analyse|

### <a name="exporting-the-core-analytics-data-to-a-csv-file"></a>Exportation des données de Core Analytics vers un fichier CSV

Pour accéder à l’analytique principale, un exemple de code pour un outil est fourni. Cet outil permet de télécharger les fichiers JSON dans un format de fichier plat avec séparation par virgules, qui peut être utilisé pour créer des graphiques ou autres agrégations.

Voici comment vous pouvez utiliser l’outil :

1.  Consultez le lien GitHub : [https://github.com/Azure-Samples/azure-cdn-samples/tree/master/CoreAnalytics-ExportToCsv](https://github.com/Azure-Samples/azure-cdn-samples/tree/master/CoreAnalytics-ExportToCsv)
2.  Téléchargez le code.
3.  Suivez les instructions pour la compilation et la configuration.
4.  Exécutez l’outil.
5.  Le fichier CSV résultant présente les données d’analyse dans une hiérarchie plate simple.

## <a name="log-data-delays"></a>Retards des données de journal

Le tableau suivant présente les retards des données de journal pour **CDN Azure Standard fourni par Microsoft**, **CDN Azure Standard fourni par Akamai** et **CDN Azure Standard/Premium fourni par Verizon**.

Retards des données de journal Microsoft | Retards des données de journal Verizon | Retards des données de journal Akamai
--- | --- | ---
Retard de 1 heure. | Retard de 1 heure et possibilité de nécessiter jusqu’à 2 heures pour commencer à apparaître après l’achèvement de la propagation du point de terminaison. | Retard de 24 heures. Si la création remonte à plus de 24 heures, il faut attendre jusqu’à 2 heures avant leur apparition. Si elle a été effectuée récemment, il peut falloir jusqu’à 25 heures pour que les journaux d’activité commencent à apparaître.

## <a name="diagnostic-log-types-for-cdn-core-analytics"></a>Types de journaux de diagnostic pour l’analytique principale CDN

Microsoft propose uniquement les journaux d’activité de l’analytique principale, qui contiennent les métriques affichant les statistiques de réponse HTTP et de sortie, comme dans les points de présence/périmètres CDN.

### <a name="core-analytics-metrics-details"></a>Détails des métriques de Core Analytics
Le tableau suivant présente la liste des métriques disponibles dans les journaux de l’analytique principale pour :

* **Azure CDN Standard par Microsoft**
* **Azure CDN Standard par Akamai**
* **Azure CDN Standard/Premium par Verizon**

Toutes les métriques ne sont pas disponibles auprès tous les fournisseurs, même si ces différences sont minimes. Ce tableau indique également si une métrique donnée est disponible à partir d’un fournisseur particulier. Les métriques sont disponibles uniquement pour les points de terminaison CDN recevant du trafic.


|Métrique                     | Description | Microsoft | Verizon | Akamai |
|---------------------------|-------------|-----------|---------|--------|
| RequestCountTotal         | Nombre total d’occurrences de requêtes pendant cette période. | Oui | Oui |Oui |
| RequestCountHttpStatus2xx | Nombre total des requêtes ayant abouti à un code HTTP 2xx (par exemple, 200, 202). | Oui | Oui |Oui |
| RequestCountHttpStatus3xx | Nombre total des requêtes ayant abouti à un code HTTP 3xx (par exemple, 300, 302). | Oui | Oui |Oui |
| RequestCountHttpStatus4xx | Nombre total des requêtes ayant abouti à un code HTTP 4xx (par exemple, 400, 404). | Oui | Oui |Oui |
| RequestCountHttpStatus5xx | Nombre total des requêtes ayant abouti à un code HTTP 5xx (par exemple, 500, 504). | Oui | Oui |Oui |
| RequestCountHttpStatusOthers | Nombre total des autres codes HTTP (en dehors de 2xx-5xx). | Oui | Oui |Oui |
| RequestCountHttpStatus200 | Nombre total des requêtes ayant abouti à un code HTTP 200. | Oui | Non  |Oui |
| RequestCountHttpStatus206 | Nombre total des requêtes ayant abouti à un code HTTP 206. | Oui | Non  |Oui |
| RequestCountHttpStatus302 | Nombre total des requêtes ayant abouti à un code HTTP 302. | Oui | Non  |Oui |
| RequestCountHttpStatus304 | Nombre total des requêtes ayant abouti à un code HTTP 304. | Oui | Non  |Oui |
| RequestCountHttpStatus404 | Nombre total des requêtes ayant abouti à un code HTTP 404. | Oui | Non  |Oui |
| RequestCountCacheHit | Nombre de toutes les requêtes qui ont abouti à un accès au cache. La ressource a été traitée directement du point de présence vers le client. | Oui | Oui | Non  |
| RequestCountCacheMiss | Nombre de toutes les requêtes qui ont abouti à un échec de cache. Un échec de cache signifie que la ressource est introuvable sur le point de présence le plus proche du client et qu’elle a été récupérée à l’origine. | Oui | Oui | Non |
| RequestCountCacheNoCache | Nombre de requêtes vers une ressource qui sont empêchées d’être mises en cache en raison d’une configuration utilisateur sur le périmètre. | Oui | Oui | Non |
| RequestCountCacheUncacheable | Nombre de requêtes vers des ressources qui sont empêchées d’être mises en cache par les en-têtes Cache-Control et/ou Expires des ressources. Ce nombre indique qu’elles ne doivent pas être mises en cache sur un point de présence ou par le client HTTP. | Oui | Oui | Non |
| RequestCountCacheOthers | Nombre de toutes les requêtes avec un état du cache non traité ci-dessus. | Non | Oui | Non  |
| EgressTotal | Transfert de données sortantes en Go | Oui |Oui |Oui |
| EgressHttpStatus2xx | Transfert de données sortantes* pour les réponses avec des codes d’état HTTP 2xx en Go. | Oui | Oui | Non  |
| EgressHttpStatus3xx | Transfert de données sortantes pour les réponses avec des codes d’état HTTP 3xx en Go. | Oui | Oui | Non  |
| EgressHttpStatus4xx | Transfert de données sortantes pour les réponses avec des codes d’état HTTP 4xx en Go. | Oui | Oui | Non  |
| EgressHttpStatus5xx | Transfert de données sortantes pour les réponses avec des codes d’état HTTP 5xx en Go. | Oui | Oui | Non |
| EgressHttpStatusOthers | Transfert de données sortantes pour les réponses avec d’autres codes d’état HTTP en Go. | Oui | Oui | Non  |
| EgressCacheHit | Transfert de données sortantes pour les réponses qui ont été livrées directement à partir du cache CDN sur les points de présence/périmètres CDN. | Oui | Oui | Non |
| EgressCacheMiss. | Transfert de données sortantes pour les réponses qui n’ont pas été trouvées sur le serveur POP le plus proche et qui ont été récupérées à partir du serveur d’origine. | Oui | Oui | Non |
| EgressCacheNoCache | Transferts de données sortants pour les ressources qui sont empêchés d’être mises en cache en raison d’une configuration l’utilisateur sur le périmètre. | Oui | Oui | Non |
| EgressCacheUncacheable | Transfert de données sortantes pour les ressources qui sont empêchées d’être mises en cache par les en-têtes Cache-Control et/ou Expires des ressources. Cela indique qu’elles ne doivent pas être mises en cache sur un point de présence ou par le client HTTP. | Oui | Oui | Non |
| EgressCacheOthers | Transfère les données sortantes pour d’autres scénarios de cache. | Non | Oui | Non |

\* Le transfert de données sortantes fait référence au trafic produit des serveurs POP CDN vers le client.


### <a name="schema-of-the-core-analytics-logs"></a>Schéma des journaux d’activité Core Analytics 

Tous les journaux d’activité sont stockés au format JSON, et chaque entrée comprend des champs de chaîne conformément au schéma suivant :

```json
    "records": [
        {
            "time": "2017-04-27T01:00:00",
            "resourceId": "<ARM Resource Id of the CDN Endpoint>",
            "operationName": "Microsoft.Cdn/profiles/endpoints/contentDelivery",
            "category": "CoreAnalytics",
            "properties": {
                "DomainName": "<Name of the domain for which the statistics is reported>",
                "RequestCountTotal": integer value,
                "RequestCountHttpStatus2xx": integer value,
                "RequestCountHttpStatus3xx": integer value,
                "RequestCountHttpStatus4xx": integer value,
                "RequestCountHttpStatus5xx": integer value,
                "RequestCountHttpStatusOthers": integer value,
                "RequestCountHttpStatus200": integer value,
                "RequestCountHttpStatus206": integer value,
                "RequestCountHttpStatus302": integer value,
                "RequestCountHttpStatus304": integer value,
                "RequestCountHttpStatus404": integer value,
                "RequestCountCacheHit": integer value,
                "RequestCountCacheMiss": integer value,
                "RequestCountCacheNoCache": integer value,
                "RequestCountCacheUncacheable": integer value,
                "RequestCountCacheOthers": integer value,
                "EgressTotal": double value,
                "EgressHttpStatus2xx": double value,
                "EgressHttpStatus3xx": double value,
                "EgressHttpStatus4xx": double value,
                "EgressHttpStatus5xx": double value,
                "EgressHttpStatusOthers": double value,
                "EgressCacheHit": double value,
                "EgressCacheMiss": double value,
                "EgressCacheNoCache": double value,
                "EgressCacheUncacheable": double value,
                "EgressCacheOthers": double value,
            }
        }

    ]
}
```

Où *time* représente l’heure de début des limites d’heure pour lesquelles les statistiques sont rapportées. Une métrique non prise en charge par un fournisseur CDN donne une valeur null et non un entier ou un double. Cette valeur null indique l’absence de métrique, et diffère de la valeur 0. Un ensemble de ces métriques par domaine est configuré sur le point de terminaison.

Exemple de propriétés :

```json
{
     "DomainName": "manlingakamaitest2.azureedge.net",
     "RequestCountTotal": 480,
     "RequestCountHttpStatus2xx": 480,
     "RequestCountHttpStatus3xx": 0,
     "RequestCountHttpStatus4xx": 0,
     "RequestCountHttpStatus5xx": 0,
     "RequestCountHttpStatusOthers": 0,
     "RequestCountHttpStatus200": 480,
     "RequestCountHttpStatus206": 0,
     "RequestCountHttpStatus302": 0,
     "RequestCountHttpStatus304": 0,
     "RequestCountHttpStatus404": 0,
     "RequestCountCacheHit": null,
     "RequestCountCacheMiss": null,
     "RequestCountCacheNoCache": null,
     "RequestCountCacheUncacheable": null,
     "RequestCountCacheOthers": null,
     "EgressTotal": 0.09,
     "EgressHttpStatus2xx": null,
     "EgressHttpStatus3xx": null,
     "EgressHttpStatus4xx": null,
     "EgressHttpStatus5xx": null,
     "EgressHttpStatusOthers": null,
     "EgressCacheHit": null,
     "EgressCacheMiss": null,
     "EgressCacheNoCache": null,
     "EgressCacheUncacheable": null,
     "EgressCacheOthers": null
}

```

## <a name="additional-resources"></a>Ressources supplémentaires

* [Journaux de diagnostic Azure](../azure-monitor/platform/platform-logs-overview.md)
* [Core Analytics via le portail supplémentaire Azure CDN](./cdn-analyze-usage-patterns.md)
* [Journaux d’activité Azure Monitor](../azure-monitor/log-query/log-query-overview.md)
* [API REST Azure Log Analytics](/rest/api/loganalytics)