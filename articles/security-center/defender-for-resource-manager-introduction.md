---
title: 'Azure Defender pour Resource Manager : Avantages et fonctionnalités'
description: Découvrir les avantages et les fonctionnalités d’Azure Defender pour Resource Manager
author: memildin
ms.author: memildin
ms.date: 12/07/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 5594e8225b615bf6e1af5077bd68422edbbeec19
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905449"
---
# <a name="introduction-to-azure-defender-for-resource-manager"></a>Présentation d’Azure Defender pour Resource Manager

[Azure Resource Manager](../azure-resource-manager/management/overview.md) est le service de déploiement et de gestion d’Azure. Il fournit une couche de gestion qui vous permet de créer, de mettre à jour et de supprimer des ressources dans votre compte Azure. Vous utilisez des fonctionnalités de gestion, telles que le contrôle d’accès, les verrous et les étiquettes, pour sécuriser et organiser vos ressources après le déploiement.

La couche de gestion cloud est un service essentiel, connecté à toutes vos ressources cloud. De ce fait, il est également une cible potentielle pour les attaquants. Par conséquent, nous recommandons aux équipes des opérations de sécurité de surveiller attentivement la couche de gestion des ressources. 

Azure Defender pour Resource Manager supervise automatiquement les opérations de gestion des ressources dans votre organisation, qu’elles soient effectuées via le portail Azure, les API REST Azure, Azure CLI ou d’autres clients programmatiques Azure. Azure Defender exécute une analytique de sécurité avancée pour détecter les menaces et vous avertir des activités suspectes.

## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|PRÉVERSION<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)] |
|Prix :|**Azure Defender pour Resource Manager** est facturé comme indiqué sur [la page des tarifs](security-center-pricing.md).|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Non](./media/icons/no-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-resource-manager"></a>Quels sont les avantages d’Azure Defender pour Resource Manager ?

Azure Defender pour Resource Manager protège contre certains problèmes, notamment :

- **Les opérations de gestion des ressources suspectes**, comme les opérations à partir d’adresses IP suspectes, la désactivation du logiciel anti-programme malveillant et les scripts douteux s’exécutant dans des extensions de machine virtuelle
- **L’utilisation de kits de ressources d’exploitation**, comme Microburst ou PowerZure
- **Le mouvement latéral**, depuis la couche de gestion Azure vers le plan de données des ressources Azure

:::image type="content" source="media/defender-for-resource-manager-introduction/consistent-management-layer-with-defender.png" alt-text="Diagramme de présentation d’Azure Resource Manager":::

La liste complète des alertes fournies par Azure Defender pour Resource Manager se trouve dans la [page de référence des alertes](alerts-reference.md#alerts-resourcemanager).


 ## <a name="how-to-investigate-alerts-from-azure-defender-for-resource-manager"></a>Comment examiner les alertes à partir d’Azure Defender pour Resource Manager

Les alertes de sécurité à partir d’Azure Defender pour Resource Manager sont basées sur les menaces détectées par la supervision des opérations Azure Resource Manager. Azure Defender utilise les sources de journaux internes d’Azure Resource Manager ainsi que le journal d’activité Azure, un journal de plateforme dans Azure qui fournit des insights sur les événements de niveau abonnement.

Découvrez-en plus sur le [Journal d’activité Azure](../azure-monitor/platform/activity-log.md).

Pour examiner les alertes de sécurité depuis Azure Defender pour Resource Manager :

1. Ouvrez le journal d’activité Azure.

    :::image type="content" source="media/defender-for-resource-manager-introduction/opening-azure-activity-log.png" alt-text="Comment ouvrir le journal d’activité Azure":::

1. Filtrez les événements sur :
    - L’abonnement mentionné dans l’alerte
    - La plage de temps de l’activité détectée
    - Le compte d’utilisateur associé (le cas échéant)

1. Recherchez des activités suspectes.

> [!TIP]
> Pour une meilleure expérience d’investigation plus complète, envoyez en streaming vos journaux d’activité Azure à Azure Sentinel, comme décrit dans [Connecter des données à partir du journal d’activité Azure](../sentinel/connect-azure-activity.md).



## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert Azure Defender pour Resource Manager. Pour des informations connexes, consultez l’article suivant : 

- Les alertes de sécurité peuvent être générées par Security Center, ou reçues par Security Center depuis d’autres produits de sécurité. Pour exporter toutes ces alertes vers Azure Sentinel, un système SIEM tiers ou tout autre outil externe, suivez les instructions indiquées dans [Exportation d’alertes vers un système SIEM](continuous-export.md).

- > [!div class="nextstepaction"]
    > [Activer Azure Defender](security-center-pricing.md)