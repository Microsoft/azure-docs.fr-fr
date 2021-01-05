---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 1dc491084f65bc90397b0897de6b6cfe4f2fd410
ms.sourcegitcommit: 74ba70139781ed854d3ad898a9c65ef70c0ba99b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/26/2020
ms.locfileid: "85448707"
---
### <a name="configure-infoplist-and-entitlementsplist"></a>Configurer Info.plist et Entitlements.plis

1. Assurez-vous que vous êtes connecté à votre **Compte de développeur Apple** sous ** > Visual Studio** **Préférences...**  > **Publication** > **Comptes de développeur Apple** et que le *Certificat* et le *Profil de provisionnement* appropriés ont été téléchargés. Normalement, vous avez créé ces ressources lors des étapes précédentes.

1. Dans **PushDemo.iOS**, ouvrez **Info.plist** et vérifiez que la valeur **BundleIdentifier** correspond à celle qui a été utilisée pour le profil de provisionnement respectif dans le [portail des développeurs Apple](https://developer.apple.com). La valeur **BundleIdentifier** suivait le format ``com.<organization>.PushDemo``.

1. Dans le même fichier, définissez **Version minimale du système** sur **13.0**.

    > [!NOTE]
    > Seuls les appareils qui exécutent **iOS 13.0 ou une version ultérieure** sont pris en charge dans ce tutoriel, mais vous pouvez l’étendre pour prendre en charge les appareils exécutant des versions antérieures.

1. Ouvrez **Options du projet** pour **PushDemo.iOS** (double-cliquez sur le projet).

1. Dans **Options du projet**, sous **Build > Signature du bundle iOS**, vérifiez que votre compte de développeur est sélectionné sous **Équipe**. Vérifiez ensuite que l’option « Automatically manage signing » (Gérer automatiquement la signature) est activée et que votre certificat de signature et votre profil de provisionnement ont été automatiquement sélectionnés.

    > [!NOTE]
    > Si votre *Certificat de signature* et votre *Profil de provisionnement* n’ont pas été sélectionnés automatiquement, choisissez **Provisionnement manuel**, puis cliquez sur **Bundle Signing Options** (Options de signature du bundle). Vérifiez que votre *Équipe* est sélectionnée sous **Identité de signature** et que votre profil de provisionnement pour *PushDemo* est sélectionné dans **Profil de provisionnement** pour les configurations **Debug** et **Release**, **iPhone** devant être sélectionné pour la **Plate-forme** dans les deux cas.

1. Dans **PushDemo.iOS**, ouvrez **Entitlements.plist** et vérifiez que l’option **Activer les notifications Push** est sélectionnée sous l’onglet **Droits**. Ensuite, assurez-vous que le paramètre **Environnement APS** est défini sur **développement** sous l’onglet **Source**.

### <a name="handle-push-notifications-for-ios"></a>Gérer les notifications Push pour iOS

1. Utilisez **Ctrl** + **clic** sur le projet **PushDemo.iOS**, choisissez **Nouveau dossier** dans le menu **Ajouter**, puis cliquez sur **Ajouter** en utilisant *Services* comme **Nom de dossier**.

1. Utilisez **Ctrl** + **clic** sur le dossier **Services**, puis choisissez **Nouveau fichier...** dans le menu **Ajouter**.

1. Sélectionnez **Général** > **Classe vide**, entrez *DeviceInstallationService.cs* pour le **Nom**, puis cliquez sur **Nouveau** en ajoutant l’implémentation suivante.

    ```csharp
    using System;
    using PushDemo.Models;
    using PushDemo.Services;
    using UIKit;

    namespace PushDemo.iOS.Services
    {
        public class DeviceInstallationService : IDeviceInstallationService
        {
            const int SupportedVersionMajor = 13;
            const int SupportedVersionMinor = 0;

            public string Token { get; set; }

            public bool NotificationsSupported
                => UIDevice.CurrentDevice.CheckSystemVersion(SupportedVersionMajor, SupportedVersionMinor);

            public string GetDeviceId()
                => UIDevice.CurrentDevice.IdentifierForVendor.ToString();

            public DeviceInstallation GetDeviceInstallation(params string[] tags)
            {
                if (!NotificationsSupported)
                    throw new Exception(GetNotificationsSupportError());

                if (string.isNullOrWhitespace(Token))
                    throw new Exception("Unable to resolve token for APNS");

                var installation = new DeviceInstallation
                {
                    InstallationId = GetDeviceId(),
                    Platform = "apns",
                    PushChannel = Token
                };

                installation.Tags.AddRange(tags);

                return installation;
            }

            string GetNotificationsSupportError()
            {
                if (!NotificationsSupported)
                    return $"This app only supports notifications on iOS {SupportedVersionMajor}.{SupportedVersionMinor} and above. You are running {UIDevice.CurrentDevice.SystemVersion}.";

                if (Token == null)
                    return $"This app can support notifications but you must enable this in your settings.";


                return "An error occurred preventing the use of push notifications";
            }
        }
    }
    ```

    > [!NOTE]
    > Cette classe fournit un ID unique (avec la valeur [UIDevice.IdentifierForVendor](https://docs.microsoft.com/dotnet/api/uikit.uidevice.identifierforvendor?view=xamarin-ios-sdk-12)) et la charge utile de l’inscription au hub de notification.

1. Ajoutez un nouveau dossier au projet **PushDemo.iOS** appelé *Extensions*, puis ajoutez une **Classe vide** à ce dossier appelée *NSDataExtensions.cs* avec l’implémentation suivante.

    ```csharp
    using System.Text;
    using Foundation;

    namespace PushDemo.iOS.Extensions
    {
        internal static class NSDataExtensions
        {
            internal static string ToHexString(this NSData data)
            {
                var bytes = data.ToArray();

                if (bytes == null)
                    return null;

                StringBuilder sb = new StringBuilder(bytes.Length * 2);

                foreach (byte b in bytes)
                    sb.AppendFormat("{0:x2}", b);

                return sb.ToString().ToUpperInvariant();
            }
        }
    }
    ```

1. Dans **AppDelegate.cs**, vérifiez que les espaces de noms suivants ont été ajoutés au début du fichier.

    ```csharp
    using System;
    using System.Diagnostics;
    using System.Threading.Tasks;
    using Foundation;
    using PushDemo.iOS.Extensions;
    using PushDemo.iOS.Services;
    using PushDemo.Services;
    using UIKit;
    using UserNotifications;
    using Xamarin.Essentials;
    ```

1. Ajoutez les propriétés privées et leurs champs de stockage respectifs pour stocker une référence aux implémentations **IPushDemoNotificationActionService**, **INotificationRegistrationService** et **IDeviceInstallationService**.

    ```csharp
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
    ```

1. Ajoutez la méthode **RegisterForRemoteNotifications** pour inscrire les paramètres de notification utilisateur, puis pour les notifications distantes avec **APNS**.

    ```csharp
    void RegisterForRemoteNotifications()
    {
        MainThread.BeginInvokeOnMainThread(() =>
        {
            var pushSettings = UIUserNotificationSettings.GetSettingsForTypes(
                UIUserNotificationType.Alert |
                UIUserNotificationType.Badge |
                UIUserNotificationType.Sound,
                new NSSet());

            UIApplication.SharedApplication.RegisterUserNotificationSettings(pushSettings);
            UIApplication.SharedApplication.RegisterForRemoteNotifications();
        });
    }
    ```

1. Ajoutez la méthode **CompleteRegistrationAsync** pour définir la valeur de la propriété `IDeviceInstallationService.Token`. Actualisez l’inscription et mettez en cache le jeton de l’appareil s’il a été mis à jour depuis son dernier stockage.

    ```csharp
    Task CompleteRegistrationAsync(NSData deviceToken)
    {
        DeviceInstallationService.Token = deviceToken.ToHexString();
        return NotificationRegistrationService.RefreshRegistrationAsync();
    }
    ```

1. Ajoutez la méthode **ProcessNotificationActions** pour le traitement des données de notification **NSDictionary** et l’appel conditionnel de **NotificationActionService.TriggerAction**.

    ```csharp
    void ProcessNotificationActions(NSDictionary userInfo)
    {
        if (userInfo == null)
            return;

        try
        {
            var actionValue = userInfo.ObjectForKey(new NSString("action")) as NSString;

            if (!string.IsNullOrWhiteSpace(actionValue?.Description))
                NotificationActionService.TriggerAction(actionValue.Description);
        }
        catch (Exception ex)
        {
            Debug.WriteLine(ex.Message);
        }
    }
    ```

1. Remplacez la méthode **RegisteredForRemoteNotifications** en passant l’argument **deviceToken** à la méthode **CompleteRegistrationAsync**.

    ```csharp
    public override void RegisteredForRemoteNotifications(
        UIApplication application,
        NSData deviceToken)
        => CompleteRegistrationAsync(deviceToken).ContinueWith((task)
            => { if (task.IsFaulted) throw task.Exception; });
    ```

1. Remplacez la méthode **ReceivedRemoteNotification** en passant l’argument **userInfo** à la méthode **ProcessNotificationActions**.

    ```csharp
    public override void ReceivedRemoteNotification(
        UIApplication application,
        NSDictionary userInfo)
        => ProcessNotificationActions(userInfo);
    ```

1. Remplacez la méthode **FailedToRegisterForRemoteNotifications** pour journaliser l’erreur.

    ```csharp
    public override void FailedToRegisterForRemoteNotifications(
        UIApplication application,
        NSError error)
        => Debug.WriteLine(error.Description);
    ```

    > [!NOTE]
    > Cette valeur s’apparente à un espace réservé. Vous pouvez implémenter une journalisation et une gestion des erreurs appropriées pour les scénarios de production.

1. Mettez à jour la méthode **FinishedLaunching** pour appeler `Bootstrap.Begin` juste après l’appel à `Forms.Init` en passant l’implémentation de **IDeviceInstallationService** propre à la plateforme.

    ```csharp
    Bootstrap.Begin(() => new DeviceInstallationService());
    ```

1. Dans la même méthode, demandez une autorisation de manière conditionnelle et ajoutez l’inscription aux notifications distantes juste après `Bootstrap.Begin`.

    ```csharp
    if (DeviceInstallationService.NotificationsSupported)
    {
        UNUserNotificationCenter.Current.RequestAuthorization(
                UNAuthorizationOptions.Alert |
                UNAuthorizationOptions.Badge |
                UNAuthorizationOptions.Sound,
                (approvalGranted, error) =>
                {
                    if (approvalGranted && error == null)
                        RegisterForRemoteNotifications();
                });
    }
    ```

1. Toujours dans **FinishedLaunching**, appelez **ProcessNotificationActions** immédiatement après l’appel à `LoadApplication` si l’argument **options** contient la méthode **UIApplication.LaunchOptionsRemoteNotificationKey** qui passe l’objet **userInfo** résultant.

    ```csharp
    using (var userInfo = options?.ObjectForKey(
        UIApplication.LaunchOptionsRemoteNotificationKey) as NSDictionary)
            ProcessNotificationActions(userInfo);
    ```
