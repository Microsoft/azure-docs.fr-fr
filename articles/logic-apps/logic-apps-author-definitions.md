---
title: Créer, modifier ou étendre des définitions de flux de travail JSON d’application logique
description: Guide pratique pour écrire, modifier et étendre vos définitions de flux de travail JSON d’application logique dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 01/01/2018
ms.openlocfilehash: 9163071237041d7c8510a644c573e3763434bb0c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014395"
---
# <a name="create-edit-or-extend-json-for-logic-app-workflow-definitions-in-azure-logic-apps"></a>Créer, modifier ou étendre JSON pour les définitions de flux de travail d’application logique dans Azure Logic Apps

Lorsque vous créez des solutions d’intégration d’entreprise avec des flux de travail automatisés dans [Azure Logic Apps](../logic-apps/logic-apps-overview.md), les définitions d’application logique sous-jacentes utilisent le JavaScript Object Notation (JSON) avec le [schéma Langue de la définition du flux de travail (WDL)](../logic-apps/logic-apps-workflow-definition-language.md) pour leur description et validation. Ces formats simplifient la lecture et la compréhension des définitions d’application sans en savoir beaucoup sur le code.
Lorsque vous désirez automatiser la création et le déploiement des applications logiques, vous pouvez inclure des définitions d’application logique en tant que [Ressources Azure](../azure-resource-manager/management/overview.md) dans des [modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md).
Pour créer, gérer et déployer des applications logiques, vous pouvez utiliser [Azure PowerShell](/powershell/module/az.logicapp), [Azure CLI](../azure-resource-manager/templates/deploy-cli.md) ou les [API REST Azure Logic Apps](/rest/api/logic/).

Pour utiliser des définitions d’application logique dans JSON, ouvrez l’éditeur en mode Code lorsque vous travaillez dans le portail Azure ou dans Visual Studio ou copiez la définition dans tout éditeur de votre choix.
Si vous débutez avec les applications logiques, consultez [procédure de création de votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

> [!NOTE]
> Certaines fonctionnalités Azure Logic Apps, telle que la définition des paramètres et de plusieurs déclencheurs dans les définitions d’application logique, sont uniquement disponibles dans JSON, et pas le Concepteur d’applications logiques.
> Par conséquent, pour ces tâches, vous devez travailler en mode Code ou sur un autre éditeur.

## <a name="edit-json---azure-portal"></a>Modifier JSON - portail Azure

1. Connectez-vous au <a href="https://portal.azure.com" target="_blank">portail Azure</a>.

2. Dans le menu de gauche, choisissez **Tous les services**.
Dans la zone de recherche, rechercher « applications logiques », puis à partir des résultats, sélectionnez votre application logique.

3. Dans le menu de l’application logique, sous **Outils de développement**, choisissez **Affichage du code de l’application logique**.

   L’éditeur Mode Code s’ouvre et affiche la définition de votre application logique au format JSON.

## <a name="edit-json---visual-studio"></a>Modifier JSON - Visual Studio

Avant de pouvoir travailler sur la définition de votre application logique dans Visual Studio, assurez-vous d’avoir [installé les outils requis](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#prerequisites).
Pour créer une application logique avec Visual Studio, consultez [Démarrage rapide : Automatiser des tâches et des processus avec Azure Logic Apps - Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

Dans Visual Studio, vous pouvez ouvrir des applications logiques précédemment créées et déployées directement depuis le portail Azure ou en tant que projets Azure Resource Manager à partir de Visual Studio.

1. Ouvrez la solution Visual Studio ou le projet [Azure Resource Manager](../azure-resource-manager/management/overview.md), qui contient votre application logique.

2. Recherchez et ouvrez la définition de votre application logique, qui par défaut s’affiche dans un [modèle Resource Manager](../azure-resource-manager/templates/overview.md), nommé **LogicApp.json**.
Vous pouvez utiliser et personnaliser ce modèle pour le déploiement vers différents environnements.

3. Ouvrez le menu contextuel de la définition et du modèle de votre application logique.
Sélectionnez **Ouvrir avec le Concepteur d’application logique**.

   ![Ouvrir une application logique dans une solution Visual Studio](./media/logic-apps-author-definitions/open-logic-app-designer.png)

   > [!TIP]
   > Si vous ne voyez pas cette commande dans Visual Studio 2019, vérifiez que vous avez les dernières mises à jour pour Visual Studio.

4. En bas du concepteur, choisissez **Mode Code**.

   L’éditeur Mode Code s’ouvre et affiche la définition de votre application logique au format JSON.

5. Pour revenir au mode concepteur, en bas de l’éditeur Mode Code, choisissez **Conception**.

## <a name="parameters"></a>Paramètres

Le cycle de vie du déploiement a généralement des environnements différents pour le développement, le test, la mise en lots et la production. Si vous souhaitez réutiliser des valeurs dans votre application logique sans codage en dur ou qui varient en fonction de vos besoins en matière de déploiement, vous pouvez créer un [modèle Azure Resource Manager](../azure-resource-manager/management/overview.md) pour votre définition de flux de travail afin de pouvoir également automatiser le déploiement d’applications logiques.

Suivez ces étapes générales pour *paramétrer*, ou définir et utiliser des paramètres, pour ces valeurs à la place. Vous pouvez ensuite fournir les valeurs dans un fichier de paramètres distinct qui transmet ces valeurs à votre modèle. De cette façon, vous pouvez modifier ces valeurs plus facilement sans avoir à mettre à jour et à redéployer votre application logique. Pour obtenir tous les détails, consultez [Vue d’ensemble : Automatiser le déploiement pour les applications logiques avec des modèles Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md).

1. Dans votre modèle, définissez les paramètres de modèle et les paramètres de définition de flux de travail pour accepter les valeurs à utiliser au moment du déploiement et de l’exécution, respectivement.

   Les paramètres de modèle sont définis dans une section de paramètres en dehors de votre définition de flux de travail, tandis que les paramètres de définition de flux de travail sont définis dans une section de paramètres à l’intérieur de votre définition de flux de travail.

1. Remplacez les valeurs codées en dur par des expressions qui font référence à ces paramètres. Les expressions de modèle utilisent une syntaxe différente des expressions de définition de flux de travail.

   Évitez de compliquer votre code en n’utilisant pas d’expressions de modèle, qui sont évaluées au moment du déploiement, à l’intérieur d’expressions de définition de flux de travail, qui sont évaluées au moment de l’exécution. Utilisez uniquement des expressions de modèle en dehors de votre définition de flux de travail. Utilisez uniquement des expressions de définition de flux de travail à l’intérieur de votre définition de flux de travail.

   Lorsque vous spécifiez les valeurs de vos paramètres de définition de flux de travail, vous pouvez référencer des paramètres de modèle à l’aide de la section de paramètres en dehors de votre définition de flux de travail, mais toujours à l’intérieur de la définition de ressource pour votre application logique. De cette façon, vous pouvez transmettre les valeurs de paramètre de modèle dans les paramètres de définition de votre flux de travail.

1. Stockez les valeurs de vos paramètres dans un [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md) séparé et incluez ce fichier dans votre déploiement.

## <a name="process-strings-with-functions"></a>Traiter des chaînes avec des fonctions

Logic Apps possède différentes fonctions permettant de manipuler des chaînes.
Par exemple, supposons que vous souhaitiez transmettre un nom de société d’une commande à un autre système.
Toutefois, vous n’êtes pas sûr de la gestion correcte du codage de caractères.
Vous pouvez effectuer un codage base64 sur cette chaîne, mais pour éviter les séquences d’échappement dans l’URL, il est conseillé de remplacer plusieurs caractères. De même, vous avez uniquement besoin d’une sous-chaîne du nom de la société, car les cinq premiers caractères ne sont pas utilisés.

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "order": {
      "defaultValue": {
        "quantity": 10,
        "id": "myorder1",
        "companyName": "NAME=Contoso"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "request": {
      "type": "Request",
      "kind": "Http"
    }
  },
  "actions": {
    "order": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://www.example.com/?id=@{replace(replace(base64(substring(parameters('order').companyName,5,sub(length(parameters('order').companyName), 5) )),'+','-') ,'/' ,'_' )}"
      }
    }
  },
  "outputs": {}
}
```

Les étapes suivantes décrivent la façon dont cet exemple traite cette chaîne, de l’intérieur vers l’extérieur :

```
"uri": "https://www.example.com/?id=@{replace(replace(base64(substring(parameters('order').companyName,5,sub(length(parameters('order').companyName), 5) )),'+','-') ,'/' ,'_' )}"
```

1. Obtenez l’élément [`length()`](../logic-apps/logic-apps-workflow-definition-language.md) pour le nom de la société afin de récupérer le nombre total de caractères.

2. Pour obtenir une chaîne plus courte, soustrayez `5`.

3. Obtenez à présent un élément [`substring()`](../logic-apps/logic-apps-workflow-definition-language.md).
Commencez à l’index `5`, puis passez au reste de la chaîne.

4. Nous convertissons cette sous-chaîne en une chaîne [`base64()`](../logic-apps/logic-apps-workflow-definition-language.md).

5. À présent, exécutez l’action [`replace()`](../logic-apps/logic-apps-workflow-definition-language.md) pour remplacer tous les caractères `+` par des caractères `-`.

6. Pour finir, exécutez l’action [`replace()`](../logic-apps/logic-apps-workflow-definition-language.md) pour remplacer tous les caractères `/` par des caractères `_`.

## <a name="map-list-items-to-property-values-then-use-maps-as-parameters"></a>Mapper des éléments de liste aux valeurs de propriété, puis utiliser des mappages en tant que paramètres

Pour obtenir des résultats différents en fonction de la valeur d’une propriété, vous pouvez créer un mappage qui associe chaque valeur de propriété à un résultat, puis utiliser ce mappage en tant que paramètre.

Par exemple, ce flux de travail définit certaines catégories en tant que paramètres et un mappage qui associe ces catégories à une URL spécifique.
Tout d’abord, le flux de travail obtient une liste d’articles. Ensuite, le flux de travail utilise le mappage pour rechercher l’URL correspondant à la catégorie de chaque article.

*   La fonction [`intersection()`](../logic-apps/logic-apps-workflow-definition-language.md) vérifie si la catégorie correspond à une catégorie définie connue.

*   Après avoir obtenu une catégorie correspondante, l’exemple extrait l’élément du mappage à l’aide de crochets : `parameters[...]`

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "specialCategories": {
      "defaultValue": [
        "science",
        "google",
        "microsoft",
        "robots",
        "NSA"
      ],
      "type": "Array"
    },
    "destinationMap": {
      "defaultValue": {
        "science": "https://www.nasa.gov",
        "microsoft": "https://www.microsoft.com/en-us/default.aspx",
        "google": "https://www.google.com",
        "robots": "https://en.wikipedia.org/wiki/Robot",
        "NSA": "https://www.nsa.gov/"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "Request": {
      "type": "Request",
      "kind": "http"
    }
  },
  "actions": {
    "getArticles": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://ajax.googleapis.com/ajax/services/feed/load?v=1.0&q=https://feeds.wired.com/wired/index"
      }
    },
    "forEachArticle": {
      "type": "foreach",
      "foreach": "@body('getArticles').responseData.feed.entries",
      "actions": {
        "ifGreater": {
          "type": "if",
          "expression": "@greater(length(intersection(item().categories, parameters('specialCategories'))), 0)",
          "actions": {
            "getSpecialPage": {
              "type": "Http",
              "inputs": {
                "method": "GET",
                "uri": "@parameters('destinationMap')[first(intersection(item().categories, parameters('specialCategories')))]"
              }
            }
          }
        }
      },
      "runAfter": {
        "getArticles": [
          "Succeeded"
        ]
      }
    }
  }
}
```

## <a name="get-data-with-date-functions"></a>Obtenir des données avec des fonctions de date

Pour obtenir des données à partir d’une source de données qui ne prend pas en charge des *déclencheurs* en mode natif, vous pouvez utiliser des fonctions de date pour utiliser plutôt des heures et des dates.
Par exemple, cette expression recherche la durée des étapes de ce flux de travail, de l’intérieur vers l’extérieur :

``` json
"expression": "@less(actions('order').startTime,addseconds(utcNow(),-1))",
```

1. Dans l’action `order`, extrayez l’élément `startTime`.
2. Obtenez l’heure en cours avec l’élément `utcNow()`.
3. Soustrayez une seconde :

   [`addseconds(..., -1)`](../logic-apps/logic-apps-workflow-definition-language.md)

   Vous pouvez utiliser d’autres unités de temps, par exemple `minutes` ou `hours`.

3. À présent, vous pouvez comparer ces deux valeurs.

   Si la première valeur est inférieure à la seconde, plus d’une seconde s’est écoulée depuis que la commande a été passée.

Pour mettre en forme les dates, vous pouvez utiliser des formateurs de chaîne. Par exemple, pour obtenir RFC1123, utilisez [`utcnow('r')`](../logic-apps/logic-apps-workflow-definition-language.md).
En savoir plus sur [la mise en forme des dates](../logic-apps/logic-apps-workflow-definition-language.md).

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "order": {
      "defaultValue": {
        "quantity": 10,
        "id": "myorder-id"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "Request": {
      "type": "request",
      "kind": "http"
    }
  },
  "actions": {
    "order": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://www.example.com/?id=@{parameters('order').id}"
      }
    },
    "ifTimingWarning": {
      "type": "If",
      "expression": "@less(actions('order').startTime,addseconds(utcNow(),-1))",
      "actions": {
        "timingWarning": {
          "type": "Http",
          "inputs": {
            "method": "GET",
            "uri": "https://www.example.com/?recordLongOrderTime=@{parameters('order').id}&currentTime=@{utcNow('r')}"
          }
        }
      },
      "runAfter": {
        "order": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Instructions conditionnelles : Exécuter des étapes en fonction d’une condition dans des applications logiques](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Instructions switch : Exécuter différentes étapes en fonction de valeurs spécifiques](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Loops: Process arrays or repeat actions until a condition is met](../logic-apps/logic-apps-control-flow-loops.md) (Boucles : Traiter des tableaux ou répéter des actions jusqu’à ce qu’une condition soit remplie)
* [Exécuter ou joindre des étapes (branches) parallèles](../logic-apps/logic-apps-control-flow-branches.md)
* [Étendues : Exécuter des étapes en fonction de l’état d’un groupe](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
* En savoir plus sur le [schéma du langage de définition du flux de travail pour Azure Logic Apps](../logic-apps/logic-apps-workflow-definition-language.md)
* En savoir plus sur les [actions et déclencheurs de flux de travail pour Azure Logic Apps](../logic-apps/logic-apps-workflow-actions-triggers.md)
