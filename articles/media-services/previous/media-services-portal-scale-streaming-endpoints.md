---
title: Mettre à l’échelle des points de terminaison de streaming avec le Portail Azure | Microsoft Docs
description: Ce didacticiel vous guide à travers les étapes de mise à l’échelle des points de terminaison de streaming avec le Portail Azure.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 1008b3a3-2fa1-4146-85bd-2cf43cd1e00e
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 814d6b55e5ae04bf27dfb70d4e7b5ea2d5a4315c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89266814"
---
# <a name="scale-streaming-endpoints-with-the-azure-portal"></a>Mettre à l’échelle des points de terminaison de streaming avec le Portail Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Vue d’ensemble

> [!NOTE]
> Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/). 
> 
> 

Les points de terminaison de streaming **Premium** sont conçus pour les charges de travail avancées et fournissent une capacité de bande passante dédiée et scalable. Les clients disposant d’un point de terminaison de streaming **Premium** ont par défaut une unité de streaming. Le point de terminaison de streaming peut être adapté avec l’ajout d’unités de streaming. Chaque unité de streaming fournit une capacité de bande passante supplémentaire à l’application. Pour plus d’informations sur les types de point de terminaison de streaming et la configuration du CDN, consultez la rubrique [Vue d’ensemble des points de terminaison de streaming](media-services-streaming-endpoints-overview.md).
 
Cette rubrique montre comment mettre à l’échelle un point de terminaison de streaming.

Pour des informations détaillées sur la tarification, consultez la page [Détails de la tarification des services de média](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="scale-streaming-endpoints"></a>Mettre à l’échelle les points de terminaison de streaming

Pour modifier le nombre d’unités de streaming, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre compte Azure Media Services.
2. Dans la fenêtre **Paramètres**, sélectionnez **Points de terminaison de diffusion en continu**.
3. Cliquez sur le point de terminaison de streaming que vous souhaitez mettre à l’échelle. 

    > [!NOTE] 
    > Vous pouvez uniquement mettre à l’échelle les points de terminaison de streaming **Premium**.

4. Déplacez le curseur pour spécifier le nombre d’unités de diffusion en continu.

    ![point de terminaison de diffusion en continu](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints3.png)

## <a name="next-steps"></a>Étapes suivantes
Consultez les parcours d’apprentissage de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

