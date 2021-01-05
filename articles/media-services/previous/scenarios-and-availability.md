---
title: Scénarios courants de Microsoft Azure Media Services | Microsoft Azure
description: Cet article donne une vue d’ensemble des scénarios de Microsoft Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 11/3/2020
ms.author: inhenkel
ms.openlocfilehash: 001c535a2b39898673f2d587ee807d43b4d5f60a
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348540"
---
# <a name="microsoft-azure-media-services-common-scenarios"></a>Scénarios courants de Microsoft Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. Découvrez la dernière version, [Media Services v3](../latest/media-services-overview.md). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Microsoft Azure Media Services (AMS) vous permet de charger, de stocker, d’encoder et d’empaqueter en toute sécurité du contenu vidéo ou audio destiné à être diffusé à la demande ou en streaming en direct sur différents clients (par exemple, téléviseurs, PC et appareils mobiles).

Cet article décrit les scénarios courants de la distribution de votre contenu en direct ou à la demande.

## <a name="overview"></a>Vue d’ensemble

### <a name="prerequisites"></a>Prérequis

* Un compte Azure. Si vous ne possédez pas de compte, vous pouvez créer un compte d’évaluation gratuit en quelques minutes. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com).
* Un compte Azure Media Services. Pour plus d’informations, consultez [Créer un compte](media-services-portal-create-account.md).
* Le point de terminaison à partir duquel vous souhaitez diffuser du contenu doit se trouver dans l’état **En cours d’exécution**.

    Une fois votre compte AMS créé, un point de terminaison de streaming **par défaut** est ajouté à votre compte à l’état **Arrêté**. Pour démarrer le streaming de votre contenu et tirer parti de l’empaquetage et du chiffrement dynamiques, le point de terminaison de streaming doit se trouver à l’état **En cours d’exécution**.

### <a name="commonly-used-objects-when-developing-against-the-ams-odata-model"></a>Objets couramment utilisés lors du développement par rapport au modèle AMS OData

L’image suivante illustre certains des objets couramment utilisés lors du développement par rapport au modèle OData de Media Services.

Cliquez sur l’image pour l’afficher en plein écran.  

[![Diagramme montrant certains des objets les plus couramment utilisés lors du développement par rapport au modèle de données d’objet Azure Media Services.](./media/media-services-overview/media-services-overview-object-model-small.png)](./media/media-services-overview/media-services-overview-object-model.png#lightbox)

Vous pouvez afficher l’ensemble du modèle [ici](https://media.windows.net/API/$metadata?api-version=2.15).  

## <a name="protect-content-in-storage-and-deliver-streaming-media-in-the-clear-non-encrypted"></a>Protéger le contenu stocké et assurer une diffusion multimédia en continu en clair (sans chiffrement)

![Flux de travail VOD](./media/scenarios-and-availability/scenarios-and-availability01.png)

1. Chargez un fichier multimédia de haute qualité dans une ressource.

    Afin de protéger votre contenu lors du chargement et quand il est au repos dans le stockage, l’application à votre élément multimédia de l’option de chiffrement de stockage est recommandée.

1. Encoder en un ensemble de fichiers MP4 à débit adaptatif.

    Afin de protéger votre contenu lorsqu’il est au repos, l’application de l’option de chiffrement de stockage à l’élément multimédia de sortie est recommandée.

1. Configurez la stratégie de remise de ressources (utilisée par l’empaquetage dynamique).

    Si votre ressource est stockée sous forme chiffrée, vous **devez** configurer une stratégie de remise de ressources.
1. Publiez la ressource en créant un localisateur à la demande.
1. Diffusez le contenu publié.

## <a name="protect-content-in-storage-deliver-dynamically-encrypted-streaming-media"></a>Protéger le contenu stocké et diffuser du contenu multimédia chiffré dynamiquement en continu

![Protéger avec PlayReady](./media/media-services-content-protection-overview/media-services-content-protection-with-multi-drm.png)

1. Chargez un fichier multimédia de haute qualité dans une ressource. Appliquez l’option de chiffrement de stockage à la ressource.
1. Encoder en un ensemble de fichiers MP4 à débit adaptatif. Appliquez l’option de chiffrement de stockage à la ressource de sortie.
1. Créez la clé de contenu de chiffrement pour la ressource que vous souhaitez chiffrer dynamiquement pendant la lecture.
1. Configurez la stratégie d’autorisation de clé de contenu.
1. Configurez la stratégie de remise de ressources (utilisée par l’empaquetage dynamique et le chiffrement dynamique).
1. Publiez la ressource en créant un localisateur à la demande.
1. Diffusez le contenu publié.

## <a name="deliver-progressive-download"></a>Remettre le téléchargement progressif

1. Chargez un fichier multimédia de haute qualité dans une ressource.
1. Encoder en un fichier MP4 unique.
1. Publiez la ressource en créant un localisateur à la demande ou de signature d’accès partagé (SAS, Shared Access Signature). Si vous utilisez un localisateur SAS, le contenu est téléchargé depuis le stockage d’objets blob Azure. Il n’est pas nécessaire de disposer de points de terminaison de streaming à l’état démarré.
1. Téléchargez le contenu de manière progressive.

## <a name="delivering-live-streaming-events"></a>Diffusion d’événements en streaming en direct

1. Ingérer du contenu en direct à l’aide de différents protocoles de streaming en direct (par exemple RTMP ou Smooth Streaming)
1. Encoder votre flux en flux à débit adaptatif (facultatif)
1. Afficher un aperçu de votre flux en direct
1. Fournir le contenu par le biais de :
    1. Protocoles de streaming courants (par exemple, MPEG DASH, Smooth, HLS) directement sur vos clients,
    1. À un réseau de distribution de contenu (CDN) en vue d’une distribution ultérieure, ou
    1. Pour enregistrer et stocker le contenu ingéré à diffuser ultérieurement (vidéo à la demande).

Lors d’un streaming en direct, vous pouvez choisir l’une des deux options suivantes :

### <a name="working-with-channels-that-receive-multi-bitrate-live-stream-from-on-premises-encoders-pass-through"></a>Utilisation des canaux recevant un flux dynamique à débit binaire multiple provenant d’encodeurs locaux (méthode directe)

Le diagramme suivant présente les principaux composants de la plateforme AMS impliqués dans ce flux de travail de **méthode directe** .

![Diagramme présentant les principaux composants de la plateforme AMS impliqués dans le flux de travail « Pass-through ».](./media/scenarios-and-availability/media-services-live-streaming-current.png)

Pour plus d’informations, consultez [Utilisation des canaux recevant un flux dynamique à débit binaire multiple provenant d’encodeurs locaux](media-services-live-streaming-with-onprem-encoders.md).

### <a name="working-with-channels-that-are-enabled-to-perform-live-encoding-with-azure-media-services"></a>Utilisation de canaux activés pour effectuer un encodage en temps réel avec Azure Media Services

Le schéma suivant présente les principaux composants de la plateforme AMS impliqués dans le flux de travail Streaming en direct, pour lequel un canal est activé en vue d’effectuer un encodage en temps réel avec Media Services.

![Flux de travail live](./media/scenarios-and-availability/media-services-live-streaming-new.png)

Pour plus d’informations, consultez [Utilisation de canaux activés pour effectuer un encodage en direct avec Azure Media Services](media-services-manage-live-encoder-enabled-channels.md).

## <a name="consuming-content"></a>Utilisation de contenu

Azure Media Services fournit les outils dont vous avez besoin pour créer des applications de lecteur clientes riches et dynamiques pour la plupart des plateformes, notamment : appareils iOS, Android, Windows, Windows Phone, Xbox et décodeurs.

## <a name="enabling-azure-cdn"></a>Activation du CDN Azure

Media Services prend en charge l’intégration avec le CDN d’Azure. Pour plus d’informations sur l’activation du CDN Azure, voir [Gestion des points de terminaison de diffusion en continu dans un compte Media Services](media-services-portal-manage-streaming-endpoints.md).

## <a name="scaling-a-media-services-account"></a>Mise à l’échelle d’un compte Media Services

Les clients AMS peuvent mettre à l’échelle les points de terminaison de streaming, le traitement multimédia et le stockage dans leurs comptes AMS.

* En fonction de leurs besoins, les clients Media Services peuvent choisir un point de terminaison de streaming **Standard** ou **Premium**. Le point de terminaison de streaming **Standard** convient à la plupart des charges de travail de streaming. Il inclut les mêmes fonctionnalités que les points de terminaison de streaming **Premium** et met automatiquement à l’échelle la bande passante sortante.

    Les points de terminaison de streaming **Premium** sont conçus pour les charges de travail avancées et fournissent une capacité de bande passante dédiée et scalable. Les clients disposant d’un point de terminaison de streaming **Premium** ont par défaut une unité de streaming. Le point de terminaison de streaming peut être adapté avec l’ajout d’unités de streaming. Chaque unité de streaming fournit une capacité de bande passante supplémentaire à l’application. Pour plus d’informations sur la scalabilité des points de terminaison de streaming **Premium** , consultez la rubrique [Mettre à l’échelle des points de terminaison de streaming](media-services-portal-scale-streaming-endpoints.md).

* Un compte Media Services est associé à un Type d'unité réservé qui détermine la vitesse à laquelle vos tâches de traitement multimédia sont traitées. Vous pouvez choisir entre les types d’unités réservées suivantes : **S1** , **S2** ou **S3**. Par exemple, un même travail d’encodage s’exécute plus rapidement quand vous utilisez le type d’unité réservée **S2** que le type **S1**.

    En plus de spécifier le type d’unité réservée, vous pouvez approvisionner votre compte avec des **unités réservées**. Le nombre d’unités réservées approvisionnées détermine le nombre de tâches multimédias qui peuvent être traitées simultanément dans un compte donné.

    > [!NOTE]
    > Les unités réservées fonctionnent pour la mise en parallèle de tout le traitement multimédia, notamment les travaux d’indexation qui utilisent Azure Media Indexer. Toutefois, contrairement à l’encodage, l’indexation des travaux n’est pas plus rapide avec des unités réservées plus rapides.

    Pour plus d’informations, consultez [Mise à l’échelle du traitement multimédia](media-services-portal-scale-media-processing.md).

* Vous pouvez aussi mettre à l’échelle votre compte Media Services en lui ajoutant des comptes de stockage. Chaque compte de stockage est limité à 500 To. Pour plus d’informations, consultez [Gérer les comptes de stockage](./media-services-managing-multiple-storage-accounts.md).

## <a name="next-steps"></a>Étapes suivantes

[Migrer vers Media Services V3](../latest/media-services-overview.md)

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]