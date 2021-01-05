---
title: Gérer les points de terminaison de streaming avec le Portail Azure | Microsoft Docs
description: Cet article montre comment gérer les points de terminaison de streaming avec le portail Azure.
services: media-services
documentationcenter: ''
author: Juliako
writer: juliako
manager: femila
editor: ''
ms.assetid: bb1aca25-d23a-4520-8c45-44ef3ecd5371
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 496c5de8f9ead0773dfef7e65051344645ffa9de
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041967"
---
# <a name="manage-streaming-endpoints-with-the-azure-portal"></a>Gérer les points de terminaison de streaming avec le Portail Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Cet article explique comment utiliser le portail Azure pour gérer les points de terminaison de streaming. 

>[!NOTE]
>Consultez l’article de [présentation](media-services-streaming-endpoints-overview.md). 

Pour plus d’informations sur la mise à l’échelle du point de terminaison de streaming, consultez [cet](media-services-portal-scale-streaming-endpoints.md) article.

## <a name="start-managing-streaming-endpoints"></a>Commencer à gérer des points de terminaison de streaming 

Pour commencer à gérer les points de terminaison de streaming de votre compte, procédez comme suit.

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre compte Azure Media Services.
2. Dans le panneau **Paramètres** , sélectionnez **Points de terminaison de streaming**.
   
    ![Capture d’écran montrant la page « Media Services » avec des « points de terminaison de streaming » sélectionnés dans le panneau « Paramètres ».](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints1.png)

> [!NOTE]
> Vous êtes facturé uniquement lorsque votre point de terminaison de streaming est en cours d’exécution.

## <a name="adddelete-a-streaming-endpoint"></a>Ajouter/supprimer un point de terminaison de streaming

>[!NOTE]
>Il n’est pas possible de supprimer le point de terminaison de streaming par défaut.

Pour ajouter/supprimer un point de terminaison de streaming à l’aide du Portail Azure, procédez comme suit :

1. Pour ajouter un point de terminaison de streaming, cliquez sur le **+ Point de terminaison** en haut de la page. 

    Vous pouvez également disposer de plusieurs points de terminaison de streaming si vous prévoyez d’avoir différents CDN ou bien un CDN et un accès direct.

2. Pour supprimer un point de terminaison de streaming, appuyez sur le bouton **Supprimer** .      
3. Cliquez sur le bouton **Démarrer** pour démarrer le point de terminaison de streaming.
   
    ![Capture d’écran montrant l’action « Point de terminaison » sélectionnée, et le volet « Détails du point de terminaison de streaming » affiché.](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints2.png)


## <a name="configuring-the-streaming-endpoint"></a><a id="configure_streaming_endpoints"></a>Configuration du point de terminaison de diffusion en continu
Le point de terminaison de streaming vous permet de configurer les propriétés suivantes :

* Contrôle d’accès
* contrôle de cache
* Stratégies d’accès intersite

Pour plus d’informations sur ces propriétés, consultez la rubrique [StreamingEndpoint](/rest/api/media/operations/streamingendpoint).

>[!NOTE]
>Si le CDN est activé, l’accès IP n’est pas disponible. Il l’est uniquement lorsque vous ne disposez pas de CDN.

Vous pouvez configurer le point de terminaison de streaming en procédant comme suit :

1. Sélectionnez le point de terminaison de streaming que vous souhaitez configurer.
2. Cliquez sur **Settings**.

Vous trouverez une brève description des champs ci-dessous.

![Capture d’écran montrant l’action « Settings » sélectionnée pour le point de terminaison de streaming.](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints4.png)

1. Stratégie de cache maximale : permet de configurer la durée de vie du cache pour les ressources traitées par le biais de ce point de terminaison de streaming. Si aucune valeur n’est définie, la valeur par défaut est utilisée. Les valeurs par défaut peuvent également être définies directement dans Azure Storage. Si Azure CDN est activé pour le point de terminaison de streaming, ne définissez pas une valeur de stratégie de cache inférieure à 600 secondes.  
2. Adresses IP autorisées : permet de définir les adresses IP autorisées à se connecter au point de terminaison de streaming publié. Si aucune adresse IP n'est spécifiée, toutes les adresses IP peuvent se connecter. Les adresses IP peuvent être définies sous forme d’adresse IP unique (par exemple, « 10.0.0.1 »), de plage d’adresses IP constituée d’une adresse IP et d’un masque de sous-réseau CIDR (par exemple, « 10.0.0.1/22 ») ou de plage d’adresses IP constituée d’une adresse IP et d’un masque de sous-réseau au format décimal séparé par des points (par exemple, « 10.0.0.1(255.255.255.0) »).
3. Configuration de l’authentification de l’en-tête de signature Akamai : permet de spécifier la configuration de la demande d’authentification de l’en-tête de signature à partir de serveurs Akamai. L’expiration est au format UTC.

## <a name="scale-your-premium-streaming-endpoint"></a>Mise à l’échelle du point de terminaison de streaming Premium

Pour plus d’informations, consultez [cet](media-services-portal-scale-streaming-endpoints.md) article.

## <a name="enable-azure-cdn-integration"></a><a id="enable_cdn"></a>Activer l’intégration au CDN Azure

Lorsque vous créez un compte, l’intégration CDN Azure du point de terminaison de streaming par défaut est activée par défaut.

Si vous souhaitez activer/désactiver le CDN ultérieurement, votre point de terminaison de streaming doit avoir l’état **Arrêté**. Il peut s’écouler jusqu’à deux heures avant que l’intégration d’Azure CDN soit active et que les modifications soient actives sur tous les comptes POP CDN. Toutefois, vous pouvez démarrer votre point de terminaison de streaming et le flux sans interruptions à partir du point de terminaison de streaming. Une fois l’intégration terminée, le flux est émis à partir du CDN. Pendant la durée de l’approvisionnement, votre point de terminaison de streaming est en état de **démarrage** et vous pouvez observer une dégradation des performances.

L’intégration du CDN est activée dans tous les centres de données Azure, sauf dans les régions Gouvernement fédéral et Chine.

Une fois qu’elle est activée, la configuration **d’Access Control** , du **nom d’hôte personnalisé et de **l’authentification de signature Akamai** est désactivée.
 
> [!IMPORTANT]
> L’intégration d’Azure Media Services au CDN Azure est implémentée sur le **CDN Azure fourni par Verizon** pour les points de terminaison de streaming Standard. Les points de terminaison de streaming Premium peuvent être configurés à l’aide de l’ensemble des **fournisseurs et niveaux de tarification Azure CDN**. Pour plus d’informations sur les fonctionnalités du CDN Azure, consultez [Vue d’ensemble du réseau de distribution de contenu (CDN)](../../cdn/cdn-overview.md).
 
### <a name="additional-considerations"></a>Considérations supplémentaires

* Lorsque le CDN est activé pour un point de terminaison de diffusion en continu, les clients ne peuvent pas demander directement le contenu à partir de l’origine. Si vous avez besoin de tester votre contenu avec ou sans CDN, vous pouvez créer un autre point de terminaison de streaming qui n’est pas activé pour le CDN.
* Le nom d’hôte de votre point de terminaison reste le même une fois le CDN activé. Vous n’avez pas besoin d’apporter de modifications à votre flux de travail Media Services une fois le CDN activé. Par exemple, si le nom d’hôte de votre point de terminaison en continu est strasbourg.streaming.mediaservices.windows.net, après avoir activé le CDN, le même nom d’hôte est utilisé.
* Pour les nouveaux points de terminaison de streaming, vous pouvez simplement activer le CDN en créant un point de terminaison. Pour les points de terminaison de streaming existants, vous devez d’abord arrêter le point de terminaison, puis activer/désactiver le CDN.
* Le point de terminaison de streaming Standard ne peut être configuré qu’à l’aide du **fournisseur CDN Standard Verizon** par le biais du portail Azure Classic. Toutefois, vous pouvez activer d’autres fournisseurs Azure CDN à l’aide d’API REST.

## <a name="configure-cdn-profile"></a>Configurer le profil CDN

Vous pouvez configurer le profil CDN en sélectionnant le bouton **Gérer le CDN** en haut.

![point de terminaison de diffusion en continu](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints6.png)

## <a name="next-steps"></a>Étapes suivantes
Consultez les parcours d’apprentissage de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
