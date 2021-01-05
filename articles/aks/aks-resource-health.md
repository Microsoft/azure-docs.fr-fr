---
title: Rechercher les événements de Resource Health qui ont un impact sur votre cluster AKS (préversion)
description: Vérifiez l’état d’intégrité de votre cluster AKS à l’aide de Azure Resource Health.
services: container-service
author: yunjchoi
ms.topic: troubleshooting
ms.date: 08/18/2020
ms.author: yunjchoi
ms.openlocfilehash: a409ac944c51d893fc344f82ae83d97559e055ef
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92070654"
---
# <a name="check-for-resource-health-events-impacting-your-aks-cluster-preview"></a>Rechercher les événements de Resource Health qui ont un impact sur votre cluster AKS (préversion)


Lors de l’exécution de vos charges de travail de conteneur sur AKS, vous devez vous assurer que vous pouvez résoudre les problèmes dès qu’ils surviennent pour minimaliser leur impact sur la disponibilité de vos charges de travail. [Azure Resource Health](../service-health/resource-health-overview.md) vous donne une visibilité sur divers événements d’intégrité susceptibles de rendre votre cluster AKS indisponible.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="open-resource-health"></a>Ouvrir Resource Health

### <a name="to-access-resource-health-for-your-aks-cluster"></a>Pour accéder à Resource Health pour votre cluster AKS :

- Accédez à votre cluster AKS dans le [portail Azure](https://portal.azure.com).
- Sélectionnez **Resource Health** dans le volet de navigation gauche.

### <a name="to-access-resource-health-for-all-clusters-on-your-subscription"></a>Pour accéder à Resource Health pour tous les clusters de votre abonnement :

- Recherchez **Service Health** dans le [portail Azure](https://portal.azure.com) et accédez-y.
- Sélectionnez **Resource health** dans le volet de navigation gauche.
- Sélectionnez votre abonnement et définissez le type de ressource sur Azure Kubernetes service (AKS).

![Capture d’écran montrant l’option Intégrité des ressources pour vos clusters AKS.](./media/aks-resource-health/resource-health-check.png)

## <a name="check-the-health-status"></a>Contrôler l’état d’intégrité

Azure Resource Health vous aide à diagnostiquer et à obtenir un support pour les problèmes de service qui affectent vos ressources Azure. Resource Health fournit des rapports sur l’intégrité actuelle et passée de vos ressources et vous aide à déterminer si le problème est causé par une action initiée par l’utilisateur ou par un événement de plateforme.

Resource Health reçoit des signaux pour que votre cluster géré détermine l’état d’intégrité du cluster. Il examine l’état d’intégrité de votre cluster AKS et signale les actions requises pour chaque signal d’intégrité. Ces signaux incluent la résolution automatique des problèmes, des mises à jour planifiées, des événements d’intégrité non planifiés et l’indisponibilité causée par des actions initiées par l’utilisateur. Ces signaux sont classés à l’aide de l’état d’intégrité dl’Azure Resource Health : *Disponible*, *Non disponible*, *Inconnu* et *Détérioré*.

- **Disponible** : lorsqu’il n’y a aucun problème connu affectant l’intégrité de votre cluster, Resource Health signale votre cluster comme *Disponible*.

- **Non disponible** : lorsqu’il y a un événement sur la plateforme ou hors de la plateforme qui affecte l’intégrité de votre cluster, Resource Health signale votre cluster comme *Non disponible*.

- **Inconnue** : En cas de perte temporaire de la connexion avec les mesures d’intégrité de votre cluster, Resource Health signale votre cluster comme *Inconnu*.

- **Détérioré** : lorsqu’un problème d’intégrité nécessite une action de votre part, Resource Health signale votre cluster comme *Détérioré*.

Pour plus d’informations sur ce qu’indique chaque état d’intégrité, consultez [Vue d’ensemble de Resource Health](../service-health/resource-health-overview.md#health-status).

### <a name="view-historical-data"></a>Afficher des données historiques

Vous pouvez également consulter les informations historiques des 30 derniers de Resource Health dans la section **Historique d’intégrité**.

## <a name="next-steps"></a>Étapes suivantes

Exécutez des vérifications sur votre cluster pour continuer à résoudre les problèmes de cluster à l’aide de [diagnostics AKS](./concepts-diagnostics.md).