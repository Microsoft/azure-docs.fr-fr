---
title: Envoyer des notifications vers des utilisateurs spécifiques à l’aide d’Azure Notification Hubs | Microsoft Docs
description: Découvrez comment envoyer des notifications vers des utilisateurs spécifiques utilisant des applications de la plate-forme Windows universelle (UWP).
documentationcenter: windows
author: sethmanheim
manager: femila
services: notification-hubs
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 08/17/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/22/2019
ms.openlocfilehash: 97a6a45ab01fc113b79a48ba7fcb246d528684be
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019478"
---
# <a name="tutorial-send-notifications-to-specific-users-by-using-azure-notification-hubs"></a>Tutoriel : Envoyer des notifications à des utilisateurs spécifiques avec Azure Notification Hubs

[!INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]

## <a name="overview"></a>Vue d’ensemble

Ce tutoriel explique comment utiliser Azure Notification Hubs pour envoyer des notifications Push à un utilisateur d’une application sur un appareil spécifique. Un serveur principal WebAPI ASP.NET est utilisé pour authentifier les clients. Lorsque le serveur principal authentifie un utilisateur de l’application client, il ajoute automatiquement une balise à l’enregistrement des notifications. Le serveur principal utilise cette balise pour envoyer des notifications à l’utilisateur spécifique.

> [!NOTE]
> Le code complet de ce didacticiel est disponible sur [GitHub](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/NotifyUsers).

Dans ce tutoriel, vous effectuez les étapes suivantes :

> [!div class="checklist"]
> * Création du projet WebAPI
> * Authentification des clients sur le serveur principal WebAPI
> * Inscription aux notifications à l’aide du serveur principal WebAPI
> * Envoyer des notifications depuis le serveur principal WebAPI
> * Publication du nouveau serveur principal WebAPI
> * Mise à jour du code pour le projet client
> * Test de l’application

## <a name="prerequisites"></a>Prérequis

Ce tutoriel fait intervenir le hub de notification et le projet Visual Studio que vous avez créés dans le [Tutoriel : Envoyer des notifications vers des applications de plateforme Windows universelle avec Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md). Par conséquent, terminez-le avant de démarrer ce didacticiel.

> [!NOTE]
> Si vous utilisez Mobile Apps dans Azure App Service comme service principal, consultez la [version Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-push) de ce didacticiel.

[!INCLUDE [notification-hubs-aspnet-backend-notifyusers](../../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## <a name="update-the-code-for-the-uwp-client"></a>Mettre à jour le code pour le client UWP

Dans cette section, vous mettez à jour le code du projet que vous avez terminé pour le [Tutoriel : Envoyer des notifications vers des applications de plateforme Windows universelle avec Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md). Le projet doit déjà être associé au Windows store. Il doit également être configuré pour utiliser votre hub de notification. Dans cette section, vous allez ajouter du code pour appeler le nouveau serveur principal WebAPI et l’utiliser pour l’enregistrement et l’envoi de notifications.

1. Dans Visual Studio, ouvrez la solution que vous avez créée pour le [Tutoriel : Envoyer des notifications vers des applications de plateforme Windows universelle avec Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).
2. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet Plateforme Windows universelle, puis cliquez sur **Gérer les packages NuGet**.
3. Sur le côté gauche, sélectionnez **Parcourir**.
4. Dans la zone **Rechercher**, entrez **Client Http**.
5. Dans la liste des résultats, cliquez sur **System.Net.Http**, puis cliquez sur **Installer**. Terminez l'installation.
6. De retour dans la zone **Recherche** NuGet, tapez **Json.net**. Installez le package **Newtonsoft.json** , puis fermez la fenêtre du Gestionnaire de package NuGet.
7. Dans l’Explorateur de solutions, dans le projet **WindowsApp**, double-cliquez sur **MainPage.xaml** pour l’ouvrir dans l’éditeur de Visual Studio.
8. Dans le fichier `MainPage.xaml`, remplacez la section `<Grid>` par le code suivant : Ce code ajoute une zone de texte Nom d’utilisateur et Mot de passe pour l’authentification de l’utilisateur. Il ajoute également des zones de texte pour le message de notification et la balise de nom d’utilisateur qui doivent recevoir la notification :

    ```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <TextBlock Grid.Row="0" Text="Notify Users" HorizontalAlignment="Center" FontSize="48"/>

        <StackPanel Grid.Row="1" VerticalAlignment="Center">
            <Grid>
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition></ColumnDefinition>
                    <ColumnDefinition></ColumnDefinition>
                    <ColumnDefinition></ColumnDefinition>
                </Grid.ColumnDefinitions>
                <TextBlock Grid.Row="0" Grid.ColumnSpan="3" Text="Username" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="UsernameTextBox" Grid.Row="1" Grid.ColumnSpan="3" Margin="20,0,20,0"/>
                <TextBlock Grid.Row="2" Grid.ColumnSpan="3" Text="Password" FontSize="24" Margin="20,0,20,0" />
                <PasswordBox Name="PasswordTextBox" Grid.Row="3" Grid.ColumnSpan="3" Margin="20,0,20,0"/>

                <Button Grid.Row="4" Grid.ColumnSpan="3" HorizontalAlignment="Center" VerticalAlignment="Center"
                            Content="1. Login and register" Click="LoginAndRegisterClick" Margin="0,0,0,20"/>

                <ToggleButton Name="toggleWNS" Grid.Row="5" Grid.Column="0" HorizontalAlignment="Right" Content="WNS" IsChecked="True" />
                <ToggleButton Name="toggleFCM" Grid.Row="5" Grid.Column="1" HorizontalAlignment="Center" Content="FCM" />
                <ToggleButton Name="toggleAPNS" Grid.Row="5" Grid.Column="2" HorizontalAlignment="Left" Content="APNS" />

                <TextBlock Grid.Row="6" Grid.ColumnSpan="3" Text="Username Tag To Send To" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="ToUserTagTextBox" Grid.Row="7" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                <TextBlock Grid.Row="8" Grid.ColumnSpan="3" Text="Enter Notification Message" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="NotificationMessageTextBox" Grid.Row="9" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                <Button Grid.Row="10" Grid.ColumnSpan="3" HorizontalAlignment="Center" Content="2. Send push" Click="PushClick" Name="SendPushButton" />
            </Grid>
        </StackPanel>
    </Grid>
    ```

9. Dans l’Explorateur de solutions, ouvrez le fichier `MainPage.xaml.cs` pour les projets **(Windows 8.1)** et **(Windows Phone 8.1)** . Ajoutez les instructions `using` suivantes au début des deux fichiers :

    ```csharp
    using System.Net.Http;
    using Windows.Storage;
    using System.Net.Http.Headers;
    using Windows.Networking.PushNotifications;
    using Windows.UI.Popups;
    using System.Threading.Tasks;
    ```

10. Dans `MainPage.xaml.cs` pour le projet **WindowsApp**, ajoutez le membre suivant à la classe `MainPage`. Remplacez bien `<Enter Your Backend Endpoint>` par le point de terminaison réel de votre serveur principal, obtenu précédemment. Par exemple : `http://mybackend.azurewebsites.net`.

    ```csharp
    private static string BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";
    ```

11. Ajoutez le code ci-dessous à la classe MainPage dans `MainPage.xaml.cs` pour les projets **(Windows 8.1)** et **(Windows Phone 8.1)** .

    La méthode `PushClick` correspond au gestionnaire de clic du bouton **Envoyer des notifications Push** . Elle appelle le serveur principal pour déclencher une notification vers tous les appareils avec une balise de nom d’utilisateur correspondant au paramètre `to_tag` . Le message de notification est envoyé en tant que contenu JSON dans le corps de la demande.

    La méthode `LoginAndRegisterClick` correspond au gestionnaire de clic du bouton **Se connecter et s’inscrire**. Elle stocke le jeton d’authentification de base (représente n’importe quel jeton utilisé par votre système d’authentification) dans un stockage local, puis utilise `RegisterClient` pour l’inscription aux notifications à l’aide du serveur principal.

    ```csharp
    private async void PushClick(object sender, RoutedEventArgs e)
    {
        if (toggleWNS.IsChecked.Value)
        {
            await sendPush("wns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
        }
        if (toggleFCM.IsChecked.Value)
        {
            await sendPush("fcm", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
        }
        if (toggleAPNS.IsChecked.Value)
        {
            await sendPush("apns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);

        }
    }

    private async Task sendPush(string pns, string userTag, string message)
    {
        var POST_URL = BACKEND_ENDPOINT + "/api/notifications?pns=" +
            pns + "&to_tag=" + userTag;

        using (var httpClient = new HttpClient())
        {
            var settings = ApplicationData.Current.LocalSettings.Values;
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

            try
            {
                await httpClient.PostAsync(POST_URL, new StringContent("\"" + message + "\"",
                    System.Text.Encoding.UTF8, "application/json"));
            }
            catch (Exception ex)
            {
                MessageDialog alert = new MessageDialog(ex.Message, "Failed to send " + pns + " message");
                alert.ShowAsync();
            }
        }
    }

    private async void LoginAndRegisterClick(object sender, RoutedEventArgs e)
    {
        SetAuthenticationTokenInLocalStorage();

        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        // The "username:<user name>" tag gets automatically added by the message handler in the backend.
        // The tag passed here can be whatever other tags you may want to use.
        try
        {
            // The device handle used is different depending on the device and PNS.
            // Windows devices use the channel uri as the PNS handle.
            await new RegisterClient(BACKEND_ENDPOINT).RegisterAsync(channel.Uri, new string[] { "myTag" });

            var dialog = new MessageDialog("Registered as: " + UsernameTextBox.Text);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
            SendPushButton.IsEnabled = true;
        }
        catch (Exception ex)
        {
            MessageDialog alert = new MessageDialog(ex.Message, "Failed to register with RegisterClient");
            alert.ShowAsync();
        }
    }

    private void SetAuthenticationTokenInLocalStorage()
    {
        string username = UsernameTextBox.Text;
        string password = PasswordTextBox.Password;

        var token = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(username + ":" + password));
        ApplicationData.Current.LocalSettings.Values["AuthenticationToken"] = token;
    }
    ```

12. Ouvrez `App.xaml.cs` et recherchez l’appel vers `InitNotificationsAsync()` dans le gestionnaire d’événements `OnLaunched()`. Commentez ou supprimez l’appel vers `InitNotificationsAsync()`. Le gestionnaire de bouton initialise les inscriptions aux notifications :

    ```csharp
    protected override void OnLaunched(LaunchActivatedEventArgs e)
    {
        //InitNotificationsAsync();
    ```

13. Cliquez avec le bouton droit sur le projet **WindowsApp**, cliquez sur **Ajouter**, puis cliquez sur **Classe**. Nommez la classe `RegisterClient.cs`, puis cliquez sur **OK** pour générer la classe.

    Cette classe encapsule les appels REST nécessaires pour contacter le service principal de l'application et inscrire cette dernière pour les notifications Push. Il enregistre également en local les informations *registrationIds* créées par le hub de notification, comme expliqué dans la rubrique [Inscription auprès du serveur principal de votre application](/previous-versions/azure/azure-services/dn743807(v=azure.100)). Il utilise un jeton d'autorisation qui se trouve dans le stockage local lorsque vous cliquez sur le bouton **Se connecter et s'inscrire**.
14. Ajoutez les instructions `using` suivantes au début du fichier RegisterClient.cs :

    ```csharp
    using Windows.Storage;
    using System.Net;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using Newtonsoft.Json;
    using System.Threading.Tasks;
    using System.Linq;
    ```

15. Ajoutez le code suivant dans la définition de classe `RegisterClient` :

    ```csharp
    private string POST_URL;

    private class DeviceRegistration
    {
        public string Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }

    public RegisterClient(string backendEndpoint)
    {
        POST_URL = backendEndpoint + "/api/register";
    }

    public async Task RegisterAsync(string handle, IEnumerable<string> tags)
    {
        var regId = await RetrieveRegistrationIdOrRequestNewOneAsync();

        var deviceRegistration = new DeviceRegistration
        {
            Platform = "wns",
            Handle = handle,
            Tags = tags.ToArray<string>()
        };

        var statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);

        if (statusCode == HttpStatusCode.Gone)
        {
            // regId is expired, deleting from local storage & recreating
            var settings = ApplicationData.Current.LocalSettings.Values;
            settings.Remove("__NHRegistrationId");
            regId = await RetrieveRegistrationIdOrRequestNewOneAsync();
            statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);
        }

        if (statusCode != HttpStatusCode.Accepted && statusCode != HttpStatusCode.OK)
        {
            // log or throw
            throw new System.Net.WebException(statusCode.ToString());
        }
    }

    private async Task<HttpStatusCode> UpdateRegistrationAsync(string regId, DeviceRegistration deviceRegistration)
    {
        using (var httpClient = new HttpClient())
        {
            var settings = ApplicationData.Current.LocalSettings.Values;
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string) settings["AuthenticationToken"]);

            var putUri = POST_URL + "/" + regId;

            string json = JsonConvert.SerializeObject(deviceRegistration);
                            var response = await httpClient.PutAsync(putUri, new StringContent(json, Encoding.UTF8, "application/json"));
            return response.StatusCode;
        }
    }

    private async Task<string> RetrieveRegistrationIdOrRequestNewOneAsync()
    {
        var settings = ApplicationData.Current.LocalSettings.Values;
        if (!settings.ContainsKey("__NHRegistrationId"))
        {
            using (var httpClient = new HttpClient())
            {
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

                var response = await httpClient.PostAsync(POST_URL, new StringContent(""));
                if (response.IsSuccessStatusCode)
                {
                    string regId = await response.Content.ReadAsStringAsync();
                    regId = regId.Substring(1, regId.Length - 2);
                    settings.Add("__NHRegistrationId", regId);
                }
                else
                {
                    throw new System.Net.WebException(response.StatusCode.ToString());
                }
            }
        }
        return (string)settings["__NHRegistrationId"];

    }
    ```

16. Enregistrez toutes vos modifications.

## <a name="test-the-application"></a>Tester l'application

1. Lancez l’application sur chaque Windows.
2. Entrez un **Nom d’utilisateur** et un **Mot de passe**, comme indiqué dans l’écran ci-dessous. Ces informations d’identification doivent différer du nom d’utilisateur et du mot de passe entrés sur Windows Phone.
3. Cliquez sur **Se connecter et s’inscrire** et vérifiez le contenu de la boîte de dialogue indiquant que vous êtes connecté. Ce code active également le bouton **Envoyer des notifications Push**.

    ![Capture d’écran de l’application Notification Hubs montrant le nom d’utilisateur et le mot de passe renseignés.][14]
4. Puis, dans le champ **Balise de nom d’utilisateur**, entrez le nom d’utilisateur enregistré. Entrez un message de notification, puis cliquez sur **Envoyer des notifications Push**.
5. Seuls les appareils enregistrés avec la balise de nom d’utilisateur correspondant reçoivent le message de notification.

    ![Capture d’écran de l’application Notification Hubs montrant le message ayant fait l’objet d’un envoi (push).][15]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à envoyer des notifications Push à des utilisateurs spécifiques ayant des balises associées à leurs enregistrements. Pour savoir comment envoyer des notifications basées sur l’emplacement, passez au tutoriel suivant :

> [!div class="nextstepaction"]
>[Notifications Push en fonction de la localisation](notification-hubs-push-bing-spatial-data-geofencing-notification.md)

[9]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push9.png
[10]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push10.png
[11]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push11.png
[12]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push12.png
[13]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-ui.png
[14]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-windows-instance-username.png
[15]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-notification-received.png
[16]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-send-message.png

<!-- URLs. -->
[Get started with Notification Hubs]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Secure Push]: notification-hubs-aspnet-backend-windows-dotnet-wns-secure-push-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
