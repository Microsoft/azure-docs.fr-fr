---
title: Créer et gérer des variables pour stocker et transmettre des valeurs
description: Découvrez comment stocker, gérer, utiliser et transmettre des valeurs à l’aide de variables dans vos tâches et flux de travail automatisés créés avec Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 09/20/2019
ms.openlocfilehash: b486b94a74d98f5630bd0bf40ebf0864c2ec5ab8
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91333900"
---
# <a name="store-and-manage-values-by-using-variables-in-azure-logic-apps"></a>Stocker et gérer des valeurs en utilisant des variables dans Azure Logic Apps

Cet article explique comment créer et utiliser des variables pour stocker des valeurs dans votre application logique. Des variables peuvent, par exemple, vous aider à suivre le nombre d’exécutions d’une boucle. Pour itérer sur un tableau ou rechercher un élément spécifique dans un tableau, vous pouvez utiliser une variable qui référence le numéro d’index de chaque élément du tableau.

Vous pouvez créer des variables pour des types de données tels que entier, flottant, booléen, chaîne, tableau et objet. Après avoir créé une variable, vous pouvez effectuer d’autres tâches, par exemple :

* Obtenir ou référencer la valeur de la variable.
* Augmenter ou réduire la variable d’une valeur constante, opérations également appelées *incrémenter* et *décrémenter*.
* Affecter une valeur différente à la variable.
* Insérer, ou *ajouter*, la valeur de la variable à la fin d’une chaîne ou d’un tableau.

Les variables existent et sont globales uniquement au sein de l’instance d’application logique qui les crée. En outre, elles sont conservées dans toutes les itérations de boucle à l’intérieur d’une instance d’application logique. Quand vous référencez une variable, utilisez le nom de celle-ci comme jeton, au lieu du nom de l’action qui est la façon habituelle de référencer les résultats d’une action.

> [!IMPORTANT]
> Par défaut, les cycles dans une boucle « For each » s’exécutent en parallèle. Quand vous utilisez des variables dans des boucles, exécutez la boucle [séquentiellement](../logic-apps/logic-apps-control-flow-loops.md#sequential-foreach-loop) afin que les variables retournent des résultats prévisibles.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* L’application logique dans laquelle vous souhaitez créer la variable

  Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un [déclencher](../logic-apps/logic-apps-overview.md#logic-app-concepts) en tant que première étape de votre application logique

  Pour que vous puissiez ajouter des actions pour la création et l’utilisation de variables, votre application logique doit démarrer avec un déclencheur.

<a name="create-variable"></a>

## <a name="initialize-variable"></a>Initialiser la variable

Vous pouvez créer une variable et déclarer son type de données et sa valeur initiale, le tout au sein d’une seule action dans votre application logique. Vous ne pouvez déclarer des variables qu’au niveau global, pas dans des étendues, des conditions ou des boucles.

1. Dans le [portail Azure](https://portal.azure.com) ou dans Visual Studio, ouvrez votre application logique dans le Concepteur d’application logique.

   Cet exemple utilise le portail Azure et une application logique avec un déclencheur existant.

1. Dans votre application logique, à l’étape où vous souhaitez ajouter une variable, effectuez l’une des opérations suivantes : 

   * Pour ajouter une action après la dernière étape, sélectionnez **Nouvelle étape**.

     ![Capture d’écran montrant l’action « Nouvelle étape » sélectionnée sur la page « Concepteur d’application logique ».](./media/logic-apps-create-variables-store-values/add-action.png)

   * Pour ajouter une action entre des étapes, positionnez la souris sur la flèche de connexion de façon à ce que le signe plus ( **+** ) s’affiche. Sélectionnez le signe plus, puis sélectionnez **Ajouter une action**.

1. Sous **Choisir une action**, dans la zone de recherche, entrez `variables` en guise de filtre. Dans la liste d’actions, sélectionnez **Initialiser la variable**.

   ![Action select](./media/logic-apps-create-variables-store-values/select-initialize-variable-action.png)

1. Entrez ces informations sur votre variable, comme indiqué ci-dessous :

   | Propriété | Obligatoire | Valeur |  Description |
   |----------|----------|-------|--------------|
   | **Nom** | Oui | <*variable-name*> | Nom de la variable à incrémenter |
   | **Type** | Oui | <*variable-type*> | Type de données de la variable. |
   | **Valeur** | Non | <*start-value*> | Valeur initiale de votre variable. <p><p>**Conseil** : Bien que cette valeur soit facultative, nous vous recommandons de la définir afin de toujours connaître la valeur initiale de votre variable. |
   |||||

   Par exemple :

   ![Initialiser la variable](./media/logic-apps-create-variables-store-values/initialize-variable.png)

1. À présent, continuez à ajouter les actions de votre choix. Lorsque c’est chose faite, dans la barre d’outils du concepteur, sélectionnez **Enregistrer**.

Si vous passez du concepteur à l’éditeur en mode Code, voici comment l’action **Initialiser la variable** s’affiche dans la définition de votre application logique, qui est au format JSON (JavaScript Object Notation) :

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "Count",
               "type": "Integer",
               "value": 0
          } ]
      },
      "runAfter": {}
   }
},
```

> [!NOTE]
> Bien que l’action **Initialiser la variable** comporte une section `variables` qui est structurée sous la forme d’un tableau, elle ne peut créer qu’une seule variable à la fois. Chaque nouvelle variable requiert une action **Initialiser la variable** individuelle.

Voici quelques exemples pour d’autres types de variables :

*Variable de chaîne*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myStringVariable",
               "type": "String",
               "value": "lorem ipsum"
          } ]
      },
      "runAfter": {}
   }
},
```

*Variable booléenne*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myBooleanVariable",
               "type": "Boolean",
               "value": false
          } ]
      },
      "runAfter": {}
   }
},
```

*Tableau d’entiers*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myArrayVariable",
               "type": "Array",
               "value": [1, 2, 3]
          } ]
      },
      "runAfter": {}
   }
},
```

*Tableau de chaînes*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myArrayVariable",
               "type": "Array",
               "value": ["red", "orange", "yellow"]
          } ]
      },
      "runAfter": {}
   }
},
```

<a name="get-value"></a>

## <a name="get-the-variables-value"></a>Obtenir la valeur de la variable

Pour extraire ou référencer le contenu d’une variable, vous pouvez également utiliser la [fonction variables()](../logic-apps/workflow-definition-language-functions-reference.md#variables) dans le Concepteur d’application logique et l’éditeur en mode Code. Lorsque vous référencez une variable, utilisez le nom de celle-ci en tant que jeton, pas le nom de l’action qui est la façon habituelle de référencer les résultats d’une action.

Par exemple, cette expression obtient les éléments de la variable tableau [créée précédemment dans cet article](#append-value) à l’aide de la fonction `variables()`. La fonction `string()` retourne le contenu de la variable sous forme de chaîne : `"1, 2, 3, red"`

```json
@{string(variables('myArrayVariable'))}
```

<a name="increment-value"></a>

## <a name="increment-variable"></a>Incrémenter une variable 

Pour augmenter ou *incrémenter* une variable d’une valeur constante, ajoutez l’action **Incrémenter une variable** à votre application logique. Cette action fonctionne uniquement avec les variables de type entier et flottant.

1. Dans le Concepteur d’application logique, à l’étape où vous souhaitez augmenter une variable existante, sélectionnez **Nouvelle étape**. 

   Par exemple, cette application logique a déjà un déclencheur et une action qui a créé une variable. Par conséquent, ajoutez une nouvelle action sous ces étapes :

   ![Ajouter une action](./media/logic-apps-create-variables-store-values/add-increment-variable-action.png)

   Pour ajouter une action entre des étapes existantes, positionnez votre souris sur la flèche de connexion de façon à ce que le signe plus (+) s’affiche. Sélectionnez le signe plus, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez « Incrémenter une variable » comme filtre. Dans la liste des actions, sélectionnez **Incrémenter une variable**.

   ![Sélectionnez l’action « Incrémenter une variable ».](./media/logic-apps-create-variables-store-values/select-increment-variable-action.png)

1. Fournissez les informations suivantes pour l’incrémentation de votre variable :

   | Propriété | Obligatoire | Valeur |  Description |
   |----------|----------|-------|--------------|
   | **Nom** | Oui | <*variable-name*> | Nom de la variable à incrémenter |
   | **Valeur** | Non | <*increment-value*> | Valeur utilisée pour incrémenter la variable. La valeur par défaut est 1. <p><p>**Conseil** : Bien que cette valeur soit facultative, nous vous recommandons de la définir afin de toujours connaître la valeur spécifique pour l’incrémentation de votre variable. |
   ||||

   Par exemple :

   ![Exemple de valeur d’incrément](./media/logic-apps-create-variables-store-values/increment-variable-action-information.png)

1. Lorsque c’est chose faite, dans la barre d’outils du concepteur, sélectionnez **Enregistrer**.

Si vous passez du concepteur à l’éditeur en mode Code, voici comment l’action **Incrémenter une variable** s’affiche dans la définition de votre application logique, qui est au format JSON :

```json
"actions": {
   "Increment_variable": {
      "type": "IncrementVariable",
      "inputs": {
         "name": "Count",
         "value": 1
      },
      "runAfter": {}
   }
},
```

## <a name="example-create-loop-counter"></a>Exemple : créer un compteur de boucle

Des variables sont couramment utilisées pour compter le nombre d’exécutions d’une boucle. Cet exemple montre comment créer et utiliser des variables pour cette tâche en créant une boucle qui compte les pièces jointes à un e-mail.

1. Dans le portail Azure, créez une application logique vide. Ajoutez un déclencheur qui vérifie la présence de nouveaux e-mails et pièces jointes.

   Cet exemple utilise le déclencheur Office 365 Outlook **quand un nouvel e-mail arrive**. Vous pouvez configurer ce déclencheur de façon à ce qu’il s’active uniquement lorsque l’e-mail contient des pièces jointes. En revanche, vous pouvez utiliser n’importe quel connecteur qui vérifie la présence de nouveaux e-mails contenant des pièces jointes, tel que le connecteur Outlook.com.

1. Dans le déclencheur, pour vérifier la présence de pièces jointes et le passage de celles-ci au workflow de votre application logique, sélectionnez **Oui** pour ces propriétés :

   * **Contient une pièce jointe**
   * **Inclure des pièces jointes**

   ![Rechercher et inclure des pièces jointes](./media/logic-apps-create-variables-store-values/check-include-attachments.png)

1. Ajoutez l’action [**Initialiser la variable**](#create-variable). Créez une variable de type entier nommée `Count` et de valeur initiale zéro.

   ![Ajouter une action pour « Initialiser la variable »](./media/logic-apps-create-variables-store-values/initialize-variable.png)

1. Pour parcourir chaque pièce jointe, ajoutez une boucle *for each*.

   1. Sous l’action **Initialiser la variable**, sélectionnez **Nouvelle étape**.

   1. Sous **Choisir une action**, sélectionnez **Intégré**. Dans la zone de recherche, entrez `for each` comme filtre de recherche, puis sélectionnez **For each**.

      ![Ajouter une boucle « pour chaque »](./media/logic-apps-create-variables-store-values/add-loop.png)

1. Dans la boucle, cliquez dans la zone **Select an output from previous steps (Sélectionner un résultat des étapes précédentes)** . Lorsque la liste de contenus dynamiques s’affiche, sélectionnez **Pièces jointes**.

   ![Sélectionner « Pièces jointes »](./media/logic-apps-create-variables-store-values/select-attachments.png)

   La propriété **Pièces jointes** passe à votre boucle un tableau qui contient les pièces jointes aux e-mails à partir de la sortie du déclencheur.

1. Dans la boucle **For each**, sélectionnez **Ajouter une action**.

   ![Sélectionner « Ajouter une action »](./media/logic-apps-create-variables-store-values/add-action-2.png)

1. Dans la zone de recherche, entrez « Incrémenter une variable » comme filtre. Dans la liste des actions, sélectionnez **Incrémenter une variable**.

   > [!NOTE]
   > Assurez-vous que l’action **Incrémenter une variable** apparaît dans la boucle. Si l’action apparaît hors de la boucle, faites-la glisser dans la boucle.

1. Dans l’action **Incrémenter une variable**, dans la liste **Nom**, sélectionnez la variable **Nombre**.

   ![Sélectionner la variable « Nombre »](./media/logic-apps-create-variables-store-values/add-increment-variable-example.png)

1. Sous la boucle, ajoutez une action qui vous envoie le nombre de pièces jointes. Dans l’action, incluez la valeur de la variable **Nombre**, par exemple :

   ![Ajouter une action qui envoie les résultats](./media/logic-apps-create-variables-store-values/send-email-results.png)

1. Enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

### <a name="test-your-logic-app"></a>Tester votre application logique

1. Si votre application logique n’est pas activée, dans le menu associé, sélectionnez **Vue d’ensemble**. Dans la barre d’outils, sélectionnez **Activer**.

1. Dans la barre d’outils du Concepteur d’application logique, sélectionnez **Exécuter**. Cette étape démarre manuellement votre application logique.

1. Envoyez un e-mail avec une ou plusieurs des pièces jointes au compte de courrier que vous avez utilisé dans cet exemple.

   Cette étape active le déclencheur de l’application logique, qui crée et exécute une instance de flux de travail de votre application logique. Ensuite, l’application logique vous envoie un message ou un e-mail indiquant le nombre de pièces jointes dans l’e-mail que vous avez envoyé.

Si vous passez du concepteur à l’éditeur en mode Code, voici comment la boucle **For each** s’affiche avec l’action **Incrémenter une variable** dans la définition de votre application logique, qui est au format JSON.

```json
"actions": {
   "For_each": {
      "type": "Foreach",
      "actions": {
         "Increment_variable": {
           "type": "IncrementVariable",
            "inputs": {
               "name": "Count",
               "value": 1
            },
            "runAfter": {}
         }
      },
      "foreach": "@triggerBody()?['Attachments']",
      "runAfter": {
         "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

<a name="decrement-value"></a>

## <a name="decrement-variable"></a>Décrémenter une variable

Pour diminuer ou *décrémenter* une variable d’une valeur constante, procédez de la même manière que pour [augmenter une variable](#increment-value), mais à la place recherchez et sélectionnez l’action **Décrémenter une variable**. Cette action fonctionne uniquement avec les variables de type entier et flottant.

Voici les propriétés disponibles pour l’action **Décrémenter une variable** :

| Propriété | Obligatoire | Valeur |  Description |
|----------|----------|-------|--------------|
| **Nom** | Oui | <*variable-name*> | Nom de la variable à décrémenter | 
| **Valeur** | Non | <*increment-value*> | Valeur de la décrémentation de la variable. La valeur par défaut est 1. <p><p>**Conseil** : Bien que cette valeur soit facultative, nous vous recommandons de la définir afin de toujours connaître la valeur spécifique pour la décrémentation de votre variable. |
||||| 

Si vous passez du concepteur à l’éditeur en mode Code, voici comment l’action **Décrémenter une variable** s’affiche dans la définition de votre application logique, qui est au format JSON.

```json
"actions": {
   "Decrement_variable": {
      "type": "DecrementVariable",
      "inputs": {
         "name": "Count",
         "value": 1
      },
      "runAfter": {}
   }
},
```

<a name="assign-value"></a>

## <a name="set-variable"></a>Définir une variable

Pour attribuer une valeur différente à une variable, procédez de la même manière que pour [augmenter une variable](#increment-value) sauf que vous devez :

1. Rechercher et sélectionner l’action **Définir une variable** à la place.

1. Fournissez le nom de la variable et la valeur à lui attribuer. Le type de données de la nouvelle valeur et de la variable doivent être le même. La valeur est requise, car cette action n’a pas de valeur par défaut.

Voici les propriétés disponibles pour l’action **Définir une variable** :

| Propriété | Obligatoire | Valeur |  Description |
|----------|----------|-------|--------------|
| **Nom** | Oui | <*variable-name*> | Nom de la variable à modifier |
| **Valeur** | Oui | <*new-value*> | Valeur que vous souhaitez attribuer à la variable. Le type de données des deux doit être identique. |
||||| 

> [!NOTE]
> Sauf si vous incrémentez ou décrémentez des variables, la modification de variables dans des boucles *peut* produire des résultats inattendus, car les boucles s’exécutent en parallèle ou simultanément par défaut. Dans ce cas, essayez de définir votre boucle pour qu’elle s’exécute de manière séquentielle. Par exemple, lorsque vous souhaitez référencer la valeur de la variable à l’intérieur de la boucle et attendez la même valeur au début et à la fin de cette instance de boucle, procédez comme suit pour modifier la façon dont la boucle s’exécute : 
>
> 1. Dans l’angle supérieur droit de votre boucle, sélectionnez le bouton représentant des points de suspension ( **...** ), puis sélectionnez **Paramètres**.
> 
> 2. Sous **Contrôle d’accès concurrentiel**, définissez le paramètre **Remplacer les valeurs par défaut** sur **Activé**.
>
> 3. Positionnez le curseur **Degré de parallélisme** sur **1**.

Si vous passez du concepteur à l’éditeur en mode Code, voici comment l’action **Définir une variable** s’affiche dans la définition de votre application logique, qui est au format JSON. Cet exemple change la valeur actuelle de la variable `Count`.

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "Count",
               "type": "Integer",
               "value": 0
          } ]
      },
      "runAfter": {}
   },
   "Set_variable": {
      "type": "SetVariable",
      "inputs": {
         "name": "Count",
         "value": 100
      },
      "runAfter": {
         "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

<a name="append-value"></a>

## <a name="append-to-variable"></a>Ajouter à une variable

Pour les variables qui stockent des chaînes ou des tableaux, vous pouvez insérer ou *ajouter* la valeur d’une variable à la fin de ces chaînes ou des tableaux. Vous pouvez procéder de la même manière que pour [augmenter une variable](#increment-value), à l’exception des étapes suivantes : 

1. Recherchez et sélectionnez une des actions suivantes, selon que votre variable est une chaîne ou un tableau : 

   * **Ajouter à une variable chaîne**
   * **Ajouter à une variable tableau** 

1. Spécifiez la valeur à ajouter à la fin de la chaîne ou du tableau. Cette valeur est requise.

Voici les propriétés disponibles pour les actions **Ajouter à...**  :

| Propriété | Obligatoire | Valeur |  Description |
|----------|----------|-------|--------------|
| **Nom** | Oui | <*variable-name*> | Nom de la variable à modifier |
| **Valeur** | Oui | <*append-value*> | Valeur que vous souhaitez ajouter, qui peut être de tout type |
|||||

Si vous passez du concepteur à l’éditeur en mode Code, voici comment l’action **Ajouter à une variable tableau** s’affiche dans la définition de votre application logique, qui est au format JSON. Cet exemple crée une variable tableau et ajoute une autre valeur à la fin du tableau. Le résultat est une variable mise à jour qui contient ce tableau : `[1,2,3,"red"]`

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
            "name": "myArrayVariable",
            "type": "Array",
            "value": [1, 2, 3]
         } ]
      },
      "runAfter": {}
   },
   "Append_to_array_variable": {
      "type": "AppendToArrayVariable",
      "inputs": {
         "name": "myArrayVariable",
         "value": "red"
      },
      "runAfter": {
        "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [connecteurs Logic Apps](../connectors/apis-list.md)
