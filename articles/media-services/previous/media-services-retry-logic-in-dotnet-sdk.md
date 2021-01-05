---
title: Logique de nouvelle tentative dans le Kit de développement logiciel (SDK) Media Services pour .NET | Microsoft Docs
description: La rubrique fournit une vue d’ensemble de la logique de nouvelle tentative dans le Kit de développement logiciel (SDK) pour .NET.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 527b61a6-c862-4bd8-bcbc-b9aea1ffdee3
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 144db6a5ceaf56a35d3ce11dd54e1dfb4c97d7e3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89264111"
---
# <a name="retry-logic-in-the-media-services-sdk-for-net"></a>Logique de nouvelle tentative dans le Kit de développement logiciel (SDK) Media Services pour .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Lorsque vous utilisez les services Microsoft Azure, des erreurs temporaires peuvent se produire. Si une erreur temporaire se produit, dans la plupart des cas, après plusieurs tentatives, l’opération aboutit. Le Kit de développement logiciel (SDK) Media Services pour .NET implémente la logique de nouvelle tentative pour gérer des défaillances temporaires liées à des exceptions et erreurs générées par des requêtes web, l’exécution de requêtes, l’enregistrement de modifications et des opérations de stockage.  Par défaut, le Kit de développement logiciel (SDK) Media Services pour .NET effectue quatre nouvelles tentatives avant de lever une nouvelle exception pour votre application. Le code de votre application doit ensuite gérer cette exception correctement.  

 Voici une brève indication des stratégies Web Request, Storage, Query et SaveChanges :  

* La stratégie Storage est utilisée pour les opérations de stockage d’objets blob (chargements ou téléchargement de fichiers de ressources).  
* La stratégie Web Request est utilisée pour les demandes web génériques (par exemple, pour obtenir un jeton d’authentification et résoudre le point de terminaison du cluster de l’utilisateur).  
* La stratégie Query permet d’interroger des entités à partir de REST (par exemple, mediaContext.Assets.Where(...)).  
* La stratégie SaveChanges permet d’effectuer toute opération qui modifie des données au sein du service (par exemple, création d’une entité, mise à jour d’une entité, appel d’une fonction de service pour une opération).  
  
  Cette rubrique répertorie les types d’exceptions et codes d’erreur qui sont gérés par le Kit de développement logiciel (SDK) Media Services pour la logique de nouvelle tentative .NET.  

## <a name="exception-types"></a>Types d'exceptions
Le tableau suivant décrit les exceptions que le Kit de développement logiciel (SDK) Media Services pour .NET gère ou ne gère pas pour certaines opérations susceptibles de provoquer des défaillances temporaires.  

| Exception | Web Request | Stockage | Requête | SaveChanges |
| --- | --- | --- | --- | --- |
| WebException<br/>Pour plus d’informations, voir la section [Codes d’état WebException](media-services-retry-logic-in-dotnet-sdk.md#WebExceptionStatus). |Oui |Oui |Oui |Oui |
| DataServiceClientException<br/> Pour plus d’informations, voir [Codes d’état d’erreur HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |Non |Oui |Oui |Oui |
| DataServiceQueryException<br/> Pour plus d’informations, voir [Codes d’état d’erreur HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |Non |Oui |Oui |Oui |
| DataServiceRequestException<br/> Pour plus d’informations, voir [Codes d’état d’erreur HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |Non |Oui |Oui |Oui |
| DataServiceTransportException |Non |Non |Oui |Oui |
| TimeoutException |Oui |Oui |Oui |Non |
| SocketException |Oui |Oui |Oui |Oui |
| StorageException |Non |Oui |Non |Non |
| IOException |Non |Oui |Non |Non |

### <a name="webexception-status-codes"></a><a name="WebExceptionStatus"></a> Codes d’état WebException
Le tableau suivant présente les codes d’erreur WebException pour lesquels la logique de nouvelle tentative est implémentée. L’énumération [WebExceptionStatus](/dotnet/api/system.net.webexceptionstatus?view=netcore-3.1) définit les codes d’état.  

| Statut | Web Request | Stockage | Requête | SaveChanges |
| --- | --- | --- | --- | --- |
| ConnectFailure |Oui |Oui |Oui |Oui |
| NameResolutionFailure |Oui |Oui |Oui |Oui |
| ProxyNameResolutionFailure |Oui |Oui |Oui |Oui |
| SendFailure |Oui |Oui |Oui |Oui |
| PipelineFailure |Oui |Oui |Oui |Non |
| ConnectionClosed |Oui |Oui |Oui |Non |
| KeepAliveFailure |Oui |Oui |Oui |Non |
| UnknownError |Oui |Oui |Oui |Non |
| ReceiveFailure |Oui |Oui |Oui |Non |
| RequestCanceled |Oui |Oui |Oui |Non |
| Délai d'expiration |Oui |Oui |Oui |Non |
| ProtocolError <br/>La nouvelle tentative sur ProtocolError est contrôlée par la gestion des codes d’état HTTP. Pour plus d’informations, voir [Codes d’état d’erreur HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |Oui |Oui |Oui |Oui |

### <a name="http-error-status-codes"></a><a name="HTTPStatusCode"></a> Codes d’état d’erreur HTTP
Lorsque les opérations Query et SaveChanges lèvent les exceptions DataServiceClientException, DataServiceQueryException ou DataServiceQueryException, le code d’état d’erreur HTTP est retourné dans la propriété StatusCode.  Le tableau suivant présente les codes d’erreur pour lesquels la logique de nouvelle tentative est implémentée.  

| Statut | Web Request | Stockage | Requête | SaveChanges |
| --- | --- | --- | --- | --- |
| 401 |Non |Oui |Non |Non |
| 403 |Non |Oui<br/>Gestion des nouvelles tentatives avec attentes plus longues. |Non |Non |
| 408 |Oui |Oui |Oui |Oui |
| 429 |Oui |Oui |Oui |Oui |
| 500 |Oui |Oui |Oui |Non |
| 502 |Oui |Oui |Oui |Non |
| 503 |Oui |Oui |Oui |Oui |
| 504 |Oui |Oui |Oui |Non |

Pour l’implémentation réelle du Kit de développement logiciel (SDK) Media Services pour la logique de nouvelle tentative .NET, voir [azure-sdk-for-media-services](https://github.com/Azure/azure-sdk-for-media-services/tree/dev/src/net/Client/TransientFaultHandling).

## <a name="next-steps"></a>Étapes suivantes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
