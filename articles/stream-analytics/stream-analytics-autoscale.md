---
title: Mettre automatiquement à l’échelle des travaux Stream Analytics
description: Cet article explique comment mettre automatiquement à l’échelle un travail Stream Analytics selon une planification prédéfinie ou les valeurs de métriques de travail.
author: sidramadoss
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/03/2020
ms.openlocfilehash: 8e5bcdaeaf1ec99387a708199f4353736b6bc60f
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93129845"
---
# <a name="autoscale-stream-analytics-jobs-using-azure-automation"></a>Mettre automatiquement à l’échelle des travaux de Stream Analytics avec Azure Automation

Vous pouvez optimiser le coût de vos travaux Stream Analytics en configurant la mise à l’échelle automatique. La mise à l’échelle automatique augmente ou diminue le nombre d’unités de streaming (SU) de votre travail pour refléter le changement de votre charge d’entrée. Au lieu de surprovisionner votre travail, vous pouvez effectuer un scale-up ou un scale-down en fonction des besoins. Il existe deux façons de configurer vos travaux pour la mise à l’échelle automatique :
1. **Prédéfinir une planification** quand votre charge d’entrée est prévisible. Par exemple, vous vous attendez à un taux plus élevé d’événements d’entrée pendant la journée et vous souhaitez exécuter votre travail avec plus de SU.
2. **Déclencher des opérations de scale-up ou de scale-down en fonction des métriques de travail** si votre charge d’entrée n’est pas prévisible. Vous pouvez changer dynamiquement le nombre de SU en fonction de vos métriques de travail, comme le nombre d’événements d’entrée ou d’événements d’entrée dans le backlog.

## <a name="prerequisites"></a>Prérequis
Avant de commencer à configurer la mise à l’échelle automatique de votre travail, effectuez les étapes suivantes.
1. Votre travail est optimisé avec une [topologie parallèle](./stream-analytics-parallelization.md). Si vous pouvez changer l’échelle de votre travail pendant qu’elle s’exécute, votre travail a une topologie parallèle et peut être configurée pour la mise à l’échelle automatique.
2. [Créez un compte Azure Automation](../automation/automation-create-standalone-account.md) avec l’option « RunAsAccount » activée. Ce compte doit avoir les autorisations nécessaires pour gérer vos travaux Stream Analytics.

## <a name="set-up-azure-automation"></a>Configurer Azure Automation
### <a name="configure-variables"></a>Configurer des variables
Ajoutez les variables suivantes dans le compte Azure Automation. Ces variables sont utilisées dans les runbooks décrits dans les étapes suivantes.

| Nom | Type | Valeur |
| --- | --- | --- |
| **jobName** | String | Nommez le travail Stream Analytics que vous souhaitez mettre à l’échelle automatiquement. |
| **resourceGroupName** | String | Nom du groupe de ressources dans lequel se trouve votre travail. |
| **subId** | String | ID de l’abonnement dans lequel se trouve votre travail. |
| **increasedSU** | Integer | Valeur SU supérieure vers laquelle vous souhaitez mettre à l’échelle votre travail dans une planification. Cette valeur doit figurer parmi les options SU valides que vous voyez dans les paramètres **Échelle** de votre travail pendant son exécution. |
| **decreasedSU** | Integer | Valeur SU inférieure vers laquelle vous souhaitez mettre à l’échelle votre travail dans une planification. Cette valeur doit figurer parmi les options SU valides que vous voyez dans les paramètres **Échelle** de votre travail pendant son exécution. |
| **maxSU** | Integer | Valeur SU maximale vers laquelle vous souhaitez mettre à l’échelle votre travail par incréments lors de la mise à l’échelle automatique par charge. Cette valeur doit figurer parmi les options SU valides que vous voyez dans les paramètres **Échelle** de votre travail pendant son exécution. |
| **minSU** | Integer | Valeur SU minimale vers laquelle vous souhaitez mettre à l’échelle votre travail par incréments lors de la mise à l’échelle automatique par charge. Cette valeur doit figurer parmi les options SU valides que vous voyez dans les paramètres **Échelle** de votre travail pendant son exécution. |

![Ajouter des variables dans Azure Automation](./media/autoscale/variables.png)

### <a name="create-runbooks"></a>Création de runbooks
L’étape suivante consiste à créer deux runbooks PowerShell : un pour le scale-up et un pour le scale-down.
1. Dans votre compte Azure Automation, accédez à **Runbooks** sous **Automatisation des processus** , puis sélectionnez **Créer un runbook**.
2. Nommez le premier runbook *ScaleUpRunbook* et choisissez le type PowerShell. Utilisez le [script PowerShell ScaleUpRunbook](https://github.com/Azure/azure-stream-analytics/blob/master/Autoscale/ScaleUpRunbook.ps1) disponible dans GitHub. Enregistrez-le et publiez-le.
3. Créez un autre runbook appelé *ScaleDownRunbook* de type PowerShell. Utilisez le [script PowerShell ScaleDownRunbook](https://github.com/Azure/azure-stream-analytics/blob/master/Autoscale/ScaleDownRunbook.ps1) disponible dans GitHub. Enregistrez-le et publiez-le.

![Mettre automatiquement à l’échelle des runbooks dans Azure Automation](./media/autoscale/runbooks.png)

Vous disposez maintenant de runbooks qui peuvent déclencher automatiquement des opérations de scale-up et de scale-down sur votre travail Stream Analytics. Ces runbooks peuvent être déclenchés selon une planification prédéfinie ou définis de manière dynamique en fonction de métriques de travail.

## <a name="autoscale-based-on-a-schedule"></a>Mise à l’échelle automatique en fonction d’une planification
Azure Automation vous permet de configurer une planification pour déclencher vos runbooks.
1. Dans votre compte Azure Automation, sélectionnez **Planifications** sous **Ressources partagées**. Sélectionnez ensuite **Ajouter une planification**.
2. Par exemple, vous pouvez créer deux planifications : une indiquant quand effectuer un scale-up de votre travail et une autre indiquant quand effectuer un scale-down de votre travail. Vous pouvez définir une périodicité pour ces planifications.

   ![Planifications dans Azure Automation](./media/autoscale/schedules.png)

3. Ouvrez votre **ScaleUpRunbook** , puis sélectionnez **Planifications** sous **Ressources**. Vous pouvez ensuite lier votre runbook à une planification que vous avez créée dans les étapes précédentes. Vous pouvez avoir plusieurs planifications liées au même runbook, ce qui peut être utile si vous souhaitez exécuter la même opération de mise à l’échelle à des moments différents de la journée.

![Planification de runbooks dans Azure Automation](./media/autoscale/schedulerunbook.png)

1. Répétez l’étape précédente pour **ScaleDownRunbook**.

## <a name="autoscale-based-on-load"></a>Mise à l’échelle automatique selon la charge
Il peut arriver que vous ne puissiez pas prédire la charge d’entrée. Dans ce cas, il est plus efficace d’effectuer un scale-up/down par incréments avec des limites minimale et maximale. Vous pouvez configurer des règles d’alerte dans vos travail Stream Analytics pour déclencher des runbooks quand les métriques de travail passent au-dessus ou en dessous d’un certain seuil.
1. Dans votre compte Azure Automation, créez deux autres variables de type Integer appelées **minSU** et **maxSU**. Ces variables définissent les limites au sein desquelles votre travail sera mis à l’échelle par incréments.
2. Créez deux autres runbooks. Vous pouvez utiliser le [script PowerShell StepScaleUp](https://github.com/Azure/azure-stream-analytics/blob/master/Autoscale/StepScaleUp.ps1) qui augmente les SU de votre travail par incréments jusqu’à la valeur **maxSU**. Vous pouvez également utiliser le [script PowerShell StepScaleDown](https://github.com/Azure/azure-stream-analytics/blob/master/Autoscale/StepScaleDown.ps1) qui réduit les SU de votre travail par incréments jusqu’à ce que la valeur **minSU** soit atteinte. Vous pouvez également utiliser les runbooks de la section précédente si vous souhaitez mettre à l’échelle votre travail selon des valeurs SU spécifiques.
3. Dans votre travail Stream Analytics, sélectionnez **Règles d’alerte** sous **Supervision**. 
4. Créez deux groupes d’actions : un pour l’opération de scale-up et un autre pour l’opération de scale-down. Sélectionnez **Gérer les actions** , puis cliquez sur **Ajouter un groupe d’actions**. 
5. Renseignez les champs obligatoires. Choisissez **Runbook Automation** quand vous sélectionnez le **Type d’action**. Sélectionnez le runbook à déclencher quand l’alerte se déclenche. Ensuite, créez le groupe d’actions.

   ![Créer un groupe d’actions](./media/autoscale/create-actiongroup.png)
6. Créez une [**Nouvelle règle d’alerte**](./stream-analytics-set-up-alerts.md#set-up-alerts-in-the-azure-portal) dans votre travail. Spécifiez une condition basée sur une métrique de votre choix. [*Événements d’entrée* , *% d’utilisation de SU* et *Événements d’entrée en backlog*](./stream-analytics-monitoring.md#metrics-available-for-stream-analytics) sont les métriques recommandées pour définir la logique de mise à l’échelle automatique. Il est également recommandé d’utiliser une *Précision d’agrégation* et une *Fréquence d’évaluation* de 1 minute pour le déclenchement d’opérations de scale-up. De cette façon, votre travail dispose de suffisamment de ressources pour faire face aux pics importants du volume d’entrée.
7. Sélectionnez le groupe d’actions créé à la dernière étape, puis créez l’alerte.
8. Répétez les étapes 2 à 4 pour toutes les opérations de mise à l’échelle supplémentaires que vous souhaitez déclencher en fonction de la condition des métriques dy travail.

Il est recommandé d’exécuter des tests de mise à l’échelle avant d’exécuter votre travail en production. En testant votre travail par rapport à différentes charges d’entrée, vous pouvez vous faire une idée du nombre de SU dont il a besoin pour des débits d’entrée différents. Cela peut vous aider à définir dans vos règles d’alerte les conditions qui déclenchent les opérations de scale-up et de scale-down. 

## <a name="next-steps"></a>Étapes suivantes
* [Créer des requêtes parallélisables dans Azure Stream Analytics](stream-analytics-parallelization.md)
* [Mettre à l’échelle des travaux Azure Stream Analytics pour augmenter le débit](stream-analytics-scale-jobs.md)