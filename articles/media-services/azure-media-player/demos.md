---
title: Démonstrations du Lecteur multimédia Azure
description: Cette page contient une liste de liens vers des démos du Lecteur multimédia Azure.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: article
ms.date: 04/24/2020
ms.openlocfilehash: 584748b23f526e6f03b543b8298927e3f202f743
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82138912"
---
# <a name="azure-media-player-demos"></a>Démonstrations du Lecteur multimédia Azure

Voici une liste de liens vers des démos du Lecteur multimédia Azure. Vous pouvez télécharger tous les [Exemples du Lecteur multimédia Azure](https://github.com/Azure-Samples/azure-media-player-samples) à partir de GitHub.

## <a name="demo-listing"></a>Liste des démos

| Nom d’exemple | Programmatique via JavaScript | Statique via un élément vidéo HTML5 | Description |
| ------------|----------------------------|-------------------------------------|--------------|
| De base |
| Définir la source | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_setsource.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_setsource.html) |Lire du contenu non protégé.|
| Fonctionnalités |
| Insertion de publicité VOD - VAST | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_vast_ads_vod.html) | N/A | Insérer des publicités VAST avant, pendant et après déroulement dans une ressource VOD. |
| Vitesse de lecture | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_playback_speed.html)| N/A | Permet de contrôler la vitesse de lecture de la vidéo. |
| Apparence de flush AMP | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_flush_skin.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_flush_skin.html) | Active une nouvelle apparence AMP. **Remarque :** Le flush AMP n’est pris en charge que dans les versions d’AMP 2.1.0+ |
| Légendes et sous-titres | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_webvtt.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_webvtt.html) | Lecture avec des sous-titres WebVTT.
| Légendes CEA 708 en direct | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_live_captions.html) | N/A | Lecture avec les légendes CEA 708 entrantes en direct alignées à gauche. |
| Diffusion en continu avec une solution de secours progressive | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_progressiveFallback.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_progressiveFallback.html) | Configuration de base de la lecture adaptative avec solution de secours progressive si la diffusion en continu n’est pas prise en charge sur la plateforme. |
| Vidéo progressive MP4 | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_progressiveVideo.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_progressiveVideo.html) | Lecture d’audio progressif MP4. |
| Audio progressif MP3 | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_progressiveAudio.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_progressiveAudio.html) | Lecture d’audio progressif MP3. |
| DD+ | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_dolbyDigitalPlus.html) | N/A | Lecture de contenu avec audio DD+. |
| Options |
| Profil heuristique | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_heuristicsProfile.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_heuristicsProfile.html) | Modification du profil heuristique |
| Localisation | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_localization.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_localization.html) |
Définition de la localisation |
| Menu Pistes audio | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_multiAudio.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_multiAudio.html) |
Options montrant comment afficher le menu des pistes audio sur l’apparence par défaut. |
| Touches d’accès rapide | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_hotKeys.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_hotKeys.html) | Cet exemple montre comment configurer les touches d’accès rapide activées dans le lecteur |
| Événements, journalisation et diagnostics |
| Inscrire des événements | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_registerEvents.html) | N/A | Lice avec les écouteurs d’événements. |
| Journalisation | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_logging.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_logging.html) | Activation de la journalisation détaillée sur la console. |
| Diagnostics | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_diagnostics.html) | N/A | Obtention des données de diagnostic. Cet exemple ne fonctionne que sur certaines technologies. |
| AES |
| Aucun jeton AES | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_aes_notoken.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_aes_notoken.html) | Lecture de contenu AES sans jeton. |
| Jeton AES | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_aes_token.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_aes_token.html) | Lecture de contenu AES avec jeton. |
| Simulation de proxy HLS AES | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_aes_token_withHLSProxy.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_aes_token_withHLSProxy.html) | Lecture de contenu AES avec jeton, en affichant un proxy pour HLS afin que ce jeton puisse être utilisé avec des appareils iOS. |
| Jeton AES forçage flash | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_aes_token_forceFlash.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_aes_token_forceFlash.html) | Lecture de contenu AES avec jeton, en forçant la technologie flashSS. |
| DRM |
| Multi-DRM avec PlayReady, Widevine et FairPlay | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_multiDRM_PlayReadyWidevineFairPlay_notoken.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_multiDRM_PlayReadyWidevineFairPlay_notoken.html) | Lecture de contenu DRM sans jeton, avec les en-têtes PlayReady, Widevine et FairPlay. |
| PlayReady sans jeton | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_playready_notoken.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_playready_notoken.html) | Lecture de contenu PlayReady sans jeton. |
| PlayReady sans jeton forçage Silverlight | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_playready_notoken_forceSilverlight.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_playready_notoken_forceSilverlight.html) | Lecture de contenu PlayReady sans jeton, en forçant la technologie silverlightSS. |
| PlayReady avec jeton | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_playready_token.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_playready_token.html) | Lecture de contenu PlayReady avec jeton. |
| PlayReady avec jeton forçage Silverlight | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_playready_token_forceSilverlight.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_playready_token_forceSilverlight.html) | Lecture de contenu PlayReady avec jeton, en forçant la technologie silverlightSS. |
| Protocole et technologie |
| Modifier techOrder | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_techOrder.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_techOrder.html) | Modification de l’ordre des technologies |
| Forcer MPEG-DASH uniquement dans UrlRewriter | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_forceDash.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_forceDash.html) | Lecture de contenu non protégé uniquement à l’aide du protocole DASH. |
| Exclure MPEG-DASH dans UrlRewriter | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_forceNoDash.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_forceNoDash.html) | Lecture de contenu non protégé uniquement à l’aide des protocoles Smooth et TLS. |
| Stratégie de remise multiple | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_multipleDeliveryPolicy.html) | [Statique](https://amp.azure.net/libs/amp/latest/samples/videotag_multipleDeliveryPolicy.html) | Définition de la source avec un contenu ayant plusieurs stratégies de remise à partir d’Azure Media Services |
| Sélection par programmation |
| Sélectionner la piste de texte | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_selectTextTrack.html) | N/A | Sélection d’une piste WebVTT dans la liste des pistes. |
| Sélectionner la vitesse de transmission | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_selectBitrate.html) | N/A | Sélection d’une vitesse de transmission dans la liste des vitesses de transmission. Cet exemple ne fonctionne que sur certaines technologies. |
| Sélectionner un flux audio | [Dynamique](https://amp.azure.net/libs/amp/latest/samples/dynamic_selectAudioStream.html) | N/A | Sélection d’un flux audio dans la liste des flux audio disponibles. Cet exemple ne fonctionne que sur certaines technologies. |

## <a name="next-steps"></a>Étapes suivantes

<!---Some context for the following links goes here--->
- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)