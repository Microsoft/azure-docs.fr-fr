---
title: Mises à jour du schéma du 1er juin 2016
description: Mise à jour du schéma version 2016-06-01 pour les définitions d’application logique dans Azure Logic Apps
services: logic-apps
ms.suite: integration
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 07/25/2016
ms.openlocfilehash: ccc7df5bfac327fabf05f210764dbe10658b5015
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000315"
---
# <a name="schema-updates-for-azure-logic-apps---june-1-2016"></a>Mises à jour de schéma pour Azure Logic Apps - 1er juin 2016

La [version mise à jour](https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json) du schéma et de la version de l’API pour Azure Logic Apps intègrent différentes améliorations clés destinées à accroître la fiabilité et la simplicité d’utilisation des applications logiques :

* Les [étendues](#scopes) vous permettent de regrouper ou d’imbriquer des actions sous la forme d’un ensemble d’actions.
* Les [conditions et les boucles](#conditions-loops) sont désormais des actions de première classe.
* La précision de l’ordre d’exécution des actions a été améliorée grâce au remplacement de la propriété `dependsOn` par la propriété `runAfter`.

Pour mettre à niveau vos applications logiques à partir du schéma en version préliminaire du 1er août 2015 vers le schéma du 1er juin 2016, [consultez la section relative à la mise à niveau](#upgrade-your-schema).

<a name="scopes"></a>

## <a name="scopes"></a>Étendues

Ce schéma comporte des étendues, qui vous permettent de regrouper ou d’imbriquer des actions. Par exemple, une condition peut en contenir une autre. Apprenez-en davantage sur la [syntaxe des étendues](./logic-apps-control-flow-loops.md), ou examinez l’exemple d’étendue de base ci-dessous :

```json
{
   "actions": {
      "Scope": {
         "type": "Scope",
         "actions": {                
            "Http": {
               "inputs": {
                   "method": "GET",
                   "uri": "https://www.bing.com"
               },
               "runAfter": {},
               "type": "Http"
            }
         }
      }
   }
}
```

<a name="conditions-loops"></a>

## <a name="conditions-and-loops-changes"></a>Modifications des conditions et boucles

Dans les versions de schéma précédentes, les conditions et les boucles étaient des paramètres associés avec une seule action. Cette limitation a été levée dans ce schéma, de sorte que les conditions et les boucles sont désormais disponibles sous forme de types d’actions. Apprenez-en davantage sur les [boucles et étendues](./logic-apps-control-flow-loops.md) et sur les [conditions](../logic-apps/logic-apps-control-flow-conditional-statement.md), ou examinez cet exemple de base qui illustre une action de condition :

```json
{
   "Condition - If trigger is some trigger": {
      "type": "If",
      "expression": "@equals(triggerBody(), '<trigger-name>')",
      "runAfter": {},
      "actions": {
         "Http_2": {
            "inputs": {
                "method": "GET",
                "uri": "https://www.bing.com"
            },
            "runAfter": {},
            "type": "Http"
         }
      },
      "else": 
      {
         "Condition - If trigger is another trigger": {}
      }  
   }
}
```

<a name="run-after"></a>

## <a name="runafter-property"></a>Propriété « runAfter »

La propriété `runAfter` remplace `dependsOn`, ce qui vous permet de spécifier plus précisément l’ordre d’exécution des actions en fonction de l’état des actions précédentes. La propriété `dependsOn` indiquait si « l’action avait été exécutée et avait réussi », selon que l’action précédente avait réussi ou avait échoué, ou si elle avait été ignorée. Elle n’indiquait pas le nombre de fois que vous vouliez exécuter l’action. La propriété `runAfter` vous offre la souplesse sous la forme d’un objet spécifiant le nom de toutes les actions après lesquelles l’objet doit s’exécuter. Cette propriété définit également un tableau d’états acceptables en tant que déclencheurs. Par exemple, si vous souhaitez exécuter une action après la réussite de l’action A, ainsi qu’après la réussite ou l’échec de l’action B, configurez la propriété `runAfter` suivante :

```json
{
   // Other parts in action definition
   "runAfter": {
      "A": ["Succeeded"],
      "B": ["Succeeded", "Failed"]
    }
}
```

## <a name="upgrade-your-schema"></a>Mettre à niveau votre schéma

Quelques étapes suffisent pour mettre à niveau vers le [schéma le plus récent](https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json). Le processus de mise à niveau implique l’exécution du script de mise à niveau, l’enregistrement en tant que nouvelle application logique et, si vous le souhaitez, le remplacement éventuel de l’application logique précédente.

1. Dans le Portail Azure, ouvrez votre application logique.

2. Accédez à **Vue d’ensemble**. Dans la barre d’outils de l’application logique, choisissez **Mettre à jour le schéma**.
   
   ![Choisir l’option Mettre à jour le schéma][1]
   
   Le portail vous renvoie la définition mise à niveau, que vous pouvez copier et coller dans une définition de ressource si nécessaire. 

   > [!IMPORTANT]
   > *Veillez* à choisir **Enregistrer sous** afin que toutes les références de connexion restent valides dans l’application logique mise à niveau.

3. Dans la barre d’outils du panneau de mise à niveau, choisissez **Enregistrer sous**.

4. Entrez le nom et l’état de l’application logique. Pour déployer votre application logique mise à niveau, choisissez **Créer**.

5. Vérifiez que votre application logique mise à niveau fonctionne comme prévu.
   
   > [!NOTE]
   > Si vous utilisez un déclencheur manuel ou sur demande, l’URL de rappel change dans votre nouvelle application logique. Testez la nouvelle URL pour vérifier son fonctionnement de bout en bout. Pour conserver les URL précédentes, vous pouvez cloner votre application logique existante.

6. *Facultatif* Pour remplacer votre application logique précédente par la nouvelle version du schéma, dans la barre d’outils, choisissez **Cloner** en regard de l’option **Mettre à jour le schéma**. Cette étape n’est requise que si vous souhaitez conserver les mêmes ID de ressource ou URL du déclencheur sur demande de votre application logique.

## <a name="upgrade-tool-notes"></a>Notes de l’outil de mise à niveau

### <a name="mapping-conditions"></a>Mappage de conditions

Dans la définition mise à niveau, l’outil s’efforce de regrouper les actions des branches true et false sous la forme d’une étendue. Le modèle de concepteur de `@equals(actions('a').status, 'Skipped')` apparaît notamment en tant qu’action `else`. Toutefois, si l’outil détecte des modèles non reconnaissables, il peut éventuellement créer des conditions distinctes pour les branches true et false. Vous pouvez remapper les actions après la mise à niveau si nécessaire.

#### <a name="foreach-loop-with-condition"></a>Boucle « foreach » avec condition

Dans le nouveau schéma, vous pouvez utiliser l’action de filtrage pour répliquer le modèle qui utilise une boucle **foreach** avec une condition par élément. Toutefois, la modification se produit automatiquement lorsque vous procédez à la mise à niveau. La condition devient une action de filtrage qui apparaît avant la boucle **foreach**, retournant ainsi uniquement un tableau d’éléments correspondant à cette condition, puis transmettant ce tableau à l’action **Pour chaque**. Pour découvrir un exemple, consultez l’article relatif aux [boucles et étendues](./logic-apps-control-flow-loops.md).

### <a name="resource-tags"></a>Balises de ressource

Après la mise à niveau, les balises de ressource sont supprimées. Vous devez donc les redéfinir pour le workflow mis à niveau.

## <a name="other-changes"></a>Autres modifications

### <a name="renamed-manual-trigger-to-request-trigger"></a>Déclencheur « manual » (manuel) renommé déclencheur « request » (sur demande)

Le type de déclencheur `manual` est déconseillé et a été renommé sous la forme `request` avec le type `http`. Cette modification est plus cohérente avec le type de modèle pour la génération duquel le déclencheur est utilisé.

### <a name="new-filter-action"></a>Nouvelle action de « filtre »

Pour filtrer un tableau d’éléments volumineux de façon à obtenir un sous-ensemble d’éléments plus restreint, le nouveau type `filter` accepte un tableau et une condition, évalue la condition pour chaque élément, puis renvoie un tableau d’éléments correspondant à la condition.

### <a name="restrictions-for-foreach-and-until-actions"></a>Restrictions relatives aux actions « foreach » et « until »

La boucle `foreach` et `until` est limitée à une seule action.

### <a name="new-trackedproperties-for-actions"></a>Nouvelle propriété « trackedProperties » relative aux actions

Les actions peuvent désormais comporter une propriété supplémentaire appelée `trackedProperties`, sœur des propriétés `runAfter` et `type`. Cet objet spécifie certaines entrées ou sorties d’action que vous souhaitez inclure dans la télémétrie de diagnostic Azure, dont l’émission intervient dans le cadre d’un workflow. Par exemple :

``` json
{
   "Http": {
      "inputs": {
         "method": "GET",
         "uri": "https://www.bing.com"
      },
      "runAfter": {},
      "type": "Http",
      "trackedProperties": {
         "responseCode": "@action().outputs.statusCode",
         "uri": "@action().inputs.uri"
      }
   }
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Créer des définitions d’application logique](../logic-apps/logic-apps-author-definitions.md)
* [Automatiser le déploiement d’application logique](logic-apps-azure-resource-manager-templates-overview.md)

<!-- Image references -->
[1]: ./media/logic-apps-schema-2016-04-01/upgradeButton.png
