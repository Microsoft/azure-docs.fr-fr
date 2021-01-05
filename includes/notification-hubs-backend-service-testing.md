---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 5e75c5d5510f596eb7911cae0310e60b6bef67bf
ms.sourcegitcommit: 5cace04239f5efef4c1eed78144191a8b7d7fee8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/08/2020
ms.locfileid: "86146451"
---
### <a name="send-a-test-notification"></a>Envoyer une notification de test

1. Ouvrez un nouvel onglet dans [Postman](https://www.postman.com/downloads/).

1. Définissez la requête sur **POST** et entrez l’adresse suivante :

    ```xml
    https://<app_name>.azurewebsites.net/api/notifications/requests
    ```

1. Si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional), veillez à configurer les en-têtes de demande pour inclure votre valeur **apikey**.

   | Clé                            | Valeur                          |
   | ------------------------------ | ------------------------------ |
   | apikey                         | <votre_clé_d’api>                 |

1. Choisissez l’option **raw** (brut) pour **Body** (Corps), choisissez **JSON** dans la liste des options de format, puis incluez du contenu **JSON** d’espace réservé :

    ```json
    {
        "text": "Message from Postman!",
        "action": "action_a"
    }
    ```

1. Sélectionnez le bouton **Code** qui apparaît dans le coin supérieur droit sous le bouton **Save** (Enregistrer). La demande doit ressembler à l’exemple suivant quand elle est affichée pour **HTML** (selon que vous avez inclus un en-tête **apikey**) :

    ```html
    POST /api/notifications/requests HTTP/1.1
    Host: https://<app_name>.azurewebsites.net
    apikey: <your_api_key>
    Content-Type: application/json

    {
        "text": "Message from backend service",
        "action": "action_a"
    }
    ```

1. Exécutez l’application **PushDemo** sur au moins l’une des plateformes cibles (**Android** et **iOS**).

    > [!NOTE]
    > Si vous effectuez un test sur **Android**, assurez-vous que vous n’êtes pas en mode **Debug**, ou si l’application a été déployée au moment de son exécution, forcez sa fermeture, puis redémarrez-la à partir du lanceur.

1. Dans l’application **PushDemo**, appuyez sur le bouton **Register** (Inscrire).

1. De retour dans **[Postman](https://www.postman.com/downloads)** , fermez la fenêtre **Generate Code Snippets** (Générer des extraits de code) (si ce n’est déjà fait), puis cliquez sur le bouton **Send** (Envoyer).

1. Vérifiez que vous recevez une réponse **200 OK** dans **[Postman](https://www.postman.com/downloads)** et que l’alerte s’affiche dans l’application sous la forme **ActionA action received** (Action ActionA reçue).  

1. Fermez l’application **PushDemo**, puis recliquez sur **Send** dans **[Postman](https://www.postman.com/downloads)** .

1. Vérifiez que vous recevez une réponse **200 OK** dans **[Postman](https://www.postman.com/downloads)** . Vérifiez qu’une notification s’affiche dans la zone de notification pour l’application **PushDemo** avec le message approprié.

1. Appuyez sur la notification pour confirmer qu’elle ouvre l’application et qu’elle a affiché l’alerte **ActionA action received**.

1. De retour dans **[Postman](https://www.postman.com/downloads)** , modifiez le corps de la demande précédent pour envoyer une notification silencieuse en spécifiant *action_b* au lieu d’*action_a* pour la valeur **action**.

    ```json
    {
        "action": "action_b",
        "silent": true
    }
    ```

1. L’application toujours ouverte, cliquez sur le bouton **Send** dans **[Postman](https://www.postman.com/downloads)** .

1. Vérifiez que vous recevez une réponse **200 OK** dans **[Postman](https://www.postman.com/downloads)** et que l’alerte s’affiche dans l’application sous la forme **ActionB action received** au lieu d’**ActionA action received**.

1. Fermez l’application **PushDemo**, puis recliquez sur **Send** dans **[Postman](https://www.postman.com/downloads)** .

1. Vérifiez que vous recevez une réponse **200 OK** dans **[Postman](https://www.postman.com/downloads)** et que la notification silencieuse n’apparaît pas dans la zone de notification.
