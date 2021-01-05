---
title: Résoudre les problèmes de dépassement de délai de demande ou HTTP 408 Azure Cosmos DB avec le SDK .NET
description: Apprenez à diagnostiquer et corriger des exceptions de dépassement de délai de demande avec le SDK .NET.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 08/06/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.custom: devx-track-dotnet
ms.openlocfilehash: c8d448cf335f328b5ae55579fd30127ef0e37e9d
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340496"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-net-sdk-request-timeout-exceptions"></a>Diagnostiquer et résoudre les exceptions de dépassement de délai de demande avec le SDK .NET
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

L'erreur HTTP 408 se produit si le SDK n'a pas pu terminer la demande avant l'expiration du délai d'attente.

## <a name="customize-the-timeout-on-the-azure-cosmos-db-net-sdk"></a>Personnaliser le délai d'expiration sur le Kit de développement logiciel (SDK) .NET Azure Cosmos

Le Kit de développement logiciel (SDK) offre deux alternatives distinctes pour contrôler les délais d’expiration, chacune avec une étendue différente.

### <a name="requesttimeout"></a>RequestTimeout

La configuration `CosmosClientOptions.RequestTimeout` (ou `ConnectionPolicy.RequestTimeout` pour le SDK v2) vous permet de définir un délai d'expiration qui concerne chaque requête réseau individuelle. Une opération lancée par un utilisateur peut couvrir plusieurs requêtes réseau (par exemple, il peut y avoir une limitation). Cette configuration s'appliquerait à chaque requête réseau lors de la nouvelle tentative. Ce délai n'est pas un délai d'expiration de demande d'opération de bout en bout.

### <a name="cancellationtoken"></a>CancellationToken

Toutes les opérations asynchrones dans le Kit de développement logiciel (SDK) ont un paramètre CancellationToken facultatif. Ce paramètre [CancellationToken](/dotnet/standard/threading/how-to-listen-for-cancellation-requests-by-polling) est utilisé tout au long de l'opération, sur toutes les requêtes réseau. Entre les requêtes réseau, le paramètre CancellationToken peut être vérifié et une opération annulée si le jeton associé a expiré. Le paramètre CancellationToken doit être utilisé pour définir un délai d'expiration approximatif sur l'étendue de l'opération.

> [!NOTE]
> Le paramètre `CancellationToken` est un mécanisme par lequel la bibliothèque vérifie l'annulation lorsqu'elle [ne provoque pas d'état non valide](https://devblogs.microsoft.com/premier-developer/recommended-patterns-for-cancellationtoken/). L'annulation de l'opération peut ne pas intervenir au moment même où le délai défini est écoulé. Une fois le délai écoulé, l'annulation peut en effet n'intervenir que lorsque les conditions de sécurité sont réunies.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
La liste suivante répertorie les causes connues et les solutions pour les exceptions de dépassement de délai de demande.

### <a name="high-cpu-utilization"></a>Utilisation élevée du processeur
L'utilisation élevée du processeur est le cas le plus courant. Pour une latence optimale, l'utilisation du processeur doit être d'environ 40 %. Utilisez 10 secondes comme intervalle pour superviser l'utilisation maximale (non moyenne) du processeur. Les pics d'utilisation du processeur sont plus courants avec les requêtes entre partitions où il peut y avoir plusieurs connexions pour une seule requête.

#### <a name="solution"></a>Solution :
L'application cliente qui utilise le SDK doit faire l'objet d'un scale-up ou d'un scale-out.

### <a name="socket-or-port-availability-might-be-low"></a>La disponibilité du socket ou du port peut être faible
Quand ils s’exécutent dans Azure, les clients qui utilisent le kit SDK .NET peuvent rencontrer un problème d’épuisement de ports Azure SNAT (PAT).

#### <a name="solution-1"></a>Solution 1 :
Si vous utilisez des machines virtuelles Azure, suivez le [guide consacré à l'insuffisance de ports SNAT](troubleshoot-dot-net-sdk.md#snat).

#### <a name="solution-2"></a>Solution 2 :
Si vous utilisez Azure App Service, suivez le [guide de dépannage des erreurs de connexion](../app-service/troubleshoot-intermittent-outbound-connection-errors.md#cause) et [utilisez les diagnostics App Service](https://azure.github.io/AppService/2018/03/01/Deep-Dive-into-TCP-Connections-in-App-Service-Diagnostics.html).

#### <a name="solution-3"></a>Solution 3 :
Si vous utilisez Azure Functions, suivez la [recommandation Azure Functions](../azure-functions/manage-connections.md#static-clients) de gestion des clients statiques ou des singletons pour tous les services impliqués (y compris Cosmos DB). Vérifiez les [limites de service](../azure-functions/functions-scale.md#service-limits) en fonction du type et de la taille de votre hébergement Function App.

#### <a name="solution-4"></a>Solution 4 :
Si vous utilisez un proxy HTTP, vérifiez qu’il peut prendre en charge le nombre de connexions configuré dans SDK `ConnectionPolicy`. Sinon, vous serez confronté à des problèmes de connexion.

### <a name="create-multiple-client-instances"></a>Créer plusieurs instances clientes
La création de plusieurs instances clientes peut entraîner des problèmes de conflit et de dépassement de délai de connexion.

#### <a name="solution"></a>Solution :
Suivez les [conseils en lien avec les performances](performance-tips-dotnet-sdk-v3-sql.md#sdk-usage) et utilisez une seule instance CosmosClient pour l’ensemble du processus.

### <a name="hot-partition-key"></a>Clé de partition à chaud
Azure Cosmos DB répartit le débit global approvisionné de manière uniforme entre les partitions physiques. Lorsqu'il existe une partition à chaud, une ou plusieurs clés de partition logique sur une partition physique consomment la totalité des unités de requête par seconde (RU/s) de la partition physique. Dans le même temps, les RU/s d'autres partitions physiques ne sont plus utilisées. Par conséquent, le nombre total de RU/s consommées sera inférieur au nombre global de RU/s approvisionnées dans la base de données ou le conteneur, mais vous verrez toujours une limitation (429s) sur les demandes par rapport à la clé de partition logique à chaud. Utilisez la [métrique Consommation d’unités de requête normalisée](monitor-normalized-request-units.md) pour voir si la charge de travail rencontre une partition à chaud. 

#### <a name="solution"></a>Solution :
Choisissez une clé de partition appropriée qui répartit uniformément le volume de demandes et le stockage. Découvrez comment [changer votre clé de partition](https://devblogs.microsoft.com/cosmosdb/how-to-change-your-partition-key/).

### <a name="high-degree-of-concurrency"></a>Degré élevé de simultanéité
L'application atteint un niveau élevé de simultanéité, ce qui peut entraîner des conflits sur le canal.

#### <a name="solution"></a>Solution :
L'application cliente qui utilise le SDK doit faire l'objet d'un scale-up ou d'un scale-out.

### <a name="large-requests-or-responses"></a>Demandes ou réponses volumineuses
Des demandes ou réponses volumineuses peuvent entraîner un blocage en tête de ligne sur le canal et exacerber les conflits, même avec un degré relativement faible de simultanéité.

#### <a name="solution"></a>Solution :
L'application cliente qui utilise le SDK doit faire l'objet d'un scale-up ou d'un scale-out.

### <a name="failure-rate-is-within-the-azure-cosmos-db-sla"></a>Le taux d'échec est compris dans le contrat de niveau de service Azure Cosmos DB
L’application doit être en mesure de gérer les défaillances temporaires et de réessayer si nécessaire. Il n'y a pas de nouvelle tentative pour les exceptions 408 car, lors de la création de chemins, il est impossible de savoir si le service a créé l'élément ou non. Le renvoi d’un même élément pour la création entraîne une exception de conflit. La logique métier des applications utilisateur peut avoir une logique personnalisée pour gérer les conflits, ce qui permet d'éviter l'ambiguïté d'un élément existant par rapport à un conflit suite à une nouvelle tentative de création.

### <a name="failure-rate-violates-the-azure-cosmos-db-sla"></a>Le taux d'échec n'est pas conforme au contrat de niveau de service Azure Cosmos DB
Contactez [Support Azure](https://aka.ms/azure-support).

## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-dot-net-sdk.md) des problèmes lors de l'utilisation du kit de développement logiciel (SDK) .NET Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) et [.NET v2](performance-tips.md).