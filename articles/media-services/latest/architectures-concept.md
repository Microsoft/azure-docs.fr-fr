---
title: Architectures Media Services
titleSuffix: Azure Media Services
description: Cet article décrit les architectures pour Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: inhenkel
ms.openlocfilehash: c14dbe4253696e01e99111a24319e455c1d551e9
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533993"
---
# <a name="media-services-architectures"></a>Architectures Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="live-streaming-digital-media"></a>Streaming en direct de contenu multimédia numérique

Une solution de streaming en direct vous permet de capturer des vidéos en temps réel et de les diffuser aux consommateurs en temps réel, comme le streaming d’entretiens, de conférences et d’événements sportifs en ligne. Dans cette solution, une vidéo est capturée par une caméra vidéo et envoyée à un point de terminaison d’entrée de canal. Le canal reçoit le flux d’entrée dynamique et le rend disponible pour la diffusion en continu via un point de terminaison de streaming vers un navigateur web ou une application mobile. Les canaux fournissent également un point de terminaison de surveillance d’aperçu pour afficher un aperçu de votre flux et le valider avant traitement et livraison. Le canal peut également enregistrer et stocker le contenu ingéré en vue d’une diffusion ultérieure (vidéo à la demande).

Cette solution est basée sur les services gérés Azure : Media Services et réseau de distribution de contenu. Ces services fonctionnent dans un environnement à haute disponibilité, avec des correctifs et un support, ce qui vous permet de vous concentrer sur votre solution plutôt que sur l’environnement dans lequel elles opèrent.

Consultez [Streaming en direct de contenu multimédia numérique](https://docs.microsoft.com/azure/architecture/solution-ideas/articles/digital-media-live-stream) dans le centre des architectures Azure.

## <a name="video-on-demand-digital-media"></a>Vidéo à la demande pour le contenu multimédia numérique

Une solution de vidéo à la demande simple qui vous donne la possibilité de diffuser du contenu vidéo enregistré, tel que des films, des clips d’infos, des segments sportifs, des vidéos de formation et des tutoriels de support client, sur n’importe quel appareil de point de terminaison, application mobile ou navigateur de bureau compatible avec la vidéo. Les fichiers vidéo sont chargés vers le stockage Blob Azure, encodés au format standard multidébit, puis distribués via tous les principaux protocoles de streaming à débit adaptatif (TLS, MPEG-DASH, Smooth) au client Lecteur multimédia Azure.

Cette solution est basée sur les services gérés Azure : Stockage Blob, Réseau de distribution de contenu et Lecteur multimédia Azure. Ces services fonctionnent dans un environnement à haute disponibilité, avec des correctifs et un support, ce qui vous permet de vous concentrer sur votre solution plutôt que sur l’environnement dans lequel elles opèrent.

Consultez [Vidéo à la demande pour le contenu multimédia numérique](https://docs.microsoft.com/azure/architecture/solution-ideas/articles/digital-media-video) dans le centre des architectures Azure.

## <a name="gridwich-media-processing-system"></a>Système de traitement multimédia Gridwich

Le système Gridwich représente les meilleures pratiques pour le traitement et la livraison de ressources multimédias sur Azure. Bien que le système Gridwich soit spécifique au média, l’infrastructure de traitement des messages et de gestion des événements peut s’appliquer à tout flux de travail de traitement d’événements sans état.

Consultez [Système de traitement multimédia Gridwich](https://docs.microsoft.com/azure/architecture/reference-architectures/media-services/gridwich-architecture) dans le centre des architectures Azure.

## <a name="next-steps"></a>Étapes suivantes

> [Vue d’ensemble d’Azure Media Services](media-services-overview.md)