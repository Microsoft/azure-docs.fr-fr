---
title: Migration d’une application de façon à utiliser le SDK .NET Azure Cosmos DB 2.0 (Microsoft.Azure.Cosmos)
description: Apprenez à mettre à niveau votre application .NET existante du Kit de développement logiciel (SDK) v1 vers le SDK .NET v2 pour l’API Core (SQL).
author: stefArroyo
ms.author: esarroyo
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/15/2020
ms.openlocfilehash: 88c40452da72ed4ab43d2d7613636136a5b78cfe
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94550020"
---
# <a name="migrate-your-application-to-use-the-azure-cosmos-db-net-sdk-v2"></a>Migration d’une application de façon à utiliser le SDK .NET Azure Cosmos DB v2

> [!IMPORTANT]
> Il est important de noter que la version 3 du Kit de développement logiciel (SDK) .NET est actuellement disponible et qu’un plan de migration de v2 vers v3 est disponible [ici](migrate-dotnet-v3.md). Pour en savoir plus sur le Kit de développement logiciel (SDK) .NET Azure Cosmos DB v2, consultez les [notes de publication](sql-api-sdk-dotnet.md), le [référentiel GitHub .NET](https://github.com/Azure/azure-cosmos-dotnet-v2), les [conseils de performances](performance-tips.md) du SDK .NET v2 et le [guide de résolution des problèmes](troubleshoot-dot-net-sdk.md).
>

Cet article souligne quelques-unes des considérations relatives à la mise à niveau de votre application .NET v1 existante vers le Kit de développement logiciel (SDK) .NET Azure Cosmos DB v2 pour l’API Core (SQL). Le Kit de développement logiciel (SDK) .NET Azure Cosmos DB v2 correspond à l’espace de noms `Microsoft.Azure.DocumentDB`. Vous pouvez utiliser les informations fournies dans ce document si vous migrez votre application depuis l’une des plateformes .NET Azure Cosmos DB suivantes pour utiliser le Kit de développement logiciel (SDK) v2 `Microsoft.Azure.Cosmos` :

* Kit de développement logiciel (SDK) .NET Framework Azure Cosmos DB v1 pour l’API SQL
* Kit SDK .NET Core Azure Cosmos DB v1 pour l’API SQL

## <a name="whats-available-in-the-net-v2-sdk"></a>Éléments disponibles dans le Kit de développement logiciel (SDK) .NET v2

Le Kit de développement logiciel (SDK) v2 contient de nombreuses améliorations en matière de convivialité et de performances, notamment :

* Prise en charge du mode direct TCP pour les clients non Windows
* Prise en charge des écritures multirégions
* Améliorations des performances des requêtes
* Prise en charge des collections géospatiales/géométriques et de l’indexation
* Améliorations des diagnostics de transport direct/TCP
* Mise à jour de la pile de transport TCP direct pour réduire le nombre de connexions établies
* Améliorations de la réduction de la latence dans le RequestTimeout

La majeure partie de la logique de nouvelle tentative et des niveaux inférieurs du Kit de développement logiciel (SDK) restent largement inchangée.

## <a name="why-migrate-to-the-net-v2-sdk"></a>Pourquoi migrer vers le Kit de développement logiciel (SDK) .NET v2

Outre les nombreuses améliorations en matière de performances, les nouvelles fonctionnalités investies dans le dernier Kit de développement logiciel (SDK) ne seront pas reportées sur les versions antérieures.

En outre, les anciens Kits de développement logiciel (SDK) seront remplacés par des versions plus récentes et le SDK v1 passera en [mode de maintenance](sql-api-sdk-dotnet.md). Pour une expérience de développement optimale, nous vous recommandons de migrer votre application vers une version ultérieure du Kit de développement logiciel (SDK).

## <a name="major-changes-from-v1-sdk-to-v2-sdk"></a>Modifications majeures de la version 1 à la version 2 du Kit de développement logiciel (SDK)

### <a name="direct-mode--tcp"></a>Mode direct + TCP

Le Kit de développement logiciel (SDK) .NET v2 prend désormais en charge le mode direct et le mode passerelle. Le mode direct prend en charge la connectivité via le protocole TCP et offre de meilleures performances, car il se connecte directement aux réplicas principaux avec moins de tronçons réseau.

Pour plus d’informations, consultez le [guide relatif aux modes de connectivité du Kit de développement logiciel (SDK) SQL Azure Cosmos DB](sql-sdk-connection-modes.md).

### <a name="session-token-formatting"></a>Mise en forme des jetons de session

Le Kit de développement logiciel (SDK) v2 n’utilise plus le format de jeton de session *simple* tel qu’il était utilisé dans la version 1, mais utilise plutôt la mise en forme *vectorielle*. Le format doit être converti lors de la transmission à l’application cliente avec des versions différentes, car les formats ne sont pas interchangeables.

Pour plus d’informations, consultez la [conversion de formats de jeton de session dans le Kit de développement logiciel (SDK) .NET](how-to-convert-session-token.md).

### <a name="using-the-net-change-feed-processor-sdk"></a>Utilisation du Kit de développement logiciel (SDK) du processeur de flux de modification .NET

La bibliothèque du processeur de flux de modification .NET 2.1.x requiert `Microsoft.Azure.DocumentDB` 2.0 ou une version ultérieure.

La bibliothèque 2.1.x présente les principales modifications suivantes :

* Amélioration de la stabilité et de la capacité de diagnostic
* Amélioration de la gestion des erreurs et des exceptions
* Prise en charge supplémentaire des collections de baux partitionnés
* Extensions avancées pour implémenter l’interface et la classe `ChangeFeedDocument` pour un traitement et un suivi supplémentaires des erreurs
* Ajout de la prise en charge de l’utilisation d’un magasin personnalisé afin de rendre persistants les jetons de continuation par partition

Pour plus d’informations, consultez les [notes de publication](sql-api-sdk-dotnet-changefeed.md) de la bibliothèque du processeur de flux de modification.

### <a name="using-the-bulk-executor-library"></a>Utilisation de la bibliothèque d’exécuteur en bloc

La bibliothèque d’exécuteur en bloc v2 a actuellement une dépendance sur le Kit de développement logiciel (SDK) .NET Azure Cosmos DB 2.5.1 ou version ultérieure.

Pour plus d’informations, consultez [Vue d’ensemble de la bibliothèque d’exécuteur en bloc Azure Cosmos DB](bulk-executor-overview.md) et les [notes de publication](sql-api-sdk-bulk-executor-dot-net.md) de la bibliothèque d’exécuteur en bloc .NET.

## <a name="next-steps"></a>Étapes suivantes

* Lire [d’autres conseils en matière de performances](sql-api-get-started.md) en utilisant l’API Azure Cosmos DB pour SQL v2 pour optimiser votre application afin d’obtenir des performances maximales
* En savoir plus sur [ce que le Kit de développement logiciel (SDK) v2 vous permet de faire](sql-api-dotnet-samples.md)