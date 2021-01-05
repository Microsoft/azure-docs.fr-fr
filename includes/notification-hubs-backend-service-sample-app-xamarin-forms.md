---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: fc479522b3fd436ff02ff6b8985bdeddb66da90a
ms.sourcegitcommit: 74ba70139781ed854d3ad898a9c65ef70c0ba99b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/26/2020
ms.locfileid: "85448728"
---
### <a name="create-the-xamarinforms-solution"></a>Créer la solution Xamarin.Forms

1. Dans **Visual Studio**, créez une solution **Xamarin.Forms** en utilisant **Application Forms vide** comme modèle et en entrant *PushDemo* comme **Nom du projet**.

    > [!NOTE]
    > Dans la boîte de dialogue **Configurer votre application Forms vide**, assurez-vous que l’**Identificateur de l’organisation** correspond à la valeur que vous avez utilisée et que les deux cibles **Android** et **iOS** sont cochées.

1. Utilisez **Ctrl** + **clic** sur la solution *PushDemo*, puis choisissez **Mettre à jour les paquets NuGet**.
1. Utilisez **Ctrl** + **clic** sur la solution *PushDemo*, puis choisissez **Gérer les packages NuGet...** .
1. Cherchez le fichier **Newtonsoft.Json** et vérifiez qu’il est coché.
1. Cliquez sur **Ajouter des packages**, puis cliquez sur **Accepter** quand vous êtes invité à accepter les termes du contrat de licence.
1. Générez et exécutez l’application sur chaque plateforme cible (**Commande** + **Entrée**) pour vérifier que l’application basée sur un modèle s’exécute sur vos appareils.

### <a name="implement-the-cross-platform-components"></a>Implémenter les composants multiplateforme

1. Utilisez **Ctrl** + **clic** sur le projet **PushDemo**, choisissez **Nouveau dossier** dans le menu **Ajouter**, puis cliquez sur **Ajouter** en utilisant *Models* comme **Nom de dossier**.

1. Utilisez **Ctrl** + **clic** sur le dossier **Models**, puis choisissez **Nouveau fichier** dans le menu **Ajouter**.

1. Sélectionnez **Général** > **Classe vide**, entrez *DeviceInstallation.cs*, puis ajoutez l’implémentation suivante.

    ```csharp
    using System.Collections.Generic;
    using Newtonsoft.Json;

    namespace PushDemo.Models
    {
        public class DeviceInstallation
        {
            [JsonProperty("installationId")]
            public string InstallationId { get; set; }

            [JsonProperty("platform")]
            public string Platform { get; set; }

            [JsonProperty("pushChannel")]
            public string PushChannel { get; set; }

            [JsonProperty("tags")]
            public List<string> Tags { get; set; } = new List<string>();
        }
    }
    ```

1. Ajoutez une **Énumération vide** au dossier **Models** appelée *PushDemoAction.cs* avec l’implémentation suivante.

    ```csharp
    namespace PushDemo.Models
    {
        public enum PushDemoAction
        {
            ActionA,
            ActionB
        }
    }
    ```

1. Ajoutez un nouveau dossier au projet **PushDemo** appelé *Services*, puis ajoutez une **Classe vide** à ce dossier appelée *ServiceContainer.cs* avec l’implémentation suivante.

     ```csharp
    using System;
    using System.Collections.Generic;

    namespace PushDemo.Services
    {
        public static class ServiceContainer
        {
            static readonly Dictionary<Type, Lazy<object>> services
                = new Dictionary<Type, Lazy<object>>();

            public static void Register<T>(Func<T> function)
                => services[typeof(T)] = new Lazy<object>(() => function());

            public static T Resolve<T>()
                => (T)Resolve(typeof(T));

            public static object Resolve(Type type)
            {
                {
                    if (services.TryGetValue(type, out var service))
                        return service.Value;

                    throw new KeyNotFoundException($"Service not found for type '{type}'");
                }
            }
        }
    }
    ```

    > [!NOTE]
    > Il s’agit d’une version réduite de la classe [ServiceContainer](https://github.com/xamcat/mobcat-library/blob/master/MobCAT/ServiceContainer.cs) à partir du dépôt [XamCAT](https://github.com/xamcat/mobcat-library). Elle sera utilisée comme conteneur IoC (inversion de contrôle) léger.

1. Ajoutez une **Interface vide** au dossier **Services** appelée *IDeviceInstallationService.cs*, puis ajoutez le code suivant.

    ```csharp
    using PushDemo.Models;

    namespace PushDemo.Services
    {
        public interface IDeviceInstallationService
        {
            string Token { get; set; }
            bool NotificationsSupported { get; }
            string GetDeviceId();
            DeviceInstallation GetDeviceInstallation(params string[] tags);
        }
    }
    ```

    > [!NOTE]
    > Cette interface sera implémentée et démarrée par chaque cible ultérieurement pour fournir les fonctionnalités propres à la plateforme et les informations **DeviceInstallation** nécessaires au service back-end.

1. Ajoutez une autre **Interface vide** au dossier **Services** appelée *INotificationRegistrationService.cs*, puis ajoutez le code suivant.  

    ```csharp
    using System.Threading.Tasks;

    namespace PushDemo.Services
    {
        public interface INotificationRegistrationService
        {
            Task DeregisterDeviceAsync();
            Task RegisterDeviceAsync(params string[] tags);
            Task RefreshRegistrationAsync();
        }
    }
    ```

    > [!NOTE]
    > Cette interface gère l’interaction entre le client et le service back-end.

1. Ajoutez une autre **Interface vide** au dossier **Services** appelée *INotificationActionService.cs*, puis ajoutez le code suivant.  

    ```csharp
    namespace PushDemo.Services
    {
        public interface INotificationActionService
        {
            void TriggerAction(string action);
        }
    }
    ```

    > [!NOTE]
    > Cette interface est utilisée comme mécanisme simple pour centraliser la gestion des actions de notification.

1. Ajoutez une **Interface vide** au dossier **Services** appelée *IPushDemoNotificationActionService.cs* qui dérive de *INotificationActionService*, avec l’implémentation suivante.  

    ```csharp
    using System;
    using PushDemo.Models;

    namespace PushDemo.Services
    {
        public interface IPushDemoNotificationActionService : INotificationActionService
        {
            event EventHandler<PushDemoAction> ActionTriggered;
        }
    }
    ```

    > [!NOTE]
    > Ce type est propre à l’application **PushDemo** et utilise l’énumération **PushDemoAction** pour identifier l’action qui est déclenchée de manière fortement typée.

1. Ajoutez au dossier **Services** une **Classe vide** appelée *NotificationRegistrationService.cs* implémentant l’*INotificationRegistrationService* avec le code suivant.

    ```csharp
    using System;
    using System.Net.Http;
    using System.Text;
    using System.Threading.Tasks;
    using Newtonsoft.Json;
    using PushDemo.Models;
    using Xamarin.Essentials;

    namespace PushDemo.Services
    {
        public class NotificationRegistrationService : INotificationRegistrationService
        {
            const string RequestUrl = "api/notifications/installations";
            const string CachedDeviceTokenKey = "cached_device_token";
            const string CachedTagsKey = "cached_tags";

            string _baseApiUrl;
            HttpClient _client;
            IDeviceInstallationService _deviceInstallationService;

            public NotificationRegistrationService(string baseApiUri, string apiKey)
            {
                _client = new HttpClient();
                _client.DefaultRequestHeaders.Add("Accept", "application/json");
                _client.DefaultRequestHeaders.Add("apikey", apiKey);

                _baseApiUrl = baseApiUri;
            }

            IDeviceInstallationService DeviceInstallationService
                => _deviceInstallationService ??
                    (_deviceInstallationService = ServiceContainer.Resolve<IDeviceInstallationService>());

            public async Task DeregisterDeviceAsync()
            {
                var cachedToken = await SecureStorage.GetAsync(CachedDeviceTokenKey)
                    .ConfigureAwait(false);

                if (cachedToken == null)
                    return;

                var deviceId = DeviceInstallationService?.GetDeviceId();

                if (string.IsNullOrWhiteSpace(deviceId))
                    throw new Exception("Unable to resolve an ID for the device.");

                await SendAsync(HttpMethod.Delete, $"{RequestUrl}/{deviceId}")
                    .ConfigureAwait(false);

                SecureStorage.Remove(CachedDeviceTokenKey);
                SecureStorage.Remove(CachedTagsKey);
            }

            public async Task RegisterDeviceAsync(params string[] tags)
            {
                var deviceInstallation = DeviceInstallationService?.GetDeviceInstallation(tags);

                await SendAsync<DeviceInstallation>(HttpMethod.Put, RequestUrl, deviceInstallation)
                    .ConfigureAwait(false);

                await SecureStorage.SetAsync(CachedDeviceTokenKey, deviceInstallation.PushChannel)
                    .ConfigureAwait(false);

                await SecureStorage.SetAsync(CachedTagsKey, JsonConvert.SerializeObject(tags));
            }

            public async Task RefreshRegistrationAsync()
            {
                var cachedToken = await SecureStorage.GetAsync(CachedDeviceTokenKey)
                    .ConfigureAwait(false);

                var serializedTags = await SecureStorage.GetAsync(CachedTagsKey)
                    .ConfigureAwait(false);

                if (string.IsNullOrWhiteSpace(cachedToken) ||
                    string.IsNullOrWhiteSpace(serializedTags) ||
                    string.IsNullOrWhiteSpace(DeviceInstallationService.Token) ||
                    cachedToken == DeviceInstallationService.Token)
                    return;

                var tags = JsonConvert.DeserializeObject<string[]>(serializedTags);

                await RegisterDeviceAsync(tags);
            }

            async Task SendAsync<T>(HttpMethod requestType, string requestUri, T obj)
            {
                string serializedContent = null;

                await Task.Run(() => serializedContent = JsonConvert.SerializeObject(obj))
                    .ConfigureAwait(false);

                await SendAsync(requestType, requestUri, serializedContent);
            }

            async Task SendAsync(
                HttpMethod requestType,
                string requestUri,
                string jsonRequest = null)
            {
                var request = new HttpRequestMessage(requestType, new Uri($"{_baseApiUrl}{requestUri}"));

                if (jsonRequest != null)
                    request.Content = new StringContent(jsonRequest, Encoding.UTF8, "application/json");

                var response = await _client.SendAsync(request).ConfigureAwait(false);

                response.EnsureSuccessStatusCode();
            }
        }
    }
    ```

    > [!NOTE]
    > L’argument **apiKey** n’est obligatoire que si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional).

1. Ajoutez au dossier **Services** une **Classe vide** appelée *PushDemoNotificationActionService.cs* implémentant l’*IPushDemoNotificationActionService* avec le code suivant.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using PushDemo.Models;

    namespace PushDemo.Services
    {
        public class PushDemoNotificationActionService : IPushDemoNotificationActionService
        {
            readonly Dictionary<string, PushDemoAction> _actionMappings = new Dictionary<string, PushDemoAction>
            {
                { "action_a", PushDemoAction.ActionA },
                { "action_b", PushDemoAction.ActionB }
            };

            public event EventHandler<PushDemoAction> ActionTriggered = delegate { };

            public void TriggerAction(string action)
            {
                if (!_actionMappings.TryGetValue(action, out var pushDemoAction))
                    return;

                List<Exception> exceptions = new List<Exception>();

                foreach (var handler in ActionTriggered?.GetInvocationList())
                {
                    try
                    {
                        handler.DynamicInvoke(this, pushDemoAction);
                    }
                    catch (Exception ex)
                    {
                        exceptions.Add(ex);
                    }
                }

                if (exceptions.Any())
                    throw new AggregateException(exceptions);
            }
        }
    }
    ```

1. Ajoutez au projet **PushDemo** une **Classe vide** appelée *Config.cs* avec l’implémentation suivante.  

    ```csharp
    namespace PushDemo
    {
        public static partial class Config
        {
            public static string ApiKey = "API_KEY";
            public static string BackendServiceEndpoint = "BACKEND_SERVICE_ENDPOINT";
        }
    }
    ```

    > [!NOTE]
    > Cette classe est un moyen simple de conserver les secrets en dehors du contrôle de code source. Vous pouvez remplacer ces valeurs dans le cadre d’une génération automatisée ou les remplacer à l’aide d’une classe partielle locale. Vous effectuerez cette opération à l’étape suivante.
    >
    > Le champ **ApiKey** n’est obligatoire que si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional).

1. Ajoutez au projet **PushDemo** une autre **Classe vide** appelée *Config.local_secrets.cs* avec l’implémentation suivante.  

    ```csharp
    namespace PushDemo
    {
        public static partial class Config
        {
            static Config()
            {
                ApiKey = "<your_api_key>";
                BackendServiceEndpoint = "<your_api_app_url>";
            }
        }
    }
    ```

    > [!NOTE]
    > Remplacez les valeurs d’espace réservé par vos propres valeurs. Vous devez en avoir pris note quand vous avez créé le service back-end. L’URL de l’**application API** doit être ``https://<api_app_name>.azurewebsites.net/``. N’oubliez pas d’ajouter ``*.local_secrets.*`` à votre fichier gitignore pour éviter de valider ce fichier.
    >
    > Le champ **ApiKey** n’est obligatoire que si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional).

1. Ajoutez au projet **PushDemo** une **Classe vide** appelée *Bootstrap.cs* avec l’implémentation suivante.  

    ```csharp
    using System;
    using PushDemo.Services;

    namespace PushDemo
    {
        public static class Bootstrap
        {
            public static void Begin(Func<IDeviceInstallationService> deviceInstallationService)
            {
                ServiceContainer.Register(deviceInstallationService);

                ServiceContainer.Register<IPushDemoNotificationActionService>(()
                    => new PushDemoNotificationActionService());

                ServiceContainer.Register<INotificationRegistrationService>(()
                    => new NotificationRegistrationService(
                        Config.BackendServiceEndpoint,
                        Config.ApiKey));
            }
        }
    }
    ```

    > [!NOTE]
    > La méthode **Begin** est appelée par chaque plateforme quand l’application se lance en passant une implémentation d’**IDeviceInstallationService** propre à la plateforme.
    >
    > L’argument de constructeur **apiKey** **NotificationRegistrationService** n’est obligatoire que si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional).

### <a name="implement-the-cross-platform-ui"></a>Implémenter l’interface utilisateur multiplateforme

1. Dans le projet **PushDemo**, ouvrez **MainPage.xaml** et remplacez le contrôle **StackLayout** par ce qui suit.

    ```xml
    <StackLayout VerticalOptions="EndAndExpand"  
                 HorizontalOptions="FillAndExpand"
                 Padding="20,40">
        <Button x:Name="RegisterButton"
                Text="Register"
                Clicked="RegisterButtonClicked" />
        <Button x:Name="DeregisterButton"
                Text="Deregister"
                Clicked="DeregisterButtonClicked" />
    </StackLayout>
    ```

1. Maintenant, dans **MainPage.xaml.cs**, ajoutez un champ de stockage **readonly** pour stocker une référence à l’implémentation **INotificationRegistrationService**.

    ```csharp
    readonly INotificationRegistrationService _notificationRegistrationService;
    ```

1. Dans le constructeur **MainPage**, résolvez l’implémentation **INotificationRegistrationService** à l’aide de **ServiceContainer** et attribuez-la au champ de stockage *_notificationRegistrationService_*.

    ```csharp
    public MainPage()
    {
        InitializeComponent();

        _notificationRegistrationService =
            ServiceContainer.Resolve<INotificationRegistrationService>();
    }
    ```

1. Implémentez les gestionnaires d’événements pour les événements **Clicked** des boutons **RegisterButton** et **DeregisterButton** en appelant les méthodes **Register**/**Deregister** correspondantes.

    ```csharp
    void RegisterButtonClicked(object sender, EventArgs e)
        => _notificationRegistrationService.RegisterDeviceAsync().ContinueWith((task)
            => { ShowAlert(task.IsFaulted ?
                    task.Exception.Message :
                    $"Device registered"); });

    void DeregisterButtonClicked(object sender, EventArgs e)
        => _notificationRegistrationService.DeregisterDeviceAsync().ContinueWith((task)
            => { ShowAlert(task.IsFaulted ?
                    task.Exception.Message :
                    $"Device deregistered"); });

    void ShowAlert(string message)
        => MainThread.BeginInvokeOnMainThread(()
            => DisplayAlert("PushDemo", message, "OK").ContinueWith((task)
                => { if (task.IsFaulted) throw task.Exception; }));
    ```

1. Maintenant, dans **App.xaml.cs**, vérifiez que les espaces de noms suivants sont référencés.

    ```csharp
    using PushDemo.Models;
    using PushDemo.Services;
    using Xamarin.Essentials;
    using Xamarin.Forms;
    ```

1. Implémentez le gestionnaire d’événements pour l’événement **ActionTriggered** d’**IPushDemoNotificationActionService**.

    ```csharp
    void NotificationActionTriggered(object sender, PushDemoAction e)
        => ShowActionAlert(e);

    void ShowActionAlert(PushDemoAction action)
        => MainThread.BeginInvokeOnMainThread(()
            => MainPage?.DisplayAlert("PushDemo", $"{action} action received", "OK")
                .ContinueWith((task) => { if (task.IsFaulted) throw task.Exception; }));
    ```

1. Dans le constructeur **App**, résolvez l’implémentation **IPushNotificationActionService** à l’aide de **ServiceContainer** et abonnez-vous à l’événement **ActionTriggered** d’**IPushDemoNotificationActionService**.

    ```csharp
    public App()
    {
        InitializeComponent();

        ServiceContainer.Resolve<IPushDemoNotificationActionService>()
            .ActionTriggered += NotificationActionTriggered;

        MainPage = new MainPage();
    }
    ```

    > [!NOTE]
    > Il s’agit simplement d’illustrer la réception et la propagation des actions de notification Push. En règle générale, celles-ci sont gérées en mode silencieux par exemple, en accédant à une vue spécifique ou en actualisant certaines données plutôt qu’en affichant une alerte via la **Page** racine, **MainPage** en l’occurrence.
