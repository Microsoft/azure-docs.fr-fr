---
title: 'Tutoriel : Gérer le calcul avec Azure Functions'
description: Découvrez comment utiliser Azure Functions pour gérer le calcul de votre pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/27/2018
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: f0731f0deaf46ec419cfe43037804e10f2b73fd4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96448380"
---
# <a name="use-azure-functions-to-manage-compute-resources-for-your-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Utiliser Azure Functions pour gérer les ressources de calcul de votre pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics

Ce tutoriel utilise Azure Functions pour gérer les ressources de calcul d’un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.

Pour utiliser un application de fonction Azure avec un pool SQL dédié (anciennement SQL DW), vous devez créer un [compte de principal de service](../../active-directory/develop/howto-create-service-principal-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Le compte de principal de service doit être doté d’un accès contributeur sous le même abonnement que votre instance de pool SQL dédié (anciennement SQL DW).

## <a name="deploy-timer-based-scaling-with-an-azure-resource-manager-template"></a>Déployer une mise à l’échelle basée sur un minuteur à l’aide d’un modèle Azure Resource Manager

Pour déployer le modèle, vous avez besoin des informations suivantes :

- Nom du groupe de ressources dans lequel se trouve votre instance de pool SQL dédié (anciennement SQL DW)
- Nom du serveur sur lequel se trouve votre instance de pool SQL dédié (anciennement SQL DW)
- Nom de votre instance de pool SQL dédié (anciennement SQL DW)
- ID de locataire (ID du répertoire) de votre annuaire Azure Active Directory
- Identifiant d’abonnement
- ID d'application du principal de service
- Clé secrète du principal de service

Une fois que vous avez les informations ci-dessus, déployez ce modèle :

[![Image affichant le bouton intitulé « Déployer sur Azure ».](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Fsql-data-warehouse-samples%2Fmaster%2Farm-templates%2FsqlDwTimerScaler%2Fazuredeploy.json)

Une fois que vous avez déployé le modèle, vous devez trouver trois nouvelles ressources : un plan Azure App Service gratuit, un plan Function App basé sur la consommation et un compte de stockage qui gère la journalisation et la file d’attente des opérations. Continuez à lire les autres sections pour savoir comment modifier les fonctions déployées pour s’ajuster à vos besoins.

## <a name="change-the-compute-level"></a>Modifier le niveau de calcul

1. Accédez à votre service Function App. Si vous avez déployé le modèle avec les valeurs par défaut, ce service doit être nommé *DWOperations*. Une fois que votre Function App est ouverte, vous devriez remarquer que cinq fonctions sont déployées dans votre service Function App.

   ![Fonctions déployées avec modèle](./media/manage-compute-with-azure-functions/five-functions.png)

2. Sélectionnez *DWScaleDownTrigger* ou *DWScaleUpTrigger* pour effectuer un scale-up ou un scale-down. Dans le menu déroulant, sélectionnez Intégrer.

   ![Sélectionnez Intégrer pour la fonction](./media/manage-compute-with-azure-functions/select-integrate.png)

3. À présent, la valeur affichée devrait indiquer *%ScaleDownTime%* ou *%ScaleUpTime%* . Ces valeurs indiquent que la planification se base sur les valeurs définies dans les [Paramètres d’application](../../azure-functions/functions-how-to-use-azure-function-app-settings.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Vous pouvez ignorer cette valeur pour le moment et définir la planification sur votre heure de préférence en fonction des étapes suivantes.

4. Dans la zone de planification, ajoutez l’expression CRON pour refléter la fréquence à laquelle vous souhaitez qu’Azure Synapse Analytics fasse l’objet d’un scale-up.

   ![Modifier la planification de fonction](./media/manage-compute-with-azure-functions/change-schedule.png)

   La valeur de `schedule` est une [expression CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression) qui contient les six champs suivants :

   ```json
   {second} {minute} {hour} {day} {month} {day-of-week}
   ```

   Par exemple, *« 0 30 9 * * 1-5 »* reflète un déclencheur qui se produit chaque jour de la semaine à 9h30. Pour plus d’informations, voir les [Exemples de planification](../../azure-functions/functions-bindings-timer.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#example) Azure Functions.

## <a name="change-the-time-of-the-scale-operation"></a>Modifier l’heure de l’opération de mise à l’échelle

1. Accédez à votre service Function App. Si vous avez déployé le modèle avec les valeurs par défaut, ce service doit être nommé *DWOperations*. Une fois que votre Function App est ouverte, vous devriez remarquer que cinq fonctions sont déployées dans votre service Function App.

2. Sélectionnez *DWScaleDownTrigger* ou *DWScaleUpTrigger* pour effectuer un scale-up ou scale-down de la valeur de calcul. Lors de la sélection des fonctions, le volet devrait afficher le fichier *index.js*.

   ![Modifiez le niveau de calcul du déclencheur de fonction](././media/manage-compute-with-azure-functions/index-js.png)

3. Définissez la valeur de *ServiceLevelObjective* sur le niveau souhaité et sélectionnez Enregistrer. *ServiceLevelObjective* correspond au niveau de calcul sur lequel votre instance d’entrepôt de données est mise à l’échelle en fonction de la planification définie dans la section Intégrer.

## <a name="use-pause-or-resume-instead-of-scale"></a>Utiliser mise en pause ou reprise au lieu de mise à l'échelle

Actuellement, les fonctions activées par défaut sont *DWScaleDownTrigger* et *DWScaleUpTrigger*. Si vous préférez utiliser les fonctionnalités de mise en pause et de reprise, vous pouvez activer *DWPauseTrigger* ou *DWResumeTrigger*.

1. Accédez au volet Fonctions.

   ![Volet Fonctions](./media/manage-compute-with-azure-functions/functions-pane.png)

2. Sélectionnez le bouton bascule correspondant aux déclencheurs que vous souhaitez activer.

3. Accédez aux onglets *Intégrer* des déclencheurs respectifs pour modifier leur planification.

   > [!NOTE]
   > La différence fonctionnelle entre les déclencheurs de mise à l’échelle et les déclencheurs de pause/reprise réside dans le message envoyé à la file d’attente. Pour plus d’informations, voir [Ajouter une nouvelle fonction de déclencheur](manage-compute-with-azure-functions.md#add-a-new-trigger-function).

## <a name="add-a-new-trigger-function"></a>Ajouter une nouvelle fonction de déclenchement

Actuellement, il n’y a que deux fonctions de mise à l’échelle incluses dans le modèle. Avec ces fonctions, vous pouvez uniquement réduire et augmenter l’échelle une seule fois par jour. Pour un contrôle plus précis, par exemple, pour réduire l’échelle plusieurs fois par jour ou pour avoir un comportement de mise à l’échelle différent les week-ends, vous devez ajouter un autre déclencheur.

1. Créez une nouvelle fonction vide. Sélectionnez le bouton *+* proche de l’emplacement de votre fonction pour faire apparaître le volet de modèle de fonction.

   ![Capture d’écran du menu « Applications de fonction » avec l’icône « Plus » à côté de « Fonctions » sélectionnée](./media/manage-compute-with-azure-functions/create-new-function.png)

2. Sous Langage, sélectionnez *JavaScript*, puis *TimerTrigger*.

   ![Créer une fonction](./media/manage-compute-with-azure-functions/timertrigger-js.png)

3. Nommez votre fonction et définissez votre planification. L’illustration montre un exemple de déclenchement de fonction tous les samedis à minuit (dans la nuit de vendredi à samedi).

   ![Mise à l’échelle inférieure samedi](./media/manage-compute-with-azure-functions/scale-down-saturday.png)

4. Copiez le contenu de *index.js* parmi les autres fonctions de déclencheur.

   ![Copiez index.js](././media/manage-compute-with-azure-functions/index-js.png)

5. Définissez la variable de l’opération sur le comportement souhaité comme suit :

   ```JavaScript
   // Resume the dedicated SQL pool (formerly SQL DW) instance
   var operation = {
       "operationType": "ResumeDw"
   }

   // Pause the dedicated SQL pool (formerly SQL DW) instance
   var operation = {
       "operationType": "PauseDw"
   }

   // Scale the dedicated SQL pool (formerly SQL DW)l instance to DW600c
   var operation = {
       "operationType": "ScaleDw",
       "ServiceLevelObjective": "DW600c"
   }
   ```

## <a name="complex-scheduling"></a>Planification complexe

Cette section décrit ce dont vous avez besoin pour réaliser une planification plus complexe des fonctionnalités de mise en pause, de reprise et de mise à l’échelle.

### <a name="example-1"></a>Exemple 1

Montée en puissance quotidienne à 8 h 00 jusqu’à DW600c et descente en puissance à 20 h 00 jusqu’à DW200c.

| Fonction  | Planifier     | Opération                                |
| :-------- | :----------- | :--------------------------------------- |
| Fonction1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW600c"}` |
| Fonction2 | 0 0 20 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200c"}` |

### <a name="example-2"></a>Exemple 2

Montée en puissance quotidienne à 8h00 jusqu’à DW1000c, descente en puissance à 16h00 jusqu’à DW600 et descente en puissance à 22h00 jusqu’à DW200c.

| Fonction  | Planifier     | Opération                                |
| :-------- | :----------- | :--------------------------------------- |
| Fonction1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000c"}` |
| Fonction2 | 0 0 16 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600c"}` |
| Fonction3 | 0 0 22 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200c"}` |

### <a name="example-3"></a>Exemple 3

Scale-up à 8 h 00 jusqu’à DW1000c, scale-down unique jusqu’à DW600c à 16 h 00 les jours ouvrables. Mise en pause le vendredi à 23 h 00, reprise à 7 h 00 le lundi matin.

| Fonction  | Planifier       | Opération                                |
| :-------- | :------------- | :--------------------------------------- |
| Fonction1 | 0 0 8 * * 1-5  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000c"}` |
| Fonction2 | 0 0 16 * * 1-5 | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600c"}` |
| Fonction3 | 0 0 23 * * 5   | `var operation = {"operationType": "PauseDw"}` |
| Fonction4 | 0 0 7 * * 1    | `var operation = {"operationType": "ResumeDw"}` |

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur les fonctions Azure [Déclencheur de minuteur](../../azure-functions/functions-create-scheduled-function.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

Consultez le [référentiel d’exemples](https://github.com/Microsoft/sql-data-warehouse-samples) du pool SQL dédié (anciennement SQL DW).
