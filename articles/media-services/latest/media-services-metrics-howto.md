---
title: Afficher les mesures avec Azure Monitor
description: Cet article montre comment surveiller les mesures avec les graphiques du portail Azure et Azure CLI.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: 09c306b85c910e8ec2021abf15013935c44b0df3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91707708"
---
# <a name="monitor-media-services-metrics"></a>Surveiller les mesures Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

[Azure Monitor](../../azure-monitor/overview.md) vous permet de surveiller les métriques et journaux de diagnostic pour vous aider à comprendre le fonctionnement de vos applications. Pour une description détaillée de cette fonctionnalité et pour comprendre pourquoi utiliser les mesures et les journaux de diagnostic Azure Media Services, consultez la section [Surveiller les mesures et les journaux de diagnostic Media Services](media-services-metrics-diagnostic-logs.md).

Azure Monitor offre plusieurs moyens d’interagir avec les mesures, y compris en créant des graphiques dans le portail, en y accédant via l’API REST ou en envoyant des requêtes avec Azure CLI. Cet article montre comment surveiller les mesures avec les graphiques du portail Azure et Azure CLI.

## <a name="prerequisites"></a>Prérequis

- [Créer un compte Media Services](./create-account-howto.md)
- Évaluation [Surveiller les mesures et les journaux de diagnostic Media Services](media-services-metrics-diagnostic-logs.md)

## <a name="view-metrics-in-azure-portal"></a>Voir les mesures depuis le portail Azure

1. Connectez-vous au portail Azure sur https://portal.azure.com.
1. Accédez à votre compte Azure Media Services et sélectionnez **Mesures**.
1. Cliquez sur la case **RESSOURCE** et sélectionnez la ressource pour laquelle vous souhaitez suivre les mesures.

    La fenêtre **Sélectionner une ressource** s’affiche sur la droite avec la liste des ressources disponibles pour vous. Dans ce cas, vous voyez :

    * &lt;Nom du compte Media Services&gt;
    * &lt;Nom du compte Media Services&gt;/&lt;nom de point de terminaison de streaming&gt;
    * &lt;Nom du compte de stockage&gt;

    Sélectionnez la ressource puis appuyez sur **Appliquer**. Pour plus d’informations sur les ressources prises en charge et les mesures, consultez [Surveiller les mesures Media Services](media-services-metrics-diagnostic-logs.md).

    ![Capture d’écran qui montre la ressource sélectionnée et met en surbrillance le bouton Appliquer.](media/media-services-metrics/metrics02.png)

    > [!NOTE]
    > Pour basculer entre les ressources pour lesquelles vous souhaitez surveiller les mesures, cliquez sur la case **RESSOURCE** à nouveau et répétez cette étape.
1. (Facultatif) Nommez votre graphique (modifier le nom en appuyant sur le crayon en haut).
1. Ajouter des mesures que vous souhaitez afficher.

    ![Mesures](media/media-services-metrics/metrics03.png)
1. Vous pouvez épingler votre graphique à votre tableau de bord.

## <a name="view-metrics-with-azure-cli"></a>Afficher les mesures avec Azure CLI

Pour obtenir des mesures de « Sortie » avec Azure CLI, vous devez exécuter la commande `az monitor metrics` suivante :

```azurecli-interactive
az monitor metrics list --resource \
   "/subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/Microsoft.Media/mediaservices/<Media Services account name>/streamingendpoints/<streaming endpoint name>" \
   --metric "Egress"
```

Pour obtenir d’autres mesures, remplacez « Sortie » par le nom de la mesure qui vous intéresse.

## <a name="see-also"></a>Voir aussi

* [Mesures Azure Monitor](../../azure-monitor/platform/data-platform.md)
* [Créer, afficher et gérer des alertes de mesure à l’aide d’Azure Monitor](../../azure-monitor/platform/alerts-metric.md).

## <a name="next-steps"></a>Étapes suivantes

[Journaux de diagnostic](media-services-diagnostic-logs-howto.md)
