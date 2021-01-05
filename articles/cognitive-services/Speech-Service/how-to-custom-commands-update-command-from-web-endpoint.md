---
title: Mettre à jour une commande à partir d’un point de terminaison web
titleSuffix: Azure Cognitive Services
description: Découvrez comment mettre à jour l’état d’une commande à l’aide d’un appel à un point de terminaison web.
services: cognitive-services
author: nitinme
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/20/2020
ms.author: nitinme
ms.openlocfilehash: d0b77e6af36f0a71405f6c032bfdd121abeb0071
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560268"
---
# <a name="update-a-command-from-a-web-endpoint"></a>Mettre à jour une commande à partir d’un point de terminaison web

Si votre application cliente a besoin de mettre à jour l’état d’une commande en cours sans entrée vocale, vous pouvez utiliser un appel à un point de terminaison web pour mettre à jour la commande.

Dans cet article, vous allez découvrir comment mettre à jour une commande en cours à partir d’un point de terminaison web.

## <a name="prerequisites"></a>Prérequis
> [!div class = "checklist"]
> * Application [Commandes personnalisées](quickstart-custom-commands-application.md) créée précédemment

## <a name="create-an-azure-function"></a>Création d’une fonction Azure 

Pour cet exemple, vous avez besoin d’une [fonction Azure](../../azure-functions/index.yml) déclenchée par HTTP, qui prend en charge l’entrée suivante (ou un sous-ensemble de cette entrée) :

```JSON
{
  "conversationId": "SomeConversationId",
  "currentCommand": {
    "name": "SomeCommandName",
    "parameters": {
      "SomeParameterName": "SomeParameterValue",
      "SomeOtherParameterName": "SomeOtherParameterValue"
    }
  },
  "currentGlobalParameters": {
      "SomeGlobalParameterName": "SomeGlobalParameterValue",
      "SomeOtherGlobalParameterName": "SomeOtherGlobalParameterValue"
  }
}
```

Passons en revue les attributs clés de cette entrée :

| Attribut | Explication |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **conversationId** | Identificateur unique de la conversation. Notez que cet ID peut être généré à partir de l’application cliente. |
| **currentCommand** | Commande actuellement active dans la conversation. |
| **name** | Nom de la commande. L’attribut `parameters` est un mappage avec les valeurs actuelles des paramètres. |
| **currentGlobalParameters** | Mappage similaire à `parameters`, mais utilisé pour les paramètres globaux. |

La sortie de la fonction Azure doit prendre en charge le format suivant :

```JSON
{
  "updatedCommand": {
    "name": "SomeCommandName",
    "updatedParameters": {
      "SomeParameterName": "SomeParameterValue"
    },
    "cancel": false
  },
  "updatedGlobalParameters": {
    "SomeGlobalParameterName": "SomeGlobalParameterValue"
  }
}
```

Vous pouvez reconnaître ce format, car il est identique à celui que vous avez utilisé durant la [mise à jour d’une commande à partir du client](./how-to-custom-commands-update-command-from-client.md). 

À présent, créez une fonction Azure basée sur Node.js. Copiez/collez ce code :

```nodejs
module.exports = async function (context, req) {
    context.log(req.body);
    context.res = {
        body: {
            updatedCommand: {
                name: "IncrementCounter",
                updatedParameters: {
                    Counter: req.body.currentCommand.parameters.Counter + 1
                }
            }
        }
    };
}
```

Quand vous appelez cette fonction Azure à partir de commandes personnalisées, vous envoyez les valeurs actuelles de la conversation. Vous retournez les paramètres que vous voulez mettre à jour ou vous voulez annuler la commande actuelle.

## <a name="update-the-existing-custom-commands-app"></a>Mettre à jour l’application Commandes personnalisées existante

Connectons la fonction Azure à l’application Commandes personnalisées existante :

1. Ajoutez une nouvelle commande nommée `IncrementCounter`.
1. Ajoutez un seul exemple de phrase avec la valeur `increment`.
1. Ajoutez un nouveau paramètre appelé `Counter` (même nom que celui spécifié dans la fonction Azure) de type `Number` avec la valeur par défaut `0`.
1. Ajoutez un nouveau point de terminaison web appelé `IncrementEndpoint` en indiquant l’URL de votre fonction Azure et avec l’option **Mises à jour distantes** définie sur **Activées**.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-commands/set-web-endpoint-with-remote-updates.png" alt-text="Capture d’écran qui montre la définition d’un point de terminaison web avec des mises à jour distantes.":::
1. Créez une règle d’interaction appelée **IncrementRule**, puis ajoutez une action **Appeler un point de terminaison web**.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-commands/increment-rule-web-endpoint.png" alt-text="Capture d’écran montrant la création d’une règle d’interaction.":::
1. Dans la configuration de l’action, sélectionnez `IncrementEndpoint`. Configurez **En cas de réussite** sur **Envoyer une réponse vocale** avec la valeur `Counter` et configurez **En cas d’échec** avec un message d’erreur.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-commands/set-increment-counter-call-endpoint.png" alt-text="Capture d’écran qui montre la définition d’un compteur d’incréments pour l’appel d’un point de terminaison web.":::
1. Définissez l’état post-exécution de la règle sur **Attendre l’entrée de l’utilisateur**.

## <a name="test-it"></a>Testez-le

1. Enregistrez et entraînez votre application.
1. Sélectionnez **Test**.
1. Envoyez `increment` plusieurs fois (il s’agit de l’exemple de phrase pour la commande `IncrementCounter`).
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-commands/increment-counter-example.png" alt-text="Capture d’écran montrant un exemple de compteur d’incréments.":::

Notez que la fonction Azure incrémente la valeur du paramètre `Counter` à chaque passage.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Activer un processus CI/CD pour votre application Commandes personnalisées](./how-to-custom-commands-deploy-cicd.md)