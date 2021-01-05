---
title: Activer et interroger les journaux de diagnostic
titleSuffix: Azure Digital Twins
description: Découvrez comment activer la journalisation avec les paramètres de diagnostic et comment interroger les journaux pour un affichage immédiat.
author: baanders
ms.author: baanders
ms.date: 11/9/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: d988617fcaf7479c7bb3356e6ef6f87824ed23a7
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94616652"
---
# <a name="troubleshooting-azure-digital-twins-diagnostics-logging"></a>Dépannage d’Azure Digital Twins : Journalisation des diagnostics

Azure Digital Twins peut collecter les journaux de votre instance de service pour superviser ses performances, son accès et d’autres données. Vous pouvez utiliser ces journaux pour obtenir un aperçu de ce qui se passe dans votre instance Azure Digital Twins et effectuer une analyse de cause racine sur des problèmes sans avoir besoin de contacter le support Azure.

Cet article explique comment [**configurer les paramètres de diagnostic**](#turn-on-diagnostic-settings) dans le [portail Azure](https://portal.azure.com) pour commencer à collecter des journaux à partir de votre instance Azure Digital Twins. Vous pouvez également spécifier l’emplacement de stockage des journaux (comme Log Analytics ou un compte de stockage de votre choix).

Cet article contient aussi les listes de toutes les [catégories de journaux](#log-categories) et tous les [schémas de journalisation](#log-schemas) qu’Azure Digital Twins collecte.

Après avoir configuré les journaux, vous pouvez aussi les [**interroger**](#view-and-query-logs) pour collecter rapidement des insights personnalisés.

## <a name="turn-on-diagnostic-settings"></a>Activer les paramètres de diagnostic 

Activez les paramètres de diagnostic pour démarrer la collecte des journaux sur votre instance Azure Digital Twins. Vous pouvez aussi choisir la destination à laquelle les journaux exportés doivent être stockés. Voici comment activer les paramètres de diagnostic pour votre instance Azure Digital Twins.

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre instance Azure Digital Twins. Vous pouvez la trouver en tapant son nom dans la barre de recherche du portail. 

2. Sélectionnez **Paramètres de diagnostic** dans le menu, puis **Ajouter un paramètre de diagnostic**.

    :::image type="content" source="media/troubleshoot-diagnostics/diagnostic-settings.png" alt-text="Capture d’écran montrant la page des paramètres de diagnostic et le bouton à ajouter" lightbox="media/troubleshoot-diagnostics/diagnostic-settings.png":::

3. Dans la page qui suit, renseignez les valeurs suivantes :
     * **Nom du paramètre de diagnostic** : Nommez les paramètres de diagnostic.
     * **Détails de la catégorie** : choisissez les opérations à surveiller, puis cochez les cases pour activer les diagnostics pour ces opérations. Les opérations sur lesquelles les paramètres de diagnostic peuvent établir des rapports sont :
        - DigitalTwinsOperation
        - EventRoutesOperation
        - ModelsOperation
        - QueryOperation
        - AllMetrics
        
        Pour plus d’informations sur ces catégories et les informations qu’elles contiennent, consultez la section [*Catégories de journaux*](#log-categories) ci-dessous.
     * **Détails de la destination** : Indiquez où vous voulez envoyer les journaux d’activité. Vous pouvez sélectionner n’importe quelle combinaison des trois options suivantes :
        - Envoyer à Log Analytics
        - Archiver dans un compte de stockage
        - Diffuser vers un hub d’événements

        Il se peut que vous soyez invité à fournir des détails supplémentaires s’ils sont nécessaires pour la sélection de votre destination.  
    
4. Enregistrez les nouveaux paramètres. 

    :::image type="content" source="media/troubleshoot-diagnostics/diagnostic-settings-details.png" alt-text="Capture d'écran montrant la page des paramètres de diagnostic où l'utilisateur a renseigné un nom de paramètre de diagnostic et coché quelques cases en rapport avec les détails de la catégorie et de la destination. Le bouton Enregistrer est en surbrillance." lightbox="media/troubleshoot-diagnostics/diagnostic-settings-details.png":::

Les nouveaux paramètres prennent effet au bout de 10 minutes environ. Après cela, les journaux réapparaissent dans la cible configurée sur la page **Paramètres de diagnostic** de votre instance. 

Pour plus d’informations sur les paramètres de diagnostic et leurs options de configuration, vous pouvez consulter [*Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations*](../azure-monitor/platform/diagnostic-settings.md).

## <a name="log-categories"></a>Catégories de journal

Voici plus d’informations sur les catégories de journaux collectées par Azure Digital Twins.

| Catégorie de journal | Description |
| --- | --- |
| ADTModelsOperation | Consigner tous les appels d’API se rapportant à des modèles |
| ADTQueryOperation | Consigner tous les appels d’API se rapportant à des requêtes |
| ADTEventRoutesOperation | Consigner tous les appels d’API se rapportant aux itinéraires d’événements, ainsi que la sortie d’événements provenant d’Azure Digital Twins vers un service de point de terminaison comme Event Grid, Event Hubs et Service Bus |
| ADTDigitalTwinsOperation | Consigner tous les appels d’API se rapportant à Azure Digital Twins |

Chaque catégorie de journal se compose d'opérations d'écriture, de lecture, de suppression et d'action.  Ces opérations sont mappées à des appels d'API REST, comme suit :

| Type d'événement | Opérations de l'API REST |
| --- | --- |
| Write | PUT et PATCH |
| Lire | GET |
| DELETE | Suppression |
| Action | POST |

Voici une liste complète des opérations et des [appels d’API REST Azure Digital Twins](/rest/api/azure-digitaltwins/) correspondants consignés dans chaque catégorie. 

>[!NOTE]
> Chaque catégorie de journal contient plusieurs opérations/appels d’API REST. Dans le tableau ci-dessous, chaque catégorie de journal est mappée à l’ensemble des opérations/appels d’API REST en dessous, jusqu’à ce que la catégorie suivante du journal soit affichée. 

| Catégorie de journal | Opération | Appels d’API REST et autres événements |
| --- | --- | --- |
| ADTModelsOperation | Microsoft.DigitalTwins/models/write | API de mise à jour des modèles de jumeaux numériques |
|  | Microsoft.DigitalTwins/models/read | API de modèles de jumeaux numériques Get by ID et Liste |
|  | Microsoft.DigitalTwins/models/delete | API de suppression des modèles de jumeaux numériques |
|  | Microsoft.DigitalTwins/models/action | API d’ajout des modèles de jumeaux numériques |
| ADTQueryOperation | Microsoft.DigitalTwins/query/action | API de jumeaux numériques de requête |
| ADTEventRoutesOperation | Microsoft.DigitalTwins/eventroutes/write | API d’ajout de routages d’événements |
|  | Microsoft.DigitalTwins/eventroutes/read | API de routages d'événements Get by ID et Liste |
|  | Microsoft.DigitalTwins/eventroutes/delete | API de suppression de routages d’événements |
|  | Microsoft.DigitalTwins/eventroutes/action | Échec lors de la tentative de publication des événements sur un service de point de terminaison (pas un appel d’API) |
| ADTDigitalTwinsOperation | Microsoft.DigitalTwins/digitaltwins/write | Ajouter des jumeaux numériques, Ajouter une relation, Mettre à jour, Mettre à jour un composant |
|  | Microsoft.DigitalTwins/digitaltwins/read | Jumeaux numériques Get by ID, Obtenir un composant, Obtenir une relation par ID, Relations entrantes, Lister les relations |
|  | Microsoft.DigitalTwins/digitaltwins/delete | Supprimer des jumeaux numériques, Supprimer une relation |
|  | Microsoft.DigitalTwins/digitaltwins/action | Envoyer des données de télémétrie de composant de jumeaux numériques, Envoyer des données de télémétrie |

## <a name="log-schemas"></a>Schémas des journaux 

Chaque catégorie de journal dispose d'un schéma qui définit la façon dont les événements de cette catégorie sont signalés. Chaque entrée de journal est stockée sous forme de texte et formatée en tant qu'objet blob JSON. Les champs du journal et des exemples de corps JSON sont fournis pour chaque type de journal ci-dessous. 

`ADTDigitalTwinsOperation`, `ADTModelsOperation` et `ADTQueryOperation` utilisent un schéma de journal d'API cohérent ; `ADTEventRoutesOperation` dispose d'un schéma distinct.

### <a name="api-log-schemas"></a>Schéma des journaux d'API

Ce schéma de journal est cohérent pour `ADTDigitalTwinsOperation`, `ADTModelsOperation` et `ADTQueryOperation`. Il contient des informations relatives aux appels d'API à une instance d'Azure Digital Twins.

Vous trouverez ci-dessous les descriptions des champs et des propriétés des journaux d'API.

| Nom du champ | Type de données | Description |
|-----|------|-------------|
| `Time` | DateTime | Date et heure (UTC) auxquelles l'événement s'est produit |
| `ResourceID` | String | ID Azure Resource Manager de la ressource sur laquelle l'événement s'est produit |
| `OperationName` | String  | Type d'action réalisée pendant l'événement |
| `OperationVersion` | String | Version de l'API utilisée pendant l'événement |
| `Category` | String | Type de ressource émise |
| `ResultType` | String | Résultat de l'événement |
| `ResultSignature` | String | Code d'état HTTP de l'événement |
| `ResultDescription` | String | Détails supplémentaires sur l'événement |
| `DurationMs` | String | Temps nécessaire pour exécuter l'événement, en millisecondes |
| `CallerIpAddress` | String | Adresse IP source masquée de l'événement |
| `CorrelationId` | Guid | Le client a fourni un identificateur unique pour l'événement |
| `Level` | String | Gravité de la journalisation de l'événement |
| `Location` | String | Région où s'est produit l'événement |
| `RequestUri` | Uri | Point de terminaison utilisé pendant l'événement |

Vous trouverez ci-dessous des exemples de corps JSON correspondant à ces types de journaux.

#### <a name="adtdigitaltwinsoperation"></a>ADTDigitalTwinsOperation

```json
{
  "time": "2020-03-14T21:11:14.9918922Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/digitaltwins/write",
  "operationVersion": "2020-10-31",
  "category": "DigitalTwinOperation",
  "resultType": "Success",
  "resultSignature": "200",
  "resultDescription": "",
  "durationMs": "314",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "2f6a8e64-94aa-492a-bc31-16b9f0b16ab3",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/digitaltwins/factory-58d81613-2e54-4faa-a930-d980e6e2a884?api-version=2020-10-31"
}
```

#### <a name="adtmodelsoperation"></a>ADTModelsOperation

```json
{
  "time": "2020-10-29T21:12:24.2337302Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/models/write",
  "operationVersion": "2020-10-31",
  "category": "ModelsOperation",
  "resultType": "Success",
  "resultSignature": "201",
  "resultDescription": "",
  "durationMs": "935",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "9dcb71ea-bb6f-46f2-ab70-78b80db76882",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/Models?api-version=2020-10-31",
}
```

#### <a name="adtqueryoperation"></a>ADTQueryOperation

```json
{
  "time": "2020-12-04T21:11:44.1690031Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/query/action",
  "operationVersion": "2020-10-31",
  "category": "QueryOperation",
  "resultType": "Success",
  "resultSignature": "200",
  "resultDescription": "",
  "durationMs": "255",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "1ee2b6e9-3af4-4873-8c7c-1a698b9ac334",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/query?api-version=2020-10-31",
}
```

### <a name="egress-log-schemas"></a>Schéma des journaux de sortie

Il s'agit du schéma des journaux `ADTEventRoutesOperation`. Ceux-ci contiennent des détails sur les exceptions et les opérations d'API relatives aux points de terminaison de sortie connectés à une instance d'Azure Digital Twins.

|Nom du champ | Type de données | Description |
|-----|------|-------------|
| `Time` | DateTime | Date et heure (UTC) auxquelles l'événement s'est produit |
| `ResourceId` | String | ID Azure Resource Manager de la ressource sur laquelle l'événement s'est produit |
| `OperationName` | String  | Type d'action réalisée pendant l'événement |
| `Category` | String | Type de ressource émise |
| `ResultDescription` | String | Détails supplémentaires sur l'événement |
| `Level` | String | Gravité de la journalisation de l'événement |
| `Location` | String | Région où s'est produit l'événement |
| `EndpointName` | String | Nom du point de terminaison de sortie créé dans Azure Digital Twins |

Vous trouverez ci-dessous des exemples de corps JSON correspondant à ces types de journaux.

#### <a name="adteventroutesoperation"></a>ADTEventRoutesOperation

```json
{
  "time": "2020-11-05T22:18:38.0708705Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/eventroutes/action",
  "category": "EventRoutesOperation",
  "resultDescription": "Unable to send EventGrid message to [my-event-grid.westus-1.eventgrid.azure.net] for event Id [f6f45831-55d0-408b-8366-058e81ca6089].",
  "correlationId": "7f73ab45-14c0-491f-a834-0827dbbf7f8e",
  "level": "3",
  "location": "southcentralus",
  "properties": {
    "endpointName": "endpointEventGridInvalidKey"
  }
}
```

## <a name="view-and-query-logs"></a>Afficher et interroger les journaux

Plus haut dans cet article, vous avez configuré les types de journaux à stocker et spécifié leur emplacement de stockage.

Pour résoudre le problème et générer des insights à partir de ces journaux, vous pouvez générer des **requêtes personnalisées**. Pour commencer, vous pouvez aussi tirer parti de quelques exemples de requêtes fournis par le service, qui répondent aux questions courantes que les clients peuvent se poser sur leur instance.

Voici comment interroger les journaux de votre instance.

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre instance Azure Digital Twins. Vous pouvez la trouver en tapant son nom dans la barre de recherche du portail. 

2. Sélectionnez **Journaux** dans le menu pour ouvrir la page d’interrogation de journal. Cette page s’ouvre dans une fenêtre nommée *Requêtes*.

    :::image type="content" source="media/troubleshoot-diagnostics/logs.png" alt-text="Capture d’écran montrant la page Journaux d’une instance Azure Digital Twins. Elle est recouverte par une fenêtre Requêtes qui montre des requêtes prédéfinies nommées selon les différentes options de journalisation, comme Latence de l’API DigitalTwin et Latence de l’API de modèle." lightbox="media/troubleshoot-diagnostics/logs.png":::

    Il s’agit d’exemples de requêtes prédéfinies écrites pour divers journaux. Vous pouvez sélectionner l’une des requêtes pour la charger dans l’éditeur de requête, puis l’exécuter afin de voir ces journaux pour votre instance.

    Vous pouvez aussi fermer la fenêtre *Requêtes* sans rien exécuter pour accéder directement à la page de l’éditeur de requête, où vous pouvez écrire ou modifier du code de requête personnalisé.

3. Après avoir quitté la fenêtre *Requêtes*, la page principale de l’éditeur de requête s’affiche. Ici, vous pouvez voir et modifier le texte des exemples de requêtes, ou écrire vos propres requêtes ex nihilo.
    :::image type="content" source="media/troubleshoot-diagnostics/logs-query.png" alt-text="Capture d’écran montrant la page Journaux d’une instance Azure Digital Twins. La fenêtre Requêtes a disparu et, à la place, figure la liste des différents journaux, un volet d’édition présentant du code de requête modifiable et un volet présentant l’historique des requêtes." lightbox="media/troubleshoot-diagnostics/logs-query.png":::

    Dans le volet gauche : 
    - L’onglet *Tables* présente les différentes [catégories de journaux](#log-categories) Azure Digital Twins utilisables dans vos requêtes. 
    - L’onglet *Requêtes* contient les exemples de requêtes que vous pouvez charger dans l’éditeur.
    - L’onglet *Filtre* vous permet de personnaliser une vue filtrée des données retournées par la requête.

Pour plus d’informations sur les requêtes de journal et sur la façon de les écrire, vous pouvez consulter [*Vue d’ensemble des requêtes de journal dans Azure Monitor*](../azure-monitor/log-query/log-query-overview.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la configuration des diagnostics, consultez [*Collecter et utiliser des données de journaux à partir de vos ressources Azure*](../azure-monitor/platform/platform-logs-overview.md).
* Pour plus d’informations sur les métriques d’Azure Digital Twins, consultez [*Dépannage : Afficher les métriques avec Azure Monitor*](troubleshoot-metrics.md).
* Pour savoir comment activer les alertes pour vos métriques, consultez [*Résolution des problèmes : Configurer des alertes*](troubleshoot-alerts.md).