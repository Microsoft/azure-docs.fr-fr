---
title: Résoudre un message « En-tête de demande trop volumineux » ou une requête incorrecte 400 dans Azure Cosmos DB
description: Découvrez comment diagnostiquer et corriger l’exception d’en-tête de demande trop volumineux.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.custom: devx-track-dotnet
ms.openlocfilehash: a14503bc56777563b6360143efaa86a136b22278
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340513"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-header-too-large-message"></a>Diagnostiquer et résoudre le message « En-tête de demande trop volumineux » dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le message « En-tête de demande trop volumineux » est généré avec un code d’erreur HTTP 400. Cette erreur se produit si la taille de l’en-tête de demande est devenue trop importante et dépasse la taille maximale autorisée. Nous vous recommandons d’utiliser la dernière version du Kit de développement logiciel (SDK). Utilisez au moins la version 3.x ou 2.x, car ces versions ajoutent un suivi de la taille d’en-tête au message d’exception.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
Le message « En-tête de demande trop volumineux » apparaît si le jeton de session ou de continuation est trop volumineux. Les sections suivantes décrivent la cause et la solution du problème dans chaque catégorie.

### <a name="session-token-is-too-large"></a>Jeton de session trop volumineux

#### <a name="cause"></a>Cause :
Une requête incorrecte 400 se produit probablement car le jeton de session est trop volumineux. Si les instructions suivantes sont vraies, cela confirme que le jeton de session est trop volumineux :

* L’erreur se produit lors d’opérations ponctuelles, par exemple, de création, de lecture et de mise à jour, quand il n’y a pas de jeton de continuation.
* L’exception a démarré sans apporter de modification à l’application. Le jeton de session croît à mesure que le nombre de partitions dans le conteneur augmente. Le nombre de partitions augmentent à mesure que croît la quantité de données ou le débit.

#### <a name="temporary-mitigation"></a>Atténuation temporaire : 
Redémarrez votre application cliente pour réinitialiser tous les jetons de session. Le jeton de session reviendra finalement à la taille à l’origine du problème. Utilisez la solution dans la section suivante pour éviter complètement ce problème.

#### <a name="solution"></a>Solution :
1. Suivez les instructions des articles contenant des conseils en termes de performances [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) ou [.NET v2](performance-tips.md). Convertissez l’application pour qu’elle utilise le mode de connexion directe avec le protocole TCP (Transmission Control Protocol). Le mode de connexion directe avec le protocole TCP n’a pas de restriction de taille d’en-tête comme le protocole HTTP, ce qui évite ce problème. Veillez à utiliser la dernière version du Kit de développement logiciel (SDK), qui contient un correctif pour les opérations de requête lorsque l’interopérabilité du service n’est pas disponible.
1. Si le mode de connexion directe avec le protocole TCP n’est pas une option pour votre charge de travail, atténuez le problème en modifiant le [niveau de cohérence du client](how-to-manage-consistency.md). Le jeton de session est utilisé uniquement pour la cohérence de session, ce qui est le niveau de cohérence par défaut pour Azure Cosmos DB. D’autres niveaux de cohérence n’utilisent pas le jeton de session.

### <a name="continuation-token-is-too-large"></a>Jeton de continuation trop volumineux

#### <a name="cause"></a>Cause :
La requête incorrecte 400 se produit sur les opérations de requête où le jeton de continuation est utilisé si le jeton de continuation est devenu trop volumineux ou si des requêtes différentes ont des tailles de jeton de continuation différentes.
    
#### <a name="solution"></a>Solution :
1. Suivez les instructions des articles contenant des conseils en termes de performances [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) ou [.NET v2](performance-tips.md). Convertissez l’application pour qu’elle utilise le mode de connexion directe avec le protocole TCP. Le mode de connexion directe avec le protocole TCP n’a pas de restriction de taille d’en-tête comme le protocole HTTP, ce qui évite ce problème. 
1. Si le mode de connexion directe avec le protocole TCP n’est pas une option pour votre charge de travail, définissez l’option `ResponseContinuationTokenLimitInKb`. Vous pouvez trouver cette option dans `FeedOptions` pour la version v2 ou dans `QueryRequestOptions` pour la version v3.

## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-dot-net-sdk.md) les problèmes lors de l’utilisation du SDK .NET Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) et [.NET v2](performance-tips.md).
