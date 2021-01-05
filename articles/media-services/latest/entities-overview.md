---
title: Filtrage, classement et pagination d’entités Media Services
titleSuffix: Azure Media Services
description: Apprenez-en davantage sur le filtrage, le tri et la pagination des entités Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: overview
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 9a8cff3685cdaad011332adf58dc76f74976cd44
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89300185"
---
# <a name="filtering-ordering-and-paging-of-media-services-entities"></a>Filtrage, classement et pagination d’entités Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cette rubrique décrit les options de requête OData et la prise en charge de la pagination disponibles dans la liste des entités Azure Media Services V3.

## <a name="considerations"></a>Considérations

* Les propriétés d’entités de type `Datetime` sont toujours au format UTC.
* Vous devez encoder l’espace blanc dans la chaîne de requête en URL avant d’envoyer votre demande.

## <a name="comparison-operators"></a>Opérateurs de comparaison

Vous pouvez utiliser les opérateurs suivants pour comparer un champ à une valeur constante :

Opérateurs d’égalité :

- `eq`: teste si un champ est *égal à* une valeur constante.
- `ne`: teste si un champ n’est *pas égal à* une valeur constante.

Opérateurs de plage :

- `gt`: teste si un champ est *supérieur à* une valeur constante.
- `lt`: teste si un champ est *inférieur à* une valeur constante.
- `ge`: teste si un champ est *supérieur ou égal à* une valeur constante.
- `le`: teste si un champ est *inférieur ou égal à* une valeur constante.

## <a name="filter"></a>Filtrer

Utilisez `$filter` pour fournir un paramètre de filtre OData afin de rechercher uniquement les objets qui vous intéressent.

L’exemple REST suivant filtre sur la valeur `alternateId` d’une ressource :

```
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01&$filter=properties/alternateId%20eq%20'unique identifier'
```

L’exemple C# suivant filtre sur la date de création de la ressource :

```csharp
var odataQuery = new ODataQuery<Asset>("properties/created lt 2018-05-11T17:39:08.387Z");
var firstPage = await MediaServicesArmClient.Assets.ListAsync(CustomerResourceGroup, CustomerAccountName, odataQuery);
```

## <a name="order-by"></a>Trier par

Utiliser `$orderby` pour trier les objets retournés par le paramètre spécifié. Par exemple :  

```
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01$orderby=properties/created%20gt%202018-05-11T17:39:08.387Z
```

Pour trier les résultats par ordre croissant ou décroissant, ajoutez `asc` ou `desc` au nom du champ, en les séparant par un espace. Par exemple : `$orderby properties/created desc`.

## <a name="skip-token"></a>Jeton d’évitement

Si une réponse de requête contient un grand nombre d’éléments, le service retourne une valeur `$skiptoken` (`@odata.nextLink`) pour obtenir la page de résultats suivante. Utilisez-la pour parcourir tout le jeu de résultats.

Dans Media Services v3, vous ne pouvez pas configurer la taille de la page. La taille de la page varie en fonction du type d’entité. Lisez les sections individuelles qui suivent pour plus d’informations.

Si des entités sont créées ou supprimées pendant que vous paginez la collection, les changements sont reflétés dans les résultats retournés (si ces changements concernent la partie de la collection qui n’a pas été téléchargée).

> [!TIP]
> Utilisez toujours `nextLink` pour énumérer la collection et ne pas dépendre d’une taille de page particulière.
>
> La valeur `nextLink` est présente uniquement s’il y a plus d’une page d’entités.

Prenons l’exemple suivant où la valeur `$skiptoken` est utilisée. Veillez à remplacer *amstestaccount* par le nom de votre compte et à définir la valeur *api-version* avec la version la plus récente.

Si vous demandez une liste de ressources comme celle-ci :

```
GET  https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01 HTTP/1.1
x-ms-client-request-id: dd57fe5d-f3be-4724-8553-4ceb1dbe5aab
Content-Type: application/json; charset=utf-8
```

Vous recevrez une réponse similaire à celle-ci :

```
HTTP/1.1 200 OK

{
"value":[
{
"name":"Asset 0","id":"/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaservices/amstestaccount/assets/Asset 0","type":"Microsoft.Media/mediaservices/assets","properties":{
"assetId":"00000000-0000-0000-0000-000000000000","created":"2018-12-11T22:12:44.98Z","lastModified":"2018-12-11T22:15:48.003Z","container":"asset-00000000-0000-0000-0000-0000000000000","storageAccountName":"amsacctname","storageEncryptionFormat":"None"
}
},
// lots more assets
{
"name":"Asset 517","id":"/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaservices/amstestaccount/assets/Asset 517","type":"Microsoft.Media/mediaservices/assets","properties":{
"assetId":"00000000-0000-0000-0000-000000000000","created":"2018-12-11T22:14:08.473Z","lastModified":"2018-12-11T22:19:29.657Z","container":"asset-00000000-0000-0000-0000-000000000000","storageAccountName":"amsacctname","storageEncryptionFormat":"None"
}
}
],"@odata.nextLink":"https:// management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01&$skiptoken=Asset+517"
}
```

Vous devez ensuite demander la page suivante en envoyant une requête GET pour :

```
https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01&$skiptoken=Asset+517
```

L’exemple C# suivant montre comment énumérer tous les localisateurs de streaming dans le compte.

```csharp
var firstPage = await MediaServicesArmClient.StreamingLocators.ListAsync(CustomerResourceGroup, CustomerAccountName);

var currentPage = firstPage;
while (currentPage.NextPageLink != null)
{
    currentPage = await MediaServicesArmClient.StreamingLocators.ListNextAsync(currentPage.NextPageLink);
}
```

## <a name="using-logical-operators-to-combine-query-options"></a>Utilisation d’opérateurs logiques pour combiner des options de requête

Media Services v3 prend en charge les opérateurs logiques **OU** et **ET**. 

L’exemple REST suivant vérifie l’état de la tâche :

```
https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/qbtest/providers/Microsoft.Media/mediaServices/qbtest/transforms/VideoAnalyzerTransform/jobs?$filter=properties/state%20eq%20Microsoft.Media.JobState'Scheduled'%20or%20properties/state%20eq%20Microsoft.Media.JobState'Processing'&api-version=2018-07-01
```

Vous construisez la même requête dans C# comme suit : 

```csharp
var odataQuery = new ODataQuery<Job>("properties/state eq Microsoft.Media.JobState'Scheduled' or properties/state eq Microsoft.Media.JobState'Processing'");
client.Jobs.List(config.ResourceGroup, config.AccountName, VideoAnalyzerTransformName, odataQuery);
```

## <a name="filtering-and-ordering-options-of-entities"></a>Filtrage et classement des options des entités

Le tableau suivant montre comment appliquer les options de filtrage et de classement à des entités différentes :

|Nom de l’entité|Nom de la propriété|Filtrer|JSON|
|---|---|---|---|
|[Éléments multimédias](/rest/api/media/assets/)|name|`eq`, `gt`, `lt`, `ge`, `le`|`asc` et `desc`|
||properties.alternateId |`eq`||
||properties.assetId |`eq`||
||properties.created| `eq`, `gt`, `lt`| `asc` et `desc`|
|[Stratégies de clé de contenu](/rest/api/media/contentkeypolicies)|name|`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` et `desc`|
||properties.created    |`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` et `desc`|
||properties.description    |`eq`, `ne`, `ge`, `le`, `gt`, `lt`||
||properties.lastModified|`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` et `desc`|
||properties.policyId|`eq`, `ne`||
|[Tâches](/rest/api/media/jobs)| name  | `eq`            | `asc` et `desc`|
||properties.state        | `eq`, `ne`        |                         |
||properties.created      | `gt`, `ge`, `lt`, `le`| `asc` et `desc`|
||properties.lastModified | `gt`, `ge`, `lt`, `le` | `asc` et `desc`| 
|[Localisateurs de Streaming](/rest/api/media/streaminglocators)|name|`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` et `desc`|
||properties.created    |`eq`, `ne`, `ge`, `le`,  `gt`, `lt`|`asc` et `desc`|
||properties.endTime    |`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` et `desc`|
|[Stratégies de streaming](/rest/api/media/streamingpolicies)|name|`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` et `desc`|
||properties.created    |`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` et `desc`|
|[Transformations](/rest/api/media/transforms)| name | `eq`            | `asc` et `desc`|
|| properties.created      | `gt`, `ge`, `lt`, `le`| `asc` et `desc`|
|| properties.lastModified | `gt`, `ge`, `lt`, `le`| `asc` et `desc`|

## <a name="next-steps"></a>Étapes suivantes

* [Répertorier les ressources](/rest/api/media/assets/list)
* [Répertorier les stratégies de clé de contenu](/rest/api/media/contentkeypolicies/list)
* [Répertorier les travaux](/rest/api/media/jobs/list)
* [Répertorier les stratégies de streaming](/rest/api/media/streamingpolicies/list)
* [Répertorier les localisateurs de streaming](/rest/api/media/streaminglocators/list)
* [Diffuser un fichier](stream-files-dotnet-quickstart.md)
* [Quotas et limites](limits-quotas-constraints.md)
