---
title: Publier du contenu dans le portail Azure | Microsoft Docs
description: Ce didacticiel vous fournit les étapes à suivre pour la publication de votre contenu dans le portail Azure.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 92c364eb-5a5f-4f4e-8816-b162c031bb40
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 9f86f7a0a7eaef7c0dc244a2e089596de3779eae
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92018954"
---
# <a name="publish-content-in-the-azure-portal"></a>Publier du contenu dans le portail Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [Portail](media-services-portal-publish.md)
> * [.NET](media-services-deliver-streaming-content.md)
> * [REST](media-services-rest-deliver-streaming-content.md)
> 
> 

## <a name="overview"></a>Vue d’ensemble
> [!NOTE]
> Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Pour plus d’informations, consultez la page [Version d’évaluation gratuite d’Azure](https://azure.microsoft.com/pricing/free-trial/). 
> 
> 

Pour fournir aux utilisateurs une URL pouvant être utilisée pour diffuser en continu ou télécharger votre contenu, vous devez d’abord publier votre actif multimédia en créant un localisateur. Les localisateurs permettent d’accéder aux fichiers d’actifs multimédias. Azure Media Services prend en charge deux types de localisateurs : 

* **Localisateurs de streaming (OnDemandOrigin)** . Les localisateurs de streaming sont utilisés pour le streaming adaptatif. Parmi les exemples de streaming adaptatif, nous pouvons citer Apple HTTP Live Streaming (HLS), Microsoft Smooth Streaming et Dynamic Adaptive Streaming sur HTTP (DASH, aussi appelé MPEG-DASH). Pour créer un localisateur de streaming, votre actif multimédia doit contenir un fichier .ism. Par exemple : `http://amstest.streaming.mediaservices.windows.net/61b3da1d-96c7-489e-bd21-c5f8a7494b03/scott.ism/manifest`.
* **Localisateurs progressifs (signature d’accès partagé)** . Les localisateurs progressifs sont utilisés pour diffuser des vidéos par téléchargement progressif.

Pour créer une URL de streaming HLS, ajoutez *(format=m3u8-aapl)* à l’URL :

`{streaming endpoint name-media services account name}/{locator ID}/{file name}.ism/Manifest(format=m3u8-aapl)`

Pour générer une URL de streaming afin de lire les actifs multimédias Smooth Streaming, utilisez le format d’URL suivant :

`{streaming endpoint name-media services account name}/{locator ID}/{file name}.ism/Manifest`

Pour créer une URL de streaming MPEG DASH, ajoutez *(format=mpd-time-csf)* à l’URL :

`{streaming endpoint name-media services account name}/{locator ID}/{file name}.ism/Manifest(format=mpd-time-csf)`

Une URL de signature d’accès partagé a le format suivant :

`{blob container name}/{asset name}/{file name}/{shared access signature}`

Pour plus d’informations, consultez [Vue d’ensemble de la fourniture de contenu](media-services-deliver-content-overview.md).

> [!NOTE]
> Les localisateurs qui ont été créés dans le portail Azure avant mars 2015 expirent au bout de deux ans.  
> 
> 

Pour mettre à jour la date d’expiration d’un localisateur, vous pouvez utiliser une [API REST](/rest/api/media/operations/locator#update_a_locator) ou une [API .NET](/dotnet/api/microsoft.windowsazure.mediaservices.client.ilocator). 

> [!NOTE]
> Lorsque vous mettez à jour la date d’expiration d’un localisateur de signature d’accès partagé, l’URL est modifiée.

### <a name="to-use-the-portal-to-publish-an-asset"></a>Pour publier un élément multimédia à l’aide du portail
1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre compte Azure Media Services.
2. Sélectionnez **Paramètres** > **Éléments multimédias**. Sélectionnez l’élément que vous souhaitez publier.
3. Cliquez sur le bouton **Publier**.
4. Sélectionnez le type de localisateur.
5. Sélectionnez **Ajouter**.
   
    ![Publier la vidéo](./media/media-services-portal-vod-get-started/media-services-publish1.png)

L’URL est ajoutée à la liste des **URL publiées**.

## <a name="play-content-in-the-portal"></a>Lecture du contenu dans le portail
Vous pouvez tester votre vidéo sur un lecteur de contenu dans le portail Azure.

Sélectionnez la vidéo, puis cliquez sur le bouton **Lecture**.

![Lire la vidéo dans le portail Azure](./media/media-services-portal-vod-get-started/media-services-play.png)

Certaines considérations s’appliquent :

* Vérifiez que la vidéo a été publiée.
* Le lecteur multimédia du portail Azure effectue la lecture à partir du point de terminaison de streaming par défaut. Si vous souhaitez lire à partir d’un autre point de terminaison de streaming que celui par défaut, sélectionnez et copiez l’URL dans un autre lecteur. Par exemple, vous pouvez tester votre vidéo avec le lecteur [Azure Media Player](https://aka.ms/azuremediaplayer).
* Le point de terminaison de streaming à partir duquel vous diffusez en continu doit être en cours d’exécution.  

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Étapes suivantes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]