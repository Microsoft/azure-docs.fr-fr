---
title: Envoyer des notifications localisées vers des applications Windows à l’aide d’Azure Notification Hubs | Microsoft Docs
description: Découvrez comment utiliser Azure Notification Hubs pour envoyer des notifications de dernières nouvelles localisées.
services: notification-hubs
documentationcenter: windows
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: c454f5a3-a06b-45ac-91c7-f91210889b25
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 03/22/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 03/22/2019
ms.openlocfilehash: d91320d8f78942ca916084cbf29a88818d95f531
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92314623"
---
# <a name="tutorial-send-localized-push-notifications-to-windows-apps-using-azure-notification-hubs"></a>Tutoriel : Envoyer des notifications Push localisées vers des applications Windows à l’aide d’Azure Notification Hubs

> [!div class="op_single_selector"]
> * [Windows Store C#](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
> * [iOS](notification-hubs-ios-xplat-localized-apns-push-notification.md)

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel vous montre comment envoyer des notifications localisées vers des appareils mobiles inscrits auprès du service Notification Hubs. Dans le didacticiel, vous mettez à jour les applications créées dans le [Tutoriel : Envoyer des notifications à des appareils spécifiques (plateforme Windows universelle)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md) pour prendre en charge les scénarios suivants :

- L'application Windows Store permet aux appareils clients de spécifier une langue et de s'abonner à différentes catégories de dernières nouvelles.
- L’application back-end diffuse les notifications à l’aide des fonctionnalités de **balise** et de **modèle** d’Azure Notification Hubs.

Lorsque vous effectuez le didacticiel, l’application mobile vous permet d’inscrire les catégories qui vous intéressent et également de spécifier une langue qui recevra les notifications. L’application de serveur principal envoie des notifications localisées par la langue et l’appareil.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Mettre à jour l’application Windows pour prendre en charge les informations des paramètres régionaux
> * Mettre à jour l’application de serveur principal pour envoyer des notifications localisées
> * Test de l'application

## <a name="prerequisites"></a>Prérequis

Suivre le [Tutoriel : Envoyer des notifications à des appareils spécifiques (plateforme Windows universelle)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md).

Dans le [Tutoriel : Envoyer des notifications à des appareils spécifiques (plateforme Windows universelle)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md), vous avez créé une application ayant utilisée des **balises** pour s’abonner aux notifications de différentes **catégories** d’actualité. Dans ce didacticiel, vous utilisez la fonctionnalité de **modèle** de Notification Hubs pour facilement envoyer des notifications de dernières nouvelles **localisées**.

À un haut niveau, les modèles permettent de spécifier le format dans lequel un appareil particulier reçoit une notification. Le modèle spécifie le format de charge utile exact en se référant aux propriétés qui font partie du message envoyé par le serveur principal de votre application. Dans ce didacticiel, l’application de serveur principal envoie un message indépendant de paramètres régionaux contenant toutes les langues prises en charge :

```json
{
    "News_English": "...",
    "News_French": "...",
    "News_Mandarin": "..."
}
```

Les appareils s'inscrivent avec un modèle qui se réfère à la bonne propriété. Par exemple, une application Windows Store qui veut recevoir un message toast en anglais doit s’inscrire pour le modèle suivant, avec les balises correspondantes :

```xml
<toast>
    <visual>
    <binding template=\"ToastText01\">
        <text id=\"1\">$(News_English)</text>
    </binding>
    </visual>
</toast>
```

Pour en savoir plus sur les modèles, consultez [Modèles Push](notification-hubs-templates-cross-platform-push-messages.md).

## <a name="update-windows-app-to-support-locale-information"></a>Mettre à jour l’application Windows pour prendre en charge les informations des paramètres régionaux

1. Ouvrez la solution Visual Studio que vous avez créée pour le [Tutoriel : Envoyer des notifications à des appareils spécifiques (plateforme Windows universelle)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md).
2. Mettez à jour le fichier `MainPage.xaml` de la solution pour y ajouter une zone de liste modifiable de paramètres régionaux :

    ```xml
    <Grid Margin="120, 58, 120, 80"  
            Background="{StaticResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition />
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top"/>
        <ComboBox Name="Locale" HorizontalAlignment="Left" VerticalAlignment="Center" Width="200" Grid.Row="1" Grid.Column="0">
            <x:String>English</x:String>
            <x:String>French</x:String>
            <x:String>Mandarin</x:String>
        </ComboBox>
        <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="2" Grid.Column="0"/>
        <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="3" Grid.Column="0"/>
        <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="4" Grid.Column="0"/>
        <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="2" Grid.Column="1"/>
        <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="3" Grid.Column="1"/>
        <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="4" Grid.Column="1"/>
        <Button Content="Subscribe" HorizontalAlignment="Center" Grid.Row="5" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click" />
    </Grid>
    ```
3. Dans la classe `Notifications`, ajoutez un paramètre régional aux méthodes `StoreCategoriesAndSubscribe` et `SubscribeToCategories`.

    ```csharp
    public async Task<Registration> StoreCategoriesAndSubscribe(string locale, IEnumerable<string> categories)
    {
        ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
        ApplicationData.Current.LocalSettings.Values["locale"] = locale;
        return await SubscribeToCategories(locale, categories);
    }

    public async Task<Registration> SubscribeToCategories(string locale, IEnumerable<string> categories = null)
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        if (categories == null)
        {
            categories = RetrieveCategories();
        }

        // Using a template registration. This makes supporting notifications across other platforms much easier.
        // Using the localized tags based on locale selected.
        string templateBodyWNS = String.Format("<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(News_{0})</text></binding></visual></toast>", locale);

        return await hub.RegisterTemplateAsync(channel.Uri, templateBodyWNS, "localizedWNSTemplateExample", categories);
    }
    ```

    Au lieu d’appeler la méthode `RegisterNativeAsync`, appelez `RegisterTemplateAsync`. Vous inscrivez un format de notification spécifique dans lequel le modèle dépend des paramètres régionaux. Vous indiquez aussi un nom pour le modèle (« localizedWNSTemplateExample »), car vous pouvez être amené à inscrire plus d’un modèle (par exemple, un pour les notifications toast et un pour les vignettes). Vous devez également leur attribuer un nom pour les mettre à jour ou les supprimer.

    Si un appareil inscrit plusieurs modèles avec la même balise, un message entrant ciblant cette balise entraîne l'envoi de plusieurs notifications à l'appareil (un pour chaque modèle). Ce comportement s'avère utile lorsque le même message logique doit générer plusieurs notifications visuelles, par exemple affichant un badge et un toast dans une application Windows Store.
4. Ajoutez la méthode suivante pour extraire les paramètres régionaux stockés :

    ```csharp
    public string RetrieveLocale()
    {
        var locale = (string) ApplicationData.Current.LocalSettings.Values["locale"];
        return locale != null ? locale : "English";
    }
    ```

5. Dans le fichier `MainPage.xaml.cs`, mettez à jour le gestionnaire de clic de bouton pour récupérer la valeur actuelle de la zone de liste modifiable Paramètres régionaux et la fournir à l’appel de la classe `Notifications` :

    ```csharp
    private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
    {
        var locale = (string)Locale.SelectedItem;

        var categories = new HashSet<string>();
        if (WorldToggle.IsOn) categories.Add("World");
        if (PoliticsToggle.IsOn) categories.Add("Politics");
        if (BusinessToggle.IsOn) categories.Add("Business");
        if (TechnologyToggle.IsOn) categories.Add("Technology");
        if (ScienceToggle.IsOn) categories.Add("Science");
        if (SportsToggle.IsOn) categories.Add("Sports");

        var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(locale,
                categories);

        var dialog = new MessageDialog("Locale: " + locale + " Subscribed to: " + 
            string.Join(",", categories) + " on registration Id: " + result.RegistrationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    ```
6. Pour finir, dans le fichier `App.xaml.cs`, mettez à jour la méthode `InitNotificationsAsync` afin de récupérer les paramètres régionaux et les utiliser lors de l’abonnement :

    ```csharp
    private async void InitNotificationsAsync()
    {
        var result = await notifications.SubscribeToCategories(notifications.RetrieveLocale());

        // Displays the registration ID so you know it was successful
        if (result.RegistrationId != null)
        {
            var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
    }
    ```



## <a name="run-the-uwp-application"></a>Exécuter l’application UWP

1. Exécutez l’application UWP. Patientez jusqu'à ce voir le message **Inscription réussie**.

    ![Application mobile et inscription](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/registration-successful.png)
2. Sélectionnez les **catégories** et les **paramètres régionaux**, puis cliquez sur **S’abonner**. L'application convertit les catégories sélectionnées en balises et demande une nouvelle inscription de l'appareil pour les balises sélectionnées depuis le Notification Hub.

    ![Application mobile](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/mobile-app.png)
3. Vous voyez un message de **confirmation** concernant les **abonnements**.

    ![Message d'abonnement](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/subscription-message.png)

## <a name="update-console-app-to-send-localized-notifications"></a>Mettre à jour l’application console pour envoyer des notifications localisées

Lorsque vous envoyez des notifications de modèle, vous devez uniquement fournir un ensemble de propriétés. Dans ce didacticiel, l’application de serveur principal envoie l’ensemble des propriétés contenant la version localisée des actualités, par exemple :

```json
{
    "News_English": "World News in English!",
    "News_French": "World News in French!",
    "News_Mandarin": "World News in Mandarin!"
}
```

Dans cette section, vous mettez à jour le projet d’application console dans la solution. Modifiez la méthode `SendTemplateNotificationAsync` dans l’application console que vous avez créée précédemment avec le code suivant :

> [!IMPORTANT]
> Spécifiez le nom et la chaîne de connexion avec un accès complet à votre concentrateur de notification dans le code.

```csharp
private static async void SendTemplateNotificationAsync()
{
    // Define the notification hub.
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(
        "<connection string with full access>", "<hub name>");

    // Sending the notification as a template notification. All template registrations that contain
    // "messageParam" or "News_<local selected>" and the proper tags will receive the notifications.
    // This includes APNS, FCM, WNS, and MPNS template registrations.
    Dictionary<string, string> templateParams = new Dictionary<string, string>();

    // Create an array of breaking news categories.
    var categories = new string[] { "World", "Politics", "Business", "Technology", "Science", "Sports"};
    var locales = new string[] { "English", "French", "Mandarin" };

    foreach (var category in categories)
    {
        templateParams["messageParam"] = "Breaking " + category + " News!";

        // Sending localized News for each tag too...
        foreach( var locale in locales)
        {
            string key = "News_" + locale;

            // Your real localized news content would go here.
            templateParams[key] = "Breaking " + category + " News in " + locale + "!";
        }

        await hub.SendTemplateNotificationAsync(templateParams, category);
    }
}
```

Ce simple appel remet l’information localisée à **tous** les appareils, indépendamment de leur plateforme, étant donné que votre hub de notification crée et remet la charge utile native qui convient à l’ensemble des appareils abonnés à une balise spécifique.

## <a name="run-console-app-to-send-localized-notification"></a>Exécuter l’application console pour envoyer une notification localisée
Exécutez l’**application console** pour envoyer des notifications pour chaque catégorie et dans chaque langue prise en charge. Vérifiez que vous avez uniquement reçu une notification pour les catégories auxquelles vous êtes abonnées et le message est pour les paramètres régionaux sélectionnés.

![Messages de notification](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/notifications.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à envoyer des notifications localisées à des appareils spécifiques ayant des balises associées à leurs enregistrements. Pour savoir comment envoyer des notifications à des utilisateurs spécifiques qui peuvent utiliser plus d’un appareil, passez au didacticiel suivant :

> [!div class="nextstepaction"]
>[Notifications Push vers des utilisateurs spécifiques](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md)

<!-- Anchors. -->
[Template concepts]: #concepts
[The app user interface]: #ui
[Building the Windows Store client app]: #building-client
[Send notifications from your back-end]: #send
[Next Steps]:#next-steps

<!-- Images. -->

<!-- URLs. -->
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs: ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Notify users with Notification Hubs: Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs/notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-dotnet
[Get started with authentication]: /develop/mobile/tutorials/get-started-with-users-dotnet
[Get started with push notifications]: /develop/mobile/tutorials/get-started-with-push-dotnet
[Push notifications to app users]: /develop/mobile/tutorials/push-notifications-to-app-users-dotnet
[Authorize users with scripts]: /develop/mobile/tutorials/authorize-users-in-scripts-dotnet
[JavaScript and HTML]: /develop/mobile/tutorials/get-started-with-push-js
[wns object]: /previous-versions/azure/reference/jj860484(v=azure.100)
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for iOS]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for Windows Store]: /previous-versions/azure/azure-services/jj927170(v=azure.100)