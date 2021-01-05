---
title: Résoudre des problèmes liés aux exceptions non autorisées Azure Cosmos DB
description: Découvrez comment diagnostiquer et corriger des exceptions non autorisées.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: aa9bf1fd706ccf6064893f1141be5e5b2f185ff3
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94411165"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-unauthorized-exceptions"></a>Diagnostiquer et résoudre des problèmes liés à des exceptions non autorisées Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

HTTP 401 : La signature MAC trouvée dans la requête HTTP n’est pas la même que la signature calculée.
Si vous avez reçu le message d’erreur 401 « La signature MAC trouvée dans la requête HTTP n’est pas la même que la signature calculée », cela peut être dû aux scénarios suivants.

Pour les kits de développement logiciel (SDK) plus anciens, l’exception peut apparaître comme une exception JSON non valide à la place de l’exception non autorisée 401 correcte. Les SDK plus récents gèrent correctement ce scénario et fournissent un message d’erreur valide.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
La liste suivante répertorie les causes connues et les solutions pour les exceptions non autorisées.

### <a name="the-key-wasnt-properly-rotated-is-the-most-common-scenario"></a>Le scénario le plus courant est que la clé n’a pas été correctement permutée
La signature MAC 401 est visible peu après une rotation de clé et finit par s’arrêter sans aucune modification. 

#### <a name="solution"></a>Solution :
La clé a été pivotée et n’a pas respecté les [meilleures pratiques](secure-access-to-data.md#key-rotation). La rotation des clés de compte Azure Cosmos DB peut prendre de quelques secondes à plusieurs jours, selon la taille du compte Azure Cosmos DB.

### <a name="the-key-is-misconfigured"></a>La clé n’est pas correctement configurée 
Le problème 401 de signature MAC est cohérent et se produit pour tous les appels avec cette clé.

#### <a name="solution"></a>Solution :
La clé n’est pas correctement configurée sur l’application et utilise une clé incorrecte pour le compte ou la clé entière n’a pas été copiée.

### <a name="the-application-is-using-the-read-only-keys-for-write-operations"></a>L’application utilise les clés en lecture seule pour les opérations d’écriture
Le problème 401 de signature MAC se produit uniquement pour les opérations d’écriture telles que créer ou remplacer, mais une demande de lecture est effectuée.

#### <a name="solution"></a>Solution :
Faites basculer l’application afin qu’elle utilise une clé de lecture/d’écriture pour permettre aux opérations de se terminer correctement.

### <a name="race-condition-with-create-container"></a>Condition de concurrence critique avec une création de conteneur
Le problème 401 de signature MAC se produit peu après la création d’un conteneur. Ce problème se produit uniquement tant que la création du conteneur n’est pas terminée.

#### <a name="solution"></a>Solution :
Il existe une condition de concurrence avec la création d’un conteneur. Une instance d’application tente d’accéder au conteneur avant que la création du conteneur soit terminée. Le scénario le plus courant pour cette condition de concurrence critique est lorsque l’application est en cours d’exécution et que le conteneur est supprimé et recréé avec le même nom. Le Kit de développement logiciel (SDK) tente d’utiliser le nouveau conteneur, mais la création du conteneur est toujours en cours, de sorte qu’il n’en a pas les clés.

## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-dot-net-sdk.md) des problèmes lors de l’utilisation du kit de développement logiciel (SDK) .NET Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) et [.NET V2](performance-tips.md).
* [Diagnostiquer et résoudre](troubleshoot-java-sdk-v4-sql.md) des problèmes lors de l'utilisation du SDK Java v4 Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour le [SDK Java v4](performance-tips-java-sdk-v4-sql.md).