---
title: Configurer l’encodeur Telestream Wirecast pour envoyer un flux en direct à débit binaire unique | Microsoft Docs
description: "Cette rubrique explique comment configurer l'encodeur live Wirecast afin d'envoyer un flux à débit binaire unique à des canaux AMS activés pour l'encodage en temps réel. "
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 0d2f1e81-51a6-4ca9-894a-6dfa51ce4c70
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.reviewer: cenkdin;anilmur
ms.openlocfilehash: ecab0ee67dd1d5cee3fd9927c00c9f9d827b2f47
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92910047"
---
# <a name="use-the-wirecast-encoder-to-send-a-single-bitrate-live-stream"></a>Utiliser l’encodeur Wirecast pour envoyer un flux en direct à débit binaire unique

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
> [!div class="op_single_selector"]
> * [Wirecast](media-services-configure-wirecast-live-encoder.md)
>

Cet article explique comment configurer l'encodeur live [Telestream Wirecast](https://www.telestream.net/wirecast/overview.htm) afin d'envoyer un flux à débit binaire unique à des canaux AMS activés pour l'encodage en temps réel. Pour plus d’informations, consultez [Utilisation de canaux activés pour effectuer un encodage en direct avec Azure Media Services](media-services-manage-live-encoder-enabled-channels.md).

Ce didacticiel montre comment gérer Azure Media Services (AMS) avec l’outil Azure Media Services Explorer (AMSE). Cet outil est uniquement compatible avec les PC Windows. Si vous êtes sous Mac ou Linux, utilisez le portail Azure pour créer des [canaux](media-services-portal-creating-live-encoder-enabled-channel.md#create-a-channel) et des [programmes](media-services-portal-creating-live-encoder-enabled-channel.md).

> [!NOTE]
> Les encodeurs doivent prendre en charge le protocole TLS 1.2 lors de l’utilisation des protocoles RTMPS. Utilisez Wirecast version 13.0.2 ou ultérieure en raison de la configuration TLS 1.2.

## <a name="prerequisites"></a>Prérequis
* [Créer un compte Azure Media Services](media-services-portal-create-account.md)
* Vérifiez qu’un point de terminaison de streaming est en cours d’exécution. Pour plus d’informations, consultez [Gestion des points de terminaison de diffusion en continu dans un compte Media Services](media-services-portal-manage-streaming-endpoints.md)
* Installez la dernière version de l’outil [AMSE](https://github.com/Azure/Azure-Media-Services-Explorer) .
* Lancez l’outil et connectez-vous à votre compte AMS.

## <a name="tips"></a>Conseils
* Si possible, utilisez une connexion Internet câblée.
* Une bonne règle pour déterminer les besoins en bande passante consiste à doubler les débits binaires de diffusion en continu. Bien qu’il ne s’agisse pas d’une obligation, cela permet de réduire l’impact de l’encombrement du réseau.
* Lors de l’utilisation d’encodeurs logiciels, fermez tous les programmes inutiles.

## <a name="create-a-channel"></a>Créer un canal
1. Dans l’outil AMSE, accédez à l’onglet **Live** , puis cliquez avec le bouton droit dans la zone des canaux. Dans le menu qui s’affiche, sélectionnez **Créer un canal...** .

    ![Capture d’écran de l’option Créer un canal sélectionnée dans un menu](./media/media-services-wirecast-live-encoder/media-services-wirecast1.png)

2. Spécifiez un nom de canal (le champ Description est facultatif). Sous Paramètres du canal, sélectionnez **Standard** pour l’option Live Encoding, avec le protocole d’entrée défini sur **RTPM**. Vous pouvez laisser tous les autres paramètres inchangés.

    Vérifiez que l’option **Démarrer maintenant le nouveau canal** est sélectionnée.

3. Cliquez sur **Créer un canal**.

   ![Capture d’écran de la boîte de dialogue Créer un canal live](./media/media-services-wirecast-live-encoder/media-services-wirecast2.png)

> [!NOTE]
> Le démarrage du canal peut prendre jusqu’à 20 minutes.
>
>

Pendant le démarrage du canal, vous pouvez [configurer l’encodeur](media-services-configure-wirecast-live-encoder.md#configure_wirecast_rtmp).

> [!IMPORTANT]
> La facturation commence dès que l’état du canal indique qu’il est prêt à être utilisé. Pour plus d’informations, consultez [États du canal](media-services-manage-live-encoder-enabled-channels.md#states).
>
>

## <a name="configure-the-telestream-wirecast-encoder"></a><a id="configure_wirecast_rtmp" />Configurer l’encodeur Telestream Wirecast
Dans ce didacticiel, les paramètres de sortie ci-dessous sont utilisés. Le reste de cette section décrit la procédure de configuration plus en détail.

**Vidéo** :

* Codec : H.264
* Profil : Élevé (niveau 4.0)
* Vitesse de transmission : 5 000 Kbits/s
* Image clé : 2 secondes (60 secondes)
* Fréquence d’images : 30

**Audio** :

* Codec : AAC (LC)
* Vitesse de transmission : 192 Kbits/s
* Taux d’échantillonnage : 44,1 kHz

### <a name="configuration-steps"></a>Configuration
1. Ouvrez l’application Telestream Wirecast sur votre ordinateur et configurez le streaming RTMP.
2. Configurez la sortie en accédant à l’onglet **Sortie** et en sélectionnant **Paramètres de sortie...** .

    Vérifiez que le champ **Destination de sortie** est défini sur **Serveur RTMP**.
3. Cliquez sur **OK**.
4. Sur la page de paramètres, définissez le champ **Destination** sur **Azure Media Services**.

    Le profil d’encodage est prédéfini sur **Azure H.264 720 p 16:9 (1280 x 720)** . Pour personnaliser ces paramètres, sélectionnez l’icône en forme d’engrenage à droite de la liste déroulante, puis sélectionnez **Nouvelle prédéfinition**.

    ![Capture d’écran de la boîte de dialogue Choisir un modèle avec l’option BlobTrigger sélectionnée](./media/media-services-wirecast-live-encoder/media-services-wirecast3.png)
5. Configurez les présélections de l’encodeur.

    Nommez la présélection et vérifiez les paramètres recommandés suivants :

    **Vidéo**

   * Encodeur : MainConcept H.264
   * Images par seconde : 30
   * Vitesse de transmission moyenne : 5 000 Kbits/s (cette valeur peut être ajustée en fonction des limitations du réseau)
   * Profil : Principal
   * Image clé toutes les : 60 images

     **Audio**

   * Vitesse de transmission cible : 192 Kbits/s
   * Taux d’échantillonnage : 44,100 kHz

     ![Capture d’écran de la présélection de l’encodeur pour AzureTest1](./media/media-services-wirecast-live-encoder/media-services-wirecast4.png)
6. Appuyez sur **Enregistrer**.

    Vous pouvez maintenant sélectionner le profil créé dans le champ Encodage.

    Assurez-vous que ce nouveau profil est sélectionné.
7. Récupérez l’URL d’entrée du canal pour l’affecter au **Point de terminaison RTMP** Wirecast.

    Revenez à l’outil AMSE et vérifiez l’état d’achèvement du canal. Une fois que l’état est passé de **Démarrage** à **En cours d’exécution** , vous pouvez obtenir l’URL d’entrée.

    Une fois le canal en cours d’exécution, cliquez avec le bouton droit sur le nom du canal, déplacez le pointeur vers le bas pour le placer sur **Copier l’URL entrée dans le Presse-papiers** , puis sélectionnez **URL d’entrée principale**.  

    ![Capture d’écran de l’option Copier l’URL dans le Presse-papiers de l’URL d’entrée principale](./media/media-services-wirecast-live-encoder/media-services-wirecast6.png)
8. Dans la fenêtre **Paramètres de sortie** de Wirecast, collez ces informations dans le champ **Adresse** de la section de sortie et indiquez un nom de flux.

    ![Capture d’écran des Paramètres de sortie](./media/media-services-wirecast-live-encoder/media-services-wirecast5.png)

1. Sélectionnez **OK**.
2. Dans l’écran principal de **Wirecast** , vérifiez que les sources d’entrée audio et vidéo sont prêtes, puis appuyez sur **Flux** dans le coin supérieur gauche.

    ![Capture d’écran du bouton Flux Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast7.png)

> [!IMPORTANT]
> Avant de cliquer sur **Flux** , vous **devez** vérifier que le canal est prêt.
> Veillez également à ne pas laisser le canal à l’état d’exécution sans un flux de contribution d’entrée pendant plus de 15 minutes.
>
>

## <a name="test-playback"></a>Tester la lecture

Accédez à l’outil AMSE et cliquez avec le bouton droit sur le canal à tester. Dans le menu, placez le pointeur sur **Lire l’aperçu** et sélectionnez **avec le Lecteur multimédia Azure**.  

![Capture d’écran de l’option Lire l’aperçu avec le lecteur multimédia Azure sélectionnée](./media/media-services-wirecast-live-encoder/media-services-wirecast8.png)

Si le flux s’affiche dans le lecteur, cela signifie que l’encodeur a été correctement configuré pour se connecter à AMS.

Si vous recevez une erreur, vous devez réinitialiser le canal et ajuster les paramètres de l’encodeur. Pour obtenir des conseils, consultez l’article sur la [résolution des problèmes](media-services-troubleshooting-live-streaming.md).  

## <a name="create-a-program"></a>Créer un programme
1. Une fois que vous avez vérifié que la lecture fonctionne sur le canal, créez un programme. Sous l’onglet **Live** de l’outil AMSE, cliquez avec le bouton droit dans la zone des programmes et sélectionnez **Créer un programme**.  

    ![Capture d’écran de l’option Créer un programme sélectionnée](./media/media-services-wirecast-live-encoder/media-services-wirecast9.png)
2. Nommez le programme et, si nécessaire, ajustez la **longueur de la fenêtre d’archive** (qui est de quatre heures par défaut). Vous pouvez également spécifier un emplacement de stockage ou conserver la valeur par défaut.  
3. Cochez la case **Démarrer le programme maintenant** .
4. Cliquez sur **Créer le programme**.  

   >[!NOTE]
   >La création d’un programme prend moins de temps que la création d’un canal.
       
5. Une fois le programme en cours d’exécution, vérifiez que la lecture fonctionne. Pour ce faire, cliquez avec le bouton droit sur le programme, placez le pointeur sur **Lire le(s) programme(s)** , puis sélectionnez **avec le Lecteur multimédia Azure**.  
6. Après confirmation, cliquez à nouveau avec le bouton droit sur le programme et sélectionnez **Copier l’URL de sortie dans le Presse-papiers** (ou obtenez cette information à l’aide de l’option **Informations et paramètres du programme** du menu).

Le flux est maintenant prêt à être incorporé dans un lecteur ou distribué à une audience pour un affichage en direct.  

## <a name="troubleshooting"></a>Dépannage
Pour obtenir des conseils, consultez l’article sur la [résolution des problèmes](media-services-troubleshooting-live-streaming.md).

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
