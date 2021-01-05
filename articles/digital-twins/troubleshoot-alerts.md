---
title: Configurer des alertes
titleSuffix: Azure Digital Twins
description: Découvrez comment activer des alertes sur les métriques Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 7/28/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 057a1b08095a71e2fca6119f5b63bfdc787eea88
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94616600"
---
# <a name="troubleshooting-azure-digital-twins-alerts"></a>Dépannage d’Azure Digital Twins : Alertes

Azure Digital Twins collecte des [métriques](troubleshoot-metrics.md) pour votre instance de service qui fournissent des informations sur l’état de vos ressources. Vous pouvez utiliser ces métriques pour évaluer l’intégrité globale du service Azure Digital Twins et des ressources qui y sont connectées.

Les **alertes** vous avertissent de façon proactive lorsque des conditions significatives sont détectées dans vos données de métriques. Elles permettent d’identifier et de résoudre les problèmes avant que les utilisateurs de votre système ne les remarquent. Pour en savoir plus sur les alertes, consultez la rubrique [*Vue d’ensemble des alertes dans Microsoft Azure*](../azure-monitor/platform/alerts-overview.md).

## <a name="turn-on-alerts"></a>Activer les alertes

Voici comment activer des alertes pour votre instance Azure Digital Twins :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre instance Azure Digital Twins. Vous pouvez la trouver en tapant son nom dans la barre de recherche du portail. 

2. Sélectionnez **Alertes** dans le menu, puis **+ Nouvelle règle d’alerte**.

3. Dans la page *Créer une règle d’alerte* qui suit, vous pouvez suivre les invites pour définir les conditions, les actions à déclencher et les détails de l’alerte.     
    * Les informations **Étendue** doivent être renseignées automatiquement avec les informations de votre instance
    * Vous allez définir les informations **Condition** et **Groupe d’actions** pour personnaliser les déclencheurs d’alerte et les réponses
    * Dans la section **Détails de la règle d’alerte**, entrez un nom et une description (facultative) pour votre règle. Vous pouvez cocher la case _Activer la règle d’alerte lors de la création_ si vous souhaitez que l’alerte devienne active dès qu’elle est créée.
        - C’est également là que vous sélectionnez un _groupe de ressources_ et un niveau de _gravité_.

4. Sélectionnez le bouton _Créer une règle d’alerte_ pour créer votre règle d’alerte.

:::image type="content" source="media/troubleshoot-alerts/create-alert-rule.png" alt-text="Capture d’écran montrant la page Créer une règle d’alerte avec des sections pour l’étendue, la condition, groupe d’actions et les détails de la règle d’alerte" lightbox="media/troubleshoot-alerts/create-alert-rule.png":::

Pour une procédure pas à pas guidée de la saisie de ces champs, consultez [*Vue d’ensemble des alertes dans Microsoft Azure*](../azure-monitor/platform/alerts-overview.md). Voici quelques exemples de ce à quoi ressemblera les étapes pour Azure Digital Twins.

### <a name="select-conditions"></a>Sélectionner les conditions

Voici un extrait du processus *Sélectionner une condition* illustrant les types de signaux d’alerte disponibles pour Azure Digital Twins. Sur cette page, vous pouvez filtrer le type de signal et sélectionner le signal souhaité dans une liste.

:::image type="content" source="media/troubleshoot-alerts/configure-signal-logic.png" alt-text="Capture d’écran montrant la première page Configurer la logique du signal. La zone Type de signal est mise en surbrillance pour sélectionner des métriques ou des journaux d’activité, ainsi qu’une liste des métriques que vous pouvez sélectionner en dessous":::

Après avoir sélectionné un signal, vous êtes invité à configurer la logique de l’alerte. Vous pouvez filtrer sur une dimension, définir une valeur seuil pour votre alerte et définir la fréquence des vérifications pour la condition. Voici un exemple de configuration d’une alerte lorsque la métrique du taux moyen d’échec de routage dépasse 5 %.

:::image type="content" source="media/troubleshoot-alerts/configure-signal-logic-2.png" alt-text="Capture d’écran montrant la deuxième page Configurer la logique du signal. La page affiche l’historique des métriques, a une zone à filtrer sur une dimension telle que les opérations Event Grid et une section pour la définition de la logique d’alerte telle que « moyenne est supérieure à 5 ».":::

### <a name="verify-success"></a>Vérifier la réussite de l’exécution

Une fois les alertes configurées, elles s’affichent sur la page *Alertes* de votre instance.
 
:::image type="content" source="media/troubleshoot-alerts/alerts-post.png" alt-text="Capture d’écran montrant la page Alertes et le bouton à ajouter. Une alerte est configurée" lightbox="media/troubleshoot-alerts/alerts-post.png":::

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations concernant les alertes dans Azure Monitor, consultez [*Vue d’ensemble des alertes dans Microsoft Azure*](../azure-monitor/platform/alerts-overview.md).
* Pour plus d’informations sur les métriques d’Azure Digital Twins, consultez [*Dépannage : Afficher les métriques avec Azure Monitor*](troubleshoot-metrics.md).
* Pour savoir comment activer la journalisation des diagnostics pour vos métriques, consultez [*Résolution des problèmes : Configurer les diagnostics*](troubleshoot-diagnostics.md).
