---
title: Scripts PowerShell utilisant le module Az.Search
titleSuffix: Azure Cognitive Search
description: Créez et configurez un service Recherche cognitive Azure avec PowerShell. Vous pouvez mettre à l'échelle un service, gérer les clés API d'administration et de requête et interroger des informations système.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: powershell
ms.topic: conceptual
ms.date: 02/11/2020
ms.openlocfilehash: d7b672b7e2c3004eba4a38bd659965b7dee24db6
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422482"
---
# <a name="manage-your-azure-cognitive-search-service-with-powershell"></a>Gérer votre service Recherche cognitive Azure avec PowerShell
> [!div class="op_single_selector"]
> * [Portail](search-manage.md)
> * [PowerShell](search-manage-powershell.md)
> * [REST API](/rest/api/searchmanagement/)
> * [Kit de développement logiciel (SDK) .NET](/dotnet/api/microsoft.azure.management.search)
> * [Python](https://pypi.python.org/pypi/azure-mgmt-search/0.1.0)> 

Vous pouvez exécuter des scripts et des applets de commande PowerShell sur Windows, Linux ou dans [Azure Cloud Shell](../cloud-shell/overview.md) pour créer et configurer Recherche cognitive Azure. Le module **Az.Search** étend [Azure PowerShell](/powershell/) avec une parité complète pour les [API REST de gestion de la recherche](/rest/api/searchmanagement) et la capacité à réaliser les tâches suivantes :

> [!div class="checklist"]
> * [Lister les services de recherche dans un abonnement](#list-search-services)
> * [Retourner les informations sur les services](#get-search-service-information)
> * [Créer ou supprimer un service](#create-or-delete-a-service)
> * [Régénérer des clés API d’administration](#regenerate-admin-keys)
> * [Créer ou supprimer des clés API de requête](#create-or-delete-query-keys)
> * [Effectuer un scale-up ou un scale-down avec les réplicas et les partitions](#scale-replicas-and-partitions)

Parfois, des questions sont posées sur des tâches qui ne figurent *pas* dans la liste ci-dessus. Vous ne pouvez pas utiliser le module **Az.Search** ou l’API REST de gestion pour changer un nom de serveur, une région ou un niveau. Des ressources dédiées sont allouées lorsqu’un service est créé. Ainsi, toute modification du matériel sous-jacent (emplacement ou type de nœud) nécessite un nouveau service. De même, il n’existe pas d’API ou d’outils pour transférer du contenu, tel qu’un index, d’un service à un autre.

Au sein d’un service, la création et la gestion de contenu s’effectuent par le biais de l’[API REST du service de recherche](/rest/api/searchservice/) ou du [SDK .NET](/dotnet/api/overview/azure/search.documents-readme). Bien qu’il n’existe aucune commande PowerShell dédiée au contenu, vous pouvez écrire un script PowerShell qui appelle des API REST ou .NET afin de créer et de charger des index.

<a name="check-versions-and-load"></a>

## <a name="check-versions-and-load-modules"></a>Vérifier les versions et charger des modules

Les exemples de cet article sont interactifs et requièrent des autorisations élevées. Azure PowerShell (module **Az** )doit être installé. Pour plus d’informations, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/).

### <a name="powershell-version-check-51-or-later"></a>Vérification de la version PowerShell (5.1 ou ultérieure)

La version locale de PowerShell doit correspondre à la version 5.1 ou ultérieure, sur n’importe quel système d’exploitation pris en charge.

```azurepowershell-interactive
$PSVersionTable.PSVersion
```

### <a name="load-azure-powershell"></a>Charger Azure PowerShell

Si vous ne savez pas si **Az** est installé, exécutez la commande suivante pour le vérifier. 

```azurepowershell-interactive
Get-InstalledModule -Name Az
```

Certains systèmes ne chargent pas automatiquement les modules. Si la commande précédente génère une erreur, essayez de charger le module, et si cela échoue, consultez à nouveau les instructions d’installation pour vous assurer que vous n'avez oublié aucune étape.

```azurepowershell-interactive
Import-Module -Name Az
```

### <a name="connect-to-azure-with-a-browser-sign-in-token"></a>Se connecter à Azure avec un jeton de connexion de navigateur

Vous pouvez utiliser vos informations d’identification de connexion au portail pour vous connecter à un abonnement dans PowerShell. Vous pouvez également [vous authentifier de manière non interactive avec un principal de service](../active-directory/develop/howto-authenticate-service-principal-powershell.md).

```azurepowershell-interactive
Connect-AzAccount
```

Si vous disposez de plusieurs abonnements Azure, définissez votre abonnement Azure à utiliser. Pour afficher une liste de vos abonnements en cours, exécutez la commande suivante.

```azurepowershell-interactive
Get-AzSubscription | sort SubscriptionName | Select SubscriptionName
```

Pour spécifier l’abonnement, exécutez la commande suivante. Dans l’exemple suivant, le nom de l’abonnement est `ContosoSubscription`.

```azurepowershell-interactive
Select-AzSubscription -SubscriptionName ContosoSubscription
```

<a name="list-search-services"></a>

## <a name="list-services-in-a-subscription"></a>Répertorier les services dans un abonnement

Les commandes suivantes proviennent d' [**Az.Resources**](/powershell/module/az.resources) et renvoient des informations sur les ressources et services existants déjà approvisionnés dans votre abonnement. Si vous ne savez pas combien de services de recherche sont déjà créés, ces commandes vous en informent et vous évitent d'accéder au portail.

La première commande renvoie tous les services de recherche.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Search/searchServices | ft
```

Dans la liste des services, elle permet d'obtenir des informations sur une ressource spécifique.

```azurepowershell-interactive
Get-AzResource -ResourceName <service-name>
```

Les résultats se présentent comme suit.

```
Name              : my-demo-searchapp
ResourceGroupName : demo-westus
ResourceType      : Microsoft.Search/searchServices
Location          : westus
ResourceId        : /subscriptions/<alpha-numeric-subscription-ID>/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="import-azsearch"></a>Importer Az.Search

Les commandes provenant d' [**Az.Search**](/powershell/module/az.search) ne sont pas disponibles tant que vous n'avez pas chargé le module.

```azurepowershell-interactive
Install-Module -Name Az.Search
```

### <a name="list-all-azsearch-commands"></a>Répertorier toutes les commandes Az.Search

En guise de vérification, renvoyez une liste de commandes fournies dans le module.

```azurepowershell-interactive
Get-Command -Module Az.Search
```

Les résultats se présentent comme suit.

```
CommandType     Name                                Version    Source
-----------     ----                                -------    ------
Cmdlet          Get-AzSearchAdminKeyPair            0.7.1      Az.Search
Cmdlet          Get-AzSearchQueryKey                0.7.1      Az.Search
Cmdlet          Get-AzSearchService                 0.7.1      Az.Search
Cmdlet          New-AzSearchAdminKey                0.7.1      Az.Search
Cmdlet          New-AzSearchQueryKey                0.7.1      Az.Search
Cmdlet          New-AzSearchService                 0.7.1      Az.Search
Cmdlet          Remove-AzSearchQueryKey             0.7.1      Az.Search
Cmdlet          Remove-AzSearchService              0.7.1      Az.Search
Cmdlet          Set-AzSearchService                 0.7.1      Az.Search
```

## <a name="get-search-service-information"></a>Obtenir des informations sur le service de recherche

Après avoir importé **Az.Search** et déterminé le groupe de ressources contenant votre service de recherche, exécutez [Get-AzSearchService](/powershell/module/az.search/get-azsearchservice) pour renvoyer la définition de service, notamment le nom, la région, le niveau, ainsi que le nombre de réplicas et de partitions.

```azurepowershell-interactive
Get-AzSearchService -ResourceGroupName <resource-group-name>
```

Les résultats se présentent comme suit.

```
Name              : my-demo-searchapp
ResourceGroupName : demo-westus
ResourceType      : Microsoft.Search/searchServices
Location          : West US
Sku               : Standard
ReplicaCount      : 1
PartitionCount    : 1
HostingMode       : Default
ResourceId        : /subscriptions/<alphanumeric-subscription-ID>/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="create-or-delete-a-service"></a>Créer ou supprimer un service

[**New-AzSearchService**](/powershell/module/az.search/new-azsearchadminkey) permet de [créer un service de recherche](search-create-service-portal.md).

```azurepowershell-interactive
New-AzSearchService -ResourceGroupName "demo-westus" -Name "my-demo-searchapp" -Sku "Standard" -Location "West US" -PartitionCount 3 -ReplicaCount 3
``` 
Les résultats se présentent comme suit.

```
ResourceGroupName : demo-westus
Name              : my-demo-searchapp
Id                : /subscriptions/<alphanumeric-subscription-ID>/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
Location          : West US
Sku               : Standard
ReplicaCount      : 3
PartitionCount    : 3
HostingMode       : Default
Tags
```     

## <a name="regenerate-admin-keys"></a>Régénération des clés d’administration

[**New- AzSearchAdminKey**](/powershell/module/az.search/new-azsearchadminkey) permet de substituer les [clés API](search-security-api-keys.md) d'administration. Deux clés d'administration sont créées avec chaque service à des fins d'accès authentifié. Ces clés sont nécessaires pour chaque requête. Les deux clés d’administration fonctionnement de la même manière, et accorde au service de recherche un accès total en écriture, avec possibilité de récupérer des informations ou de créer et de supprimer des objets. Deux clés sont disponibles pour vous permettre d'en utiliser une lorsque vous remplacez l'autre. 

Vous ne pouvez régénérer qu'une clé à la fois, spécifiée en tant que clé `primary` ou `secondary`. Pour ne pas interrompre le service, pensez à mettre à jour la totalité du code client de manière à utiliser une clé secondaire lors de la substitution de la clé primaire. Évitez tout changement de clés lorsque des opérations sont en cours.

En effet, si vous régénérez les clés sans mettre à jour le code client, les requêtes utilisant l'ancienne clé n'aboutiront pas. La régénération de toutes les nouvelles clés ne bloque indéfiniment votre service, et vous en conservez l'accès via le portail. Une fois les clés primaire et secondaire régénérées, vous pouvez mettre à jour le code client pour utiliser les nouvelles clés, après quoi les opérations reprennent.

Les valeurs correspondant aux clés API sont générées par le service. Vous ne pouvez pas fournir de clé personnalisée à des fins d'utilisation par Recherche cognitive Azure. De même, il n'existe aucun nom défini par l'utilisateur pour les clés API d'administration. Les références à la clé correspondent à des chaînes fixes, `primary` ou `secondary`. 

```azurepowershell-interactive
New-AzSearchAdminKey -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -KeyKind Primary
```

Les résultats se présentent comme suit. Les deux clés sont renvoyées, même si vous n'en modifiez qu'une.

```
Primary                    Secondary
-------                    ---------
<alphanumeric-guid>        <alphanumeric-guid>  
```

## <a name="create-or-delete-query-keys"></a>Créer ou supprimer des clés de requête

[**New- AzSearchQueryKey**](/powershell/module/az.search/new-azsearchquerykey) permet de créer des [clés API](search-security-api-keys.md) de requête à des fins d'accès en écriture seule à partir d’applications clientes vers un index Recherche cognitive Azure. Les clés de requête sont utilisées pour s’authentifier auprès d’un index spécifique et récupérer les résultats de la recherche. Les clés de requête n’accordent pas d’accès en lecture seule à d’autres éléments du service, comme un index, une source de données ou un indexeur.

Vous ne pouvez pas fournir de clé à des fins d'utilisation par Recherche cognitive Azure. Les clés API sont générées par le service.

```azurepowershell-interactive
New-AzSearchQueryKey -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <query-key-name> 
```

## <a name="scale-replicas-and-partitions"></a>Mettre à l'échelle des réplicas et des partitions

[**Set-AzSearchService**](/powershell/module/az.search/set-azsearchservice) permet d' [augmenter ou de diminuer les réplicas et les partitions](search-capacity-planning.md) pour réajuster les ressources facturables de votre service. Augmenter les réplicas ou les partitions se répercute sur votre facture, qui présente des frais fixes et variables. En cas de besoin temporaire de plus de puissance de traitement, vous pouvez augmenter les réplicas et les partitions pour gérer la charge de travail. La zone de surveillance de la page Vue d’ensemble du portail présente des vignettes sur la latence de requête, les requêtes par seconde et les limitations, indiquant si la capacité actuelle est suffisante.

L'ajout ou la suppression de ressources peut prendre un certain temps. Des ajustements de la capacité interviennent en arrière-plan, ce qui permet la poursuite des charges de travail existantes. Une capacité supplémentaire est utilisée pour les demandes entrantes, dès qu’elle est prête, sans configuration supplémentaire. 

La suppression de la capacité peut entraîner des perturbations. Il est recommandé d'arrêter tous les travaux d'indexation avant de réduire la capacité pour éviter la suppression de requêtes. Si ce n’est pas possible, essayez de réduire la capacité de façon incrémentielle, un seul réplica et une seule partition à la fois, jusqu'à ce que vos nouveaux niveaux cibles soient atteints.

Il n'est pas possible d'arrêter la commande une fois celle-ci en cours d'exécution. Vous devez attendre qu'elle prenne fin avant d'examiner les résultats.

```azurepowershell-interactive
Set-AzSearchService -ResourceGroupName <resource-group-name> -Name <search-service-name> -PartitionCount 6 -ReplicaCount 6
```

Les résultats se présentent comme suit.

```
ResourceGroupName : demo-westus
Name              : my-demo-searchapp
Location          : West US
Sku               : Standard
ReplicaCount      : 6
PartitionCount    : 6
HostingMode       : Default
Id                : /subscriptions/65a1016d-0f67-45d2-b838-b8f373d6d52e/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="next-steps"></a>Étapes suivantes

Génération d'un [index](search-what-is-an-index.md), [interrogation d'un index](search-query-overview.md) à l’aide du portail, des API REST ou du kit de développement logiciel (SDK) .NET.

* [Créer un index Recherche cognitive Azure dans le portail Azure](search-get-started-portal.md)
* [Configuration d’un indexeur pour charger des données provenant d’autres services](search-indexer-overview.md)
* [Interrogation d’un index de Recherche cognitive Azure à l’aide de l’Explorateur de recherche dans le Portail Azure](search-explorer.md)
* [Utilisation de la Recherche cognitive Azure dans .NET](search-howto-dotnet-sdk.md)