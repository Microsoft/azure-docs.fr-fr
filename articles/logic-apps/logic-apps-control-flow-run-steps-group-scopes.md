---
title: Regrouper et exécuter des actions par étendue
description: Créer des actions d’étendue qui s’exécutent en fonction de l’état du groupe dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.date: 10/03/2018
ms.topic: article
ms.openlocfilehash: 95b5cc191ac6857bf8e1b09e70b22d928473fe03
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92314840"
---
# <a name="run-actions-based-on-group-status-by-using-scopes-in-azure-logic-apps"></a>Exécuter des actions en fonction de l’état du groupe à l’aide d’étendues dans Azure Logic Apps

Pour exécuter des actions uniquement après l’échec ou la réussite d’un groupe d’actions, placez ce groupe dans une *étendue*. Cette structure est utile lorsque vous souhaitez organiser les actions en tant que groupe logique, évaluer l’état de ce groupe et effectuer des actions qui sont basées sur l’état de l’étendue. Une fois que toutes les actions d’une étendue ont été exécutées, l’étendue récupère également son propre état. Par exemple, vous pouvez utiliser des étendues lorsque vous souhaitez implémenter la [gestion des erreurs et des exceptions](../logic-apps/logic-apps-exception-handling.md#scopes). 

Pour vérifier l’état d’une étendue, vous pouvez utiliser les mêmes critères que ceux utilisés pour déterminer l’état d’exécution des applications logiques, tels que **Réussi**, **Échec**, **Annulé**, etc. Par défaut, quand toutes les actions de l’étendue réussissent, l’état de l’étendue est marqué comme **Réussi**. Mais quand une action dans l’étendue échoue ou est annulée, l’état de l’étendue est défini sur **Échec**. Pour les limites sur les étendues, consultez [Limites et configuration](../logic-apps/logic-apps-limits-and-config.md). 

Par exemple, voici une application logique de haut niveau qui utilise une étendue pour exécuter des actions spécifiques et une condition pour vérifier l’état de l’étendue. Si des actions de l’étendue échouent ou se ferment de façon inattendue, l’étendue est définie sur **Échec** ou **Abandonné**, respectivement ; l’application logique envoie un message « Scope failed » (Échec de l’étendue). Si toutes les actions de l’étendue réussissent, l’application logique envoie un message « Scope succeeded » (Réussite de l’étendue).

![Diagramme montrant le flux de l’étendue de l’application logique avec des exemples d’échec et de réussite de l’étendue](./media/logic-apps-control-flow-run-steps-group-scopes/scope-high-level.png)

## <a name="prerequisites"></a>Prérequis

Pour suivre l’exemple de cet article, vous avez besoin de ces éléments :

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/). 

* Un compte de messagerie de n’importe quel fournisseur de messagerie pris en charge par Logic Apps. Cet exemple utilise Outlook.com. Si vous utilisez un autre fournisseur, le flux général reste le même, mais votre interface utilisateur peut s’afficher différemment.

* Une clé Bing Maps. Pour obtenir cette clé, consultez <a href="/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key" target="_blank">Getting a Bing Maps Key</a> (Obtention d’une clé Bing Maps).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

## <a name="create-sample-logic-app"></a>Créer un exemple d’application logique

Commencez par créer cet exemple d’application logique pour que vous puissiez ajouter une étendue plus tard :

![Créer un exemple d’application logique](./media/logic-apps-control-flow-run-steps-group-scopes/finished-sample-app.png)

* Un déclencheur **Planification - Périodicité** qui vérifie le service Bing Maps à un intervalle que vous spécifiez
* Une action **Bing Maps - Obtenir un itinéraire** qui vérifie le temps de trajet entre deux emplacements
* Une instruction conditionnelle qui vérifie si le temps de trajet dépasse votre temps de trajet spécifié
* Une action qui vous envoie un e-mail indiquant que ce temps de trajet actuel dépasse celui que vous avez spécifié

Vous pouvez enregistrer votre application logique à tout moment, par conséquent, enregistrez votre travail souvent.

1. Si ce n’est pas déjà fait, connectez-vous au <a href="https://portal.azure.com" target="_blank">portail Azure</a>. Créez une application logique vide.

1. Ajoutez le déclencheur **Planification - Périodicité** avec ces paramètres : **Intervalle** = « 1 » et **Fréquence** = « Minute »

   ![Configurer le déclencheur « Planification - Périodicité »](./media/logic-apps-control-flow-run-steps-group-scopes/recurrence.png)

   > [!TIP]
   > Pour simplifier l’affichage et masquer les détails de chaque action dans le concepteur, réduisez la forme de chaque action au cours de ces étapes.

1. Ajoutez l’action **Bing Maps - Obtenir un itinéraire**. 

   1. Si vous ne disposez pas d’une connexion à Bing Maps, vous êtes invité à en créer une.

      | Paramètre | Valeur | Description |
      | ------- | ----- | ----------- |
      | **Nom de connexion** | BingMapsConnection | Donnez un nom à votre connexion. | 
      | **Clé API** | <*your-Bing-Maps-key*> | Entrez la clé Bing Cartes que vous avez reçue précédemment. | 
      ||||  

   1. Configurez votre action **Obtenir un itinéraire**, comme le montre le tableau sous cette image :

      ![Configurer l’action « Bing Maps - Obtenir un itinéraire »](./media/logic-apps-control-flow-run-steps-group-scopes/get-route.png) 

      Pour plus d’informations sur ces paramètres, voir [Calculate a route (Calculer un itinéraire)](/bingmaps/rest-services/routes/calculate-a-route).

      | Paramètre | Valeur | Description |
      | ------- | ----- | ----------- |
      | **Étape 1** | <*start*> | Entrez l’origine de votre itinéraire. | 
      | **Étape 2** | <*end*> | Entrez la destination de votre itinéraire. | 
      | **Avoid** | None | Entrez les éléments à éviter sur votre itinéraire, par exemple les autoroutes, les péages, etc. Pour les valeurs possibles, consultez [Calculate a route](/bingmaps/rest-services/routes/calculate-a-route) (Calculer un itinéraire). | 
      | **Optimize** | timeWithTraffic | Sélectionnez un paramètre permettant d’optimiser votre itinéraire, par exemple la distance, la durée du trajet avec les informations de circulation actuelle, etc. Cet exemple utilise cette valeur : « timeWithTraffic » | 
      | **Unité de distance** | <*your-preference*> | Entrez l’unité de distance pour calculer votre itinéraire. Cet exemple utilise cette valeur : « Mile » | 
      | **Mode de déplacement** | Conduite | Entrez le mode de déplacement pour votre itinéraire. Cet exemple utilise cette valeur : « Driving » | 
      | **Date et heure de transit** | None | S’applique au mode transit uniquement. | 
      | **Type de date et heure de transit** | None | S’applique au mode transit uniquement. | 
      ||||  

1. [Ajoutez une condition](../logic-apps/logic-apps-control-flow-conditional-statement.md) qui vérifie si le temps de trajet actuel dépasse une durée spécifiée. 
   Pour cet exemple, suivez ces étapes :

   1. Renommez la condition avec cette description : **Si le temps de trafic est supérieur à la durée spécifiée**

   1. Dans la colonne de gauche, cliquez à l’intérieur de la zone **Choisir une valeur** pour afficher la liste de contenu dynamique. Dans cette liste, sélectionnez le champ **Travel Duration Traffic** (Trafic correspondant à la durée du trajet), qui est exprimé en secondes. 

      ![Créer une condition](./media/logic-apps-control-flow-run-steps-group-scopes/build-condition.png)

   1. Dans la zone du milieu, sélectionnez cet opérateur : **est supérieur à**.

   1. Dans la colonne la plus à droite, entrez cette valeur de comparaison, équivalente à 10 minutes en secondes : **600**

      Une fois que vous avez terminé, votre condition ressemble à cet exemple :

      ![Condition terminée](./media/logic-apps-control-flow-run-steps-group-scopes/finished-condition.png)

1. Dans de la branche **If true** (Si true), ajoutez une action « Envoyer e-mail » pour votre fournisseur d’e-mail. 
   Définissez cette action en suivant les étapes décrites sous cette image :

   ![Ajouter une action « Envoyer e-mail » à la branche « If true » (Si true)](./media/logic-apps-control-flow-run-steps-group-scopes/send-email.png)

   1. Dans le champ **À**, entrez votre adresse e-mail à des fins de test.

   1. Dans le champ **Sujet**, entrez ce texte :

      ```Time to leave: Traffic more than 10 minutes```

   1. Dans le champ **Corps**, entrez ce texte avec un espace de fin : 

      ```Travel time:```

      Alors que le curseur s’affiche dans le champ **Corps**, la liste de contenu dynamique reste ouverte afin que vous puissiez sélectionner tous les paramètres qui sont disponibles à ce stade.

   1. Dans la liste de contenu dynamique, choisissez **Expression**.

   1. Recherchez et sélectionnez la fonction **div()** . 
      Placez votre curseur dans les parenthèses de la fonction.

   1. Alors que votre curseur se trouve dans les parenthèses de la fonction, choisissez **Contenu dynamique** pour afficher la liste de contenu dynamique. 
   
   1. À partir de la section **Obtenir un itinéraire**, sélectionnez le champ **Travel Duration Traffic** (Trafic correspondant à la durée du trajet).

      ![Sélectionner « Travel Duration Traffic » (Trafic correspondant à la durée du trajet)](./media/logic-apps-control-flow-run-steps-group-scopes/send-email-2.png)

   1. Une fois que le champ correspond au format JSON, ajoutez une **virgule** (```,```) suivie du nombre ```60``` afin de convertir la valeur dans **Travel Duration Traffic** (Trafic correspondant à la durée du trajet) exprimée en secondes en minutes. 
   
      ```
      div(body('Get_route')?['travelDurationTraffic'],60)
      ```

      Votre expression ressemble maintenant à cet exemple :

      ![Expression finale](./media/logic-apps-control-flow-run-steps-group-scopes/send-email-3.png)  

   1. Une fois que vous avez terminé, sélectionnez **OK**.

   <!-- markdownlint-disable MD038 -->
   1. Une fois que l’expression est résolue, ajoutez ce texte avec un espace de début : ``` minutes```
  
       Votre champ **Corps** ressemble maintenant à cet exemple :

       ![Champ « Corps » final](./media/logic-apps-control-flow-run-steps-group-scopes/send-email-4.png)
   <!-- markdownlint-enable MD038 -->

1. Enregistrez votre application logique.

Ensuite, ajoutez une étendue afin que vous puissiez regrouper des actions spécifiques et évaluer leur état.

## <a name="add-a-scope"></a>Ajouter une étendue

1. Si ce n’est pas déjà le cas, ouvrez votre application logique dans le concepteur d’application logique. 

1. Ajoutez une étendue à l’emplacement du flux de travail que vous souhaitez. Par exemple, pour ajouter une étendue entre des étapes existantes dans le workflow de l’application logique, effectuez les étapes suivantes : 

   1. Déplacez votre pointeur sur la flèche où vous souhaitez ajouter l’étendue. 
   Choisissez le **signe plus** ( **+** ) > **Ajouter une action**.

      ![Ajouter une étendue](./media/logic-apps-control-flow-run-steps-group-scopes/add-scope.png)

   1. Dans la zone de recherche, entrez « étendue » comme filtre. 
   Sélectionnez l’action **Étendue**.

## <a name="add-steps-to-scope"></a>Ajouter des étapes à l’étendue

1. Ajoutez maintenant les étapes ou faites glisser les étapes que vous souhaitez exécuter dans l’étendue. Pour cet exemple, faites glisser ces actions dans l’étendue :
      
   * **Obtenir un itinéraire**
   * **Si le temps de trajet est supérieur au temps spécifié**, qui comprend les branches **true** et **false**

   Votre application logique ressemble maintenant à cet exemple :

   ![Étendue ajoutée](./media/logic-apps-control-flow-run-steps-group-scopes/scope-added.png)

1. Sous l’étendue, ajoutez une condition qui vérifie l’état de l’étendue. Renommez la condition avec cette description : **En cas d’échec de l’étendue**

   ![Ajouter une condition pour vérifier l’état de l’étendue](./media/logic-apps-control-flow-run-steps-group-scopes/add-condition-check-scope-status.png)
  
1. Dans la condition, ajoutez ces expressions qui vérifient si l’état de l’étendue est égal à « Échec » ou « Abandonné ». 

   1. Pour ajouter une ligne, choisissez **Ajouter**. 

   1. Dans chaque ligne, cliquez dans la zone de gauche pour afficher la liste de contenu dynamique. 
   Dans la liste de contenu dynamique, choisissez **Expression**. Dans la zone d’édition, entrez cette expression, puis choisissez **OK** : 
   
      `result('Scope')[0]['status']`

      ![Capture d’écran montrant la zone « Expression » avec mise en évidence de l’expression de résultat](./media/logic-apps-control-flow-run-steps-group-scopes/check-scope-status.png)

   1. Pour les deux lignes, sélectionnez **est égal à** comme opérateur. 
   
   1. Pour les valeurs de comparaison, dans la première ligne, entrez `Failed`. 
   Dans la deuxième ligne, entrez `Aborted`. 

      Une fois que vous avez terminé, votre condition ressemble à cet exemple :

      ![Ajouter l’expression qui vérifie l’état de l’étendue](./media/logic-apps-control-flow-run-steps-group-scopes/check-scope-status-finished.png)

      Maintenant, définissez la propriété `runAfter` de la condition afin que celle-ci vérifie l’état de l’étendue et exécute l’action correspondante que vous définissez dans les étapes ultérieures.

   1. Sur la condition **Si l’étendue a échoué**, choisissez le bouton **points de suspension** (...), puis choisissez **Configurer la propriété Exécuter après**.

      ![Configurer la propriété « runAfter »](./media/logic-apps-control-flow-run-steps-group-scopes/configure-run-after.png)

   1. Sélectionnez tous ces états d’étendue : **a réussi**, **a échoué**, **est ignorée** et **a expiré**

      ![Sélectionner les états de l’étendue](./media/logic-apps-control-flow-run-steps-group-scopes/select-run-after-statuses.png)

   1. Quand vous avez fini, choisissez **Terminé**. 
   La condition affiche maintenant une icône « informations ».

1. Dans les branches **If true** (Si true) et **If false** (Si false), ajoutez les actions que vous souhaitez effectuer en fonction de chaque état d’étendue, par exemple, envoyer un e-mail ou un message.

   ![Ajouter les actions à effectuer en fonction de l’état de l’étendue](./media/logic-apps-control-flow-run-steps-group-scopes/handle-true-false-branches.png)

1. Enregistrez votre application logique.

Votre application logique terminée ressemble maintenant à cet exemple :

![Application logique finale avec étendue](./media/logic-apps-control-flow-run-steps-group-scopes/scopes-overview.png)

## <a name="test-your-work"></a>Tester votre travail

Dans la barre d’outils du concepteur, choisissez **Exécuter**. Si toutes les actions de l’étendue réussissent, vous recevez un message « Scope succeeded » (Réussite de l’étendue). Si des actions de l’étendue échouent, vous recevez un message « Scope failed » (Échec de l’étendue). 

<a name="scopes-json"></a>

## <a name="json-definition"></a>Définition JSON

Si vous travaillez en mode code, vous pouvez définir une structure d’étendue dans la définition JSON de votre application logique à la place. Par exemple, voici la définition JSON du déclencheur et des actions dans l’application logique précédente :

``` json
"triggers": {
  "Recurrence": {
    "type": "Recurrence",
    "recurrence": {
       "frequency": "Minute",
       "interval": 1
    }
  }
}
```

```json
"actions": {
  "If_scope_failed": {
    "type": "If",
    "actions": {
      "Scope_failed": {
        "type": "ApiConnection",
        "inputs": {
          "body": {
            "Body": "Scope failed. Scope status: @{result('Scope')[0]['status']}",
            "Subject": "Scope failed",
            "To": "<your-email@domain.com>"
          },
          "host": {
            "connection": {
              "name": "@parameters('$connections')['outlook']['connectionId']"
            }
          },
          "method": "post",
          "path": "/Mail"
        },
        "runAfter": {}
      }
    },
    "else": {
      "actions": {
        "Scope_succeded": {
          "type": "ApiConnection",
          "inputs": {
            "body": {
              "Body": "Scope succeeded. Scope status: @{result('Scope')[0]['status']}",
              "Subject": "Scope succeeded",
              "To": "<your-email@domain.com>"
            },
            "host": {
              "connection": {
               "name": "@parameters('$connections')['outlook']['connectionId']"
              }
            },
            "method": "post",
            "path": "/Mail"
          },
          "runAfter": {}
        }
      }
    },
    "expression": {
      "or": [ 
         {
            "equals": [ 
              "@result('Scope')[0]['status']", 
              "Failed"
            ]
         },
         {
            "equals": [
               "@result('Scope')[0]['status']", 
               "Aborted"
            ]
         } 
      ]
    },
    "runAfter": {
      "Scope": [
        "Failed",
        "Skipped",
        "Succeeded",
        "TimedOut"
      ]
    }
  },
  "Scope": {
    "type": "Scope",
    "actions": {
      "Get_route": {
        "type": "ApiConnection",
        "inputs": {
          "host": {
            "connection": {
              "name": "@parameters('$connections')['bingmaps']['connectionId']"
            }
          },
          "method": "get",
          "path": "/REST/V1/Routes/Driving",
          "queries": {
            "distanceUnit": "Mile",
            "optimize": "timeWithTraffic",
            "travelMode": "Driving",
            "wp.0": "<start>",
            "wp.1": "<end>"
          }
        },
        "runAfter": {}
      },
      "If_traffic_time_is_more_than_specified_time": {
        "type": "If",
        "actions": {
          "Send_mail_when_traffic_exceeds_10_minutes": {
            "type": "ApiConnection",
            "inputs": {
              "body": {
                 "Body": "Travel time:@{div(body('Get_route')?['travelDurationTraffic'],60)} minutes",
                 "Subject": "Time to leave: Traffic more than 10 minutes",
                 "To": "<your-email@domain.com>"
              },
              "host": {
                "connection": {
                   "name": "@parameters('$connections')['outlook']['connectionId']"
                }
              },
              "method": "post",
              "path": "/Mail"
            },
            "runAfter": {}
          }
        },
        "expression": {
          "and" : [
            {
               "greater": [ 
                  "@body('Get_route')?['travelDurationTraffic']", 
                  600
               ]
            }
          ]
        },
        "runAfter": {
          "Get_route": [
            "Succeeded"
          ]
        }
      }
    },
    "runAfter": {}
  }
},
```

## <a name="get-support"></a>Obtenir de l’aide

* Pour toute question, consultez la page [Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Pour voter pour des fonctionnalités et suggestions ou pour en soumettre, visitez le [site de commentaires des utilisateurs Azure Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Étapes suivantes

* [Instructions conditionnelles : Exécuter des étapes en fonction d’une condition dans des applications logiques](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Instructions switch : Exécuter différentes étapes en fonction de valeurs spécifiques](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Loops: Process arrays or repeat actions until a condition is met](../logic-apps/logic-apps-control-flow-loops.md) (Boucles : Traiter des tableaux ou répéter des actions jusqu’à ce qu’une condition soit remplie)
* [Exécuter ou joindre des étapes (branches) parallèles](../logic-apps/logic-apps-control-flow-branches.md)