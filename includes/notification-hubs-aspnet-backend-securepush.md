---
author: sethmanheim
ms.service: notification-hubs
ms.topic: include
ms.date: 09/14/2020
ms.author: sethm
ms.openlocfilehash: fb3c95b74128f1da7b29a290e17fefe21987dd76
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019415"
---
## <a name="webapi-project"></a>Projet WebAPI

1. Dans Visual Studio, ouvrez le projet **AppBackend** que vous avez créé dans le didacticiel **Notification des utilisateurs** .
2. Dans le fichier Notifications.cs, remplacez la totalité de la classe **Notifications** par le code suivant. Veillez à remplacer les espaces réservés par la chaîne de connexion (avec accès complet) de votre hub de notification et par le nom de celui-ci. Ces valeurs sont disponibles dans le [portail Azure](https://portal.azure.com). Ce module représente maintenant les différentes notifications sécurisées qui seront envoyées. Dans les implémentations complètes, les notifications sont stockées dans une base de données. Par souci de simplification, nous les stockons ici en mémoire.

   ```csharp
    public class Notification
    {
        public int Id { get; set; }
        public string Payload { get; set; }
        public bool Read { get; set; }
    }

    public class Notifications
    {
        public static Notifications Instance = new Notifications();

        private List<Notification> notifications = new List<Notification>();

        public NotificationHubClient Hub { get; set; }

        private Notifications() {
            Hub = NotificationHubClient.CreateClientFromConnectionString("{conn string with full access}",     "{hub name}");
        }

        public Notification CreateNotification(string payload)
        {
            var notification = new Notification() {
            Id = notifications.Count,
            Payload = payload,
            Read = false
            };

            notifications.Add(notification);

            return notification;
        }

        public Notification ReadNotification(int id)
        {
            return notifications.ElementAt(id);
        }
    }
    ```

3. Dans le fichier NotificationsController.cs, remplacez le code de la définition de classe **NotificationsController** par le code suivant. Ce composant permet à l'appareil de récupérer la notification en toute sécurité. Dans l'exemple de ce didacticiel, il vous permet également de déclencher une notification push sécurisée sur vos appareils. Notez que la notification envoyée ici au hub de notification est brute puisqu’elle comporte uniquement son ID (sans message) :

   ```csharp
    public NotificationsController()
    {
        Notifications.Instance.CreateNotification("This is a secure notification!");
    }

    // GET api/notifications/id
    public Notification Get(int id)
    {
        return Notifications.Instance.ReadNotification(id);
    }

    public async Task<HttpResponseMessage> Post()
    {
        var secureNotificationInTheBackend = Notifications.Instance.CreateNotification("Secure confirmation.");
        var usernameTag = "username:" + HttpContext.Current.User.Identity.Name;

        // windows
        var rawNotificationToBeSent = new Microsoft.Azure.NotificationHubs.WindowsNotification(secureNotificationInTheBackend.Id.ToString(),
                        new Dictionary<string, string> {
                            {"X-WNS-Type", "wns/raw"}
                        });
        await Notifications.Instance.Hub.SendNotificationAsync(rawNotificationToBeSent, usernameTag);

        // apns
        await Notifications.Instance.Hub.SendAppleNativeNotificationAsync("{\"aps\": {\"content-available\": 1}, \"secureId\": \"" + secureNotificationInTheBackend.Id.ToString() + "\"}", usernameTag);

        // gcm
        await Notifications.Instance.Hub.SendGcmNativeNotificationAsync("{\"data\": {\"secureId\": \"" + secureNotificationInTheBackend.Id.ToString() + "\"}}", usernameTag);

        return Request.CreateResponse(HttpStatusCode.OK);
    }
    ```

Notez que la méthode `Post` n'envoie pas de notification toast. Elle envoie une notification brute qui contient uniquement l’ID de la notification, sans aucun contenu sensible. Veillez également à commenter l'opération d'envoi pour les plateformes pour lesquelles aucune information d'identification n'est configurée sur votre hub de notification, car celles-ci généreront des erreurs.

1. Nous allons maintenant redéployer cette application sur un site web Azure afin de la rendre accessible à tous les appareils. Cliquez avec le bouton droit sur le projet **AppBackend**, puis sélectionnez **Publier**.
2. Sélectionnez Site web Azure comme cible de publication. Connectez-vous avec votre compte Azure, sélectionnez un site web (nouveau ou existant), puis notez la valeur de la propriété **URL de destination** dans l’onglet **Connexion**. Plus loin dans ce didacticiel, nous utiliserons cette URL comme *point de terminaison principal* . Cliquez sur **Publier**.
