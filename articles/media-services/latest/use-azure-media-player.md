---
title: Lecture à l’aide du lecteur multimédia Azure - Azure | Microsoft Docs
description: Le lecteur multimédia Azure est un lecteur vidéo web conçu pour lire du contenu multimédia à partir de Microsoft Azure Media Services sur une grande variété de navigateurs et d’appareils.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 07/17/2019
ms.author: inhenkel
ms.openlocfilehash: a3b32110a7236d47946c3a1f901aaedf51b47775
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022215"
---
# <a name="playback-with-azure-media-player"></a>Lecture avec le lecteur multimédia Azure

Le lecteur multimédia Azure est un lecteur vidéo web conçu pour lire du contenu multimédia à partir de Microsoft Azure Media Services sur une grande variété de navigateurs et d’appareils. Azure Media Player exploite des normes du secteur, telles que HTML5, Media Source Extensions (MSE) et Encrypted Media Extensions (EME), pour offrir une expérience enrichie de diffusion en continu adaptative. Lorsque ces normes ne sont pas disponibles sur un périphérique ou dans un navigateur, Azure Media Player utilise Flash et Silverlight comme technologies de secours. Quelle que soit la technologie de lecture utilisée, les développeurs bénéficient d’une interface JavaScript unifiée pour accéder aux API. Le contenu fourni par Azure Media Services peut ainsi être lu sur un large éventail de périphériques et de navigateurs sans effort supplémentaire.

Microsoft Azure Media Services permet au contenu d’être diffusé dans les formats de streaming HLS, DASH et Smooth Streaming. Azure Media Player prend en compte des différents formats et lit automatiquement le lien le mieux adapté aux capacités de la plateforme/du navigateur. Media Services assure également le chiffrement dynamique des ressources avec le chiffrement PlayReady ou le chiffrement d’enveloppe AES 128 bits. Lorsqu’il est configuré de manière appropriée, Azure Media Player permet le déchiffrement du contenu chiffré avec PlayReady et AES 128 bits.

> [!NOTE]
> La lecture HTTPS est requise pour le contenu chiffré Widevine.

## <a name="use-azure-media-player-demo-page"></a>Utiliser la page de démonstration du lecteur multimédia Azure

### <a name="start-using"></a>Démarrer l’utilisation

Vous pouvez utiliser la [page de démonstration du lecteur multimédia Azure](https://aka.ms/azuremediaplayer) pour lire les exemples Azure Media Services ou votre propre flux de données.  

Pour lire une nouvelle vidéo, collez une autre URL et appuyez sur **Mettre à jour**.

Pour configurer diverses options de lecture (relatives à la technologie, à la langue ou au chiffrement), appuyez sur **Options avancées**.

![Azure Media Player](./media/azure-media-player/home-page.png)

### <a name="monitor-diagnostics-of-a-video-stream"></a>Superviser les diagnostics d’un flux vidéo

Vous pouvez utiliser la [page de démonstration du lecteur multimédia Azure](https://aka.ms/azuremediaplayer) pour superviser les diagnostics d’un flux vidéo.

![Diagnostics Azure Media Player](./media/azure-media-player/diagnostics.png)

## <a name="set-up-azure-media-player-in-your-html"></a>Configurer le lecteur multimédia Azure dans votre code HTML

Le lecteur multimédia Azure est facile à configurer. Seuls quelques instants sont nécessaires pour lire du contenu multimédia à partir de votre compte Media Services. Pour plus d’informations sur l’installation et la configuration du lecteur multimédia Azure, consultez la [documentation relative au lecteur multimédia Azure](../azure-media-player/azure-media-player-overview.md).

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="next-steps"></a>Étapes suivantes

* [Documentation relative au lecteur multimédia Azure](../azure-media-player/azure-media-player-overview.md)
* [Exemples du lecteur multimédia Azure](https://github.com/Azure-Samples/azure-media-player-samples).