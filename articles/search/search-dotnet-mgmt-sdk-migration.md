---
title: Mettre à niveau vers le SDK .NET Management de Recherche Azure
titleSuffix: Azure Cognitive Search
description: Effectuez la mise à niveau vers le SDK .NET Management de Recherche Azure à partir d’une version précédente. Découvrez les nouvelles fonctionnalités et les modifications de code nécessaires pour la migration.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 07/08/2020
ms.openlocfilehash: 8648347eb48081389cf360fa949b31bbd0b8c71e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936705"
---
# <a name="upgrading-versions-of-the-azure-search-net-management-sdk"></a>Mise à niveau des versions du Kit de développement logiciel (SDK) .NET Management

Cet article explique comment migrer vers des versions ultérieures du SDK .NET Management de Recherche Azure, qui sert à provisionner ou à déprovisionner les services de recherche, à ajuster la capacité et à gérer les clés API.

Les kits SDK de gestion ciblent une version spécifique de l’API REST de gestion. Pour plus d’informations sur les concepts et les opérations, consultez [Gestion des recherches (REST)](/rest/api/searchmanagement/).

## <a name="versions"></a>Versions

| Version du SDK | Version de l’API REST correspondante | Ajout de fonctionnalités ou changement de comportement |
|-------------|--------------------------------|-------------------------------------|
| [3.0](https://www.nuget.org/packages/Microsoft.Azure.Management.Search/3.0.0) | api-version=2020-30-20 | Ajout de la sécurité de point de terminaison (pare-feu IP et intégration avec [Azure Private Link](../private-link/private-endpoint-overview.md)) |
| [2.0](https://www.nuget.org/packages/Microsoft.Azure.Management.Search/2.0.0) | api-version=2019-10-01 | Améliorations de la convivialité. Rupture de la modification de la [Liste des clés de requête](/rest/api/searchmanagement/querykeys/listbysearchservice) (GET n'existe plus). |
| [1.0](https://www.nuget.org/packages/Microsoft.Azure.Management.Search/1.0.1) | api-version=2015-08-19  | Première version |

## <a name="how-to-upgrade"></a>Mise à niveau

1. Mettez à jour vos références NuGet `Microsoft.Azure.Management.Search`, soit en utilisant la console NuGet Package, soit en effectuant un clic droit sur les références de votre projet et en sélectionnant « Gérer les packages NuGet » dans Visual Studio.

1. Une fois que NuGet a téléchargé les nouveaux packages et leurs dépendances, recompilez votre projet. En fonction de la structure de votre code, la recompilation peut réussir, auquel cas vous avez terminé.

1. Si votre build échoue, cela est peut-être dû au fait que vous avez implémenté certaines des interfaces du Kit SDK (par exemple, à des fins de tests unitaires) qui ont été modifiées. Pour résoudre ce problème, vous devez implémenter de nouvelles méthodes telles que `BeginCreateOrUpdateWithHttpMessagesAsync`.

1. Une fois les erreurs de build résolues, vous pouvez apporter des modifications à votre application pour bénéficier des nouvelles fonctionnalités. 

## <a name="upgrade-to-30"></a>Mettre à niveau vers la version 3.0

La version 3.0 ajoute la protection de point de terminaison privée en limitant l’accès à des plages d’adresses IP, et en intégrant éventuellement Azure Private Link pour les services de recherche qui ne doivent pas être visibles sur l’Internet public.

### <a name="new-apis"></a>Nouvelles API

| API | Category| Détails |
|-----|--------|------------------|
| [NetworkRuleSet](/rest/api/searchmanagement/services/createorupdate#networkruleset) | Pare-feu IP | Limitez l’accès à un point de terminaison de service à une liste d’adresses IP autorisées. Pour plus d’informations sur les concepts et pour savoir comment procéder dans le portail, consultez [Configurer le pare-feu IP](service-configure-firewall.md). |
| [Ressource de liaison privée partagée](/rest/api/searchmanagement/sharedprivatelinkresources) | Private Link | Créez une ressource de liaison privée partagée qui sera utilisée par un service de recherche.  |
| [Connexions de point de terminaison privé](/rest/api/searchmanagement/privateendpointconnections) | Private Link | Établissez et gérez les connexions à un service de recherche par le biais d’un point de terminaison privé. Pour plus d’informations sur les concepts et pour savoir comment procéder dans le portail, consultez [Créer un point de terminaison privé](service-create-private-endpoint.md).|
| [Ressource de liaison privée](/rest/api/searchmanagement/privatelinkresources/) | Private Link | Pour un service de recherche qui a une connexion de point de terminaison privé, obtenez la liste de tous les services utilisés sur le même réseau virtuel. Si votre solution de recherche comprend des indexeurs qui extraient des données à partir de sources Azure (Stockage Azure, Cosmos DB, Azure SQL), ou qui utilisent Cognitive Services ou Key Vault, toutes ces ressources doivent avoir des points de terminaison sur le réseau virtuel, et cette API doit retourner une liste. |
| [PublicNetworkAccess](/rest/api/searchmanagement/services/createorupdate#publicnetworkaccess)| Private Link | Il s’agit d’une propriété sur les requêtes de création ou de mise à jour de service. Quand elle est désactivée, la liaison privée est la seule modalité d’accès. |

### <a name="breaking-changes"></a>Changements cassants

Vous ne pouvez plus utiliser GET sur une requête [Lister les clés de requête](/rest/api/searchmanagement/querykeys/listbysearchservice). Dans les versions précédentes, vous pouviez utiliser GET ou POST. Dans cette version, et dans toutes les versions ultérieures, seul POST est pris en charge. 

## <a name="upgrade-to-20"></a>Mettre à niveau vers la version 2.0

La version 2 du SDK .NET Management de Recherche Azure est une mise à niveau mineure. La modification de votre code ne devrait donc nécessiter que peu d’efforts. Les modifications apportées au SDK sont strictement des modifications côté client destinées à améliorer la convivialité du SDK. Elles incluent notamment les suivantes :

* `Services.CreateOrUpdate` et ses versions asynchrones interrogent désormais automatiquement le provisionnement `SearchService` et ne retournent aucune donnée avant la fin du provisionnement du service. Cela vous évite d’avoir à écrire vous-même ce code d’interrogation.

* Si vous souhaitez continuer à interroger manuellement le provisionnement du service, vous pouvez utiliser la nouvelle méthode `Services.BeginCreateOrUpdate` ou l’une de ses versions asynchrones.

* De nouvelles méthodes `Services.Update` et leurs versions asynchrones ont été ajoutées au Kit SDK. Ces méthodes utilisent HTTP PATCH pour prendre en charge la mise à jour incrémentielle d’un service. Par exemple, vous pouvez désormais mettre à l’échelle un service en appliquant ces méthodes à une instance `SearchService` contenant uniquement les propriétés `partitionCount` et `replicaCount` souhaitées. L’ancienne manière d’appeler `Services.Get`, qui modifiait l’instance `SearchService` retournée et lui appliquait `Services.CreateOrUpdate`, est toujours prise en charge, mais elle n’est plus nécessaire. 

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des problèmes, le meilleur forum pour publier des questions est [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-cognitive-search?tab=Newest). Si vous trouvez un bogue, vous pouvez signaler un problème dans le [Référentiel GitHub du Kit de développement logiciel (SDK) Azure .NET](https://github.com/Azure/azure-sdk-for-net/issues). N’oubliez pas d’étiqueter le titre de votre problème avec « [Search] ».