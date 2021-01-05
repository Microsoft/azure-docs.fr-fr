---
title: Envoyer des notifications à des appareils spécifiques (plateforme Windows universelle) | Microsoft Docs
description: Utilisez Azure Notification Hubs avec des balises dans l’inscription pour envoyer les dernières nouvelles à une application de plateforme Windows universelle.
services: notification-hubs
documentationcenter: windows
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 994d2eed-f62e-433c-bf65-4afebf1c0561
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 09/30/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 03/22/2019
ms.openlocfilehash: 2c77eba69fd914e8ecc7d08a1b16f61ceefe101b
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92320568"
---
# <a name="tutorial-send-notifications-to-specific-devices-running-universal-windows-platform-applications"></a>Tutoriel : Envoyer des notifications vers des appareils spécifiques exécutant des applications de plateforme Windows universelle

[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

## <a name="overview"></a>Vue d’ensemble

Ce tutoriel vous montre comment utiliser Azure Notification Hubs pour diffuser des notifications relatives aux dernières nouvelles. Ce tutoriel traite des applications Windows Store ou Windows Phone 8.1 (non-Silverlight). Si vous ciblez Windows Phone 8.1 Silverlight, consultez [Notifications Push vers des appareils Windows Phone spécifiques à l’aide d’Azure Notification Hubs](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md).

Dans ce tutoriel, vous découvrirez comment utiliser Azure Notification Hubs pour envoyer des notifications Push à des appareils Windows spécifiques exécutant une application de plateforme Windows universelle (UWP). Après avoir suivi ce tutoriel, vous pourrez vous inscrire aux catégories relatives aux dernières nouvelles, qui vous intéressent. Vous recevrez des notifications Push pour ces catégories uniquement.

Pour rendre possibles les scénarios de diffusion, incluez une ou plusieurs *balises* quand vous créez une inscription dans le hub de notification. Quand des notifications sont envoyées à une balise, tous les appareils inscrits à cette balise reçoivent la notification. Pour plus d’informations sur les balises, consultez [Routage et expressions de balise](notification-hubs-tags-segment-push-message.md).

> [!NOTE]
> Les projets Windows Store et Windows Phone versions 8.1 et antérieures ne sont pas pris en charge dans Visual Studio 2019. Pour en savoir plus, consultez [Plateforme cible et compatibilité dans Visual Studio 2019](/visualstudio/releases/2019/compatibility).

Dans ce tutoriel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
> * Ajout d’une sélection de catégories à l’application mobile
> * Inscription à des notifications
> * Envoyer des notifications avec balises
> * Exécution de l’application et génération de notifications

## <a name="prerequisites"></a>Prérequis

Suivre le [Tutoriel : Envoyer des notifications à des applications de plateforme Windows universelle avec Azure Notification Hubs][get-started] avant de commencer ce tutoriel.  

## <a name="add-category-selection-to-the-app"></a>Ajout d’une sélection de catégories à l’application

La première étape consiste à ajouter des éléments d’interface utilisateur à votre page principale existante pour permettre aux utilisateurs de sélectionner les catégories auxquelles s’inscrire. Les catégories sélectionnées sont stockées sur l’appareil. Quand l’application démarre, elle crée une inscription d’appareil dans votre hub de notification, avec les catégories sélectionnées se présentant sous forme de balises.

1. Ouvrez le fichier projet *MainPage.xaml*, puis copiez le code suivant dans l’élément `Grid` :

    ```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="1" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="2" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="3" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="1" Grid.Column="1" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="2" Grid.Column="1" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="3" Grid.Column="1" HorizontalAlignment="Center"/>
        <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click"/>
    </Grid>
    ```

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Ajouter** > **Classe**. Sous **Ajouter un nouvel élément**, nommez la classe *Notifications* , puis sélectionnez **Ajouter**. Si nécessaire, ajoutez le modificateur `public` à la définition de la classe.

1. Ajoutez les instructions `using` suivantes au nouveau fichier :

    ```csharp
    using Windows.Networking.PushNotifications;
    using Microsoft.WindowsAzure.Messaging;
    using Windows.Storage;
    using System.Threading.Tasks;
    ```

1. Copiez le code suivant dans la nouvelle classe `Notifications` :

    ```csharp
    private NotificationHub hub;

    public Notifications(string hubName, string listenConnectionString)
    {
        hub = new NotificationHub(hubName, listenConnectionString);
    }

    public async Task<Registration> StoreCategoriesAndSubscribe(IEnumerable<string> categories)
    {
        ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
        return await SubscribeToCategories(categories);
    }

    public IEnumerable<string> RetrieveCategories()
    {
        var categories = (string) ApplicationData.Current.LocalSettings.Values["categories"];
        return categories != null ? categories.Split(','): new string[0];
    }

    public async Task<Registration> SubscribeToCategories(IEnumerable<string> categories = null)
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        if (categories == null)
        {
            categories = RetrieveCategories();
        }

        // Using a template registration to support notifications across platforms.
        // Any template notifications that contain messageParam and a corresponding tag expression
        // will be delivered for this registration.

        const string templateBodyWNS = "<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(messageParam)</text></binding></visual></toast>";

        return await hub.RegisterTemplateAsync(channel.Uri, templateBodyWNS, "simpleWNSTemplateExample",
                categories);
    }
    ```

    Cette classe utilise le stockage local pour stocker les catégories de nouvelles que cet appareil doit recevoir. Au lieu d’appeler la méthode `RegisterNativeAsync`, appelez `RegisterTemplateAsync` pour vous inscrire aux catégories en utilisant une inscription de modèle.

    Si vous désirez inscrire plusieurs modèles, indiquez un nom de modèle, par exemple *simpleWNSTemplateExample*. Vous devez nommer les modèles pour pouvoir les mettre à jour ou les supprimer. Vous pouvez inscrire plusieurs modèles afin de disposer d’un modèle pour les notifications toast et d’un autre pour les vignettes.

    >[!NOTE]
    > Avec Notification Hubs, un appareil peut inscrire plusieurs modèles en utilisant la même balise. Dans ce cas, un message entrant qui cible cette balise déclenche l’envoi de plusieurs notifications à destination de l’appareil (une pour chaque modèle). Ce processus vous permet d’afficher un même message dans plusieurs notifications visuelles, par exemple, sous la forme d’un badge et d’une notification toast dans une application du Windows Store.

    Pour plus d’informations, consultez [Modèles](notification-hubs-templates-cross-platform-push-messages.md).

1. Dans le fichier projet *App.xaml.cs*, ajoutez la propriété suivante à la classe `App` :

    ```csharp
    public Notifications notifications = new Notifications("<hub name>", "<connection string with listen access>");
    ```

    Cette propriété sert à créer une instance `Notifications` et à y accéder.

    Dans le code, remplacez les espaces réservés `<hub name>` et `<connection string with listen access>` par le nom de votre hub de notification et la chaîne de connexion de **DefaultListenSharedAccessSignature**, que vous avez obtenue précédemment.

   > [!NOTE]
   > Les informations d’identification distribuées avec une application cliente n’étant généralement pas sécurisées, ne distribuez que la clé d’accès d’*écoute* avec votre application cliente. Grâce à l’accès d’écoute, votre application peut s’inscrire à des notifications, mais les inscriptions existantes ne peuvent pas être modifiées et les notifications ne peuvent pas être envoyées. La clé d’accès complet est utilisée dans un service backend sécurisé pour l’envoi de notifications et la modification d’inscriptions existantes.

1. Dans le fichier *MainPage.xaml.cs*, ajoutez la ligne suivante :

    ```csharp
    using Windows.UI.Popups;
    ```

1. Dans le fichier *MainPage.xaml.cs*, ajoutez la méthode suivante :

    ```csharp
    private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
    {
        var categories = new HashSet<string>();
        if (WorldToggle.IsOn) categories.Add("World");
        if (PoliticsToggle.IsOn) categories.Add("Politics");
        if (BusinessToggle.IsOn) categories.Add("Business");
        if (TechnologyToggle.IsOn) categories.Add("Technology");
        if (ScienceToggle.IsOn) categories.Add("Science");
        if (SportsToggle.IsOn) categories.Add("Sports");

        var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(categories);

        var dialog = new MessageDialog("Subscribed to: " + string.Join(",", categories) + " on registration Id: " + result.RegistrationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    ```

    Cette méthode crée une liste de catégories et utilise la classe `Notifications` pour stocker la liste dans le stockage local. Elle inscrit aussi les balises correspondantes auprès de votre hub de notification. Quand les catégories changent, l’inscription est recréée avec les nouvelles catégories.

Votre application peut alors stocker un ensemble de catégories dans le stockage local de l’appareil. L’application s’inscrit auprès du hub de notification chaque fois que des utilisateurs modifient la sélection de catégories.

## <a name="register-for-notifications"></a>Inscription à des notifications

Dans cette section, vous allez vous inscrire auprès du hub de notification au démarrage en utilisant les catégories que vous avez stockées dans le stockage local.

> [!NOTE]
> L’URI de canal affecté par le service de notification Windows (WNS) pouvant changer à tout moment, vous devez vous inscrire fréquemment aux notifications pour éviter des échecs de notifications. Cet exemple s’inscrit aux notifications chaque fois que l’application démarre. Pour les applications que vous exécutez fréquemment (plus d’une fois par jour), vous pouvez probablement ignorer l’inscription de façon à préserver la bande passante si moins d’un jour s’est écoulé depuis l’inscription précédente.

1. Pour utiliser la classe `notifications` pour vous abonner en fonction des catégories, ouvrez le fichier *App.xaml.cs* et mettez à jour la méthode `InitNotificationsAsync`.

    ```csharp
    // *** Remove or comment out these lines **_
    //var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();
    //var hub = new NotificationHub("your hub name", "your listen connection string");
    //var result = await hub.RegisterNativeAsync(channel.Uri);

    var result = await notifications.SubscribeToCategories();
    ```

    Ce processus garantit que l’application récupère les catégories dans le stockage local quand elle démarre. Elle demande ensuite une inscription à ces catégories. Vous avez créé la méthode `InitNotificationsAsync` dans le cadre du tutoriel [Envoyer des notifications vers des applications de plateforme Windows universelle avec Azure Notification Hubs][get-started].
2. Dans le fichier projet *MainPage.xaml.cs*, ajoutez le code suivant à la méthode `OnNavigatedTo` :

    ```csharp
    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        var categories = ((App)Application.Current).notifications.RetrieveCategories();

        if (categories.Contains("World")) WorldToggle.IsOn = true;
        if (categories.Contains("Politics")) PoliticsToggle.IsOn = true;
        if (categories.Contains("Business")) BusinessToggle.IsOn = true;
        if (categories.Contains("Technology")) TechnologyToggle.IsOn = true;
        if (categories.Contains("Science")) ScienceToggle.IsOn = true;
        if (categories.Contains("Sports")) SportsToggle.IsOn = true;
    }
    ```

    Ce code met à jour la page principale en fonction de l’état des catégories enregistrées précédemment.

L’application est maintenant terminée. Elle peut stocker un ensemble de catégories dans le stockage local de l’appareil. Quand les utilisateurs modifient la sélection de catégories, les catégories enregistrées sont utilisées pour l’inscription auprès du hub de notification. Dans la section suivante, vous allez définir un backend capable d’envoyer des notifications de catégorie à cette application.

## <a name="run-the-uwp-app"></a>Exécuter l’application UWP

1. Dans Visual Studio, appuyez sur F5 pour compiler et démarrer l’application. L’interface utilisateur de l’application fournit un ensemble de bascules qui vous permet de choisir les catégories auxquelles vous abonner.

   ![Application de diffusion des dernières nouvelles](./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-breaking-news.png)

1. Activez une ou plusieurs bascules de catégories, puis sélectionnez **S’abonner**.

   L'application convertit les catégories sélectionnées en balises et demande une nouvelle inscription de l'appareil pour les balises sélectionnées depuis le Notification Hub. L’application affiche les catégories inscrites dans une boîte de dialogue.

    ![Bascules de catégories et bouton S’abonner](./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-toast.png)

## <a name="create-a-console-app-to-send-tagged-notifications"></a>Créer une application console pour envoyer des notifications étiquetées

[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="run-the-console-app-to-send-tagged-notifications"></a>Exécuter l’application console pour envoyer des notifications étiquetées

Exécutez l’application créée dans la section précédente. Les notifications pour les catégories sélectionnées apparaissent comme notifications toast.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à diffuser les dernières nouvelles par catégorie. L’application back-end envoie des notifications étiquetées aux appareils inscrits pour recevoir des notifications de cette balise. Pour savoir comment envoyer des notifications à des utilisateurs spécifiques quel que soit l’appareil qu’ils utilisent, passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Envoyer des notifications localisées vers des applications Windows avec Azure Notification Hubs](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)

<!-- Anchors. -->
[Add category selection to the app]: #adding-categories
[Register for notifications]: #register
[Send notifications from your back-end]: #send
[Run the app and generate notifications]: #test-app
[Next Steps]: #next-steps

<!-- URLs.-->
[get-started]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started/
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for Windows Store]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[wns object]: /previous-versions/azure/reference/jj860484(v=azure.100)