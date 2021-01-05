---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: de961aa36d690cf03e42707758bc70e5495f692e
ms.sourcegitcommit: 74ba70139781ed854d3ad898a9c65ef70c0ba99b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/26/2020
ms.locfileid: "85448708"
---
### <a name="validate-package-name-and-permissions"></a>Valider le nom et les autorisations de package

1. Dans **PushDemo.Android**, ouvrez **Options du projet** puis **Application Android** dans la section **Générer**.

1. Vérifiez que le **Nom du package** correspond à la valeur que vous avez utilisée dans le projet **PushDemo** au sein de [Firebase Console](https://console.firebase.google.com). Le **nom du package** était au format ``com.<organization>.pushdemo``.

1. Définissez la **Version Android minimale** sur **Android 8.0 (niveau d’API 26)** et la **Version cible d’Android** sur le dernier **niveau d’API**.

    > [!NOTE]
    > Seuls les appareils qui exécutent les **niveaux d’API 26 et supérieurs** sont pris en charge dans le cadre de ce tutoriel, mais vous pouvez l’étendre pour prendre en charge les appareils exécutant des versions antérieures.

1. Assurez-vous que les autorisations **INTERNET** et **READ_PHONE_STATE** sont activées sous **Autorisations requises**.

1. Cliquez sur **OK**

### <a name="add-the-xamarin-google-play-services-base-and-xamarinfirebasemessaging-packages"></a>Ajouter les packages Xamarin Google Play Services Base et Xamarin.Firebase.Messaging

1. Dans **PushDemo.Android**, utilisez **Ctrl** + **clic** sur le dossier **Packages**, puis choisissez **Gérer les packages NuGet...** .

1. Recherchez **Xamarin.GooglePlayServices.Base** (pas **Basement**) et assurez-vous qu’il est activé.

1. Recherchez **Xamarin.Firebase.Messaging** et assurez-vous qu’il est activé.

1. Cliquez sur **Ajouter des packages**, puis cliquez sur **Accepter** quand vous êtes invité à accepter les **termes du contrat de licence**.

### <a name="add-the-google-services-json-file"></a>Ajout du fichier JSON Google Services

1. Utilisez **Ctrl** + **clic** sur le projet `PushDemo.Android`, puis choisissez **Fichier existant...** dans le menu **Ajouter**.

1. Choisissez le fichier *google-services.json* que vous avez téléchargé lors de la configuration du projet **PushDemo** dans [Firebase Console](https://console.firebase.google.com), puis cliquez sur **Ouvrir**.

1. Quand vous y êtes invité, choisissez de **copier le fichier dans le répertoire**.

1. Utilisez **Ctrl** + **Clic** sur le fichier *google-services.json* à partir du projet `PushDemo.Android`, puis assurez-vous que **GoogleServicesJson** est défini comme **Action de génération**.

### <a name="handle-push-notifications-for-android"></a>Gérer les notifications Push pour Android

1. Utilisez **Ctrl** + **clic** sur le projet `PushDemo.Android`, choisissez **Nouveau dossier** dans le menu **Ajouter**, puis cliquez sur **Ajouter** en utilisant *Services* comme **Nom de dossier**.

1. Utilisez **Ctrl** + **clic** sur le dossier **Services**, puis choisissez **Nouveau fichier** dans le menu **Ajouter**.

1. Sélectionnez **Général** > **Classe vide**, entrez *DeviceInstallationService.cs* pour le **Nom**, puis cliquez sur **Nouveau** en ajoutant l’implémentation suivante.

    ```csharp
    using System;
    using Android.App;
    using Android.Gms.Common;
    using PushDemo.Models;
    using PushDemo.Services;
    using static Android.Provider.Settings;

    namespace PushDemo.Droid.Services
    {
        public class DeviceInstallationService : IDeviceInstallationService
        {
            public string Token { get; set; }

            public bool NotificationsSupported
                => GoogleApiAvailability.Instance
                    .IsGooglePlayServicesAvailable(Application.Context) == ConnectionResult.Success;

            public string GetDeviceId()
                => Secure.GetString(Application.Context.ContentResolver, Secure.AndroidId);

            public DeviceInstallation GetDeviceInstallation(params string[] tags)
            {
                if (!NotificationsSupported)
                    throw new Exception(GetPlayServicesError());

                if (string.isNullOrWhitespace(Token))
                    throw new Exception("Unable to resolve token for FCM");

                var installation = new DeviceInstallation
                {
                    InstallationId = GetDeviceId(),
                    Platform = "fcm",
                    PushChannel = Token
                };

                installation.Tags.AddRange(tags);

                return installation;
            }

            string GetPlayServicesError()
            {
                int resultCode = GoogleApiAvailability.Instance.IsGooglePlayServicesAvailable(Application.Context);

                if (resultCode != ConnectionResult.Success)
                    return GoogleApiAvailability.Instance.IsUserResolvableError(resultCode) ?
                               GoogleApiAvailability.Instance.GetErrorString(resultCode) :
                               "This device is not supported";

                return "An error occurred preventing the use of push notifications";
            }
        }
    }
    ```

    > [!NOTE]
    > Cette classe fournit un ID unique (à l’aide de [Secure.AndroidId](https://docs.microsoft.com/dotnet/api/android.provider.settings.secure.androidid?view=xamarin-android-sdk-9)) dans le cadre de la charge utile de l’inscription au hub de notification.

1. Ajoutez une autre **Classe vide** au dossier **Services** appelée *PushNotificationFirebaseMessagingService.cs*, puis ajoutez l’implémentation suivante.

    ```csharp
    using Android.App;
    using Android.Content;
    using Firebase.Messaging;
    using PushDemo.Services;

    namespace PushDemo.Droid.Services
    {
        [Service]
        [IntentFilter(new[] { "com.google.firebase.MESSAGING_EVENT" })]
        public class PushNotificationFirebaseMessagingService : FirebaseMessagingService
        {
            IPushDemoNotificationActionService _notificationActionService;
            INotificationRegistrationService _notificationRegistrationService;
            IDeviceInstallationService _deviceInstallationService;

            IPushDemoNotificationActionService NotificationActionService
                => _notificationActionService ??
                    (_notificationActionService =
                    ServiceContainer.Resolve<IPushDemoNotificationActionService>());

            INotificationRegistrationService NotificationRegistrationService
                => _notificationRegistrationService ??
                    (_notificationRegistrationService =
                    ServiceContainer.Resolve<INotificationRegistrationService>());

            IDeviceInstallationService DeviceInstallationService
                => _deviceInstallationService ??
                    (_deviceInstallationService =
                    ServiceContainer.Resolve<IDeviceInstallationService>());

            public override void OnNewToken(string token)
            {
                DeviceInstallationService.Token = token;

                NotificationRegistrationService.RefreshRegistrationAsync()
                    .ContinueWith((task) => { if (task.IsFaulted) throw task.Exception; });
            }

            public override void OnMessageReceived(RemoteMessage message)
            {
                if(message.Data.TryGetValue("action", out var messageAction))
                    NotificationActionService.TriggerAction(messageAction);
            }
        }
    }
    ```

1. Dans **MainActivity.cs**, vérifiez que les espaces de noms suivants ont été ajoutés au début du fichier.

    ```csharp
    using System;
    using Android.App;
    using Android.Content;
    using Android.Content.PM;
    using Android.OS;
    using Android.Runtime;
    using Firebase.Iid;
    using PushDemo.Droid.Services;
    using PushDemo.Services;
    ```

1. Dans **MainActivity.cs**, définissez **LaunchMode** sur **SingleTop** afin que **MainActivity** ne soit pas recréé lors de son ouverture.

    ```csharp
    [Activity(
        Label = "PushDemo",
        LaunchMode = LaunchMode.SingleTop,
        Icon = "@mipmap/icon",
        Theme = "@style/MainTheme",
        MainLauncher = true,
        ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation)]
    ```

1. Ajoutez des propriétés privées et des champs de stockage correspondants pour stocker une référence aux implémentations **IPushNotificationActionService** et **IDeviceInstallationService**.

    ```csharp
    IPushDemoNotificationActionService _notificationActionService;
    IDeviceInstallationService _deviceInstallationService;

    IPushDemoNotificationActionService NotificationActionService
        => _notificationActionService ??
            (_notificationActionService =
            ServiceContainer.Resolve<IPushDemoNotificationActionService>());

    IDeviceInstallationService DeviceInstallationService
        => _deviceInstallationService ??
            (_deviceInstallationService =
            ServiceContainer.Resolve<IDeviceInstallationService>());
    ```

1. Implémentez l’interface **IOnSuccessListener** pour récupérer et stocker le jeton **Firebase**.

    ```csharp
    public class MainActivity : global::Xamarin.Forms.Platform.Android.FormsAppCompatActivity, Android.Gms.Tasks.IOnSuccessListener
    {
        ...

        public void OnSuccess(Java.Lang.Object result)
            => DeviceInstallationService.Token =
                result.Class.GetMethod("getToken").Invoke(result).ToString();
    }
    ```

1. Ajoutez une nouvelle méthode appelée **ProcessNotificationActions** qui vérifiera si une intention (**Intent**) donnée a une valeur supplémentaire nommée *action*. Déclenchez cette action de manière conditionnelle à l’aide de l’implémentation **IPushDemoNotificationActionService**.

    ```csharp
    void ProcessNotificationActions(Intent intent)
    {
        try
        {
            if (intent?.HasExtra("action") == true)
            {
                var action = intent.GetStringExtra("action");

                if (!string.IsNullOrEmpty(action))
                    NotificationActionService.TriggerAction(action);
            }
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine(ex.Message);
        }
    }
    ```

1. Substituez la méthode **OnNewIntent** pour appeler la méthode **ProcessNotificationActions**.

    ```csharp
    protected override void OnNewIntent(Intent intent)
    {
        base.OnNewIntent(intent);
        ProcessNotificationActions(intent);
    }
    ```

    > [!NOTE]
    > Dans la mesure où **LaunchMode** pour **Activity** est défini sur **SingleTop**, une intention (**Intent**) sera envoyée à l’instance **Activity** par le biais de la méthode **OnNewIntent**, plutôt que de la méthode **OnCreate**. Vous devez donc gérer une intention entrante dans les méthodes **OnCreate** et **OnNewIntent**.

1. Mettez à jour la méthode **OnCreate** pour appeler `Bootstrap.Begin` juste après l’appel à `base.OnCreate` en passant l’implémentation de **IDeviceInstallationService** propre à la plateforme.

    ```csharp
    Bootstrap.Begin(() => new DeviceInstallationService());
    ```

1. Dans la même méthode, appelez de manière conditionnelle **GetInstanceId** sur l’instance **FirebaseApp**, juste après l’appel à `Bootstrap.Begin`, en ajoutant **MainActivity** comme **IOnSuccessListener**.

    ```csharp
    if (DeviceInstallationService.NotificationsSupported)
    {
        FirebaseInstanceId.GetInstance(Firebase.FirebaseApp.Instance)
            .GetInstanceId()
            .AddOnSuccessListener(this);
    }
    ```

1. Toujours dans **OnCreate**, appelez **ProcessNotificationActions** immédiatement après l’appel à `LoadApplication` en passant l’intention (**Intent**) actuelle.

    ```cs
    ...

    LoadApplication(new App());

    ProcessNotificationActions(Intent);
    ```

> [!NOTE]
> Pour continuer à recevoir des notifications Push, vous devez réinscrire l’application chaque fois que vous l’exécutez et l’arrêtez à partir d’une session de débogage.
