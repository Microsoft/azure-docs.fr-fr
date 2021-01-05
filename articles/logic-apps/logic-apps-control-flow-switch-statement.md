---
title: Ajouter des instructions switch à des workflows
description: Comment créer des instructions switch qui contrôlent des actions de workflow basées sur des valeurs spécifiques dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/08/2018
ms.openlocfilehash: 62f147104cf9e0e1605b02a420cb6d20190361b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89657455"
---
# <a name="create-switch-statements-that-run-workflow-actions-based-on-specific-values-in-azure-logic-apps"></a>Créer des instructions switch qui exécutent des actions de workflow basées sur des valeurs spécifiques dans Azure Logic Apps

Pour exécuter des actions spécifiques basées sur les valeurs d’objets, d’expressions ou de jetons, ajoutez une instruction *switch*,. Cette structure évalue l’objet, l’expression ou le jeton, choisit le cas qui correspond au résultat et exécute des actions spécifiques à ce cas seulement. Lors de l’exécution de l’instruction switch, un seul cas doit correspondre au résultat.

Par exemple, supposez que vous souhaitiez une application logique qui prenne des étapes différentes selon l’option sélectionnée dans l’e-mail. Dans cet exemple, l’application logique vérifie le flux RSS d’un site web pour y rechercher un nouveau contenu. Si un nouvel élément s’affiche dans le flux RSS, l’application logique envoie un e-mail à un approbateur. Selon que l’approbateur sélectionne « Approuver » ou « Rejeter », l’application logique suit des étapes différentes.

> [!TIP]
> Comme dans tous les langages de programmation, les instructions switch ne prennent en charge que les opérateurs d’égalité. Si vous avez besoin d’autres opérateurs de relation, par exemple « supérieur à », utilisez une [instruction de condition](../logic-apps/logic-apps-control-flow-conditional-statement.md).
> Pour garantir un comportement d’exécution déterministe, les cas doivent contenir une valeur statique unique et non des jetons ou des expressions dynamiques.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Pour suivre l’exemple présenté dans cet article, [créez cet exemple d’application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md) avec un compte Outlook.com ou avec un compte professionnel ou scolaire.

  1. Si vous ajoutez l’action permettant d’envoyer un e-mail, recherchez et sélectionnez plutôt cette action : **Envoyer un e-mail d’approbation**

     ![Sélectionner « Envoyer un e-mail d’approbation »](./media/logic-apps-control-flow-switch-statement/send-approval-email-action.png)

  1. Renseignez les champs obligatoires, tels que l’adresse e-mail de la personne qui reçoit l’e-mail d’approbation. 
  Sous **Options utilisateur**, entrez « Approuver, Rejeter ».

     ![Entrer les détails de l’e-mail](./media/logic-apps-control-flow-switch-statement/send-approval-email-details.png)

## <a name="add-switch-statement"></a>Ajout d’une instruction switch

1. Pour cet exemple, ajoutez une instruction switch à la fin de votre exemple de flux de travail. Après la dernière étape, choisissez **Nouvelle étape**.

   Si vous souhaitez ajouter une instruction switch entre les étapes, placez le pointeur sur la flèche là où vous souhaitez l’ajouter. Cliquez sur le **signe plus** ( **+** ) qui s’affiche, puis choisissez **Ajouter une action**.

1. Dans la zone de recherche, entrez le filtre « switch ». Sélectionnez cette action : **Switch - Contrôle**

   ![Ajout d’une instruction switch](./media/logic-apps-control-flow-switch-statement/add-switch-statement.png)

   Une instruction switch s’affiche avec un cas et un cas par défaut. 
   Par défaut, une instruction switch requiert au moins un cas plus le cas par défaut. 

   ![Instruction switch vide par défaut](./media/logic-apps-control-flow-switch-statement/empty-switch.png)

1. Cliquez dans la zone **Activé** pour afficher la liste du contenu dynamique. Dans cette liste, sélectionnez le champ **SelectedOption** dont la sortie détermine l’action à effectuer. 

   ![Sélection du champ SelectedOption](./media/logic-apps-control-flow-switch-statement/select-selected-option.png)

1. Pour gérer les cas dans lesquels l’approbateur sélectionne `Approve` ou `Reject`, ajoutez un autre cas entre **Cas** et **Par défaut**. 

   ![Ajout d’un autre cas](./media/logic-apps-control-flow-switch-statement/switch-plus.png)

1. Ajoutez ces actions aux cas correspondants :

   | N° de cas | **SelectedOption** | Action |
   |--------|--------------------|--------|
   | Cas 1 | **Approuver** | Ajoutez l’action **Envoyer un e-mail** Outlook pour envoyer des détails sur l’élément RSS uniquement si l’approbateur a sélectionné **Approuver**. |
   | Cas 2 | **Rejeter** | Ajoutez l’action **Envoyer un e-mail** Outlook pour informer les autres approbateurs que l’élément RSS a été rejeté. |
   | Default | None | Aucune action requise. Dans cet exemple, le cas **Par défaut** n’est pas renseigné, car **SelectedOption** possède uniquement deux options. |
   |||

   ![Instruction switch terminée](./media/logic-apps-control-flow-switch-statement/finished-switch.png)

1. Enregistrez votre application logique. 

   Pour tester manuellement cet exemple, choisissez **Exécuter** jusqu’à ce que l’application logique trouve un nouvel élément RSS et envoie un e-mail d’approbation. 
   Sélectionnez **Approuver** pour observer les résultats.

## <a name="json-definition"></a>Définition JSON

Maintenant que vous avez créé une application logique avec une instruction switch, examinons la définition de code générale derrière l’instruction switch.

``` json
"Switch": {
   "type": "Switch",
   "expression": "@body('Send_approval_email')?['SelectedOption']",
   "cases": {
      "Case": {
         "actions": {
           "Send_an_email": {}
         },
         "case" : "Approve"
      },
      "Case_2": {
         "actions": {
           "Send_an_email_2": {}
         },
         "case": "Reject"
      }
   },
   "default": {
      "actions": {}
   },
   "runAfter": {
      "Send_approval_email": [
         "Succeeded"
      ]
   }
}
```

| Étiquette | Description |
|-------|-------------|
| `"Switch"`         | Nom de l’instruction switch que vous pouvez modifier pour plus de lisibilité. |
| `"type": "Switch"` | Indique que l’action est une instruction switch. |
| `"expression"`     | Dans cet exemple, spécifie l’option sélectionnée par l’approbateur qui est confrontée à chacun des cas déclarés plus loin dans la définition. |
| `"cases"` | Définit un nombre quelconque de cas. Pour chaque cas, `"Case_*"` est le nom par défaut du cas donné, que vous pouvez modifier pour plus de lisibilité. |
| `"case"` | Spécifie la valeur du cas qui doit être une valeur unique et constante utilisée par l’instruction switch pour la comparaison. Si aucun cas ne correspond au résultat de l’expression switch, les actions de la section `"default"` sont exécutées. | 
| | | 

## <a name="get-support"></a>Obtenir de l’aide

* Pour toute question, consultez la page [Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Pour voter pour des fonctionnalités ou suggestions ou pour en soumettre, visitez le [site de commentaires des utilisateurs Azure Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Étapes suivantes

* [Instructions conditionnelles : Exécuter des étapes en fonction d’une condition dans des applications logiques](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Loops: Process arrays or repeat actions until a condition is met](../logic-apps/logic-apps-control-flow-loops.md) (Boucles : Traiter des tableaux ou répéter des actions jusqu’à ce qu’une condition soit remplie)
* [Exécuter ou joindre des étapes (branches) parallèles](../logic-apps/logic-apps-control-flow-branches.md)
* [Étendues : Exécuter des étapes en fonction de l’état d’un groupe](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
