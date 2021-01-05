---
title: Migrer d’Azure Media Services v2 vers v3 | Microsoft Docs
description: Cet article décrit les changements qui ont été introduits dans Azure Media Services v3 et montre les différences entre les deux versions. Il fournit également des conseils de migration pour le passage de Media Services v2 à Media Services v3.
services: media-services
documentationcenter: na
author: IngridAtMicrosoft
manager: femila
editor: ''
tags: ''
keywords: azure media services, flux, diffusion, en direct, hors connexion
ms.service: media-services
ms.devlang: multiple
ms.topic: conceptual
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 0cdf4699f5dc00087845ee6ca0d24ad6493c320b
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92426873"
---
# <a name="migration-guidance-for-moving-from-media-services-v2-to-v3"></a>Conseils de migration pour le passage de Media Services v2 à Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

>Soyez notifié de la disponibilité des mises à jour sur cette page en faisant un copier-coller de cette URL : `https://docs.microsoft.com/api/search/rss?search=%22Migrate+from+Azure+Media+Services+v2+to+v3%22&locale=en-us` dans votre lecteur de flux RSS.

Cet article fournit des conseils pour la migration de Media Services v2 à Media Services v3.

Si vous avez un service vidéo développé aujourd’hui sur la base des [API héritées de Media Services v2](../previous/media-services-overview.md), vous devez prendre en compte les instructions et considérations suivantes avant d’opérer une migration vers les API v3. Les API v3 présentent un grand nombre d’avantages et de fonctionnalités nouvelles qui améliorent l’expérience de développement et les fonctionnalités de Media Services. Toutefois, comme mentionné dans la section [Problèmes connus](#known-issues) de cet article, il existe certaines limitations résultant de différences entre les versions d’API. Cette page sera actualisée à mesure que l’équipe Media Services apportera des améliorations continues aux API v3, et traitera des différences entre les versions. 

## <a name="prerequisites"></a>Prérequis

* Consultez [Media Services v2 par rapport à Media Services v3](media-services-v2-vs-v3.md)
* [!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="benefits-of-media-services-v3"></a>Avantages de Media Services v3
  
### <a name="api-is-more-approachable"></a>L’API est plus accessible

*  v3 est basée sur une surface d’API unifiée qui expose des fonctionnalités de gestion et de fonctionnement qui s’appuient sur Azure Resource Manager. Vous pouvez utiliser les modèles Azure Resource Manager pour créer et déployer des transformations, des points de terminaison de streaming, des événements en direct, etc.
* Document sur la [spécification OpenAPI (anciennement Swagger)](https://aka.ms/ams-v3-rest-sdk).
    Expose le schéma pour tous les composants de service, dont l’encodage basé sur un fichier.
* Kits de développement logiciel (SDK) disponibles pour [.NET](/dotnet/api/overview/azure/mediaservices/management), .NET Core, [Node.js](/javascript/api/overview/azure/mediaservices/management), [Python](/python/api/overview/azure/mediaservices/management), [Java](/java/api/overview/azure/mediaservices/management), [Go](https://aka.ms/ams-v3-go-ref) et Ruby.
* Intégration d’[Azure CLI](/cli/azure/ams) pour la prise en charge de scripts simples.

### <a name="new-features"></a>Nouvelles fonctionnalités

* Pour le traitement de travaux basé sur un fichier, vous pouvez utiliser une URL HTTP(S) comme entrée.<br/>Vous n’avez pas besoin de contenu déjà stocké dans Azure, ou de créer des ressources.
* Introduit le concept de [transformations](transforms-jobs-concept.md) pour le traitement de travaux basé sur fichier. Vous pouvez utiliser une transformation pour créer des configurations réutilisables, créer des modèles Azure Resource Manager, et isoler des paramètres de traitement entre plusieurs clients ou locataires.
* Un actif multimédia peut avoir plusieurs [localisateurs de streaming](streaming-locators-concept.md), chacun avec des paramètres différents d’[empaquetage dynamique](dynamic-packaging-overview.md) et de chiffrement dynamique.
* La [protection du contenu](content-key-policy-concept.md) prend en charge les fonctionnalités à plusieurs clés.
* Vous pouvez diffuser des événements en direct d’une durée maximale de 24 heures quand vous utilisez Media Services pour transcoder un flux de contribution à une seule vitesse de transmission en un flux de sortie à vitesse de transmission multiple.
* Nouvelle prise en charge du streaming en direct à faible latence sur des événements en direct. Pour plus d’informations, consultez [latence](live-event-latency.md).
* L’aperçu des événements en direct prend en charge l’[empaquetage dynamique](dynamic-packaging-overview.md) et le chiffrement dynamique. Ceci permet la protection du contenu sur l’aperçu, ainsi que l’empaquetage DASH et HLS.
* La sortie en direct est plus simple à utiliser que l’entité Program dans les API v2. 
* Prise en charge améliorée de RTMP (stabilité accrue et meilleure prise en charge de l’encodeur source).
* Ingestion sécurisée RTMPS.<br/>Quand vous créez un événement en direct, vous obtenez 4 URL de réception. Les 4 URL d’ingestion sont presque identiques, ont le même jeton de streaming (AppId) ; seule la partie du numéro de port est différente. Il existe deux URL principales et de secours pour RTMPS.   
* Vous disposez d'un contrôle d'accès en fonction du rôle Azure (Azure RBAC) sur vos entités. 

## <a name="known-issues"></a>Problèmes connus

*  Vous pouvez actuellement utiliser le portail [Azure](https://portal.azure.com/) pour :

    * Gérer les [événements en direct](live-events-outputs-concept.md) Media Services v3. 
    * Consulter (et non gérer) les [actifs multimédias](assets-concept.md) v3. 
    * [Obtenir des informations sur l’accès aux API](./access-api-howto.md). 

    Pour toutes les autres tâches de gestion (par exemple, les [transformations et travaux](transforms-jobs-concept.md) et la [protection de contenu](content-protection-overview.md)), utilisez l’[API REST](/rest/api/media/), l’interface [CLI](/cli/azure/ams) ou l’un des [kits de développement logiciel (SDK)](media-services-apis-overview.md#sdks) pris en charge.
* Vous devez provisionner des unités réservées Multimédia (MRU) dans votre compte afin de contrôler la concurrence et les performances de vos travaux, en particulier ceux impliquant une analyse audio ou vidéo. Pour plus d’informations, consultez [Mise à l’échelle du traitement multimédia](../previous/media-services-scale-media-processing-overview.md). Vous pouvez gérer les MRU à l’aide de [CLI 2.0 pour Media Services v3](media-reserved-units-cli-how-to.md), du [portail Azure](../previous/media-services-portal-scale-media-processing.md) ou des [API v2](../previous/media-services-dotnet-encoding-units.md). Vous devez provisionner des unités réservées Multimédia, indépendamment de la version (v2 ou v3) des API Media Services que vous utilisez.
* Des entités Media Services créées avec l’API v3 ne peuvent pas être gérées avec l’API v2.  
* Toutes les entités de l’API V2 ne s’affichent pas automatiquement dans l’API V3.  Voici quelques exemples d’entités dans les deux versions et qui sont incompatibles :  
    * Les travaux et tâches créés dans v2 n’apparaissent pas dans v3, car elles ne sont pas associées à une transformation. Il est recommandé de basculer vers des transformations et travaux v3. Il y aura une période de temps relativement courte du basculement durant laquelle il sera nécessaire de surveiller les travaux v2 en cours.
    * Les canaux et programmes créés avec v2 (qui sont mappés à des événements et sorties en direct dans v3) ne peuvent plus être gérés avec v3. Il est recommandé de basculer vers des événements et sorties en direct v3 à un point d’arrêt approprié du canal.<br/>Actuellement, vous ne pouvez pas migrer en continu des canaux en cours d’exécution.  

> [!NOTE]
> Cette page sera actualisée à mesure que l’équipe Media Services apportera des améliorations continues aux API v3, et traitera des différences entre les versions.

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

[Tutoriel : Encoder un fichier distant basé sur une URL et diffuser la vidéo en continu – .NET](stream-files-dotnet-quickstart.md)