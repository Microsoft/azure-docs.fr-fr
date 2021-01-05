---
title: Résoudre des problèmes liés aux exceptions Taux de requêtes trop élevé Azure Cosmos DB
description: Découvrez comment diagnostiquer et corriger les exceptions Taux de requêtes trop élevé.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: c3a692a8df93931bd8fd7982e69fe2f9118444c2
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94411335"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-rate-too-large-exceptions"></a>Diagnostiquer et résoudre des problèmes liés aux exceptions Taux de requêtes trop élevé Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Un message « Taux de requêtes trop élevé » ou le code d’erreur 429 indique que vos requêtes sont limitées.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
La section suivante répertorie les causes et les solutions connues pour un trop grand nombre de requêtes.

### <a name="check-the-metrics"></a>Vérifier les métriques
Vérifiez [Analyse Azure Cosmos DB](monitor-cosmos-db.md) pour connaître le nombre d’exceptions 429.

#### <a name="cause"></a>Cause :
Le débit consommé (Unité de requête/s) a dépassé le [débit configuré](set-throughput.md). Le kit de développement logiciel (SDK) retente automatiquement de nouvelles requêtes en fonction de la stratégie de nouvelles tentatives spécifiée. Si cet échec se produit souvent, envisagez d’augmenter le débit de la collection. Vérifiez les mesures du portail pour voir si vous obtenez des erreurs 429. Examinez votre clé de partition pour vous assurer qu’elle entraîne une [répartition uniforme du stockage et du volume de demandes](partitioning-overview.md).

#### <a name="solution"></a>Solution :
1. Utilisez le [portail ou SDK](set-throughput.md) pour augmenter le débit provisionné.
1. Basculez la base de données ou le conteneur vers la [mise à l’échelle automatique](provision-throughput-autoscale.md).

## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-dot-net-sdk.md) des problèmes lors de l’utilisation du kit de développement logiciel (SDK) .NET Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) et [.NET V2](performance-tips.md).
* [Diagnostiquer et résoudre](troubleshoot-java-sdk-v4-sql.md) des problèmes lors de l'utilisation du SDK Java v4 Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour le [SDK Java v4](performance-tips-java-sdk-v4-sql.md).