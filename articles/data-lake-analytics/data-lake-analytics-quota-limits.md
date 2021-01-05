---
title: Ajuster les quotas et limites dans Azure Data Lake Analytics
description: Découvrez comment ajuster et augmenter les quotas et limites dans les comptes Azure Data Lake Analytics (ADLA).
ms.service: data-lake-analytics
author: omidm1
ms.author: omidm
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 03/15/2018
ms.openlocfilehash: fd032235f286b5db1930e9c9c6d730b5424aa4eb
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220837"
---
# <a name="adjust-quotas-and-limits-in-azure-data-lake-analytics"></a>Ajuster les quotas et limites dans Azure Data Lake Analytics

Découvrez comment ajuster et augmenter les quotas et limites dans les comptes Azure Data Lake Analytics (ADLA). Il vous sera plus simple d’appréhender le comportement de vos tâches U-SQL si vous connaissez ces limites. Toutes ces limites sont souples. Pour les augmenter, contactez le support Azure.

## <a name="azure-subscriptions-limits"></a>Limites des abonnements Azure

**Nombre maximal de comptes ADLA par abonnement et par région :**  5

Lorsque vous essayez de créer le sixième compte ADLA, le message d’erreur suivant s’affiche : « Vous avez atteint le nombre maximal de comptes Data Lake Analytics autorisés (5) dans (zone) sous l’abonnement (nom). »

Si vous souhaitez aller au-delà de cette limite, vous pouvez essayer ces options :

- choisissez une autre région le cas échéant
- contactez le support technique Azure en [ouvrant un ticket de support](#increase-maximum-quota-limits) afin de demander une augmentation du quota.

## <a name="default-adla-account-limits"></a>Limites par compte ADLA par défaut

**Nombre maximal d’unités Analytique par compte :** 250, par défaut : 32

Il s’agit du nombre maximal d’unités Analytics pouvant s’exécuter simultanément dans votre compte. Si le nombre total d’unités Analytics exécutées dans l’ensemble des tâches dépasse cette limite, les tâches les plus récentes sont automatiquement placées dans la file d’attente. Par exemple :

- Si une seule tâche est exécutée avec 32 unités Analytics, lorsque vous envoyez une deuxième tâche, celle-ci est placée dans la file d’attente jusqu’à ce que la première tâche soit terminée.
- Si vous avez déjà quatre tâches en cours d’exécution et que chacune utilise 8 unités Analytics, lorsque vous envoyez une cinquième tâche nécessitant 8 unités Analytics, celle-ci est placée dans la file d’attente jusqu’à ce que les 8 unités Analytics soient disponibles.

    ![Page limites et quota d’Azure Data Lake Analytics](./media/data-lake-analytics-quota-limits/adjust-quota-limits.png)

**Nombre maximal d’unités Analytique par travail :** 250, par défaut : 32

Il s’agit du nombre maximal d’unités Analytics pouvant être assignées à chaque tâche dans votre compte. Les tâches auxquelles est assigné un nombre supérieur à cette limite sont rejetées, à moins que l’expéditeur soit concerné par une stratégie de calcul (limite d’envoi de travaux) qui lui accorde davantage d’unités Analytics par tâche. La limite supérieure de cette valeur correspond à la limite d’unités Analytics pour le compte.

**Nombre maximal de travaux U-SQL simultanés par compte :** 20

Il s’agit du nombre maximal de tâches pouvant s’exécuter simultanément dans votre compte. Au-dessus de cette valeur, les tâches les plus récentes sont automatiquement placées dans la file d’attente.

## <a name="adjust-adla-account-limits"></a>Ajuster les limites par compte ADLA

1. Connectez-vous au [Portail Azure](https://portal.azure.com).
2. Choisissez un compte ADLA existant.
3. Cliquez sur **Propriétés** .
4. Ajustez les valeurs **Nombre maximal d’unités Analytics** , **Nombre maximal de travaux en cours d’exécution** et **Limites d’envoi de travaux** en fonction de vos besoins.

## <a name="increase-maximum-quota-limits"></a>Augmenter les limites de quota maximales

Vous trouverez plus d’informations sur les limites d’Azure dans la [documentation sur les limites spécifiques au service Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#data-lake-analytics-limits).

1. Ouvrez une demande de support dans le portail Azure.

   ![Portail Azure Data Lake Analytics – Aide et support](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-help-support.png)

   ![Portail Azure Data Lake Analytics – Nouvelle demande de support](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request.png)

2. Sélectionnez **Quota** comme type de problème.

3. Sélectionnez votre **Abonnement** (vérifiez qu’il ne s’agit pas d’une version d’essai).

4. Sélectionnez le type de quota **Data Lake Analytics** .

   ![Type de quota de la demande de support Azure Data Lake Analytics](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request-basics.png)

5. Dans la page du problème, expliquez le motif de votre demande d’augmentation de limite. Indiquez pourquoi vous avez besoin de cette capacité supplémentaire dans la zone **Détails** .

   ![Détails de la demande de support Azure Data Lake Analytics](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request-details.png)

6. Vérifiez vos informations de contact et créez la demande de support.

Microsoft examine votre demande et essaie de répondre aux besoins de votre activité dans les plus brefs délais.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d'ensemble de Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
- [Gestion d'Azure Data Lake Analytics à l'aide d'Azure PowerShell](data-lake-analytics-manage-use-powershell.md)
- [Surveiller et résoudre les problèmes des tâches Azure Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
