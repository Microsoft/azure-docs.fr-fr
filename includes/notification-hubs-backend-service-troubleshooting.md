---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 343fef941870b6deaad777fac9b9d258d454aa2b
ms.sourcegitcommit: 971a3a63cf7da95f19808964ea9a2ccb60990f64
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2020
ms.locfileid: "85095294"
---
#### <a name="no-response-from-the-backend-service"></a>Aucune réponse du service back-end

Lors du test local, vérifiez que le service back-end est en cours d’exécution et qu’il utilise le port approprié.

Si vous effectuez le test par rapport à l’**application API Azure**, vérifiez que le service est en cours d’exécution, qu’il a été déployé et qu’il a démarré sans erreur.

Veillez à vérifier que vous avez correctement spécifié l’adresse de base dans **[Postman](https://www.postman.com/downloads)** ou dans la configuration de l’application mobile lors du test par le biais du client. L’adresse de base doit être `https://<api_name>.azurewebsites.net/` ou `https://localhost:5001/` lors du test local.

#### <a name="not-receiving-notifications-on-android-after-starting-or-stopping-a-debug-session"></a>Aucune notification reçue sur Android après le démarrage ou l’arrêt d’une session de débogage

Veillez à vous réinscrire après le démarrage ou l’arrêt d’une session de débogage. Le débogueur déclenche la génération d’un nouveau jeton **Firebase**. L’installation du hub de notification doit également être mise à jour.

#### <a name="receiving-a-401-status-code-from-the-backend-service"></a>Réception d’un code d’état 401 du service back-end

Vérifiez que vous définissez l’en-tête de demande **apikey** et que cette valeur correspond à celle que vous avez configurée pour le service back-end.

Si vous recevez cette erreur quand vous procédez à un test local, vérifiez que la valeur de clé que vous avez définie dans la configuration du client correspond à la valeur du paramètre utilisateur **Authentication:ApiKey** utilisée par l’[API](#create-the-api-app).

Si vous effectuez un test avec une **application API**, assurez-vous que la valeur de clé dans le fichier de configuration du client correspond au paramètre d’application **Authentication:ApiKey** que vous utilisez dans l’[application API](#create-the-api-app).

> [!NOTE]
> Si vous avez créé ou modifié ce paramètre après avoir déployé le service back-end, vous devez redémarrer le service afin qu’il prenne effet.

Si vous avez choisi de ne pas effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional), assurez-vous que vous n’avez pas appliqué l’attribut **Authorize** à la classe **NotificationsController**.

#### <a name="receiving-a-404-status-code-from-the-backend-service"></a>Réception d’un code d’état 404 du service back-end

Vérifiez que le point de terminaison et la méthode de demande HTTP sont corrects. Par exemple, les points de terminaison doivent être :

- **[PUT]** `https://<api_name>.azurewebsites.net/api/notifications/installations`
- **[DELETE]** `https://<api_name>.azurewebsites.net/api/notifications/installations/<installation_id>`
- **[POST]** `https://<api_name>.azurewebsites.net/api/notifications/requests`

Ou lors du test local :

- **[PUT]** `https://localhost:5001/api/notifications/installations`
- **[DELETE]** `https://localhost:5001/api/notifications/installations/<installation_id>`
- **[POST]** `https://localhost:5001/api/notifications/requests`

Quand vous spécifiez l’adresse de base dans l’application cliente, assurez-vous qu’elle se termine par un caractère `/`. L’adresse de base doit être `https://<api_name>.azurewebsites.net/` ou `https://localhost:5001/` lors du test local.

#### <a name="unable-to-register-and-a-notification-hub-error-message-is-displayed"></a>Inscription impossible et affichage d’un message d’erreur du hub de notification

Vérifiez que l’appareil de test dispose d’une connectivité réseau. Ensuite, déterminez le code d’état de la réponse HTTP en définissant un point d’arrêt pour inspecter la valeur de propriété **StatusCode** dans **HttpResponse**.

Passez en revue les suggestions de résolution des problèmes précédentes, le cas échéant, en fonction du code d’État.

Définissez un point d’arrêt sur les lignes qui retournent ces codes d’état pour l’API concernée. Essayez ensuite d’appeler le service back-end lors du débogage local.

Vérifiez que le service back-end fonctionne comme prévu via **[Postman](https://www.postman.com/downloads)** à l’aide de la charge utile appropriée. Utilisez la charge utile réelle créée par le code client pour la plateforme en question.

Passez en revue les sections de configuration propres à la plateforme pour vous assurer qu’aucune étape n’a été manquée. Vérifiez que les valeurs appropriées sont résolues pour les variables `installation id` et `token` pour la plateforme appropriée.

#### <a name="unable-to-resolve-an-id-for-the-device-error-message-is-displayed"></a>Affichage du message d’erreur selon lequel il est impossible de résoudre un ID pour l’appareil

Passez en revue les sections de configuration propres à la plateforme pour vous assurer qu’aucune étape n’a été manquée.
