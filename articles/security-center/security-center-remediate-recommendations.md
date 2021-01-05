---
title: Appliquer des recommandations dans Azure Security Center | Microsoft Docs
description: Ce article explique comment appliquer les recommandations d’Azure Security Center pour protéger vos ressources et être conforme avec les stratégies de sécurité.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 8be947cc-cc86-421d-87a6-b1e23077fd50
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/08/2020
ms.author: memildin
ms.openlocfilehash: 3d4b968d0d5e091d72cf718d58b11ade68ddf8c6
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94637084"
---
# <a name="remediate-recommendations-in-azure-security-center"></a>Appliquer les recommandations d’Azure Security Center

Les recommandations fournissent des suggestions visant à mieux sécuriser vos ressources. Pour appliquer une recommandation, vous devez suivre les étapes de correction qu’elle fournit.

## <a name="remediation-steps"></a>Étapes de correction <a name="remediation-steps"></a>

Après avoir lu toutes les recommandations, vous devez décider laquelle appliquer en premier. Nous vous recommandons de vous baser sur l’[impact du niveau de sécurité](security-center-recommendations.md#monitor-recommendations) pour hiérarchiser les actions à effectuer en priorité.

1. Dans la liste, cliquez sur la recommandation.

1. Suivez les instructions dans la section **Étapes de correction**. Chaque recommandation comprend un ensemble d’instructions. La capture d’écran suivante montre les étapes de correction permettant de configurer les applications de manière à autoriser uniquement le trafic via HTTPS.

    ![Détails de la recommandation](./media/security-center-remediate-recommendations/security-center-remediate-recommendation.png)

1. Une fois l’opération terminée, une notification s’affiche pour vous informer si la mise à jour a réussi.

## <a name="quick-fix-remediation"></a>Application d’un correctif rapide<a name="one-click"></a>

Le correctif rapide simplifie le processus de correction et vous permet d’accroître rapidement votre niveau de sécurité, ce qui entraîne ainsi une amélioration de la sécurité de votre environnement.

Le correctif rapide vous permet d’appliquer rapidement une recommandation à plusieurs ressources. 

> [!TIP]
> Le correctif rapide est disponible uniquement pour des recommandations spécifiques. Pour trouver les recommandations incluant l’option de correctif rapide, utilisez le filtre dédié en haut de la liste des recommandations :
> 
> :::image type="content" source="media/security-center-remediate-recommendations/quick-fix-filter.png" alt-text="Utilisez les filtres au-dessus de la liste des recommandations pour rechercher des recommandations incluant l’option correctif rapide":::

Pour mettre en œuvre une application rapide des correctifs :

1. Dans la liste des recommandations qui ont l’étiquette **Correctif rapide !** , cliquez sur la recommandation.

    [![Sélectionnez Correctif rapide !](media/security-center-remediate-recommendations/security-center-one-click-fix-select.png)](media/security-center-remediate-recommendations/security-center-one-click-fix-select.png#lightbox)

1. Sous l’onglet **Ressources défectueuses**, sélectionnez les ressources auxquelles vous souhaitez implémenter la recommandation, puis cliquez sur **Corriger**.

    > [!NOTE]
    > Certaines des ressources listées peuvent être désactivées, ce qui est le cas si vous n’avez pas les autorisations appropriées pour les modifier.

1. Dans la zone de confirmation, lisez les détails et les implications de la correction.

    ![Correctif rapide](./media/security-center-remediate-recommendations/security-center-one-click-fix-view.png)

    > [!NOTE]
    > Les implications sont listées dans la zone grise de la fenêtre **Corriger les ressources** qui s’ouvre après que vous avez cliqué sur **Corriger**. Elles indiquent les changements apportés si vous implémentez le correctif rapide.

1. Ajoutez les paramètres appropriés, si nécessaire, et approuvez la correction.

    > [!NOTE]
    > Une fois la correction terminée, il peut s’écouler plusieurs minutes avant que les ressources apparaissent sous l’onglet **Ressources saines**. Pour voir les actions de correction, consultez le [journal d’activité](#activity-log).

1. Une fois l’opération terminée, une notification s’affiche pour vous informer si la mise à jour a réussi.

## <a name="quick-fix-remediation-logging-in-the-activity-log"></a>Journalisation de l’application du correctif rapide dans le journal d’activité <a name="activity-log"></a>

L’opération de correction applique la configuration à la ressource à l’aide d’un déploiement de modèle ou d’un appel à l’API REST PATCH. Ces opérations sont consignées dans le [journal d’activité Azure](../azure-resource-manager/management/view-activity-logs.md).


## <a name="next-steps"></a>Étapes suivantes

Ce document vous a montré comment appliquer les recommandations fournies par Security Center. Pour plus d’informations sur Security Center, consultez les rubriques suivantes :

* [Définition des stratégies de sécurité dans Azure Security Center](tutorial-security-policy.md) : découvrez comment configurer des stratégies de sécurité pour vos groupes de ressources et abonnements Azure.
* [Surveillance de l’intégrité de la sécurité dans Azure Security Center](security-center-monitoring.md) : découvrez comment surveiller l’intégrité de vos ressources Azure.
