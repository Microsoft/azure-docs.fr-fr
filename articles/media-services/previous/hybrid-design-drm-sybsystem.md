---
title: Conception de modèles hybrides de sous-systèmes de DRM à l’aide d’Azure Media Services | Microsoft Docs
description: Cette rubrique explique comment concevoir des modèles hybrides de sous-systèmes de gestion des droits numériques (DRM) à l’aide d’Azure Media Services.
services: media-services
documentationcenter: ''
author: willzhan
manager: femila
editor: ''
ms.assetid: 18213fc1-74f5-4074-a32b-02846fe90601
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/31/2020
ms.author: willzhan
ms.reviewer: juliako
ms.openlocfilehash: 1bf01e98e418edfe8f15043d16a199b276b71020
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89266899"
---
# <a name="hybrid-design-of-drm-subsystems"></a>Conception de modèles hybrides de sous-systèmes de DRM

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Cette rubrique explique comment concevoir des modèles hybrides de sous-systèmes de gestion des droits numériques (DRM) à l’aide d’Azure Media Services.

## <a name="overview"></a>Vue d’ensemble

Azure Media Services assure la prise en charge des trois systèmes de DRM suivants :

* PlayReady
* Widevine (Modular)
* FairPlay

La prise en charge DRM inclut le chiffrement (chiffrement dynamique) et la remise de licence DRM, Azure Media Player prenant en charge les trois systèmes de DRM en tant que Kit de développement logiciel (SDK) de lecteur de navigateur.

Pour un examen détaillé de la conception et de la mise en œuvre d’un sous-système de DRM/CENC, consultez l’article [CENC avec Multi-DRM et contrôle d’accès](media-services-cenc-with-multidrm-access-control.md).

Même si nous offrons une prise en charge complète des trois systèmes de DRM, il arrive que des clients aient besoin d’utiliser différents composants de leurs propres sous-systèmes/infrastructure en plus d’Azure Media Services pour créer un sous-système de DRM hybride.

Voici quelques-unes des questions que les clients nous posent souvent :

* « Puis-je utiliser mes propres serveurs de licences DRM ? » (Dans ce cas, les clients ont investi dans une batterie de serveurs de licences DRM avec une logique métier incorporée.)
* « Puis-je utiliser votre service de remise de licence DRM sans héberger de contenu dans Azure Media Services ? »

## <a name="modularity-of-the-ams-drm-platform"></a>Modularité de la plateforme DRM d’AMS

Partie intégrante d’une plateforme vidéo cloud complète, le système de DRM d’Azure Media Services a été conçu dans un souci de flexibilité et de modularité. Vous pouvez utiliser Azure Media Services avec n’importe laquelle des différentes combinaisons décrites dans le tableau ci-dessous (une explication de la notation utilisée est donnée plus bas). 

|**Hébergement et origine du contenu**|**Chiffrement du contenu**|**Remise de licence DRM**|
|---|---|---|
|AMS|AMS|AMS|
|AMS|AMS|Tiers|
|AMS|Tiers|AMS|
|AMS|Tiers|Tiers|
|Tiers|Tiers|AMS|

### <a name="content-hosting--origin"></a>Hébergement et origine du contenu

* AMS : la ressource vidéo est hébergée dans AMS et la diffusion en continu se fait via les points de terminaison de streaming d’AMS (mais pas nécessairement l’empaquetage dynamique).
* Tiers : la vidéo est hébergée et diffusée sur une plateforme de streaming tierce en dehors d’AMS.

### <a name="content-encryption"></a>Chiffrement du contenu

* AMS : le chiffrement du contenu est effectué dynamiquement/à la demande à l’aide du chiffrement dynamique d’AMS.
* Tiers : le chiffrement du contenu est effectué en dehors d’AMS à l’aide d’un workflow de prétraitement.

### <a name="drm-license-delivery"></a>Remise de licence DRM

* AMS : la licence DRM est distribuée par le service de remise de licence d’AMS.
* Tiers : la licence DRM est distribuée par un serveur de licences DRM tiers en dehors d’AMS.

## <a name="configure-based-on-your-hybrid-scenario"></a>Configuration en fonction de votre scénario hybride

### <a name="content-key"></a>Clé de contenu

En configurant une clé de contenu, vous pouvez contrôler les attributs suivants du chiffrement dynamique et du service de remise de licence d’AMS :

* La clé de contenu utilisée pour le chiffrement DRM dynamique.
* Le contenu de la licence DRM que les services de remise de licence doivent distribuer : droits, clé de contenu et restrictions.
* Le type de **restriction de la stratégie d’autorisation de clé de contenu** : restriction ouverte, IP ou par jeton.
* Si le type de restriction **par jeton** est utilisé pour la **stratégie d’autorisation de clé contenu**, la **restriction de la stratégie d’autorisation de clé de contenu** doit être respectée pour qu’une licence soit émise.

### <a name="asset-delivery-policy"></a>Stratégie de remise d’élément multimédia

En configurant une stratégie de remise de ressources, vous pouvez contrôler les attributs suivants utilisés par l’empaquetage dynamique d’AMS et le chiffrement dynamique d’un point de terminaison de streaming d’AMS :

* L’association d’un protocole de diffusion en continu et d’un chiffrement DRM, telle que DASH sous CENC (PlayReady et Widevine), la diffusion en continu lisse sous PlayReady, ou encore HLS sous Widevine ou PlayReady.
* Les URL de remise de licence par défaut/incorporées pour chacun des systèmes de DRM impliqués.
* Si les URL d’acquisition de licence (LA_URL) dans un fichier DASH MPD ou une playlist HLS contiennent une chaîne de requête d’ID de clé (KID) pour Widevine et FairPlay, respectivement.

## <a name="scenarios-and-samples"></a>Scénarios et exemples

Sur la base des explications de la section précédente, les cinq scénarios hybrides utilisent différentes combinaisons **clé de contenu**-**stratégie de remise de ressources** (les exemples mentionnés dans la dernière colonne suivent le tableau) :

|**Hébergement et origine du contenu**|**Chiffrement DRM**|**Remise de licence DRM**|**Configuration de la clé de contenu**|**Configurer une stratégie de distribution d’éléments multimédias**|**Exemple**|
|---|---|---|---|---|---|
|AMS|AMS|AMS|Oui|Oui|Exemple 1|
|AMS|AMS|Tiers|Oui|Oui|Exemple 2|
|AMS|Tiers|AMS|Oui|Non|Exemple 3|
|AMS|Tiers|Extérieur|Non|Non|Exemple 4|
|Tiers|Tiers|AMS|Oui|Non|    

Dans les exemples, la protection PlayReady fonctionne à la fois pour DASH et la diffusion en continu lisse. Les URL de vidéos ci-dessous sont des URL de diffusion en continu lisse. Pour obtenir l’URL DASH correspondante, il vous suffit d’ajouter « (format=mpd-time-csf) ». Vous pouvez utiliser le lecteur [Azure Media Test](https://aka.ms/amtest) pour effectuer des tests dans un navigateur. Il vous permet de configurer le protocole de diffusion en continu à utiliser, sous la technologie de votre choix. IE11 et Microsoft Edge sur Windows 10 prennent en charge PlayReady via EME. Pour plus d’informations, consultez [ce billet de blog sur l’outil de test](./offline-playready-streaming-windows-10.md).

### <a name="sample-1"></a>Exemple 1

* URL de la source (base) : `https://willzhanmswest.streaming.mediaservices.windows.net/1efbd6bb-1e66-4e53-88c3-f7e5657a9bbd/RussianWaltz.ism/manifest` 
* PlayReady LA_URL (DASH et diffusion en continu lisse) : `https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/` 
* Widevine LA_URL (DASH) : `https://willzhanmswest.keydelivery.mediaservices.windows.net/Widevine/?kid=78de73ae-6d0f-470a-8f13-5c91f7c4` 
* FairPlay LA_URL (HLS) : `https://willzhanmswest.keydelivery.mediaservices.windows.net/FairPlay/?kid=ba7e8fb0-ee22-4291-9654-6222ac611bd8` 

### <a name="sample-2"></a>Exemple 2

* URL de la source (base) : https://willzhanmswest.streaming.mediaservices.windows.net/1a670626-4515-49ee-9e7f-cd50853e41d8/Microsoft_HoloLens_TransformYourWorld_816p23.ism/Manifest 
* PlayReady LA_URL (DASH et diffusion en continu lisse) : `http://willzhan12.cloudapp.net/PlayReady/RightsManager.asmx` 

### <a name="sample-3"></a>Exemple 3

* URL de la source : https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500.ism/manifest 
* PlayReady LA_URL (DASH et diffusion en continu lisse) : `https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/` 

### <a name="sample-4"></a>Exemple 4

* URL de la source : https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500.ism/manifest 
* PlayReady LA_URL (DASH et diffusion en continu lisse) : `https://willzhan12.cloudapp.net/playready/rightsmanager.asmx` 

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="summary"></a>Résumé

En résumé, les composants DRM d’Azure Media Services sont flexibles, vous pouvez les utiliser dans un scénario hybride en configurant correctement la clé de contenu et la stratégie de remise de ressources, comme indiqué dans cette rubrique.

## <a name="next-steps"></a>Étapes suivantes
Afficher les parcours d’apprentissage de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]