---
title: Configurer Azure Sentinel pour Defender pour IoT (préversion)
description: Explique comment configurer Azure Sentinel pour recevoir des données de votre solution Defender pour IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: c5c0f74ed8a5688b20eea4e74f747d3ff6dd0e63
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96339997"
---
# <a name="connect-your-data-from-defender-for-iot-to-azure-sentinel-preview"></a>Connecter vos données de Defender pour IoT à Azure Sentinel (préversion)

Le connecteur de données Azure Defender pour IoT dans Azure Sentinel est actuellement disponible en préversion publique. Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/).

Dans ce guide, découvrez comment connecter vos données Defender pour IoT à Azure Sentinel.

> [!div class="checklist"]
> * Prérequis
> * Paramètres de connexion
> * Affichage des alertes Log Analytics

Connectez les alertes de Defender pour IoT et diffusez-les directement dans Azure Sentinel.

En intégrant plus étroitement Azure Defender pour IoT à Azure Sentinel, le premier SIEM natif du cloud et le premier SIEM avec sécurité IoT et OT native, Microsoft propose une approche plus simple de la sécurité unifiée sur les réseaux informatiques et industriels. Combinée à l’apprentissage automatique d’Azure Sentinel, cette intégration permet aux organisations de détecter rapidement les attaques multiphases qui, souvent, franchissent les limites de l’informatique et de la formation organisationnelle. En outre, l’intégration d’Azure Defender pour IoT aux capacités d’orchestration, d’automatisation et de réponse de la sécurité (SOAR) d’Azure Sentinel permet une réponse et une prévention automatisées grâce à des playbooks intégrés optimisés pour la formation organisationnelle. 

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer des autorisations de **lecture** et d’**écriture** pour l’espace de travail.
- **Defender pour IoT** doit être **activé** sur vos hubs IoT correspondants.
- Vous devez disposer des autorisations de **lecture** et d’**écriture** sur l’**IoT Hub** auquel vous souhaitez vous connecter.
- Vous devez également avoir des autorisations de **lecture** et d’**écriture** sur le **groupe de ressources IoT Hub**.


## <a name="connect-to-defender-for-iot"></a>Se connecter à Defender pour IoT

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données** puis cliquez sur la vignette **Defender pour IoT**.
1. Au bas du volet droit, cliquez sur **Ouvrir la page des connecteurs**.
1. Cliquez sur **Se connecter** à côté de chaque abonnement IoT Hub dont vous souhaitez diffuser les alertes et les alertes d’appareil dans Azure Sentinel.
    - Si Defender pour IoT n’est pas activé sur ce hub, vous verrez s’afficher un message d’avertissement Activer. Cliquez sur le lien **Activer** pour démarrer et activer le service.
1. Vous pouvez décider si vous souhaitez que les alertes de Defender pour IoT génèrent automatiquement des incidents dans Azure Sentinel. Sous **Créer des incidents**, sélectionnez **Activer** pour permettre à la règle de créer automatiquement des incidents à partir des alertes générées.  Cette règle est modifiable sous **Analytics** > **Règles actives**.

> [!NOTE]
>L’actualisation de la liste des hubs après y avoir apporté des modifications de connexion peut prendre 10 secondes ou plus.

## <a name="using-log-analytics-for-alert-display"></a>Utilisation de Log Analytics pour l’affichage des alertes

Pour utiliser le schéma pertinent dans Log Analytics afin d’afficher les alertes Defender pour IoT :

1. Ouvrez **Journaux** > **SecurityInsights** > **SecurityAlert** ou recherchez **SecurityAlert**.
1. Filtrez les résultats pour afficher uniquement les alertes générées par Defender pour IoT à l’aide du filtre kql suivant :

```kusto
SecurityAlert | where ProductName == "Defender for IoT"
```

### <a name="service-notes"></a>Notes de service

Après vous être connecté à un IoT Hub, les données du hub sont disponibles dans Azure Sentinel environ 15 minutes plus tard.

## <a name="next-steps"></a>Étapes suivantes

Ce document vous a montré comment connecter Defender pour IoT à Azure Sentinel. Pour en savoir plus sur la détection des menaces et l’accès aux données de sécurité, consultez les articles suivants :

- Découvrez comment utiliser Azure Sentinel pour [obtenir une visibilité sur vos données et les menaces potentielles](../sentinel/quickstart-get-visibility.md).

- Découvrez comment [accéder à vos données de sécurité IoT](how-to-security-data-access.md).