---
title: Comprendre l’intégrité de vos ressources
titleSuffix: Azure Digital Twins
description: Découvrez comment utiliser Azure Resource Health pour vérifier l’intégrité de votre instance Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 10/6/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: b64ee4d1538276bbbcab3fe09054f399ad20c162
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94616547"
---
# <a name="troubleshooting-azure-digital-twins-resource-health"></a>Dépannage d’Azure Digital Twins : Intégrité des ressources

[Azure Resource Health](../service-health/resource-health-overview.md) vous aide à diagnostiquer et à obtenir un support pour les problèmes de service qui affectent vos ressources Azure. Il rend compte de l’intégrité actuelle et passée de vos ressources.

Cet article vous montre comment obtenir des informations d’**intégrité des ressources** pour vos instances Azure Digital Twins.

## <a name="use-azure-resource-health"></a>Utilisation d’Azure Resource Health

Azure Resource Health peut vous aider à analyser si votre instance Azure Digital Twins est en cours d’exécution. Vous pouvez également l’utiliser pour savoir si une panne régionale a un impact sur l’intégrité de votre instance.

Pour vérifier l’intégrité de votre instance, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre instance Azure Digital Twins. Vous pouvez la trouver en tapant son nom dans la barre de recherche du portail. 

2. Dans le menu de votre instance, sélectionnez _**Intégrité des ressources**_ sous *Support + résolution des problèmes*. Cela vous permet d’accéder à la page d’affichage de l’historique d’intégrité des ressources. 

    :::image type="content" source="media/troubleshoot-resource-health/resource-health.png" alt-text="Capture d’écran montrant la page « Intégrité des ressources ». La section « Historique d’intégrité » présente un rapport quotidien pour les neuf derniers jours. Chaque jour indique un état « Disponible ».":::

Dans l’image ci-dessus, cette instance est affichée comme étant *Disponible*, et ce depuis neuf jours. Pour en savoir plus sur l’état *Disponible* et les autres types d’état qui peuvent apparaître, consultez [*Vue d’ensemble d’Azure Resource Health*](../service-health/resource-health-overview.md).

Vous pouvez également en savoir plus sur les différentes vérifications de l’intégrité des ressources pour différents types de ressources Azure dans [*Types de ressources et contrôles d’intégrité dans Azure Resource Health*](../service-health/resource-health-checks-resource-types.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez d’autres façons d’analyser votre instance Azure Digital Twins dans les articles suivants :
* [*Résolution des problèmes : Afficher les métriques avec Azure Monitor*](troubleshoot-metrics.md)
* [*Résolution des problèmes : Configurer les diagnostics*](troubleshoot-diagnostics.md).
* [*Résolution des problèmes : Configurer des alertes*](troubleshoot-alerts.md).
