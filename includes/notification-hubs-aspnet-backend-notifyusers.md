---
title: Fichier Include
description: Fichier Include qui contient du code pour créer un projet de serveur principal WebAPI ASP.NET.
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 09/11/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 6f490b6f25112ed8a10bbd865070bd07ea3ee84f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016940"
---
## <a name="create-the-webapi-project"></a>Création du projet WebAPI

Les sections suivantes traitent de la création d’un serveur principal WebAPI ASP.NET. Ce processus a trois objectifs principaux :

- **Authentification des clients** : ajoutez un gestionnaire de messages pour authentifier les demandes des clients et associer l’utilisateur à la demande.
- **Inscriptions aux notifications à l’aide du serveur principal WebAPI** : ajoutez un contrôleur pour gérer les nouvelles inscriptions afin qu’un appareil client puisse recevoir des notifications. Le nom d’utilisateur authentifié est automatiquement ajouté à l’inscription en tant que [balise](../articles/notification-hubs/notification-hubs-tags-segment-push-message.md).
- **Envoi de notifications aux clients** : ajoutez un contrôleur permettant aux utilisateurs de déclencher une notification push sécurisée pour les appareils et les clients associés à la balise.

Pour créer le serveur principal WebAPI ASP.NET, procédez comme suit :

> [!IMPORTANT]
> Si vous utilisez Visual Studio 2015 ou une version antérieure, avant de commencer ce didacticiel, vérifiez que la version la plus récente du Gestionnaire de package NuGet pour Visual Studio est installée.
>
>Pour ce faire, lancez Visual Studio. Dans le menu **Outils**, sélectionnez **Extensions et mises à jour**. Recherchez **Gestionnaire de package NuGet** pour votre version de Visual Studio et vérifiez que vous disposez de la dernière version. Si vous ne disposez pas de la version la plus récente, désinstallez votre version actuelle et réinstallez le Gestionnaire de package NuGet.

![Capture d’écran de la boîte de dialogue Extensions et mises à jour avec le package Gestionnaire de package NuGet pour Visual Studio mis en évidence.][B4]

> [!NOTE]
> Assurez-vous que le [Kit de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/) Visual Studio est installé, pour le déploiement du site web.

1. Démarrez Visual Studio ou Visual Studio Express.

2. Sélectionnez **Explorateur de serveurs** et connectez-vous à votre compte Azure. Pour créer les ressources du site web sur votre compte, vous devez être connecté.

3. Dans Visual Studio, cliquez avec le bouton droit de la souris sur la solution Visual Studio, pointez sur **Ajouter**, puis cliquez sur **Nouveau projet**.
4. Développez **Visual C#** , sélectionnez **Web** puis cliquez sur **Application web ASP.NET**.

5. Dans la zone **Nom**, tapez **AppBackend**, puis sélectionnez **OK**.

    ![Fenêtre Nouveau projet][B1]

6. Dans la fenêtre **Nouveau projet ASP.NET**, sélectionnez la case **Web API** puis **OK**.

    ![Fenêtre Nouveau projet ASP.NET][B2]

7. Dans la fenêtre **Configurer l’application web Microsoft Azure**, sélectionnez l’abonnement puis effectuez l’une des actions suivantes dans la liste **Plan App Service** :

    * Sélectionnez un plan App Service que vous avez déjà créé.
    * Sélectionnez **Créer un plan App Service**, et créez-en un.

   Vous n'avez pas besoin de base de données pour ce didacticiel. Une fois que vous avez sélectionné votre plan App Service, sélectionnez **OK** pour créer le projet.

    ![Fenêtre Configurer l’application web Microsoft Azure][B5]

    Si vous ne voyez pas la page Configurer le plan App Service, poursuivez le tutoriel. Vous pourrez le configurer plus tard, lors de la publication de l’application. 

## <a name="authenticate-clients-to-the-webapi-backend"></a>Authentification des clients sur le serveur principal WebAPI

Dans cette section, vous créez une classe de gestionnaire de messages nommée **AuthenticationTestHandler** pour le nouveau serveur principal. Cette classe est dérivée de [DelegatingHandler](/previous-versions/visualstudio/hh193679(v=vs.118)) et ajoutée comme gestionnaire de messages afin de pouvoir traiter toutes les demandes entrantes dans le serveur principal.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **AppBackend**, puis sélectionnez **Ajouter** et **Classe**.
2. Nommez la nouvelle classe **AuthenticationTestHandler.cs**, puis sélectionnez **Ajouter** pour générer la classe. Par souci de simplification, cette classe permet d’authentifier les utilisateurs via l’ *Authentification de base* . Votre application peut utiliser n’importe quel schéma d’authentification.
3. Dans AuthenticationTestHandler.cs, ajoutez les instructions `using` suivantes :

    ```csharp
    using System.Net.Http;
    using System.Threading;
    using System.Security.Principal;
    using System.Net;
    using System.Text;
    using System.Threading.Tasks;
    ```

4. Dans AuthenticationTestHandler.cs, remplacez la définition de classe `AuthenticationTestHandler` par le code suivant :

    Ce gestionnaire autorise la demande lorsque les trois conditions suivantes sont vraies :

   * La demande comprenait un en-tête d’*autorisation*.
   * La demande utilise l’authentification *de base* .
   * Les chaînes du nom d’utilisateur et du mot de passe sont identiques.

   Sinon, la demande est rejetée. Cette authentification ne répond pas à une véritable approche d’authentification et d’autorisation. Il s’agit uniquement d’un exemple très simple pour ce didacticiel.

   Si le message de la demande est authentifié et autorisé par `AuthenticationTestHandler`, l’utilisateur de l’authentification de base est attaché à la demande actuelle sur [HttpContext](/dotnet/api/system.web.httpcontext.current). Les informations utilisateur dans HttpContext sont utilisées ultérieurement par un autre contrôleur (RegisterController) pour ajouter une [balise](/previous-versions/azure/azure-services/dn530749(v=azure.100)) à la demande d’inscription aux notifications.

    ```csharp
    public class AuthenticationTestHandler : DelegatingHandler
    {
        protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
        {
            var authorizationHeader = request.Headers.GetValues("Authorization").First();

            if (authorizationHeader != null && authorizationHeader
                .StartsWith("Basic ", StringComparison.InvariantCultureIgnoreCase))
            {
                string authorizationUserAndPwdBase64 =
                    authorizationHeader.Substring("Basic ".Length);
                string authorizationUserAndPwd = Encoding.Default
                    .GetString(Convert.FromBase64String(authorizationUserAndPwdBase64));
                string user = authorizationUserAndPwd.Split(':')[0];
                string password = authorizationUserAndPwd.Split(':')[1];

                if (VerifyUserAndPwd(user, password))
                {
                    // Attach the new principal object to the current HttpContext object
                    HttpContext.Current.User =
                        new GenericPrincipal(new GenericIdentity(user), new string[0]);
                    System.Threading.Thread.CurrentPrincipal =
                        System.Web.HttpContext.Current.User;
                }
                else return Unauthorized();
            }
            else return Unauthorized();

            return base.SendAsync(request, cancellationToken);
        }

        private bool VerifyUserAndPwd(string user, string password)
        {
            // This is not a real authentication scheme.
            return user == password;
        }

        private Task<HttpResponseMessage> Unauthorized()
        {
            var response = new HttpResponseMessage(HttpStatusCode.Forbidden);
            var tsc = new TaskCompletionSource<HttpResponseMessage>();
            tsc.SetResult(response);
            return tsc.Task;
        }
    }
    ```

    > [!NOTE]
    > Note de sécurité : la classe `AuthenticationTestHandler` n’offre pas de véritable authentification. Elle sert uniquement à simuler l’authentification de base et n’est pas sécurisée. Vous devez mettre en œuvre un mécanisme d’authentification sécurisé dans vos applications de production et vos services.
5. Ajoutez le code suivant à la fin de la méthode `Register` dans la classe **App_Start/WebApiConfig.cs** pour inscrire le gestionnaire de messages :

    ```csharp
    config.MessageHandlers.Add(new AuthenticationTestHandler());
    ```
6. Enregistrez vos modifications.

## <a name="register-for-notifications-by-using-the-webapi-backend"></a>Inscription aux notifications à l’aide du serveur principal WebAPI

Dans cette section, vous ajoutez un nouveau contrôleur au serveur principal WebAPI pour gérer les demandes d’inscription d’un utilisateur et d’un appareil aux notifications à l’aide de la bibliothèque cliente pour les hubs de notification. Le contrôleur ajoute une balise d’utilisateur pour l’utilisateur qui a été authentifié et attaché à HttpContext par `AuthenticationTestHandler`. La balise a le format de chaîne `"username:<actual username>"`.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **AppBackend**, puis sélectionnez **Gérer les packages NuGet**.

2. Dans le volet gauche, sélectionnez **En ligne** puis tapez **Microsoft.Azure.NotificationHubs** dans la zone **Recherche**.

3. Dans la liste de résultats, sélectionnez **Microsoft Azure Notification Hubs**, puis **Installer**. Procédez à l’installation, puis fermez la fenêtre du Gestionnaire de package NuGet.

    Cette action ajoute une référence au Kit de développement logiciel (SDK) Azure Notification Hubs à l’aide du [package NuGet Microsoft.Azure.Notification Hubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

4. Créez un nouveau fichier de classe qui représente la connexion avec le hub de notification utilisé pour envoyer des notifications. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le dossier **Modèles**, sélectionnez **Ajouter**, puis **Classe**. Nommez la nouvelle classe **Notifications.cs**, puis sélectionnez **Ajouter** pour générer la classe.

    ![Fenêtre Ajouter un nouvel élément][B6]

5. Dans Notifications.cs, ajoutez l’instruction `using` suivante en haut du fichier :

    ```csharp
    using Microsoft.Azure.NotificationHubs;
    ```

6. Remplacez la définition de classe `Notifications` par le code suivant et les deux espaces réservés par la chaîne de connexion (avec accès complet) de votre centre Notification Hubs et le nom du hub (disponible sur le [Portail Azure](https://portal.azure.com)) :

    ```csharp
    public class Notifications
    {
        public static Notifications Instance = new Notifications();

        public NotificationHubClient Hub { get; set; }

        private Notifications() {
            Hub = NotificationHubClient.CreateClientFromConnectionString("<your hub's DefaultFullSharedAccessSignature>",
                                                                            "<hub name>");
        }
    }
    ```
    > [!IMPORTANT]
    > Entrez le **nom** et la **DefaultFullSharedAccessSignature** de votre hub avant de continuer. 
    
7. Ensuite, créez un contrôleur nommé **RegisterController**. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le dossier **Contrôleurs**, sélectionnez **Ajouter**, puis **Contrôleur**.

8. Sélectionnez **Contrôleur API 2 web - Vide**, puis **Ajouter**.

    ![Fenêtre Ajouter un modèle automatique][B7]

9. Dans la boîte **Nom du contrôleur**, tapez **RegisterController** pour nommer la nouvelle classe, puis sélectionnez **Ajouter**.

    ![Fenêtre Ajouter un contrôleur][B8]

10. Dans RegisterController.cs, ajoutez les instructions `using` suivantes :

    ```csharp
    using Microsoft.Azure.NotificationHubs;
    using Microsoft.Azure.NotificationHubs.Messaging;
    using AppBackend.Models;
    using System.Threading.Tasks;
    using System.Web;
    ```
11. Ajoutez le code suivant dans la définition de classe `RegisterController` . Dans ce code, vous ajoutez une balise d’utilisateur pour l’utilisateur attaché à HttpContext. L’utilisateur a été authentifié et attaché à HttpContext par le filtre de messages que vous avez ajouté, `AuthenticationTestHandler`. Vous pouvez également ajouter des contrôles facultatifs afin de vérifier que l’utilisateur dispose des droits d’inscription nécessaires pour les balises requises.

    ```csharp
    private NotificationHubClient hub;

    public RegisterController()
    {
        hub = Notifications.Instance.Hub;
    }

    public class DeviceRegistration
    {
        public string Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }

    // POST api/register
    // This creates a registration id
    public async Task<string> Post(string handle = null)
    {
        string newRegistrationId = null;

        // make sure there are no existing registrations for this push handle (used for iOS and Android)
        if (handle != null)
        {
            var registrations = await hub.GetRegistrationsByChannelAsync(handle, 100);

            foreach (RegistrationDescription registration in registrations)
            {
                if (newRegistrationId == null)
                {
                    newRegistrationId = registration.RegistrationId;
                }
                else
                {
                    await hub.DeleteRegistrationAsync(registration);
                }
            }
        }

        if (newRegistrationId == null) 
            newRegistrationId = await hub.CreateRegistrationIdAsync();

        return newRegistrationId;
    }

    // PUT api/register/5
    // This creates or updates a registration (with provided channelURI) at the specified id
    public async Task<HttpResponseMessage> Put(string id, DeviceRegistration deviceUpdate)
    {
        RegistrationDescription registration = null;
        switch (deviceUpdate.Platform)
        {
            case "mpns":
                registration = new MpnsRegistrationDescription(deviceUpdate.Handle);
                break;
            case "wns":
                registration = new WindowsRegistrationDescription(deviceUpdate.Handle);
                break;
            case "apns":
                registration = new AppleRegistrationDescription(deviceUpdate.Handle);
                break;
            case "fcm":
                registration = new FcmRegistrationDescription(deviceUpdate.Handle);
                break;
            default:
                throw new HttpResponseException(HttpStatusCode.BadRequest);
        }

        registration.RegistrationId = id;
        var username = HttpContext.Current.User.Identity.Name;

        // add check if user is allowed to add these tags
        registration.Tags = new HashSet<string>(deviceUpdate.Tags);
        registration.Tags.Add("username:" + username);

        try
        {
            await hub.CreateOrUpdateRegistrationAsync(registration);
        }
        catch (MessagingException e)
        {
            ReturnGoneIfHubResponseIsGone(e);
        }

        return Request.CreateResponse(HttpStatusCode.OK);
    }

    // DELETE api/register/5
    public async Task<HttpResponseMessage> Delete(string id)
    {
        await hub.DeleteRegistrationAsync(id);
        return Request.CreateResponse(HttpStatusCode.OK);
    }

    private static void ReturnGoneIfHubResponseIsGone(MessagingException e)
    {
        var webex = e.InnerException as WebException;
        if (webex.Status == WebExceptionStatus.ProtocolError)
        {
            var response = (HttpWebResponse)webex.Response;
            if (response.StatusCode == HttpStatusCode.Gone)
                throw new HttpRequestException(HttpStatusCode.Gone.ToString());
        }
    }
    ```
12. Enregistrez vos modifications.

## <a name="send-notifications-from-the-webapi-backend"></a>Envoyer des notifications depuis le serveur principal WebAPI

Dans cette section, vous ajoutez un nouveau contrôleur qui montre aux appareils clients comment envoyer une notification. La notification se base sur la balise du nom d’utilisateur qui utilise la bibliothèque .NET du service d’Azure Notification Hubs dans le serveur principal WebAPI ASP.NET.

1. Créez un nouveau contrôleur nommé **NotificationsController**, de la même façon que vous avez créé **RegisterController** dans la section précédente.

2. Dans NotificationsController.cs, ajoutez les instructions `using` suivantes :

    ```csharp
    using AppBackend.Models;
    using System.Threading.Tasks;
    using System.Web;
    ```
3. Ajoutez la méthode suivante à la classe **NotificationsController** :

    Ce code envoie un type de notification basé sur le paramètre `pns` du service de notification de plateforme. La valeur de `to_tag` permet de définir la balise *username* sur le message. Cette balise doit correspondre à une balise de nom d’utilisateur d’une inscription de hub de notification active. Le message de notification est extrait du corps de la demande POST et mis en forme pour le PNS cible.

    Selon le service Platform Notification Service (PNS) que vos appareils pris en charge utilisent pour recevoir des notifications, ces dernières sont prises en charge dans plusieurs formats. Par exemple sur des appareils Windows, vous pouvez utiliser une [notification toast avec WNS](/uwp/schemas/tiles/toastschema/schema-root) qui n’est pas directement prise en charge par un autre service PNS. Dans une instance de ce type, votre serveur principal doit formater la notification dans un format pris en charge pour le service PNS des appareils que vous envisagez de prendre en charge. Utilisez ensuite l’API d’envoi appropriée sur la [classe NotificationHubClient](/dotnet/api/microsoft.azure.notificationhubs.notificationhubclient).

    ```csharp
    public async Task<HttpResponseMessage> Post(string pns, [FromBody]string message, string to_tag)
    {
        var user = HttpContext.Current.User.Identity.Name;
        string[] userTag = new string[2];
        userTag[0] = "username:" + to_tag;
        userTag[1] = "from:" + user;

        Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;
        HttpStatusCode ret = HttpStatusCode.InternalServerError;

        switch (pns.ToLower())
        {
            case "wns":
                // Windows 8.1 / Windows Phone 8.1
                var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" + 
                            "From " + user + ": " + message + "</text></binding></visual></toast>";
                outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
                break;
            case "apns":
                // iOS
                var alert = "{\"aps\":{\"alert\":\"" + "From " + user + ": " + message + "\"}}";
                outcome = await Notifications.Instance.Hub.SendAppleNativeNotificationAsync(alert, userTag);
                break;
            case "fcm":
                // Android
                var notif = "{ \"data\" : {\"message\":\"" + "From " + user + ": " + message + "\"}}";
                outcome = await Notifications.Instance.Hub.SendFcmNativeNotificationAsync(notif, userTag);
                break;
        }

        if (outcome != null)
        {
            if (!((outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Abandoned) ||
                (outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Unknown)))
            {
                ret = HttpStatusCode.OK;
            }
        }

        return Request.CreateResponse(ret);
    }
    ```
4. Appuyez sur la touche **F5** pour exécuter l’application et vérifier le travail que vous avez accompli jusqu’à présent. L’application ouvre un navigateur web, qui s’affiche sur la page d’accueil ASP.NET.

## <a name="publish-the-new-webapi-backend"></a>Publication du nouveau serveur principal WebAPI

Ensuite, vous déployez l’application sur un site web Azure afin de la rendre accessible à tous les appareils.

1. Cliquez avec le bouton droit sur le projet **AppBackend**, puis sélectionnez **Publier**.

2. Sélectionnez **Microsoft Azure App Service** comme cible de publication, puis cliquez sur \*\*Publier. La fenêtre Créer App Service s’ouvre. Vous pouvez créer toutes les ressources Azure nécessaires pour exécuter l’application web ASP.NET dans Azure.

    ![Vignette Microsoft Azure App Service][B15]

3. Dans la fenêtre **Créer App Service**, sélectionnez votre compte Azure. Sélectionnez **Modifier le type** > **Application web**. Conservez le **nom de l’application web** par défaut et sélectionnez l’**abonnement**, le **groupe de ressources** et le **plan App Service**.

4. Sélectionnez **Create** (Créer).

5. Notez la valeur de la propriété **URL du site** dans la section **Résumé**. Cette URL constitue le *point de terminaison de votre serveur principal* plus loin dans ce didacticiel.

6. Sélectionnez **Publier**.

Après en avoir terminé avec l’Assistant, il publie l’application web ASP.NET dans Azure, puis ouvre l’application dans le navigateur par défaut.  Votre application est visible dans Azure App Services.

L’URL utilise le nom de l’application web que vous avez spécifié précédemment, au format http://<app_name>.azurewebsites.net.

[B1]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push1.png
[B2]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push2.png
[B3]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push3.png
[B4]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push4.png
[B5]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push5.png
[B6]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push6.png
[B7]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push7.png
[B8]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push8.png
[B14]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push14.png
[B15]: ./media/notification-hubs-aspnet-backend-notifyusers/publish-to-app-service.png
[B16]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users16.PNG
[B18]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users18.PNG