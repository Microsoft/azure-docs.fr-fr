---
title: Résoudre les problèmes liés à des exceptions d’expiration du délai de demande de service Azure Cosmos DB
description: Découvrez comment diagnostiquer et corriger des exceptions d’expiration du délai de demande de service Azure Cosmos DB.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 5b188021de30561222f098e2b5782bada25d7ce0
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94411250"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-timeout-exceptions"></a>Diagnostiquer et résoudre les problèmes liés à des exceptions d’expiration du délai de demande Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB a retourné une exception HTTP 408 Expiration du délai de la demande.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
La liste suivante répertorie les causes connues et les solutions pour les exceptions de dépassement de délai de demande.

### <a name="check-the-sla"></a>Vérification du contrat de niveau de service
Vérifiez les [données de supervision Azure Cosmos DB](monitor-cosmos-db.md) pour voir si le nombre d’exceptions de 408 enfreint le contrat de niveau de service Azure Cosmos DB.

#### <a name="solution-1-it-didnt-violate-the-azure-cosmos-db-sla"></a>Solution 1 : Il n’a pas enfreint le contrat de niveau de service Azure Cosmos DB
L’application doit gérer ce scénario et réessayer sur ces échecs temporaires.

#### <a name="solution-2-it-did-violate-the-azure-cosmos-db-sla"></a>Solution 2 : Il a enfreint le contrat de niveau de service Azure Cosmos DB
Contactez [Support Azure](https://aka.ms/azure-support).
 
### <a name="hot-partition-key"></a>Clé de partition à chaud
Azure Cosmos DB répartit le débit global approvisionné de manière uniforme entre les partitions physiques. Lorsqu'il existe une partition à chaud, une ou plusieurs clés de partition logique sur une partition physique consomment la totalité des unités de requête par seconde (RU/s) de la partition physique. Dans le même temps, les RU/s d'autres partitions physiques ne sont plus utilisées. Par conséquent, le nombre total de RU/s consommées sera inférieur au nombre global de RU/s approvisionnées dans la base de données ou le conteneur. Vous verrez toujours une limitation (429s) sur les demandes par rapport à la clé de partition logique à chaud. Utilisez la [métrique Consommation d’unités de requête normalisée](monitor-normalized-request-units.md) pour voir si la charge de travail rencontre une partition à chaud. 

#### <a name="solution"></a>Solution :
Choisissez une clé de partition appropriée qui répartit uniformément le volume de demandes et le stockage. Découvrez comment [changer votre clé de partition](https://devblogs.microsoft.com/cosmosdb/how-to-change-your-partition-key/).

## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-dot-net-sdk.md) des problèmes lors de l'utilisation du kit de développement logiciel (SDK) .NET Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) et [.NET v2](performance-tips.md).
* [Diagnostiquer et résoudre](troubleshoot-java-sdk-v4-sql.md) des problèmes lors de l'utilisation du SDK Java v4 Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour le [SDK Java v4](performance-tips-java-sdk-v4-sql.md).