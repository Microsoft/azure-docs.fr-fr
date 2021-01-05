---
title: Mettre à jour une commande à partir d’une application cliente
titleSuffix: Azure Cognitive Services
description: Découvrez comment mettre à jour une commande à partir d’une application cliente.
services: cognitive-services
author: nitinme
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/20/2020
ms.author: nitinme
ms.openlocfilehash: 08c674a7a7ec060a4273836064cb1c21e979e725
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560285"
---
# <a name="update-a-command-from-a-client-app"></a>Mettre à jour une commande à partir d’une application cliente

Dans cet article, vous allez apprendre à mettre à jour une commande en cours à partir d’une application cliente.

## <a name="prerequisites"></a>Prérequis
> [!div class = "checklist"]
> * Application [Commandes personnalisées](quickstart-custom-commands-application.md) créée précédemment

## <a name="update-the-state-of-a-command"></a>Mettre à jour l’état d’une commande

Si votre application cliente nécessite la mise à jour de l’état d’une commande en cours sans entrée vocale, vous pouvez envoyer un événement pour mettre à jour la commande.

Pour illustrer ce scénario, envoyez l’activité d’événement suivante pour mettre à jour l’état d’une commande en cours (`TurnOnOff`) : 

```json
{
  "type": "event",
  "name": "RemoteUpdate",
  "value": {
    "updatedCommand": {
      "name": "TurnOnOff",
      "updatedParameters": {
        "OnOff": "on"
      },
      "cancel": false
    },
    "updatedGlobalParameters": {},
    "processTurn": true
  }
}
```

Passons en revue les attributs clés de cette activité :

| Attribut | Explication |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **type** | L’activité est de type `"event"` et le nom de l’événement doit être `"RemoteUpdate"`. |
| **value** | L’attribut `"value"` contient les attributs nécessaires pour mettre à jour la commande en cours. |
| **updatedCommand** | L’attribut `"updatedCommand"` contient le nom de la commande. Au sein de cet attribut, `"updatedParameters"` est un mappage qui inclut les noms des paramètres et leurs valeurs mises à jour. |
| **cancel** | Si la commande en cours doit être annulée, affectez la valeur `true` à l’attribut `"cancel"`. |
| **updatedGlobalParameters** | L’attribut `"updatedGlobalParameters"` est un mappage comme `"updatedParameters"`, mais il est utilisé pour les paramètres globaux. |
| **processTurn** | Si le passage doit être traité après l’envoi de l’activité, la valeur `true` est affectée à l’attribut `"processTurn"`. |

Vous pouvez tester ce scénario dans le portail de l’application Commandes personnalisées :

1. Ouvrez l’application Commandes personnalisées que vous avez précédemment créée. 
1. Sélectionnez **Entraîner**, puis **Tester**.
1. Envoyez `turn`.
1. Ouvrez le panneau latéral, puis sélectionnez **Éditeur d’activités**.
1. Tapez et envoyez l’événement `RemoteCommand` spécifié dans la section précédente.
    > [!div class="mx-imgBorder"]
    > ![Capture d’écran qui illustre l’événement pour une commande à distance.](media/custom-commands/send-remote-command-activity.png)

Notez que la valeur du paramètre `"OnOff"` a été définie sur `"on"` à l’aide d’une activité du client, et non à partir d’une entrée vocale ou de texte.

## <a name="update-the-catalog-of-the-parameter-for-a-command"></a>Mettre à jour le catalogue du paramètre d’une commande

Quand vous configurez la liste des options valides pour un paramètre, les valeurs du paramètre sont définies globalement pour l’application. 

Dans notre exemple, le paramètre `SubjectDevice` a une liste fixe de valeurs prises en charge, quelle que soit la conversation.

Si vous voulez ajouter de nouvelles entrées au catalogue des paramètres pour chaque conversation, vous pouvez envoyer l’activité suivante :

```json
{
  "type": "event",
  "name": "RemoteUpdate",
  "value": {
    "catalogUpdate": {
      "commandParameterCatalogs": {
        "TurnOnOff": [
          {
            "name": "SubjectDevice",
            "values": {
              "stereo": [
                "cd player"
              ]
            }
          }
        ]
      }
    },
    "processTurn": false
  }
}
```
Avec cette activité, vous avez ajouté une entrée pour `"stereo"` au catalogue du paramètre `"SubjectDevice"` dans la commande `"TurnOnOff"`.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-commands/update-catalog-with-remote-activity.png" alt-text="Capture d’écran montrant une mise à jour du catalogue.":::

Notez quelques points :
- Vous ne devez envoyer cette activité qu’une seule fois (dans l’idéal, juste après le démarrage d’une connexion).
- Après avoir envoyé cette activité, vous devez attendre que l’événement `ParameterCatalogsUpdated` soit renvoyé au client.

## <a name="add-more-context-from-the-client-application"></a>Ajouter plus de contexte à partir de l’application cliente

Vous pouvez définir un contexte supplémentaire à partir de l’application cliente pour chaque conversation. Vous pouvez ensuite utiliser ce contexte dans votre application Commandes personnalisées. 

Par exemple, pensez au scénario dans lequel vous souhaitez envoyer l’ID et le nom de l’appareil connecté à l’application Commandes personnalisées.

Pour tester ce scénario, créons une commande dans l’application actuelle :
1. Créez une commande appelée `GetDeviceInfo`.
1. Ajoutez un exemple de phrase `get device info`.
1. Dans la règle de complétion **Terminé**, ajoutez une action d’**envoi de réponse vocale** qui contient les attributs de `clientContext`.
   ![Capture d’écran montrant une réponse pour l’envoi de données vocales avec un contexte.](media/custom-commands/send-speech-response-context.png)
1. Enregistrez, entraînez et testez votre application.
1. Dans la fenêtre de test, envoyez une activité pour mettre à jour le contexte du client.

    ```json
    {
       "type": "event",
       "name": "RemoteUpdate",
       "value": {
         "clientContext": {
           "deviceId": "12345",
           "deviceName": "My device"
         },
         "processTurn": false
       }
    }
    ```
1. Envoyez le texte `get device info`.
   ![Capture d’écran montrant une activité pour l’envoi du contexte du client.](media/custom-commands/send-client-context-activity.png)

Notez quelques points :
- Vous ne devez envoyer cette activité qu’une seule fois (dans l’idéal, juste après le démarrage d’une connexion).
- Vous pouvez utiliser des objets complexes pour `clientContext`.
- Vous pouvez utiliser `clientContext` dans les réponses vocales, pour l’envoi d’activités et pour l’appel de points de terminaison web.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Mettre à jour une commande à partir d’un point de terminaison web](./how-to-custom-commands-update-command-from-web-endpoint.md)
