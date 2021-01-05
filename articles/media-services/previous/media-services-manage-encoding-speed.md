---
title: Gérer la rapidité et la simultanéité de votre encodage avec Azure Media Services | Microsoft Docs
description: Cet article donne un bref aperçu de la manière dont vous pouvez gérer la rapidité et la simultanéité de vos travaux/tâches d’encodage avec Azure Media Services.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 676313f8-a158-4e3a-a99b-2c29a341ecc9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 4b6f843678d64bddd276f6123a432699efc89ad9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89269282"
---
#  <a name="manage-speed-and-concurrency-of-your-encoding"></a>Gérer la vitesse et la simultanéité de votre encodage

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

Cet article donne un bref aperçu de la manière dont vous pouvez gérer la rapidité et la simultanéité de vos travaux/tâches d’encodage.

## <a name="overview"></a>Vue d’ensemble

Dans Media Services, un **Type d’unité réservée** détermine la vitesse à laquelle vos tâches de traitement multimédia sont traitées. Vous pouvez choisir entre les types d’unités réservées suivantes : **S1**, **S2** ou **S3**. Par exemple, un même travail d’encodage s’exécute plus rapidement quand vous utilisez le type d’unité réservée **S2** que le type **S1**. La rubrique relative à la [mise à l’échelle des unités d’encodage](media-services-scale-media-processing-overview.md) contient un tableau qui vous aide à prendre une décision lors du choix entre les différentes vitesses d’encodage.

En plus de spécifier le type d’unité réservée, vous pouvez approvisionner votre compte avec des **unités réservées**. Le nombre d’unités réservées approvisionnées détermine le nombre de tâches multimédias qui peuvent être traitées simultanément dans un compte donné. Si, par exemple, votre compte a cinq unités réservées, les cinq tâches multimédias sont exécutées simultanément tant qu’il y a des tâches à traiter. Les autres tâches restent dans la file d'attente et sont sélectionnées séquentiellement pour le traitement quand l'exécution d'une tâche se termine. Si aucune unité réservée n'est approvisionnée pour un compte donné, les tâches sont sélectionnées séquentiellement. Dans ce cas, le temps d'attente entre l'achèvement d'une tâche et le démarrage de la suivante dépend de la disponibilité des ressources dans le système.

Pour obtenir des informations détaillées et des exemples sur la manière de mettre à l’échelle les unités d’encodage, consultez [cette rubrique](media-services-scale-media-processing-overview.md).

## <a name="next-step"></a>Étape suivante

[Mise à l’échelle des unités d’encodage](media-services-scale-media-processing-overview.md)

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

