---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: f0e83548d3ba3b353b471e2e3429a23421aec7b2
ms.sourcegitcommit: 971a3a63cf7da95f19808964ea9a2ccb60990f64
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2020
ms.locfileid: "85095300"
---
### <a name="create-a-web-project"></a>Créer un projet web

1. Dans **Visual Studio**, sélectionnez **Fichier** > **Nouvelle solution**.

1. Sélectionnez **.NET Core** > **Application** > **ASP.NET Core** > **API** > **Suivant**.
  
1. Dans la boîte de dialogue **Configurer votre nouvelle API web ASP.NET Core**, sélectionnez **.NET Core 3.1** comme **Framework cible**.

1. Entrez *PushDemoApi* comme **Nom du projet**, puis sélectionnez **Créer**.

1. Démarrez le débogage (**Commande** + **Entrée**) pour tester l’application basée sur un modèle.

    > [!NOTE]
    > L’application basée sur un modèle est configurée pour utiliser le **WeatherForecastController** en tant que *launchUrl*. Ce dernier est défini dans **Propriétés** > **launchSettings.json**.  
    >
    > Si un message indiquant qu’un **certificat de développement non valide a été trouvé** apparaît :
    >
    > 1. Cliquez sur **Oui** pour accepter d’exécuter l’outil « dotnet dev-certs https » afin de résoudre ce problème. L’outil « dotnet dev-certs https » vous invite ensuite à entrer un mot de passe pour le certificat et le mot de passe pour votre trousseau.
    >
    > 1. Cliquez sur **Oui** quand vous êtes invité à **installer et à approuver le nouveau certificat**, puis entrez le mot de passe de votre trousseau.

1. Développez le dossier **Controllers**, puis supprimez **WeatherForecastController.cs**.

1. Supprimez **WeatherForecast.cs**.

1. Utilisez **Ctrl** + **clic** sur le projet **PushDemoApi**, puis choisissez **Nouveau fichier** dans le menu **Ajouter**.

1. Configurez les valeurs de configuration locales à l’aide de l’[outil Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-3.1&tabs=linux#secret-manager). Le découplage des secrets de la solution garantit qu’ils ne se trouvent pas dans le contrôle de code source. Ouvrez **Terminal**, puis accédez au répertoire du fichier projet et exécutez les commandes suivantes :

    ```bash
    dotnet user-secrets init
    dotnet user-secrets set "NotificationHub:Name" <value>
    dotnet user-secrets set "NotificationHub:ConnectionString" <value>
    ```

    Remplacez les valeurs d’espace réservé par vos propres valeurs de nom de hub de notification et de chaîne de connexion. Vous les avez notées dans la section [Créer un hub de notification](#create-a-notification-hub). Sinon, vous pouvez les rechercher dans [Azure](https://portal.azure.com).

    **NotificationsHub:Name** :  
    Consultez *Nom* dans le résumé **Bases** en haut de **Vue d’ensemble**.  

    **NotificationHub:ConnectionString** :  
    Consultez *DefaultFullSharedAccessSignature* dans **Stratégies d’accès**.

    > [!NOTE]
    > Pour les scénarios de production, vous pouvez envisager des options telles qu’[Azure KeyVault](https://azure.microsoft.com/services/key-vault) pour stocker la chaîne de connexion de manière sécurisée. Par souci de simplicité, les secrets sont ajoutés aux paramètres d’application d’[Azure App Service](https://azure.microsoft.com/services/app-service/).

### <a name="authenticate-clients-using-an-api-key-optional"></a>Authentifier les clients à l’aide d’une clé API (facultatif)

Les clés API ne sont pas aussi sécurisées que les jetons, mais sont suffisantes dans le cadre de ce tutoriel. Une clé API peut être facilement configurée via le [middleware (intergiciel) ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/middleware/?view=aspnetcore-3.1).

1. Ajoutez la **clé API** aux valeurs de configuration locales.

    ```bash
    dotnet user-secrets set "Authentication:ApiKey" <value>
    ```

    > [!NOTE]
    > Vous devez remplacer la valeur d’espace réservé par la vôtre et en prendre note.

1. Utilisez **Ctrl** + **clic** sur le projet **PushDemoApi**, choisissez **Nouveau dossier** dans le menu **Ajouter**, puis cliquez sur **Ajouter** en utilisant *Authentication* comme **Nom de dossier**.

1. Utilisez **Ctrl** + **clic** sur le dossier **Authentication**, puis choisissez **Nouveau fichier** dans le menu **Ajouter**.

1. Sélectionnez **Général** > **Classe vide**, entrez *ApiKeyAuthOptions.cs* pour le **Nom**, puis cliquez sur **Nouveau** en ajoutant l’implémentation suivante.

    ```csharp
    using Microsoft.AspNetCore.Authentication;

    namespace PushDemoApi.Authentication
    {
        public class ApiKeyAuthOptions : AuthenticationSchemeOptions
        {
            public const string DefaultScheme = "ApiKey";
            public string Scheme => DefaultScheme;
            public string ApiKey { get; set; }
        }
    }
    ```

1. Ajoutez une autre **Classe vide** au dossier **Authentication** appelée *ApiKeyAuthHandler.cs*, puis ajoutez l’implémentation suivante.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Security.Claims;
    using System.Text.Encodings.Web;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authentication;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;

    namespace PushDemoApi.Authentication
    {
        public class ApiKeyAuthHandler : AuthenticationHandler<ApiKeyAuthOptions>
        {
            const string ApiKeyIdentifier = "apikey";

            public ApiKeyAuthHandler(
                IOptionsMonitor<ApiKeyAuthOptions> options,
                ILoggerFactory logger,
                UrlEncoder encoder,
                ISystemClock clock)
                : base(options, logger, encoder, clock) {}

            protected override Task<AuthenticateResult> HandleAuthenticateAsync()
            {
                string key = string.Empty;

                if (Request.Headers[ApiKeyIdentifier].Any())
                {
                    key = Request.Headers[ApiKeyIdentifier].FirstOrDefault();
                }
                else if (Request.Query.ContainsKey(ApiKeyIdentifier))
                {
                    if (Request.Query.TryGetValue(ApiKeyIdentifier, out var queryKey))
                        key = queryKey;
                }

                if (string.IsNullOrWhiteSpace(key))
                    return Task.FromResult(AuthenticateResult.Fail("No api key provided"));

                if (!string.Equals(key, Options.ApiKey, StringComparison.Ordinal))
                    return Task.FromResult(AuthenticateResult.Fail("Invalid api key."));

                var identities = new List<ClaimsIdentity> {
                    new ClaimsIdentity("ApiKeyIdentity")
                };

                var ticket = new AuthenticationTicket(
                    new ClaimsPrincipal(identities), Options.Scheme);

                return Task.FromResult(AuthenticateResult.Success(ticket));
            }
        }
    }
    ```

    > [!NOTE]
    > Un [gestionnaire d’authentification](https://docs.microsoft.com/aspnet/core/security/authentication/?view=aspnetcore-3.1#authentication-handler) est un type qui implémente le comportement d’un schéma, dans ce cas un modèle de clé API personnalisé.

1. Ajoutez une autre **Classe vide** au dossier **Authentication** appelée *ApiKeyAuthenticationBuilderExtensions.cs*, puis ajoutez l’implémentation suivante.

    ```csharp
    using System;
    using Microsoft.AspNetCore.Authentication;

    namespace PushDemoApi.Authentication
    {
        public static class AuthenticationBuilderExtensions
        {
            public static AuthenticationBuilder AddApiKeyAuth(
                this AuthenticationBuilder builder,
                Action<ApiKeyAuthOptions> configureOptions)
            {
                return builder
                    .AddScheme<ApiKeyAuthOptions, ApiKeyAuthHandler>(
                        ApiKeyAuthOptions.DefaultScheme,
                        configureOptions);
            }
        }
    }
    ```

    > [!NOTE]
    > Cette méthode d’extension simplifie le code de configuration du middleware dans **Startup.cs** ce qui le rend plus lisible et généralement plus facile à suivre.

1. Dans **Startup.cs**, mettez à jour la méthode **ConfigureServices** pour configurer l’authentification de la clé API sous l’appel à la méthode **services.AddControllers**.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();

        services.AddAuthentication(options =>
        {
            options.DefaultAuthenticateScheme = ApiKeyAuthOptions.DefaultScheme;
            options.DefaultChallengeScheme = ApiKeyAuthOptions.DefaultScheme;
        }).AddApiKeyAuth(Configuration.GetSection("Authentication").Bind);
    }
    ```

1. Toujours dans **Startup.cs**, mettez à jour la méthode **Configure** pour appeler les méthodes d’extension **UseAuthentication** et **UseAuthorization** sur l’**IApplicationBuilder** de l’application. Assurez-vous que ces méthodes sont appelées après **UseRouting** et avant **app.UseEndpoints**.

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseHttpsRedirection();

        app.UseRouting();

        app.UseAuthentication();

        app.UseAuthorization();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
    ```

    > [!NOTE]
    > L’appel d’**UseAuthentication** inscrit le middleware qui utilise les schémas d’authentification enregistrés (à partir de **ConfigureServices**). Elle doit être appelée avant tout middleware qui dépend de l’authentification d’utilisateurs.

### <a name="add-dependencies-and-configure-services"></a>Ajouter des dépendances et configurer des services

ASP.NET Core prend en charge le [modèle de conception logicielle d’injection de dépendances](https://docs.microsoft.com/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-3.1), technique qui permet d’obtenir une [inversion de contrôle](https://docs.microsoft.com/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) entre les classes et leurs dépendances.  

L’utilisation du hub de notification et du [SDK Notification Hubs pour les opérations de back-end](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/) est encapsulée dans un service. Le service est inscrit et mis à disposition par le biais d’une abstraction appropriée.

1. Utilisez **Ctrl** + **clic** sur le dossier **Dependencies**, puis choisissez **Gérer les packages NuGet...** .

1. Recherchez **Microsoft.Azure.NotificationHubs** et assurez-vous qu’il est activé.

1. Cliquez sur **Ajouter des packages**, puis cliquez sur **Accepter** quand vous êtes invité à accepter les termes du contrat de licence.

1. Utilisez **Ctrl** + **clic** sur le projet **PushDemoApi**, choisissez **Nouveau dossier** dans le menu **Ajouter**, puis cliquez sur **Ajouter** en utilisant *Models* comme **Nom de dossier**.

1. Utilisez **Ctrl** + **clic** sur le dossier **Models**, puis choisissez **Nouveau fichier** dans le menu **Ajouter**.

1. Sélectionnez **Général** > **Classe vide**, entrez *PushTemplates.cs* pour le **Nom**, puis cliquez sur **Nouveau** en ajoutant l’implémentation suivante.

    ```csharp
    namespace PushDemoApi.Models
    {
        public class PushTemplates
        {
            public class Generic
            {
                public const string Android = "{ \"notification\": { \"title\" : \"PushDemo\", \"body\" : \"$(alertMessage)\"}, \"data\" : { \"action\" : \"$(alertAction)\" } }";
                public const string iOS = "{ \"aps\" : {\"alert\" : \"$(alertMessage)\"}, \"action\" : \"$(alertAction)\" }";
            }

            public class Silent
            {
                public const string Android = "{ \"data\" : {\"message\" : \"$(alertMessage)\", \"action\" : \"$(alertAction)\"} }";
                public const string iOS = "{ \"aps\" : {\"content-available\" : 1, \"apns-priority\": 5, \"sound\" : \"\", \"badge\" : 0}, \"message\" : \"$(alertMessage)\", \"action\" : \"$(alertAction)\" }";
            }
        }
    }
    ```

    > [!NOTE]
    > Cette classe contient les charges utiles de notification sous forme de jetons pour les notifications génériques et silencieuses requises par ce scénario. Les charges utiles sont définies en dehors de l’[installation ](https://docs.microsoft.com/dotnet/api/microsoft.azure.notificationhubs.installation?view=azure-dotnet) pour permettre l’expérimentation sans avoir à mettre à jour les installations existantes via le service. La gestion des modifications apportées aux installations de cette façon n’entre pas dans le cadre de ce tutoriel. Pour la production, envisagez des [modèles personnalisés](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-templates-cross-platform-push-messages).

1. Sélectionnez **Général** > **Classe vide**, entrez *DeviceInstallation.cs* pour le **Nom**, puis cliquez sur **Nouveau** en ajoutant l’implémentation suivante.

    ```csharp
    using System.Collections.Generic;
    using System.ComponentModel.DataAnnotations;

    namespace PushDemoApi.Models
    {
        public class DeviceInstallation
        {
            [Required]
            public string InstallationId { get; set; }

            [Required]
            public string Platform { get; set; }

            [Required]
            public string PushChannel { get; set; }

            public IList<string> Tags { get; set; } = Array.Empty<string>();
        }
    }
    ```

1. Ajoutez une autre **Classe vide** au dossier **Models** appelée *NotificationRequest.cs*, puis ajoutez l’implémentation suivante.

    ```csharp
    using System;

    namespace PushDemoApi.Models
    {
        public class NotificationRequest
        {
            public string Text { get; set; }
            public string Action { get; set; }
            public string[] Tags { get; set; } = Array.Empty<string>();
            public bool Silent { get; set; }
        }
    }
    ```

1. Ajoutez une autre **Classe vide** au dossier **Models** appelée *NotificationHubOptions.cs*, puis ajoutez l’implémentation suivante.

    ```csharp
    using System.ComponentModel.DataAnnotations;

    namespace PushDemoApi.Models
    {
        public class NotificationHubOptions
        {
            [Required]
            public string Name { get; set; }

            [Required]
            public string ConnectionString { get; set; }
        }
    }
    ```

1. Ajoutez au projet **PushDemoApi** un nouveau dossier appelé *Services*.

1. Ajoutez une **Interface vide** au dossier **Services** appelée *INotificationService.cs*, puis ajoutez l’implémentation suivante.

    ```csharp
    using System.Threading.Tasks;
    using PushDemoApi.Models;

    namespace PushDemoApi.Services
    {
        public interface INotificationService
        {
            Task<bool> CreateOrUpdateInstallationAsync(DeviceInstallation deviceInstallation, CancellationToken token);
            Task<bool> DeleteInstallationByIdAsync(string installationId, CancellationToken token);
            Task<bool> RequestNotificationAsync(NotificationRequest notificationRequest, CancellationToken token);
        }
    }
    ```

1. Ajoutez au dossier **Services** une **Classe vide** appelée *NotificationHubsService.cs*, puis ajoutez le code suivant pour implémenter l’interface **INotificationService** :

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.NotificationHubs;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    using PushDemoApi.Models;

    namespace PushDemoApi.Services
    {
        public class NotificationHubService : INotificationService
        {
            readonly NotificationHubClient _hub;
            readonly Dictionary<string, NotificationPlatform> _installationPlatform;
            readonly ILogger<NotificationHubService> _logger;

            public NotificationHubService(IOptions<NotificationHubOptions> options, ILogger<NotificationHubService> logger)
            {
                _logger = logger;
                _hub = NotificationHubClient.CreateClientFromConnectionString(
                    options.Value.ConnectionString,
                    options.Value.Name);

                _installationPlatform = new Dictionary<string, NotificationPlatform>
                {
                    { nameof(NotificationPlatform.Apns).ToLower(), NotificationPlatform.Apns },
                    { nameof(NotificationPlatform.Fcm).ToLower(), NotificationPlatform.Fcm }
                };
            }

            public async Task<bool> CreateOrUpdateInstallationAsync(DeviceInstallation deviceInstallation, CancellationToken token)
            {
                if (string.IsNullOrWhiteSpace(deviceInstallation?.InstallationId) ||
                    string.IsNullOrWhiteSpace(deviceInstallation?.Platform) ||
                    string.IsNullOrWhiteSpace(deviceInstallation?.PushChannel))
                    return false;

                var installation = new Installation()
                {
                    InstallationId = deviceInstallation.InstallationId,
                    PushChannel = deviceInstallation.PushChannel,
                    Tags = deviceInstallation.Tags
                };

                if (_installationPlatform.TryGetValue(deviceInstallation.Platform, out var platform))
                    installation.Platform = platform;
                else
                    return false;

                try
                {
                    await _hub.CreateOrUpdateInstallationAsync(installation, token);
                }
                catch
                {
                    return false;
                }

                return true;
            }

            public async Task<bool> DeleteInstallationByIdAsync(string installationId, CancellationToken token)
            {
                if (string.IsNullOrWhiteSpace(installationId))
                    return false;

                try
                {
                    await _hub.DeleteInstallationAsync(installationId, token);
                }
                catch
                {
                    return false;
                }

                return true;
            }

            public async Task<bool> RequestNotificationAsync(NotificationRequest notificationRequest, CancellationToken token)
            {
                if ((notificationRequest.Silent &&
                    string.IsNullOrWhiteSpace(notificationRequest?.Action)) ||
                    (!notificationRequest.Silent &&
                    (string.IsNullOrWhiteSpace(notificationRequest?.Text)) ||
                    string.IsNullOrWhiteSpace(notificationRequest?.Action)))
                    return false;

                var androidPushTemplate = notificationRequest.Silent ?
                    PushTemplates.Silent.Android :
                    PushTemplates.Generic.Android;

                var iOSPushTemplate = notificationRequest.Silent ?
                    PushTemplates.Silent.iOS :
                    PushTemplates.Generic.iOS;

                var androidPayload = PrepareNotificationPayload(
                    androidPushTemplate,
                    notificationRequest.Text,
                    notificationRequest.Action);

                var iOSPayload = PrepareNotificationPayload(
                    iOSPushTemplate,
                    notificationRequest.Text,
                    notificationRequest.Action);

                try
                {
                    if (notificationRequest.Tags.Length == 0)
                    {
                        // This will broadcast to all users registered in the notification hub
                        await SendPlatformNotificationsAsync(androidPayload, iOSPayload, token);
                    }
                    else if (notificationRequest.Tags.Length <= 20)
                    {
                        await SendPlatformNotificationsAsync(androidPayload, iOSPayload, notificationRequest.Tags, token);
                    }
                    else
                    {
                        var notificationTasks = notificationRequest.Tags
                            .Select((value, index) => (value, index))
                            .GroupBy(g => g.index / 20, i => i.value)
                            .Select(tags => SendPlatformNotificationsAsync(androidPayload, iOSPayload, tags, token));

                        await Task.WhenAll(notificationTasks);
                    }

                    return true;
                }
                catch (Exception e)
                {
                    _logger.LogError(e, "Unexpected error sending notification");
                    return false;
                }
            }

            string PrepareNotificationPayload(string template, string text, string action) => template
                .Replace("$(alertMessage)", text, StringComparison.InvariantCulture)
                .Replace("$(alertAction)", action, StringComparison.InvariantCulture);

            Task SendPlatformNotificationsAsync(string androidPayload, string iOSPayload, CancellationToken token)
            {
                var sendTasks = new Task[]
                {
                    _hub.SendFcmNativeNotificationAsync(androidPayload, token),
                    _hub.SendAppleNativeNotificationAsync(iOSPayload, token)
                };

                return Task.WhenAll(sendTasks);
            }

            Task SendPlatformNotificationsAsync(string androidPayload, string iOSPayload, IEnumerable<string> tags, CancellationToken token)
            {
                var sendTasks = new Task[]
                {
                    _hub.SendFcmNativeNotificationAsync(androidPayload, tags, token),
                    _hub.SendAppleNativeNotificationAsync(iOSPayload, tags, token)
                };

                return Task.WhenAll(sendTasks);
            }
        }
    }
    ```

    > [!NOTE]
    > L’expression d’étiquette fournie à **SendTemplateNotificationAsync** est limitée à 20 étiquettes. Elle est limitée à 6 pour la plupart des opérateurs, mais l’expression contient uniquement des opérateurs OU (||) dans ce cas. S’il y a plus de 20 étiquettes dans la demande, elles doivent être réparties en plusieurs demandes. Pour plus d’informations, consultez la documentation [Routage et expressions d’étiquette](https://msdn.microsoft.com/library/azure/Dn530749.aspx?f=255&MSPPError=-2147217396).

1. Dans **Startup.cs**, mettez à jour la méthode **ConfigureServices** pour ajouter **NotificationHubsService** en tant qu’implémentation singleton d’**INotificationService**.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        ...

        services.AddSingleton<INotificationService, NotificationHubService>();

        services.AddOptions<NotificationHubOptions>()
            .Configure(Configuration.GetSection("NotificationHub").Bind)
            .ValidateDataAnnotations();
    }
    ```

### <a name="create-the-notifications-api"></a>Créer l’API des notifications

1. Utilisez **Ctrl** + **clic** sur le dossier **Controllers**, puis choisissez **Nouveau fichier** dans le menu **Ajouter**.

1. Sélectionnez **ASP.NET Core** > **Classe de contrôleur d’API web**, entrez *NotificationsController* comme **Nom**, puis cliquez sur **Nouveau**.

1. Ajoutez les espaces de nom suivants en haut du fichier.

    ```csharp
    using System.ComponentModel.DataAnnotations;
    using System.Net;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    using PushDemoApi.Models;
    using PushDemoApi.Services;
    ```

1. Mettez à jour le contrôleur basé sur un modèle pour le faire dériver de **ControllerBase** et le décorer avec l’attribut **ApiController**.

    ```cs
    [ApiController]
    [Route("api/[controller]")]
    public class NotificationsController : ControllerBase
    {
        // Templated methods here
    }
    ```

    > [!NOTE]
    > La classe de base **Controller** fournit la prise en charge des vues, mais cela n’est pas nécessaire dans ce cas ; **ControllerBase** peut donc être utilisé à la place.

1. Si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional), vous devez décorer **NotificationsController** avec l’attribut **Authorize** également.

    ```cs
    [Authorize]
    ```

1. Mettez à jour le constructeur pour accepter l’instance inscrite d’**INotificationService** en tant qu’argument et assignez-la à un membre readonly.

    ```cs
    readonly INotificationService _notificationService;

    public NotificationsController(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }
    ```

1. Dans **launchSettings.json** (dans le dossier **Propriétés**), changez **launchUrl** de `weatherforecast` en *api/notifications* pour qu’elle corresponde à l’URL spécifiée dans l’attribut **Route** de **RegistrationsController**.

1. Démarrer le débogage (**commande** + **Entrée**) pour vérifier que l’application utilise le nouveau **NotificationsController** et retourne un état **401 Non autorisé**.

    > [!NOTE]
    > Visual Studio risque de ne pas lancer automatiquement l’application dans le navigateur. Vous allez dorénavant utiliser [Postman](https://www.postman.com/downloads) pour tester l’API.

1. Sous un nouvel onglet **[Postman](https://www.postman.com/downloads)** , définissez la demande sur **GET** et entrez l’adresse ci-dessous.

    ```bash
    https://localhost:5001/api/notifications
    ```

    > [!NOTE]
    > L’adresse localhost doit correspondre à la valeur **applicationUrl** qui se trouve dans **Propriétés** > **launchSettings.json**. La valeur par défaut doit être `https://localhost:5001;http://localhost:5000`, mais cette information doit être vérifiée si vous recevez une réponse 404.

1. Si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional), veillez à configurer les en-têtes de demande pour inclure votre valeur **apikey**.

   | Clé                            | Valeur                          |
   | ------------------------------ | ------------------------------ |
   | apikey                         | <votre_clé_d’api>                 |

1. Cliquez sur le bouton **Envoyer**.

    > [!NOTE]
    > Vous devez recevoir un état **200 OK** avec du contenu **JSON**.
    >
    > Si vous recevez un avertissement **Vérification du certificat SSL**, vous pouvez désactiver le paramètre **[Postman](https://www.postman.com/downloads)** de vérification de certificat SSL de la demande dans les **Paramètres**.

1. Remplacez les modèles de classe basée sur un modèle par le code suivant.

    ```csharp
    [HttpPut]
    [Route("installations")]
    [ProducesResponseType((int)HttpStatusCode.OK)]
    [ProducesResponseType((int)HttpStatusCode.BadRequest)]
    [ProducesResponseType((int)HttpStatusCode.UnprocessableEntity)]
    public async Task<IActionResult> UpdateInstallation(
        [Required]DeviceInstallation deviceInstallation)
    {
        var success = await _notificationService
            .CreateOrUpdateInstallationAsync(deviceInstallation, HttpContext.RequestAborted);

        if (!success)
            return new UnprocessableEntityResult();

        return new OkResult();
    }

    [HttpDelete()]
    [Route("installations/{installationId}")]
    [ProducesResponseType((int)HttpStatusCode.OK)]
    [ProducesResponseType((int)HttpStatusCode.BadRequest)]
    [ProducesResponseType((int)HttpStatusCode.UnprocessableEntity)]
    public async Task<ActionResult> DeleteInstallation(
        [Required][FromRoute]string installationId)
    {
        var success = await _notificationService
            .DeleteInstallationByIdAsync(installationId, CancellationToken.None);

        if (!success)
            return new UnprocessableEntityResult();

        return new OkResult();
    }

    [HttpPost]
    [Route("requests")]
    [ProducesResponseType((int)HttpStatusCode.OK)]
    [ProducesResponseType((int)HttpStatusCode.BadRequest)]
    [ProducesResponseType((int)HttpStatusCode.UnprocessableEntity)]
    public async Task<IActionResult> RequestPush(
        [Required]NotificationRequest notificationRequest)
    {
        if ((notificationRequest.Silent &&
            string.IsNullOrWhiteSpace(notificationRequest?.Action)) ||
            (!notificationRequest.Silent &&
            string.IsNullOrWhiteSpace(notificationRequest?.Text)))
            return new BadRequestResult();

        var success = await _notificationService
            .RequestNotificationAsync(notificationRequest, HttpContext.RequestAborted);

        if (!success)
            return new UnprocessableEntityResult();

        return new OkResult();
    }
    ```

### <a name="create-the-api-app"></a>Créer l’application API

Vous créez maintenant une [application API ](https://azure.microsoft.com/services/app-service/api/) dans [Azure App Service](https://docs.microsoft.com/azure/app-service/) pour héberger le service back-end.  

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Cliquez sur **Créer une ressource**, recherchez et choisissez **Application API**, puis cliquez sur **Créer**.

1. Mettez à jour les champs suivants, puis cliquez sur **Créer**.

    **Nom de l’application :**  
    entrez un nom global unique pour l’**application API**.

    **Abonnement :**  
    Choisissez l’**Abonnement** cible dans lequel vous avez créé le hub de notification.

    **Groupe de ressources :**  
    Choisissez le **Groupe de ressources** cible dans lequel vous avez créé le hub de notification.

    **Plan App Service/Emplacement :**  
    Créez un **plan App Service**.  

    > [!NOTE]
    > Passez de l’option par défaut à un plan qui comprend la prise en charge de **SSL**. Sinon, vous devrez prendre les mesures appropriées lors de l’utilisation de l’application mobile pour empêcher le blocage des demandes **http**.

    **Application Insights :**  
    Conservez l’option suggérée (une nouvelle ressource sera créée avec ce nom) ou choisissez une ressource existante.

1. Une fois l’**application API** provisionnée, accédez à cette ressource.

1. Notez la propriété **URL** dans le résumé **Bases** en haut de la **Vue d’ensemble**. Cette URL représente le *point de terminaison back-end* qui sera utilisé plus loin dans ce tutoriel.

    > [!NOTE]
    > L’URL utilise le nom de l’application API que vous avez spécifié précédemment, avec le format `https://<app_name>.azurewebsites.net`.

1. Sélectionnez **Configuration** dans la liste (sous **Paramètres**).  

1. Pour chacun des paramètres ci-dessous, cliquez sur **Nouveau paramètre d’application** afin d’entrer le **Nom** et une **Valeur**, puis cliquez sur **OK**.

   | Nom                               | Valeur                          |
   | ---------------------------------- | ------------------------------ |
   | `Authentication:ApiKey`            | <valeur_clé_api>                |
   | `NotificationHub:Name`             | <valeur_nom_hub>               |
   | `NotificationHub:ConnectionString` | <valeur_chaîne_de_connexion_hub>  |

   > [!NOTE]
   > Ce sont les mêmes paramètres que ceux que vous avez définis dans les paramètres utilisateur. Vous devez être en mesure de les reprendre. Le paramètre **Authentication:ApiKey** est uniquement obligatoire si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional). Pour les scénarios de production, vous pouvez envisager des options telles qu’[Azure KeyVault](https://azure.microsoft.com/services/key-vault). Ceux-ci ont été ajoutés en tant que paramètres d’application pour des raisons de simplicité dans ce cas.

1. Une fois tous les paramètres d’application ajoutés, cliquez sur **Enregistrer**, puis **Continuer**.

### <a name="publish-the-backend-service"></a>Publier le service back-end

Ensuite, vous déployez l’application sur l’application API afin de la rendre accessible à tous les appareils.  

1. Si ce n’est déjà fait, modifiez votre configuration de **Debug** en **Release**.

1. Utilisez **Ctrl** + **Clic** sur le projet **PushDemoApi**, puis choisissez **Publier sur Azure...** dans le menu **Publier**.

1. Suivez le processus d’authentification si vous y êtes invité. Utilisez le compte que vous avez utilisé dans la section [Créer l’application API](#create-the-api-app) précédente.

1. Sélectionnez l’**application API Azure App Service** que vous avez créée dans la liste comme cible de publication, puis cliquez sur **Publier**.

Après en avoir terminé avec l’Assistant, il publie l’application dans Azure, puis ouvre l’application. Prenez note de l’**URL** si ce n’est déjà fait. Cette URL représente le *point de terminaison back-end* qui est utilisé plus loin dans ce tutoriel.

### <a name="validating-the-published-api"></a>Validation de l’API publiée

1. Dans **[Postman](https://www.postman.com/downloads)** , ouvrez un nouvel onglet, définissez la demande sur **POST** et entrez l’adresse ci-dessous. Remplacez l’espace réservé par l’adresse de base que vous avez notée dans la section précédente [Publier le service back-end](#publish-the-backend-service).

    ```csharp
    https://<app_name>.azurewebsites.net/api/notifications/installations
    ```

    > [!NOTE]
    > L’adresse de base doit être au format ``https://<app_name>.azurewebsites.net/``.

1. Si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional), veillez à configurer les en-têtes de demande pour inclure votre valeur **apikey**.

   | Clé                            | Valeur                          |
   | ------------------------------ | ------------------------------ |
   | apikey                         | <votre_clé_d’api>                 |

1. Choisissez l’option **raw** (brut) pour **Body** (Corps), choisissez **JSON** dans la liste des options de format, puis incluez du contenu **JSON** d’espace réservé :

    ```json
    {}
    ```

1. Cliquez sur **Envoyer**.

    > [!NOTE]
    > Vous devez recevoir un état **400 Demande incorrecte** du service.

1. Effectuez à nouveau les étapes 1 à 4, mais cette fois, en spécifiant le point de terminaison des demandes pour vérifier que vous recevez la même réponse **400 Demande incorrecte**.

    ```bash
    https://<app_name>.azurewebsites.net/api/notifications/requests
    ```

> [!NOTE]
> Il n’est pas encore possible de tester l’API à l’aide de données de demande valides, car cela nécessite des informations propres à la plateforme de la part de l’application mobile cliente.
