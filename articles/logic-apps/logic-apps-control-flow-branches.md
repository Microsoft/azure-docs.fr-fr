---
title: Créer ou joindre des branches parallèles pour des actions dans des flux de travail
description: Découvrez comment créer ou fusionner des branches en cours d’exécution parallèle pour des actions de flux de travail indépendantes dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/10/2018
ms.openlocfilehash: 3514ce966e1de9af1741de6b966964aca2599610
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91269231"
---
# <a name="create-or-join-parallel-branches-for-workflow-actions-in-azure-logic-apps"></a>Créer ou joindre des branches parallèles pour des actions de workflow dans Azure Logic Apps

Par défaut, vos actions dans les workflows d’application logique s’exécutent de manière séquentielle. Pour effectuer des actions indépendantes en même temps, vous pouvez créer des [branches parallèles](#parallel-branches), puis [joindre ces branches](#join-branches) plus tard dans votre flux. 

> [!TIP] 
> Si vous disposez d’un déclencheur qui reçoit un tableau et souhaite exécuter un workflow pour chaque élément du tableau, vous pouvez *dégrouper* ce tableau avec le déclencheur de propriété [**SplitOn**](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/). 

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

<a name="parallel-branches"></a>

## <a name="add-parallel-branch"></a>Ajouter une branche parallèle

Pour exécuter des étapes indépendantes en même temps, vous pouvez ajouter des branches parallèles en regard d’une étape existante. 

![Exécuter des étapes en parallèle](media/logic-apps-control-flow-branches/parallel.png)

Votre application logique attend que toutes les branches se terminent avant de poursuivre le flux de travail. Les branches parallèles s’exécutent uniquement si leurs valeurs de propriété `runAfter` correspondent à l’état de l’étape parente terminée. Par exemple, les deux branches `branchAction1` et `branchAction2` sont définies pour s’exécuter uniquement lorsque l’étape `parentAction` est terminée avec l’état `Succeeded`.

> [!NOTE]
> Avant que vous ne commenciez, votre application logique doit déjà disposer d’une étape dans laquelle vous pouvez ajouter des branches parallèles.

1. Dans le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Placez le pointeur sur la flèche au-dessus de l’étape où vous souhaitez ajouter des branches parallèles. Cliquez sur le signe **plus** ( **+** ) qui s’affiche, puis choisissez **Ajouter une branche parallèle**. 

   ![Ajouter une branche parallèle](media/logic-apps-control-flow-branches/add-parallel-branch.png)

1. Dans la zone de recherche, recherchez et sélectionnez l’action souhaitée.

   ![Capture d’écran montrant la fenêtre « Choisir une action » dans le Concepteur d’application logique.](media/logic-apps-control-flow-branches/find-select-parallel-action.png)

   Maintenant, votre action sélectionnée s’affiche dans la branche parallèle, par exemple :

   ![Rechercher et sélectionner l’action souhaitée](media/logic-apps-control-flow-branches/added-parallel-branch.png)

1. Dans chaque branche parallèle, ajoutez les étapes de votre choix. Pour ajouter une autre action à une branche, déplacez le pointeur sous l’action où vous souhaitez ajouter une action séquentielle. Cliquez sur le signe **plus** ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

   ![Ajouter une action séquentielle à une branche parallèle](media/logic-apps-control-flow-branches/add-sequential-action.png)

1. Dans la zone de recherche, recherchez et sélectionnez l’action souhaitée.

   ![Capture d’écran montrant la fenêtre « Choisir une action » et la zone de recherche dans le Concepteur d’application logique.](media/logic-apps-control-flow-branches/find-select-sequential-action.png)

   Maintenant, votre action sélectionnée s’affiche dans la branche actuelle, par exemple :

   ![Rechercher et sélectionner une action séquentielle](media/logic-apps-control-flow-branches/added-sequential-action.png)

Pour fusionner à nouveau des branches, [joignez vos branches parallèles](#join-branches). 

<a name="parallel-json"></a>

## <a name="parallel-branch-definition-json"></a>Définition de branche parallèle (JSON)

Si vous travaillez en mode Code, vous pouvez définir la structure parallèle dans la définition JSON de votre application logique à la place. Exemple :

``` json
{
  "triggers": {
    "myTrigger": {}
  },
  "actions": {
    "parentAction": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {}
    },
    "branchAction1": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "branchAction2": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

<a name="join-branches"></a>

## <a name="join-parallel-branches"></a>Joindre des branches parallèles

Pour fusionner des branches parallèles, ajoutez simplement une étape en bas, sous toutes les branches. Cette étape s’exécute à l’issue de l’exécution de toutes les branches parallèles.

![Joindre des branches parallèles](media/logic-apps-control-flow-branches/join.png)

1. Dans le [portail Azure](https://portal.azure.com), recherchez et ouvrez votre application logique dans le concepteur d’application logique. 

1. Sous les branches parallèles que vous souhaitez joindre, choisissez **Nouvelle étape**. 

   ![Ajouter une étape à joindre](media/logic-apps-control-flow-branches/add-join-step.png)

1. Dans la zone de recherche, recherchez et sélectionnez l’action que vous voulez comme étape qui joint les branches.

   ![Rechercher et sélectionner l’action qui joint les branches parallèles](media/logic-apps-control-flow-branches/join-steps.png)

   À présent, les branches parallèles sont fusionnées.

   ![Branches jointes](media/logic-apps-control-flow-branches/joined-branches.png)

<a name="join-json"></a>

## <a name="join-definition-json"></a>Définition de jointure (JSON)

Si vous travaillez en mode Code, vous pouvez définir la structure de jointure dans la définition JSON de votre application logique à la place. Exemple :

``` json
{
  "triggers": {
    "myTrigger": { }
  },
  "actions": {
    "parentAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {}
    },
    "branchAction1": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "branchAction2": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "joinAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "branchAction1": [
          "Succeeded"
        ],
        "branchAction2": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

## <a name="get-support"></a>Obtenir de l’aide

* Pour toute question, consultez la page [Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Pour voter pour des fonctionnalités et suggestions ou pour en soumettre, visitez le [site de commentaires des utilisateurs Azure Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Étapes suivantes

* [Instructions conditionnelles : Exécuter des étapes en fonction d’une condition dans des applications logiques](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Instructions switch : Exécuter différentes étapes en fonction de valeurs spécifiques](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Loops: Process arrays or repeat actions until a condition is met](../logic-apps/logic-apps-control-flow-loops.md) (Boucles : Traiter des tableaux ou répéter des actions jusqu’à ce qu’une condition soit remplie)
* [Étendues : Exécuter des étapes en fonction de l’état d’un groupe](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
