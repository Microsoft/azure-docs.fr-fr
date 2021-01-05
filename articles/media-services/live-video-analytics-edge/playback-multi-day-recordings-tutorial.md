---
title: Lecture d’enregistrements sur plusieurs jours - Azure
description: Dans ce tutoriel, vous allez apprendre à utiliser les API Azure Media Services pour lire un enregistrement vidéo continu de plusieurs jours.
ms.topic: tutorial
ms.date: 05/27/2020
ms.openlocfilehash: 1ec9260be7241057478b06446ac2aa53c14bcb47
ms.sourcegitcommit: 23aa0cf152b8f04a294c3fca56f7ae3ba562d272
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91803430"
---
# <a name="tutorial-playback-of-multi-day-recordings"></a>Tutoriel : Lecture d’enregistrements sur plusieurs jours  

Ce tutoriel se base sur le tutoriel [Enregistrement vidéo en continu](continuous-video-recording-concept.md) (CVR). Dans ce tutoriel, vous allez apprendre à utiliser les API Azure Media Services pour lire un enregistrement vidéo continu de plusieurs jours à partir du cloud. 

Cette opération peut s’avérer utile dans des scénarios tels que la sécurité publique, où il est nécessaire de conserver un enregistrement des images d’une caméra pendant plusieurs jours (ou semaines), et occasionnellement de regarder des parties spécifiques de ces images.

Ce didacticiel explique les procédures suivantes :

> [!div class="checklist"]
> * Exécuter l’exemple d’application qui montre comment parcourir le contenu d’un enregistrement de plusieurs jours
> * Afficher certaines parties de cet enregistrement

## <a name="suggested-pre-reading"></a>Lecture préalable suggérée  

Nous vous recommandons de lire les pages de documentation suivantes :

* [Présentation de Live Video Analytics sur IoT Edge](overview.md)
* [Terminologie de Live Video Analytics sur IoT Edge](terminology.md)
* [Concept de graphe multimédia](media-graph-concept.md)
* [Enregistrement vidéo en continu](continuous-video-recording-concept.md) 
* [Guide pratique : Lecture d’enregistrements](playback-recordings-how-to.md)
* [Tutoriel : Enregistrement vidéo en continu](continuous-video-recording-tutorial.md)

## <a name="prerequisites"></a>Prérequis

* Suivre le [Tutoriel CVR](continuous-video-recording-tutorial.md). Ce tutoriel et les API pertinentes abordés dans [Tutoriel : Enregistrement vidéo en continu](continuous-video-recording-tutorial.md) s’appliquent à des enregistrements de 5 minutes ou plus. Nous vous recommandons toutefois d’enregistrer des vidéos de 5 heures, voire plus. Les API utilisées pour parcourir les enregistrements offrent les meilleurs résultats avec de longs enregistrements.
* Nous vous recommandons d’exécuter ce tutoriel pendant l’exécution de [Tutoriel : Enregistrement vidéo en continu](continuous-video-recording-tutorial.md) ; autrement dit, pendant que vous continuez à enregistrer des vidéos dans le cloud.

## <a name="run-the-sample"></a>Exécution de l'exemple 

Dans le cadre du [tutoriel CVR](continuous-video-recording-tutorial.md), vous avez créé un compte Media Services. Pour les besoins de ce tutoriel, vous devez disposer d’un accès API complet à ce compte. Vous pouvez utiliser les étapes décrites dans [Récupérer des informations d’identification afin d’accéder à l’API Media Services](../latest/access-api-howto.md?tabs=portal) pour créer un principal de service. Vous devez pouvoir récupérer un bloc JSON à partir du portail Azure, ressemblant à ceci :

```
{
    "AadClientId": "<<INSERT_AZURE_AD_APP_ID_HERE>>",
    "AadSecret": "<<INSERT_AZURE_AD_APP_SECRET_HERE>>",
    "AadTenantDomain": "<<YOUR_TENANT_DOMAIN>>",
    "AadTenantId": "<<YOUR_TENANT_ID>>",
    "AccountName": "<<YOUR_ACCOUNT_NAME>>",
    "Location": "<<YOUR_REGION_NAME>>",
    "ResourceGroup": "<<YOUR_RESOURCE_GROUP>>",
    "SubscriptionId": "<<YOUR_SUBSCRIPTION_ID>>",
    "ArmAadAudience": "https://management.core.windows.net",
    "ArmEndpoint": "https://management.azure.com"
}
```

Ensuite, dans Visual Studio Code, ouvrez src/ams-asset-player. Ce dossier contient les fichiers nécessaires à ce tutoriel. Ouvrez le fichier appsettings.json et copiez son contenu dans un nouveau fichier, appsettings.development.json. Apportez les modifications suivantes à ce fichier :

```
  "AMS" : {
    "subscriptionId" : "Use value of SubscriptionId above",
    "resourceGroup" : "Use value of ResourceGroup above",
    "accountId" : "Use value of AccountName above",
    "aadTenantId" : "Use value of AadTenantId above",
    "aadClientId" : "Use value of AadClientId above",
    "aadSecret" : "Use value of AadSecret above"
} 
```

1. Dans Visual Studio Code, ouvrez l’onglet **Extensions** (ou appuyez sur Ctrl+Maj+X) et recherchez Azure IoT Hub.
1. Cliquez avec le bouton droit et sélectionnez **Paramètres d’extension**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Paramètres d’extension":::
1. Recherchez et activez « Afficher le message détaillé ».

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Paramètres d’extension":::
1. <!--In Visual Studio Code, you can click-->Cliquez sur l’icône Exécuter sur la gauche (ou Ctrl+Maj+D) pour lancer l’exécution des applications disponibles :

    ![Capture d’écran montrant un menu dans Visual Studio Code avec l’élément Exécuter sélectionné.](./media/playback-multi-day-recordings-tutorial/run.png)
1. Sélectionnez l’application AMS Asset Player dans la zone de liste déroulante, comme illustré ci-dessous, et appuyez sur F5 pour lancer le débogage.

    ![Capture d’écran montrant un menu dans Visual Studio Code avec l’élément AMS Asset Player sélectionné.](./media/playback-multi-day-recordings-tutorial/debug.png)

L’exemple d’application va créer et lancer votre application de navigation par défaut, et ouvrir la page AMS Asset Player.

> [!NOTE]
> En fonction des paramètres de sécurité de votre navigateur, il est possible qu’un avertissement s’affiche. Étant donné que la page web s’exécute localement, vous pouvez choisir d’ignorer l’avertissement.

Le lecteur AMS Asset Player vous invite à entrer le nom d’une ressource Media Service. Utilisez le nom de la ressource que vous avez utilisée pour enregistrer des vidéos dans [Tutoriel : Enregistrement de vidéo en continu](continuous-video-recording-tutorial.md).

Lorsque vous tapez le nom de la ressource et que vous appuyez sur Envoyer, le code du lecteur charge l’URL de diffusion en continu. Pour plus d'informations, consultez le [Guide pratique : Lecture d’enregistrements](playback-recordings-how-to.md). Si, comme nous le recommandons, vous êtes toujours en train d’enregistrer sur la ressource, le lecteur le détectera et tentera d’orienter la lecture vers la partie la plus récente de la vidéo enregistrée. Vous pouvez voir l’horodatage (au format UTC) en haut à gauche du lecteur. Dans la capture d’écran ci-dessous, notez comment le bouton « Live » est sélectionné.

![STREAM](./media/playback-multi-day-recordings-tutorial/assetplayer1.png)
 
Sur le côté droit du lecteur, vous pouvez voir les contrôles pour parcourir l’archive. Les années, les mois et les dates de ce contrôle sont renseignés à l’aide de l’API availableMedia documentée dans le [Guide pratique : Lecture d’enregistrements](playback-recordings-how-to.md).
Lorsque vous développez la journée, si vous avez laissé le tutoriel CVR s’exécuter pendant plusieurs heures, vous obtenez un résultat semblable à ceci :

![Parcourir l’archive](./media/playback-multi-day-recordings-tutorial/results.png)

La source du flux vidéo dans le tutoriel est un fichier MKV. Lorsque le simulateur RSTP (voir [Serveur multimédia Live555](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555)) atteint la fin du fichier, il stoppe le flux. Le nœud source RTSP dans le graphe multimédia le détecte et rétablit la connexion, et la vidéo reprend. Entre chaque fin de fichier et la reconnexion, il y a un écart dans l’archive enregistrée, qui apparaît sous la forme d’une nouvelle entrée dans les résultats availableMedia.

![Résultats](./media/playback-multi-day-recordings-tutorial/assetplayer2.png)
 
Lorsque vous cliquez sur une entrée quelconque de la liste, l’application crée une URL de diffusion en continu avec le filtre approprié, par exemple https://hostname}/{locatorId}/content. ism/manifest(format=mpd-time-csf,startTime=YYYY-MM-DDTHH:MM:SS). Le lecteur chargera cette URL et la lecture commencera à l’heure de début souhaitée.

Vous pouvez également utiliser des heures de début et de fin spécifiques, à l’aide des contrôles situés au bas de la page. Vous pouvez utiliser les résultats de l’appel availableMedia, comme indiqué sur le côté droit, comme guide pour les valeurs autorisées des heures de début et de fin. Les contraintes détaillées pour les filtres startTime et endTime sont documentées dans le [Guide pratique : Lecture d’enregistrements](playback-recordings-how-to.md). Après avoir sélectionné les valeurs d’heure, lorsque vous cliquez sur Envoyer, l’application charge le lecteur avec une URL de diffusion en continu telle que : https://{hostname}/{locatorId}/content.ism/manifest(format=mpd-time-csf,startTime=YYYY-MM-DDTHH:MM:SS,endTime= YYYY-MM-DDTHH:MM:SS) La lecture commence à l’heure de début souhaitée.

## <a name="next-steps"></a>Étapes suivantes

[Enregistrement vidéo basé sur les événements et lecture dans le cloud](event-based-video-recording-tutorial.md)
