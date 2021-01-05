---
title: Ajouter des boucles pour répéter des actions
description: Créer des boucles qui répètent des actions de workflow ou des tableaux de processus dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 01/05/2019
ms.openlocfilehash: aa4be5852b4f8af00346a3ea9a86b13a85f99824
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93358454"
---
# <a name="create-loops-that-repeat-workflow-actions-or-process-arrays-in-azure-logic-apps"></a>Créer des boucles qui répètent des actions de workflow ou des tableaux de processus dans Azure Logic Apps

Pour traiter un tableau dans votre application logique, vous pouvez créer une [boucle « Foreach »](#foreach-loop). Cette boucle répète une ou plusieurs actions sur chaque élément du tableau. Pour connaître la limite du nombre d’éléments de tableau qu’une boucle « Foreach » peut traiter, consultez [Limites de simultanéité, de bouclage et de décomposition](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits).

Pour répéter des actions jusqu’à ce qu’une condition soit remplie ou qu’un état change, vous pouvez créer une [boucle « Until »](#until-loop). Votre application logique exécute d’abord toutes les actions dans la boucle, puis elle vérifie la condition ou l’état. Si la condition est remplie, la boucle s’arrête. Dans le cas contraire, la boucle se répète. Pour connaître les limites par défaut et maximale du nombre de boucles « Until » que l’exécution d’une application logique peut avoir, consultez [Limites de simultanéité, de bouclage et de décomposition](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits).

> [!TIP]
> Si vous disposez d’un déclencheur qui reçoit un tableau et souhaite exécuter un workflow pour chaque élément du tableau, vous pouvez *dégrouper* ce tableau avec le déclencheur de propriété [**SplitOn**](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch).

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/). 

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

<a name="foreach-loop"></a>

## <a name="foreach-loop"></a>Boucle « Foreach »

Une boucle « Foreach » répète une ou plusieurs actions sur chaque élément du tableau et fonctionne uniquement sur les tableaux. Voici quelques considérations liées à l’utilisation des boucles « Foreach » :

* La boucle « Foreach » peut traiter un nombre limité d’éléments de tableau. Pour connaître cette limite, consultez [Limites de concurrence, de bouclage et de décomposition](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits).

* Par défaut, les itérations dans une boucle « Foreach » s’exécutent simultanément, ou parallèlement. Ce comportement diffère de [la boucle **Appliquer à chaque** de Power Automate](/power-automate/apply-to-each) où les itérations s’exécutent l’une après l’autre, ou séquentiellement. Toutefois, vous pouvez [configurer des itérations de boucle « Foreach » séquentielles](#sequential-foreach-loop). Par exemple, si vous souhaitez suspendre l’itération suivante dans une boucle « Foreach » à l’aide de l’[Action Retarder](../connectors/connectors-native-delay.md), vous devez définir la boucle pour qu’elle s’exécute de façon séquentielle.

  L’exception au comportement par défaut est celle de boucles imbriquées où les itérations s’exécutent toujours séquentiellement, non en parallèle. Pour exécuter des opérations en parallèle pour les éléments d’une boucle imbriquée, créez et [appelez une application logique enfant](../logic-apps/logic-apps-http-endpoint.md).

* Pour obtenir des résultats prévisibles à partir d’opérations exécutées sur des variables pendant chaque itération de boucle, exécutez ces boucles de manière séquentielle. Par exemple, quand une boucle exécutée simultanément se termine, les opérations d’incrémentation, de décrémentation et d’ajout aux variables retournent des résultats prévisibles. Toutefois, pendant chaque itération de la boucle s’exécutant simultanément, ces opérations peuvent retourner des résultats imprévisibles. 

* Les actions dans une boucle « Foreach » utilisent l’expression [`@item()`](../logic-apps/workflow-definition-language-functions-reference.md#item) 
pour référencer et traiter chaque élément du tableau. Si vous spécifiez des données qui ne sont pas présentes dans le tableau, le workflow de l’application logique échoue. 

Cet exemple d’application logique envoie un résumé quotidien pour le flux RSS d’un site web. L’application utilise une boucle « Foreach » qui envoie un e-mail pour chaque nouvel élément.

1. [Créez cet exemple d'application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md) avec un compte Outlook.com ou avec un compte professionnel ou scolaire.

2. Entre le déclencheur RSS et l’action Envoyer un courrier électronique, ajoutez une bouche « Foreach ». 

   1. Pour ajouter une boucle entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. 
   Choisissez le **signe plus** ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

      ![Sélectionner « Ajouter une action »](media/logic-apps-control-flow-loops/add-for-each-loop.png)

   1. Sous la zone de recherche, choisissez **Tout**. Dans la zone de recherche, tapez « for each » comme filtre. Dans la liste des actions, sélectionnez cette action : **For each - Contrôle**

      ![Ajouter une boucle « for each »](media/logic-apps-control-flow-loops/select-for-each.png)

3. Générez maintenant la boucle. Sous **Sélectionner une sortie des étapes précédentes**, après que la liste **Ajouter contenu dynamique** s’affiche, sélectionnez le tableau **Liens du flux**, qui sort du déclencheur RSS. 

   ![Sélectionner depuis la liste de contenu dynamique](media/logic-apps-control-flow-loops/for-each-loop-dynamic-content-list.png)

   > [!NOTE] 
   > Vous pouvez sélectionner *seulement* des sorties de tableau de l’étape précédente.

   Le tableau sélectionné s’affiche maintenant ici :

   ![Sélectionner le tableau](media/logic-apps-control-flow-loops/for-each-loop-select-array.png)

4. Pour exécuter une action sur chaque élément du tableau, faites glisser l’action **Envoyer un message électronique** dans la boucle. 

   Votre application logique peut ressembler à cet exemple :

   ![Ajouter des étapes à la boucle « Foreach »](media/logic-apps-control-flow-loops/for-each-loop-with-step.png)

5. Enregistrez votre application logique. Pour tester manuellement votre application logique, sélectionnez **Exécuter** dans la barre d’outils du concepteur.

<a name="for-each-json"></a>

## <a name="foreach-loop-definition-json"></a>Définition de la boucle « Foreach » (JSON)

Si vous travaillez en mode code pour votre application logique, vous pouvez définir la boucle `Foreach` dans la définition JSON de votre application logique à la place, par exemple :

``` json
"actions": {
   "myForEachLoopName": {
      "type": "Foreach",
      "actions": {
         "Send_an_email": {
            "type": "ApiConnection",
            "inputs": {
               "body": {
                  "Body": "@{item()}",
                  "Subject": "New CNN post @{triggerBody()?['publishDate']}",
                  "To": "me@contoso.com"
               },
               "host": {
                  "api": {
                     "runtimeUrl": "https://logic-apis-westus.azure-apim.net/apim/office365"
                  },
                  "connection": {
                     "name": "@parameters('$connections')['office365']['connectionId']"
                  }
               },
               "method": "post",
               "path": "/Mail"
            },
            "runAfter": {}
         }
      },
      "foreach": "@triggerBody()?['links']",
      "runAfter": {}
   }
}
```

<a name="sequential-foreach-loop"></a>

## <a name="foreach-loop-sequential"></a>Boucle « Foreach » : Sequential

Par défaut, les cycles dans une boucle « Foreach » s’exécutent en parallèle. Pour exécuter chaque cycle de manière séquentielle, définissez l’option **Sequential** de la boucle. Les boucles « Foreach » doivent s’exécuter séquentiellement quand vous avez des boucles imbriquées ou des variables à l’intérieur de boucles où vous attendez des résultats prévisibles. 

1. En haut à droite de la boucle, choisissez **Ellipses** ( **...** ) > **Paramètres**.

   ![Dans la boucle « Foreach », choisissez « ... » > « Paramètres »](media/logic-apps-control-flow-loops/for-each-loop-settings.png)

1. Sous **Contrôle d’accès concurrentiel**, affectez la valeur **On** au paramètre **Contrôle d’accès concurrentiel**. Déplacez le curseur **Degré de parallélisme** sur **1**, puis choisissez **Terminé**.

   ![Activer le contrôle d’accès concurrentiel](media/logic-apps-control-flow-loops/for-each-loop-sequential-setting.png)

Si vous travaillez avec la définition JSON de votre application logique, vous pouvez utiliser l’option `Sequential` en ajoutant le paramètre `operationOptions`, par exemple :

``` json
"actions": {
   "myForEachLoopName": {
      "type": "Foreach",
      "actions": {
         "Send_an_email": { }
      },
      "foreach": "@triggerBody()?['links']",
      "runAfter": {},
      "operationOptions": "Sequential"
   }
}
```

<a name="until-loop"></a>

## <a name="until-loop"></a>Boucle « Until »
  
Pour exécuter et répéter des actions jusqu’à ce qu’une condition soit remplie ou qu’un état change, placez ces actions dans une boucle « Until ». Votre application logique exécute d’abord toutes les actions dans la boucle, puis elle vérifie la condition ou l’état. Si la condition est remplie, la boucle s’arrête. Dans le cas contraire, la boucle se répète. Pour connaître les limites par défaut et maximale du nombre de boucles « Until » que l’exécution d’une application logique peut avoir, consultez [Limites de simultanéité, de bouclage et de décomposition](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits).

Voici quelques scénarios courants dans lesquels vous pouvez utiliser une boucle « Until » :

* Appeler un point de terminaison jusqu’à obtenir la réponse souhaitée.

* Créer un enregistrement dans une base de données. Attendre qu’un champ spécifique dans cet enregistrement soit approuvé. Continuer le traitement. 

À partir de 8h00 chaque jour, cet exemple d’application logique incrémente une variable jusqu’à ce que sa valeur soit égale à 10. L’application logique envoie ensuite un e-mail qui confirme la valeur actuelle. 

> [!NOTE]
> Ces étapes utilisent Office 365 Outlook, mais vous pouvez utiliser n’importe quel fournisseur de messagerie pris en charge par Logic Apps. 
> [Vérifiez la liste des connecteurs ici](/connectors/). Si vous utilisez un autre compte de messagerie, les étapes générales sont identiques, mais l’affichage de l’interface utilisateur peut être légèrement différent. 

1. Créez une application logique vide. Dans le Concepteur d’application logique, sous la zone de recherche, choisissez **Tout**. Recherchez « récurrence ». 
   Dans la liste des déclencheurs, sélectionnez ce déclencheur : **Récurrence - Planification**

   ![Ajouter le déclencheur « Récurrence - Planification »](./media/logic-apps-control-flow-loops/do-until-loop-add-trigger.png)

1. Spécifiez l’envoi du déclencheur en définissant l’intervalle, la fréquence et l’heure du jour. Pour définir l’heure, choisissez **Afficher les options avancées**.

   ![Configurer la planification de la récurrence](./media/logic-apps-control-flow-loops/do-until-loop-set-trigger-properties.png)

   | Propriété | Valeur |
   | -------- | ----- |
   | **Intervalle** | 1 | 
   | **Fréquence** | jour |
   | **Aux heures indiquées** | 8 |
   ||| 

1. Sous le déclencheur, choisissez **Nouvelle étape**. 
   Recherchez « variables », puis sélectionnez cette action : **Initialiser la variable - Variables**

   ![Ajouter une action « Initialiser la variable - Variables »](./media/logic-apps-control-flow-loops/do-until-loop-add-variable.png)

1. Configurez votre variable avec ces valeurs :

   ![Définir les propriétés de la variable](./media/logic-apps-control-flow-loops/do-until-loop-set-variable-properties.png)

   | Propriété | Valeur | Description |
   | -------- | ----- | ----------- |
   | **Nom** | Limite | Nom de votre variable | 
   | **Type** | Integer | Type de données de votre variable | 
   | **Valeur** | 0 | Valeur de départ de votre variable | 
   |||| 

1. Sous l’action **Initialiser la variable**, choisissez **Nouvelle étape**. 

1. Sous la zone de recherche, choisissez **Tout**. Recherchez « until », puis sélectionnez cette action : **Until - Contrôle**

   ![Ajouter une boucle « Until »](./media/logic-apps-control-flow-loops/do-until-loop-add-until-loop.png)

1. Générez la condition de sortie de la boucle en sélectionnant la variable **Limite** et l’opérateur **Est égal à**. 
   Entrez **10** comme valeur de comparaison.

   ![Générer la connexion de sortie pour arrêter la boucle](./media/logic-apps-control-flow-loops/do-until-loop-settings.png)

1. Dans la boucle, choisissez **Ajouter une action**. 

1. Sous la zone de recherche, choisissez **Tout**. Recherchez « variables », puis sélectionnez cette action : **Incrémenter une variable - Variables**

   ![Ajouter une action pour incrémenter une variable](./media/logic-apps-control-flow-loops/do-until-loop-increment-variable.png)

1. Comme **Nom**, sélectionnez la variable **Limite**. Comme **Valeur**, entrez « 1 ». 

     ![Incrémenter la variable « Limite » de 1](./media/logic-apps-control-flow-loops/do-until-loop-increment-variable-settings.png)

1. En dehors et sous la boucle, choisissez **Nouvelle étape**. 

1. Sous la zone de recherche, choisissez **Tout**. 
     Recherchez et ajoutez une action qui envoie un e-mail, par exemple : 

     ![Ajouter une action d’envoi de message électronique](media/logic-apps-control-flow-loops/do-until-loop-send-email.png)

1. Si vous y êtes invité, connectez-vous à votre compte e-mail.

1. Définissez les propriétés de l’action d’e-mail. Ajoutez la variable **Limite** au sujet. De cette façon, vous pouvez confirmer que la valeur actuelle de la variable corresponde aux conditions que vous avez spécifiées. Par exemple :

      ![Configurer les propriétés du message électronique](./media/logic-apps-control-flow-loops/do-until-loop-send-email-settings.png)

      | Propriété | Valeur | Description |
      | -------- | ----- | ----------- | 
      | **To** | *\<email-address\@domain>* | Adresse e-mail du destinataire. Pour effectuer le test, utilisez votre propre adresse e-mail. | 
      | **Subject** | La valeur actuelle de la variable « Limite » est **Limite** | Spécifiez l’objet du message électronique. Pour cet exemple, assurez-vous d’inclure la variable **Limite**. | 
      | **Corps** | <*email-content*> | Spécifiez le contenu du message électronique à envoyer. Pour cet exemple, écrivez ce que vous voulez. | 
      |||| 

1. Enregistrez votre application logique. Pour tester manuellement votre application logique, sélectionnez **Exécuter** dans la barre d’outils du concepteur.

      Lorsque votre application logique s’exécute, vous recevez un message électronique avec le contenu spécifié :

      ![Message électronique reçu](./media/logic-apps-control-flow-loops/do-until-loop-sent-email.png)

<a name="prevent-endless-loops"></a>

## <a name="prevent-endless-loops"></a>Empêcher les boucles infinies

La boucle « Until » arrête l’exécution en fonction de ces propriétés. Veillez donc à définir leurs valeurs en conséquence :

* **Nombre** : Cette valeur est la quantité maximale de boucles qui s’exécutent avant que la boucle ne sorte. Pour connaître les limites par défaut et maximale du nombre de boucles « Until » que l’exécution d’une application logique peut avoir, consultez [Limites de simultanéité, de bouclage et de décomposition](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits).

* **Timeout** (Expiration du délai) : Cette valeur est la durée la plus longue d’exécution de la boucle avant qu’elle ne se termine, spécifiée au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). Pour obtenir les limites par défaut et maximale de la valeur **Délai d’attente**, consultez [Limites de simultanéité, de bouclage et de décomposition](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits).

  La valeur du délai d’attente est évaluée pour chaque cycle de boucle. Si une action dans la boucle dure plus longtemps que la limite de délai d’attente, le cycle actuel ne s’arrête pas. Toutefois, le cycle suivant ne démarre pas, car la condition de limite n’est pas remplie.

Pour modifier ces limites, dans l’action de la boucle, sélectionnez **Modifier les limites**.

<a name="until-json"></a>

## <a name="until-definition-json"></a>Définition (JSON) de la boucle « Until »

Si vous travaillez en mode code pour votre application logique, vous pouvez définir une boucle `Until` dans la définition JSON de votre application logique à la place, par exemple :

``` json
"actions": {
   "Initialize_variable": {
      // Definition for initialize variable action
   },
   "Send_an_email": {
      // Definition for send email action
   },
   "Until": {
      "type": "Until",
      "actions": {
         "Increment_variable": {
            "type": "IncrementVariable",
            "inputs": {
               "name": "Limit",
               "value": 1
            },
            "runAfter": {}
         }
      },
      "expression": "@equals(variables('Limit'), 10)",
      // To prevent endless loops, an "Until" loop 
      // includes these default limits that stop the loop. 
      "limit": { 
         "count": 60,
         "timeout": "PT1H"
      },
      "runAfter": {
         "Initialize_variable": [
            "Succeeded"
         ]
      }
   }
}
```

Cet exemple de boucle « Until » appelle un point de terminaison HTTP, qui crée une ressource. La boucle s’arrête quand le corps de réponse HTTP est retourné avec l’état `Completed`. Pour empêcher les boucles infinies, la boucle s’arrête aussi si l’un de ces conditions est remplie :

* La boucle s’est exécutée 10 fois comme spécifié par l’attribut `count`. La valeur par défaut est de 60 fois. 

* La boucle s’est exécutée pendant deux heures comme spécifié par l’attribut `timeout` au format ISO 8601. La valeur par défaut est d’une heure.
  
``` json
"actions": {
   "myUntilLoopName": {
      "type": "Until",
      "actions": {
         "Create_new_resource": {
            "type": "Http",
            "inputs": {
               "body": {
                  "resourceId": "@triggerBody()"
               },
               "url": "https://domain.com/provisionResource/create-resource",
               "body": {
                  "resourceId": "@triggerBody()"
               }
            },
            "runAfter": {},
            "type": "ApiConnection"
         }
      },
      "expression": "@equals(triggerBody(), 'Completed')",
      "limit": {
         "count": 10,
         "timeout": "PT2H"
      },
      "runAfter": {}
   }
}
```

## <a name="get-support"></a>Obtenir de l’aide

* Pour toute question, consultez la [page de questions Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Pour voter pour des idées et suggestions de fonctionnalités ou pour en soumettre, rendez-vous sur le [site de commentaires des utilisateurs Azure Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Étapes suivantes

* [Instructions conditionnelles : Exécuter des étapes en fonction d’une condition dans des applications logiques](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Instructions switch : Exécuter différentes étapes en fonction de valeurs spécifiques](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Exécuter ou joindre des étapes (branches) parallèles](../logic-apps/logic-apps-control-flow-branches.md)
* [Étendues : Exécuter des étapes en fonction de l’état d’un groupe](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
