---
title: Ajouter et exécuter des extraits de code avec du code inline
description: Découvrez comment créer et exécuter des extraits de code en utilisant des actions de code intégrées pour les travaux et flux de travail automatisés que vous créez avec Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, logicappspm
ms.topic: article
ms.date: 11/19/2020
ms.custom: devx-track-js
ms.openlocfilehash: 589420d96a3a6dfcc1c17a1b204765022b1ce412
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94916642"
---
# <a name="add-and-run-code-snippets-by-using-inline-code-in-azure-logic-apps"></a>Ajoutez et exécutez des extraits de code en utilisant du code inclus dans Azure Logic Apps

Lorsque vous souhaitez exécuter un extrait de code au sein de votre application logique, vous pouvez ajouter l’action **Code inclus** intégrée sous la forme d’une étape dans le flux de travail de votre application logique. Cette action fonctionne mieux lorsque vous souhaitez exécuter du code qui correspond à ce scénario :

* S’exécute dans JavaScript. D’autres langues seront bientôt disponibles.

* S’exécute en cinq secondes maximum.

* Gère les données dont la taille ne dépasse pas 50 Mo.

* Ne nécessite pas l’utilisation des [actions de **Variables**](../logic-apps/logic-apps-create-variables-store-values.md), qui ne sont pas encore prises en charge.

* Utilise Node.js version 8.11.1. Pour en savoir plus, voir [Objets globaux standards (par catégorie)](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects).

  > [!NOTE]
  > La fonction `require()` n'est pas prise en charge par l'action **Code inclus** pour l'exécution de JavaScript.

Cette action exécute l’extrait de code et renvoie la sortie de cet extrait de code sous la forme d’un jeton nommé **Result**, que vous pouvez utiliser pour les actions suivantes au sein de votre application logique. Pour d’autres scénarios visant à créer une fonction pour votre code, essayez de [créer et d’appeler une fonction Azure](../logic-apps/logic-apps-azure-functions.md) dans votre application logique.

Dans cet article, l’exemple d’application logique se déclenche lorsqu’un nouvel e-mail arrive dans un compte professionnel ou scolaire. L’extrait de code extrait et renvoie toutes les adresses e-mail qui s’affichent dans le corps du message.

![Présentation des exemples](./media/logic-apps-add-run-inline-code/inline-code-example-overview.png)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Application logique dans laquelle vous souhaitez ajouter votre extrait de code, y compris un déclencheur. Si vous n’avez pas d’application logique, consultez la section [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

   L’exemple d’application logique de cette rubrique utilise ce déclencheur Outlook Office 365 : **When a new email arrives** (Quand un nouveau courrier électronique arrive)

* [Compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) lié à votre application logique. Si vous ne souhaitez pas créer ou utiliser un compte d’intégration, essayez de créer une application logique dans le portail Azure à l’aide du nouveau type de ressource **Logic App (préversion)** ou dans Visual Studio Code à l’aide de la nouvelle [extension en préversion Azure Logic Apps](../logic-apps/create-stateful-stateless-workflows-visual-studio-code.md).

  > [!NOTE]
  > Veillez à utiliser un compte d'intégration adapté à votre cas d'usage ou scénario. Par exemple, les comptes d'intégration de [niveau gratuit](../logic-apps/logic-apps-pricing.md#integration-accounts) sont uniquement destinés aux charges de travail et aux scénarios exploratoires, et non aux scénarios de production ; ils sont limités en termes d'utilisation et de débit, et ne sont pris en charge par aucun contrat de niveau de service (SLA). Les autres niveaux sont payants, mais incluent la prise en charge des contrats SLA, fournissent davantage de débit et offrent des limites plus élevées. Apprenez-en davantage sur les [niveaux](../logic-apps/logic-apps-pricing.md#integration-accounts), la [tarification](https://azure.microsoft.com/pricing/details/logic-apps/) et les [limites](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits) des comptes d'intégration.

## <a name="add-inline-code"></a>Ajouter du code inclus

1. Si ce n’est déjà fait, dans le [Portail Microsoft Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’application logique.

1. Dans le concepteur, ajoutez l’action **Code inclus** à l’emplacement souhaité dans le flux de travail de votre application logique.

   * Pour ajouter l’action à la fin de votre flux de travail, choisissez **Nouvelle étape**.

   * Pour ajouter l’action entre des étapes existantes, déplacez votre pointeur de souris sur la flèche qui connecte ces étapes. Cliquez sur le signe plus ( **+** ), puis sélectionnez **Ajouter une action**.

   Cet exemple ajoute l’action **Code inclus** sous le déclencheur Office 365 Outlook.

   ![Ajouter une nouvelle étape](./media/logic-apps-add-run-inline-code/add-new-step.png)

1. Sous **Choisir une action**, dans la zone de recherche, entrez « code inclus » à titre de filtre. Dans la liste des actions, sélectionnez cette action : **Exécuter du code JavaScript**

   ![Sélectionnez « Exécuter du code JavaScript »](./media/logic-apps-add-run-inline-code/select-inline-code-action.png)

   Cette action s’affiche dans le concepteur. Elle contient un exemple de code par défaut, y compris une instruction return.

   ![Action Code inclus avec un exemple de code par défaut](./media/logic-apps-add-run-inline-code/inline-code-action-default.png)

1. Dans la zone **Code**, supprimez l’exemple de code et saisissez le code que vous souhaitez exécuter. Écrivez le code que vous voulez placer à l’intérieur d’une méthode, mais sans définir la signature de méthode.

   Lorsque vous tapez un mot clé reconnu, la liste de saisie semi-automatique s’affiche afin que vous puissiez sélectionner des mots clés parmi ceux qui sont disponibles, par exemple :

   ![Liste de saisie semi-automatique de mots clés](./media/logic-apps-add-run-inline-code/auto-complete.png)

   Cet exemple d’extrait de code crée d’abord une variable qui stocke une *expression régulière*, laquelle spécifie un modèle à faire correspondre dans le texte d’entrée. Ce code crée ensuite une variable, qui stocke les données du corps de l’e-mail provenant du déclencheur.

   ![Créer des variables](./media/logic-apps-add-run-inline-code/save-email-body-variable.png)

   Afin de faciliter la référence aux résultats provenant du déclencheur et les actions précédentes, la liste de contenu dynamique s’affiche lorsque le curseur est placé dans la zone **Code**. Pour cet exemple, la liste affiche les résultats disponibles provenant du déclencheur, y compris le jeton **Body**, que vous pouvez maintenant sélectionner.

   Une fois que vous avez sélectionné le jeton **Body**, l’action de code inclus résout le jeton sur un objet `workflowContext` qui fait référence à la valeur de propriété `Body` de l’e-mail :

   ![Sélection d’un résultat](./media/logic-apps-add-run-inline-code/inline-code-example-select-outputs.png)

   Dans la zone **Code**, votre extrait de code peut utiliser l’objet `workflowContext` en lecture seule. Cet objet possède des sous-propriétés qui permettent à votre code d’accéder aux résultats provenant du déclencheur, ainsi que les actions précédentes, dans votre flux de travail. Pour en savoir plus, consultez la section suivante du présent document : [Faire référence aux résultats des actions et du déclencheur dans votre code](#workflowcontext).

   > [!NOTE]
   >
   > Si votre extrait de code fait référence à des noms d’action qui utilisent l’opérateur point (.), vous devez ajouter ces noms d’actions au paramètre [**Actions**](#add-parameters). Ces références doivent également placer les noms d’action entre des crochets ([]) et des guillemets, par exemple :
   >
   > `// Correct`</br> 
   > `workflowContext.actions["my.action.name"].body`</br>
   >
   > `// Incorrect`</br>
   > `workflowContext.actions.my.action.name.body`

   L’action de code inclus ne nécessite aucune instruction `return`, mais les résultats d’une instruction `return` sont disponibles à des fins de référence dans les actions ultérieures, via le jeton **Result**. Par exemple, l’extrait de code renvoie le résultat en appelant la fonction `match()`, qui recherche des correspondances dans le corps de l’e-mail, en fonction de l’expression régulière. L’action **Composer** utilise le jeton **Result** pour référencer les résultats de l’action Code inclus, puis crée un résultat unique.

   ![Application logique terminée](./media/logic-apps-add-run-inline-code/inline-code-complete-example.png)

1. Lorsque vous avez terminé, enregistrez votre application logique.

<a name="workflowcontext"></a>

### <a name="reference-trigger-and-action-results-in-your-code"></a>Faire référence aux résultats des actions et du déclencheur dans votre code

L’objet `workflowContext` présente la structure suivante, qui inclut les sous-propriétés `actions`, `trigger` et `workflow` :

```json
{
   "workflowContext": {
      "actions": {
         "<action-name-1>": @actions('<action-name-1>'),
         "<action-name-2>": @actions('<action-name-2>')
      },
      "trigger": {
         @trigger()
      },
      "workflow": {
         @workflow()
      }
   }
}
```

Cette table contient plus d’informations sur ces sous-propriétés :

| Propriété | Type | Description |
|----------|------|-------|
| `actions` | Collection d’objets | Objets de résultats des actions qui s’exécutent avant votre extrait de code. Chaque objet possède une paire *clé-valeur*, dans laquelle la clé correspond au nom d’une action, et la valeur équivaut à appeler la fonction [actions()](../logic-apps/workflow-definition-language-functions-reference.md#actions) avec `@actions('<action-name>')`. Le nom de l’action tire parti du nom d’action utilisé dans la définition de flux de travail sous-jacente, qui remplace les espaces (« ») figurant dans le nom de l’action par des caractères de soulignement (_). Cet objet permet d’accéder aux valeurs de propriétés d’action de l’instance de workflow exécutée. |
| `trigger` | Object | Objet de résultat du déclencheur et équivalent à l’appel de la fonction [trigger()](../logic-apps/workflow-definition-language-functions-reference.md#trigger). Cet objet permet d’accéder aux valeurs de propriétés du déclencheur de l’instance de workflow exécutée. |
| `workflow` | Object | Objet de flux de travail et équivalent à l’appel de la fonction [workflow()](../logic-apps/workflow-definition-language-functions-reference.md#workflow). Cet objet permet d’accéder aux valeurs de propriétés du flux de travail (nom du flux de travail, ID d’exécution etc.) à partir de l’instance de flux de travail actuellement exécutée. |
|||

Dans l’exemple de cette rubrique, l’objet `workflowContext` présente les propriétés suivantes, auxquelles votre code peut accéder :

```json
{
   "workflowContext": {
      "trigger": {
         "name": "When_a_new_email_arrives",
         "inputs": {
            "host": {
               "connection": {
                  "name": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Web/connections/office365"
               }
            },
            "method": "get",
            "path": "/Mail/OnNewEmail",
            "queries": {
               "includeAttachments": "False"
            }
         },
         "outputs": {
            "headers": {
               "Pragma": "no-cache",
               "Content-Type": "application/json; charset=utf-8",
               "Expires": "-1",
               "Content-Length": "962095"
            },
            "body": {
               "Id": "AAMkADY0NGZhNjdhLTRmZTQtNGFhOC1iYjFlLTk0MjZlZjczMWRhNgBGAAAAAABmZwxUQtCGTqSPpjjMQeD",
               "DateTimeReceived": "2019-03-28T19:42:16+00:00",
               "HasAttachment": false,
               "Subject": "Hello World",
               "BodyPreview": "Hello World",
               "Importance": 1,
               "ConversationId": "AAQkADY0NGZhNjdhLTRmZTQtNGFhOC1iYjFlLTk0MjZlZjczMWRhNgAQ",
               "IsRead": false,
               "IsHtml": true,
               "Body": "Hello World",
               "From": "<sender>@<domain>.com",
               "To": "<recipient-2>@<domain>.com;<recipient-2>@<domain>.com",
               "Cc": null,
               "Bcc": null,
               "Attachments": []
            }
         },
         "startTime": "2019-05-03T14:30:45.971564Z",
         "endTime": "2019-05-03T14:30:50.1746874Z",
         "scheduledTime": "2019-05-03T14:30:45.8778117Z",
         "trackingId": "1cd5ffbd-f989-4df5-a96a-6e9ce31d03c5",
         "clientTrackingId": "08586447130394969981639729333CU06",
         "originHistoryName": "08586447130394969981639729333CU06",
         "code": "OK",
         "status": "Succeeded"
      },
      "workflow": {
         "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<logic-app-workflow-name>",
         "name": "<logic-app-workflow-name>",
         "type": "Microsoft.Logic/workflows",
         "location": "<Azure-region>",
         "run": {
            "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<logic-app-workflow-name>/runs/08586453954668694173655267965CU00",
            "name": "08586453954668694173655267965CU00",
            "type": "Microsoft.Logic/workflows/runs"
         }
      }
   }
}
```

<a name="add-parameters"></a>

## <a name="add-parameters"></a>Ajout de paramètres

Dans certains cas, vous devrez peut-être demander explicitement à l’action **Code inclus** d’inclure les résultats du déclencheur ou d’actions spécifiques référencées par votre code en tant que dépendances, en ajoutant les paramètres **Déclencheur** ou **Actions**. Cette option est utile pour les scénarios dans lesquels le système ne trouve pas les résultats référencés au moment de l’exécution.

> [!TIP]
> Si vous projetez de réutiliser votre code, ajoutez des références aux propriétés à l’aide de la zone **Code**, afin que votre code inclue les références des jetons résolus, plutôt que d’ajouter le déclencheur ou les actions en tant que dépendances explicites.

Par exemple, supposons que vous disposiez du code qui fait référence au résultat **SelectedOption** de l’action **Envoyer un e-mail d’approbation** pour le connecteur Office 365 Outlook. Lors de la création, le moteur Logic Apps analyse votre code pour déterminer si vous avez référencé des résultats d’actions ou de déclencheur et inclut ces résultats automatiquement. Lors de l’exécution, si vous obtenez une erreur indiquant que le résultat d’action ou de déclencheur référencé n’est pas disponible dans l’objet `workflowContext` spécifié, vous pouvez ajouter ce déclencheur ou cette action en tant que dépendance explicite. Dans cet exemple, vous ajoutez le paramètre **Actions** et spécifiez que l’action **Code inclus** doit inclure explicitement le résultat de l’action **Envoyer un e-mail d’approbation**.

Pour ajouter ces paramètres, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les paramètres de votre choix :

   ![Ajout de paramètres](./media/logic-apps-add-run-inline-code/inline-code-action-add-parameters.png)

   | Paramètre | Description |
   |-----------|-------------|
   | **Actions** | Inclusion des résultats des actions précédentes. Voir [Inclure les résultats d’actions](#action-results). |
   | **Déclencheur** | Inclusion des résultats du déclencheur. Voir [Inclure les résultats du déclencheur](#trigger-results). |
   |||

<a name="trigger-results"></a>

### <a name="include-trigger-results"></a>Inclure les résultats du déclencheur

Si vous sélectionnez **Déclencheurs**, vous êtes invité à inclure les résultats du déclencheur.

* Dans la liste **Déclencheur**, sélectionnez **Oui**.

<a name="action-results"></a>

### <a name="include-action-results"></a>Inclure les résultats d’actions

Si vous sélectionnez **Actions**, vous êtes invité à indiquer les actions que vous souhaitez ajouter. Toutefois, avant de commencer à ajouter des actions, vous avez besoin de la version du nom d’action qui s’affiche dans la définition de flux de travail sous-jacente de l’application logique.

* Cette fonctionnalité ne prend pas en charge les variables, les boucles et les index d’itération.

* Dans les noms de la définition de flux de travail de votre application logique, vous pouvez utiliser un trait de soulignement (_), et non un espace.

* Pour les noms d’action qui utilisent l’opérateur point (.), incluez ces opérateurs, par exemple :

  `My.Action.Name`

1. Dans la barre d’outils du concepteur, sélectionnez **Mode Code**, puis recherchez le nom de l’action dans l’attribut `actions`.

   Par exemple, `Send_approval_email_` est le nom JSON de l’action **Envoyer un e-mail d’approbation**.

   ![Rechercher le nom d’une action dans JSON](./media/logic-apps-add-run-inline-code/find-action-name-json.png)

1. Pour revenir au mode Création, dans la barre d’outils du mode Code, choisissez **Concepteur**.

1. Pour ajouter la première action dans la zone **Élément d’action - 1**, saisissez le nom JSON de l’action.

   ![Indiquer la première action](./media/logic-apps-add-run-inline-code/add-action-parameter.png)

1. Pour ajouter une autre action, choisissez **Ajouter un nouvel élément**.

## <a name="reference"></a>Informations de référence

Pour en savoir plus sur la structure de l’action **exécuter le JavaScript Code** et sa syntaxe dans la définition de flux de travail sous-jacente de votre application logique, à l’aide du langage de définition du flux de travail, reportez-vous à la [section de référence](../logic-apps/logic-apps-workflow-actions-triggers.md#run-javascript-code) de cette action.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [les autres connecteurs pour Azure Logic Apps](../connectors/apis-list.md)
