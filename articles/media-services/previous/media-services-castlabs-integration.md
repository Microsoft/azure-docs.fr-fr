---
title: Utilisation de castLabs pour fournir des licences Widevine à Azure Media Services | Microsoft Docs
description: Cet article décrit comment vous pouvez utiliser Azure Media Services (AMS) pour fournir un flux chiffré dynamiquement par AMS avec des DRM PlayReady et Widevine.
services: media-services
documentationcenter: ''
author: Mingfeiy
manager: femila
editor: ''
ms.assetid: 2a9a408a-a995-49e1-8d8f-ac5b51e17d40
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: Juliako
ms.reviewer: willzhan
ms.openlocfilehash: 17b54b2adb21419de61d2309752987f6e4a48e41
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89264485"
---
# <a name="using-castlabs-to-deliver-widevine-licenses-to-azure-media-services"></a>Utilisation de castLabs pour fournir des licences Widevine à Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
> [!div class="op_single_selector"]
> * [Axinom](media-services-axinom-integration.md)
> * [castLabs](media-services-castlabs-integration.md)
> 
> 

## <a name="overview"></a>Vue d’ensemble

Cet article décrit comment vous pouvez utiliser Azure Media Services (AMS) pour fournir un flux chiffré dynamiquement par AMS avec des DRM PlayReady et Widevine. La licence PlayReady provient du serveur de licences Media Services PlayReady et la licence Widevine est délivrée par le serveur de licences **castLabs** .

Pour lire un contenu en streaming protégé par CENC (PlayReady et/ou Widevine), vous pouvez utiliser le [Lecteur multimédia Azure](https://aka.ms/azuremediaplayer). Pour plus d’informations, consultez le [document AMP](https://amp.azure.net/libs/amp/latest/docs/) .

Le diagramme suivant représente une architecture qui intègre Azure Media Services et castLabs.

![intégration](./media/media-services-castlabs-integration/media-services-castlabs-integration.png)

## <a name="typical-system-set-up"></a>Configuration système classique

* Le contenu multimédia est stocké dans AMS.
* Les ID des clés de contenu sont stockées à la fois dans castLabs et AMS.
* castLabs et AMS intègrent une authentification par jeton. Les sections suivantes décrivent les jetons d'authentification. 
* Quand un client demande à diffuser en continu la vidéo, le contenu est chiffré dynamiquement par **chiffrement commun** (CENC) et empaqueté dynamiquement par AMS en Smooth Streaming ou DASH. Nous offrons également un chiffrement de flux élémentaire PlayReady M2TS pour le protocole de diffusion en continu HLS.
* La licence PlayReady est récupérée auprès du serveur de licences AMS et licence Widevine est récupérée auprès du serveur de licences castLabs. 
* Media Player détermine automatiquement quelles licences récupérer en fonction de la capacité de la plateforme client. 

## <a name="authentication-token-generation-for-getting-a-license"></a>Génération de jetons d'authentification pour l'obtention d'une licence

AMS et castLabs prennent en charge le format de jeton JWT (jeton web JSON) utilisé pour autoriser une licence. 

### <a name="jwt-token-in-ams"></a>Jeton JWT dans AMS

Le tableau suivant décrit le jeton JWT dans AMS. 

| Émetteur | Chaîne d'émetteur issue du service de jeton sécurisé (STS) choisi |
| --- | --- |
| Public visé |Chaîne de public issue du STS utilisé |
| Revendications |Ensemble de revendications |
| NotBefore |Début de validité du jeton |
| Expires |Fin de validité du jeton |
| SigningCredentials |Clé partagée entre le serveur de licences PlayReady, le serveur de licences castLabs et le STS. Elle peut être symétrique ou asymétrique. |

### <a name="jwt-token-in-castlabs"></a>Jeton JWT dans castLabs

Le tableau suivant décrit le jeton JWT dans castLabs. 

| Nom | Description |
| --- | --- |
| optData |Chaîne JSON contenant des informations sur vous. |
| crt |Chaîne JSON contenant des informations sur l'élément multimédia, ses informations de licence et ses droits de lecture. |
| iat |Date/heure actuelle de l'époque. |
| jti |Identificateur unique relatif à ce jeton (chaque jeton peut être utilisé une seule fois dans le système castLabs). |

## <a name="sample-solution-setup"></a>Exemple de configuration de solution

L' [exemple de solution](https://github.com/AzureMediaServicesSamples/CastlabsIntegration) se compose de deux projets :

* Une application console qui peut être utilisée pour définir des restrictions DRM sur un élément multimédia déjà ingéré, à la fois pour PlayReady et Widevine.
* Une application web qui délivre les jetons, qui peut être considérée comme une version TRÈS SIMPLIFIÉE d'un STS.

Pour utiliser l'application console :

1. Modifiez le fichier app.config pour configurer les informations d'identification AMS, les informations d'identification castLabs, la configuration du STS et la clé partagée.
2. Téléchargez un élément multimédia dans AMS.
3. Obtenez l'UUID de l'élément multimédia téléchargé et modifiez la ligne 32 dans le fichier Program.cs :
   
      var objIAsset = _context.Assets.Where(x => x.Id == "nb:cid:UUID:dac53a5d-1500-80bd-b864-f1e4b62594cf").FirstOrDefault();
4. Utilisez un AssetId pour nommer l'élément multimédia dans le système castLabs (ligne 44 dans le fichier Program.cs).
   
   Vous devez définir AssetId pour **castLabs**; il doit s'agir d'une chaîne alphanumérique unique.
5. Exécutez le programme.

Pour utiliser l'application web (STS) :

1. Modifiez le fichier web.config pour configurer l'ID du prestataire castlabs, le STS et la clé partagée.
2. Déployez-le sur des sites web Azure.
3. Accédez au site web.

## <a name="playing-back-a-video"></a>Lecture d'une vidéo

Pour lire une vidéo chiffrée par chiffrement commun (PlayReady et/ou Widevine), vous pouvez utiliser le [Lecteur multimédia Azure ](https://aka.ms/azuremediaplayer). Pendant l'exécution de l'application console, l'ID de la clé de contenu et l'URL du manifeste sont renvoyés.

1. Ouvrez un nouvel onglet et lancez votre STS : http://[nom_sts].azurewebsites.net/api/token/assetid/[id_élément_multimédia_castlabs]/contentkeyid/[id_clé_contenu].
2. Accédez à [Azure Media Player](https://aka.ms/azuremediaplayer).
3. Collez l'URL de diffusion en continu.
4. Cochez la case **Options avancées** .
5. Dans la liste déroulante **Protection** , sélectionnez PlayReady et/ou Widevine.
6. Collez le jeton que vous avez obtenu de votre STS dans la zone de texte Jeton. 
   
   Le serveur de licences castLab n'a pas besoin du préfixe « Bearer= » devant le jeton. Par conséquent, supprimez-le avant d'envoyer le jeton.
7. Mettez à jour le lecteur.
8. La vidéo doit pouvoir être lue.

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

