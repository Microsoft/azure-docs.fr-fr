---
title: Gestion des inscriptions
description: Cette rubrique explique comment inscrire des appareils auprès des hubs de notification en vue de recevoir des notifications Push.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: fd0ee230-132c-4143-b4f9-65cef7f463a1
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 07/07/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 04/08/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 7807d28da459656938acb399eb8c621e4c292372
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89001567"
---
# <a name="registration-management"></a>Gestion des inscriptions

Cette rubrique explique comment inscrire des appareils auprès des hubs de notification en vue de recevoir des notifications Push. Elle donne une description générale des inscriptions, puis présente les deux principaux modes d’inscription des appareils : l’inscription de l’appareil directement auprès du hub de notification et l’inscription via un serveur principal d’application.

## <a name="what-is-device-registration"></a>Présentation de l’inscription d’un appareil

L’inscription d’un appareil auprès d’un hub de notification s’effectue à l’aide d’une **inscription** ou d’une **installation**.

### <a name="registrations"></a>Inscriptions

Une inscription associe le handle PNS (Platform Notification Service, service de notification de plateforme) à des balises et éventuellement à un modèle. Le handle PNS peut être un ChannelURI, un jeton d’appareil ou un ID d’inscription FCM. Les balises permettent d’acheminer des notifications vers l’ensemble approprié de handles d’appareils. Pour plus d’informations, consultez [Routage et expressions de balises](notification-hubs-tags-segment-push-message.md). Les modèles permettent d’implémenter la transformation par inscription. Pour plus d’informations, consultez [Modèles](notification-hubs-templates-cross-platform-push-messages.md).

> [!NOTE]
> Azure Notification Hubs prend en charge un maximum de 60 balises par appareil.

### <a name="installations"></a>Installations

Une installation est une inscription améliorée qui inclut un conteneur de propriétés associées à des opérations push. C’est la meilleure approche, et la plus récente, pour inscrire vos appareils. Toutefois, elle n’est actuellement pas prise en charge par le Kit de développement logiciel .NET côté client ([SDK Notification Hub pour les opérations de serveur principal](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)).  Cela signifie que si vous vous inscrivez à partir de l'appareil client lui-même, vous devrez utiliser l’approche [API REST Notification Hubs](/rest/api/notificationhubs/create-overwrite-installation) pour prendre en charge ces installations. Si vous utilisez un service principal, vous devriez être en mesure d'utiliser [SDK Notification Hub pour les opérations de serveur principal](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

Voici certains avantages essentiels que présente l’utilisation d’installations :

- La création ou la mise à jour d’une installation est entièrement idempotente. Vous pouvez donc la retenter sans vous soucier des inscriptions en double.
- Le modèle d’installation prend en charge un format de balise spécial (`$InstallationId:{INSTALLATION_ID}`) qui permet l’envoi d’une notification directement à un appareil spécifique. Par exemple, si le code de l’application définit l’ID d’installation `joe93developer` pour cet appareil particulier, un développeur peut cibler cet appareil lors de l’envoi d’une notification à la balise `$InstallationId:{joe93developer}`. Ceci vous permet de cibler un appareil spécifique sans avoir à effectuer aucun codage supplémentaire.
- L’utilisation d’installations vous permet également d’effectuer des mises à jour partielles d’inscriptions. La mise à jour partielle d’une installation est demandée avec une méthode PATCH utilisant la [norme JSON-Patch](https://tools.ietf.org/html/rfc6902). Cette opération est utile lorsque vous voulez mettre à jour des balises sur l’inscription. Il n’est pas nécessaire de retirer la totalité de l’inscription et de renvoyer à nouveau toutes les balises précédentes.

Une installation peut contenir les propriétés suivantes. Pour obtenir la liste complète des propriétés d’installation, consultez [Créer ou remplacer une installation avec l’API REST](/rest/api/notificationhubs/create-overwrite-installation) ou [Propriétés d’installation](/dotnet/api/microsoft.azure.notificationhubs.installation).

```json
// Example installation format to show some supported properties
{
    installationId: "",
    expirationTime: "",
    tags: [],
    platform: "",
    pushChannel: "",
    ………
    templates: {
        "templateName1" : {
            body: "",
            tags: [] },
        "templateName2" : {
            body: "",
            // Headers are for Windows Store only
            headers: {
                "X-WNS-Type": "wns/tile" }
            tags: [] }
    },
    secondaryTiles: {
        "tileId1": {
            pushChannel: "",
            tags: [],
            templates: {
                "otherTemplate": {
                    bodyTemplate: "",
                    headers: {
                        ... }
                    tags: [] }
            }
        }
    }
}
```

> [!NOTE]
> Par défaut, les inscriptions et les installations n’expirent pas.

Les inscriptions et les installations doivent contenir un handle PNS valide pour chaque appareil/canal. Comme les handles PNS peuvent être obtenus uniquement dans une application cliente sur l’appareil, l’une des méthodes consiste à effectuer l’inscription directement sur cet appareil avec l’application cliente. D’un autre côté, les considérations sur la sécurité et la logique métier associée aux balises impliquent que vous devez gérer l’inscription des appareils dans le serveur principal de l’application.

> [!NOTE]
> L’API Installations ne prend pas en charge le service Baidu (bien que l’API Registrations le fasse). 

### <a name="templates"></a>Modèles

Si vous voulez utiliser des [modèles](notification-hubs-templates-cross-platform-push-messages.md), l’installation de l’appareil contient également tous les modèles associés à cet appareil au format JSON (voir l’exemple ci-dessus). Les noms de modèles permettent de cibler différents modèles pour le même appareil.

Le nom de chaque modèle correspond à un corps de modèle et à un ensemble de balises en option. De plus, chaque plateforme peut avoir des propriétés de modèle supplémentaires. Dans le cas de Windows Store (via WNS) et de Windows Phone 8 (à l’aide de MPNS), un ensemble supplémentaire d’en-têtes peut faire partie du modèle. Dans le cas d’APNs, vous pouvez définir une propriété d’expiration sur une constante ou une expression de modèle. Pour obtenir la liste complète des propriétés d’installation, consultez [Créer ou remplacer une installation avec REST](/rest/api/notificationhubs/create-overwrite-installation) .

### <a name="secondary-tiles-for-windows-store-apps"></a>Vignettes secondaires pour les applications Windows Store

Pour les applications clientes Windows Store, le fait d’envoyer des notifications à des vignettes secondaires revient à les envoyer à la vignette principale. Ce comportement est également pris en charge dans les installations. Les vignettes secondaires ont un ChannelUri différent, ce que le Kit de développement logiciel (SDK) sur votre application cliente gère en toute transparence.

Le dictionnaire SecondaryTiles utilise le même paramètre TileId que celui qui permet de créer l’objet SecondaryTiles dans votre application Windows Store. Comme pour le paramètre principal ChannelUri, le paramètre ChannelUris des vignettes secondaires peut changer à tout moment. Afin de garder à jour les inscriptions dans le hub de notification, l’appareil doit les actualiser avec le paramètre ChannelUris actuel des vignettes secondaires.

## <a name="registration-management-from-the-device"></a>Gestion de l’inscription à partir de l’appareil

Lors de la gestion de l’inscription d’un appareil à partir des applications clientes, le serveur principal est uniquement chargé d’envoyer des notifications. Les applications clientes conservent les handles PNS à jour et inscrivent les balises. L’image suivante illustre ce modèle :

![Inscription à partir de l’appareil](./media/notification-hubs-registration-management/notification-hubs-registering-on-device.png)

L’appareil extrait d’abord le handle PNS du PNS, puis s’inscrit directement auprès du hub de notification. Une fois l’inscription effectuée, le serveur principal de l’application peut envoyer une notification ciblant cette inscription. Pour plus d’informations sur l’envoi de notifications, consultez [Routage et expressions de balises](notification-hubs-tags-segment-push-message.md).

Dans ce cas, vous utilisez uniquement les droits Listen (Écouter) pour accéder à vos hubs de notification à partir de l’appareil. Pour plus d’informations, consultez [Sécurité](notification-hubs-push-notification-security.md).

L’inscription à partir de l’appareil est la méthode la plus simple, mais présente quelques inconvénients :

- Une application cliente peut mettre à jour ses balises uniquement lorsqu’elle est active. Par exemple, si un utilisateur a deux appareils qui inscrivent des balises associées à des équipes sportives, lorsque le premier appareil inscrit une balise supplémentaire (par exemple, Seahawks), le second appareil ne reçoit pas les notifications concernant les Seahawks tant que l’application présente sur le second appareil n’a pas été exécutée une deuxième fois. De manière plus générale, lorsque les balises sont affectées par plusieurs appareils, la gestion des balises à partir du serveur principal est une option intéressante.
- Étant donné que les applications peuvent être piratées, la limitation de l’inscription à des balises spécifiques nécessite de prendre des précautions supplémentaires, comme décrit dans la section [Sécurité](notification-hubs-push-notification-security.md).

### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-an-installation"></a>Exemple de code permettant de s’inscrire auprès d’un hub de notification à partir d’un appareil à l’aide d’une installation

Pour l’instant, il n’est pris en charge qu’à l’aide de l’ [API REST Notification Hubs](/rest/api/notificationhubs/create-overwrite-installation).

Vous pouvez également utiliser la méthode PATCH utilisant la [norme JSON-Patch](https://tools.ietf.org/html/rfc6902) pour la mise à jour de l’installation.

```csharp
class DeviceInstallation
{
    public string installationId { get; set; }
    public string platform { get; set; }
    public string pushChannel { get; set; }
    public string[] tags { get; set; }

    private async Task<HttpStatusCode> CreateOrUpdateInstallationAsync(DeviceInstallation deviceInstallation,
        string hubName, string listenConnectionString)
    {
        if (deviceInstallation.installationId == null)
            return HttpStatusCode.BadRequest;

        // Parse connection string (https://msdn.microsoft.com/library/azure/dn495627.aspx)
        ConnectionStringUtility connectionSaSUtil = new ConnectionStringUtility(listenConnectionString);
        string hubResource = "installations/" + deviceInstallation.installationId + "?";
        string apiVersion = "api-version=2015-04";

        // Determine the targetUri that we will sign
        string uri = connectionSaSUtil.Endpoint + hubName + "/" + hubResource + apiVersion;

        //=== Generate SaS Security Token for Authorization header ===
        // See https://msdn.microsoft.com/library/azure/dn495627.aspx
        string SasToken = connectionSaSUtil.getSaSToken(uri, 60);

        using (var httpClient = new HttpClient())
        {
            string json = JsonConvert.SerializeObject(deviceInstallation);

            httpClient.DefaultRequestHeaders.Add("Authorization", SasToken);

            var response = await httpClient.PutAsync(uri, new StringContent(json, System.Text.Encoding.UTF8, "application/json"));
            return response.StatusCode;
        }
    }

    var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

    string installationId = null;
    var settings = ApplicationData.Current.LocalSettings.Values;

    // If we have not stored an installation ID in application data, create and store as application data.
    if (!settings.ContainsKey("__NHInstallationId"))
    {
        installationId = Guid.NewGuid().ToString();
        settings.Add("__NHInstallationId", installationId);
    }

    installationId = (string)settings["__NHInstallationId"];

    var deviceInstallation = new DeviceInstallation
    {
        installationId = installationId,
        platform = "wns",
        pushChannel = channel.Uri,
        //tags = tags.ToArray<string>()
    };

    var statusCode = await CreateOrUpdateInstallationAsync(deviceInstallation, 
                    "<HUBNAME>", "<SHARED LISTEN CONNECTION STRING>");

    if (statusCode != HttpStatusCode.Accepted)
    {
        var dialog = new MessageDialog(statusCode.ToString(), "Registration failed. Installation Id : " + installationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    else
    {
        var dialog = new MessageDialog("Registration successful using installation Id : " + installationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
}
```

### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-a-registration"></a>Exemple de code permettant de s’inscrire auprès d’un hub de notification à partir d’un appareil à l’aide d’une inscription

Ces méthodes créent ou mettent à jour une inscription de l’appareil sur lequel elles sont appelées. Cela signifie que pour mettre à jour le handle ou les balises, vous devez remplacer la totalité de l’inscription. Sachez que les inscriptions sont temporaires, et que vous devez donc toujours disposer d’un stockage fiable avec les balises actuelles dont a besoin un appareil spécifique.

```csharp
// Initialize the Notification Hub
NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(listenConnString, hubName);

// The Device ID from the PNS
var pushChannel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

// If you are registering from the client itself, then store this registration ID in device
// storage. Then when the app starts, you can check if a registration ID already exists or not before
// creating.
var settings = ApplicationData.Current.LocalSettings.Values;

// If we have not stored a registration ID in application data, store in application data.
if (!settings.ContainsKey("__NHRegistrationId"))
{
    // make sure there are no existing registrations for this push handle (used for iOS and Android)    
    string newRegistrationId = null;
    var registrations = await hub.GetRegistrationsByChannelAsync(pushChannel.Uri, 100);
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

    newRegistrationId = await hub.CreateRegistrationIdAsync();

    settings.Add("__NHRegistrationId", newRegistrationId);
}

string regId = (string)settings["__NHRegistrationId"];

RegistrationDescription registration = new WindowsRegistrationDescription(pushChannel.Uri);
registration.RegistrationId = regId;
registration.Tags = new HashSet<string>(YourTags);

try
{
    await hub.CreateOrUpdateRegistrationAsync(registration);
}
catch (Microsoft.WindowsAzure.Messaging.RegistrationGoneException e)
{
    settings.Remove("__NHRegistrationId");
}
```

## <a name="registration-management-from-a-backend"></a>Gestion des inscriptions à partir d’un serveur principal

La gestion des inscriptions à partir du serveur principal nécessite l’écriture de code supplémentaire. L’application de l’appareil doit fournir le handle PNS mis à jour au serveur principal à chaque démarrage de l’application (avec les balises et les modèles), et le serveur principal doit mettre à jour ce handle sur le hub de notification. L’image suivante illustre cette conception :

![Gestion des inscriptions](./media/notification-hubs-registration-management/notification-hubs-registering-on-backend.png)

Les avantages de la gestion des inscriptions à partir du serveur principal incluent la possibilité de modifier les balises aux inscriptions, même lorsque l’application correspondante est inactive sur l’appareil, et d’authentifier l’application cliente avant d’ajouter une balise à son inscription.

### <a name="example-code-to-register-with-a-notification-hub-from-a-backend-using-an-installation"></a>Exemple de code permettant de s’inscrire auprès d’un hub de notification à partir d’un serveur principal à l’aide d’une installation

L’appareil client obtient toujours son handle PNS et les propriétés d’installation pertinentes comme auparavant et appelle une API personnalisée sur le serveur principal qui peut effectuer l’inscription, autoriser les balises, etc. Le serveur principal peut tirer parti du [SDK Notification Hubs pour les opérations de serveur principal](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

Vous pouvez également utiliser la méthode PATCH utilisant la [norme JSON-Patch](https://tools.ietf.org/html/rfc6902) pour la mise à jour de l’installation.

```csharp
// Initialize the Notification Hub
NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(listenConnString, hubName);

// Custom API on the backend
public async Task<HttpResponseMessage> Put(DeviceInstallation deviceUpdate)
{

    Installation installation = new Installation();
    installation.InstallationId = deviceUpdate.InstallationId;
    installation.PushChannel = deviceUpdate.Handle;
    installation.Tags = deviceUpdate.Tags;

    switch (deviceUpdate.Platform)
    {
        case "mpns":
            installation.Platform = NotificationPlatform.Mpns;
            break;
        case "wns":
            installation.Platform = NotificationPlatform.Wns;
            break;
        case "apns":
            installation.Platform = NotificationPlatform.Apns;
            break;
        case "fcm":
            installation.Platform = NotificationPlatform.Fcm;
            break;
        default:
            throw new HttpResponseException(HttpStatusCode.BadRequest);
    }


    // In the backend we can control if a user is allowed to add tags
    //installation.Tags = new List<string>(deviceUpdate.Tags);
    //installation.Tags.Add("username:" + username);

    await hub.CreateOrUpdateInstallationAsync(installation);

    return Request.CreateResponse(HttpStatusCode.OK);
}
```

### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-a-registration-id"></a>Exemple de code permettant de s’inscrire auprès d’un hub de notification à partir d’un appareil à l’aide d’un ID d’inscription

À partir de votre serveur principal d’application, vous pouvez effectuer les opérations CRUD de base sur les inscriptions. Par exemple :

```csharp
var hub = NotificationHubClient.CreateClientFromConnectionString("{connectionString}", "hubName");

// create a registration description object of the correct type, e.g.
var reg = new WindowsRegistrationDescription(channelUri, tags);

// Create
await hub.CreateRegistrationAsync(reg);

// Get by ID
var r = await hub.GetRegistrationAsync<RegistrationDescription>("id");

// update
r.Tags.Add("myTag");

// update on hub
await hub.UpdateRegistrationAsync(r);

// delete
await hub.DeleteRegistrationAsync(r);
```

Le serveur principal doit gérer l’accès concurrentiel entre les mises à jour des inscriptions. Service Bus offre un contrôle de l’accès concurrentiel optimiste pour la gestion des inscriptions. Au niveau HTTP, cette implémentation est effectuée à l’aide d’ETag lors des opérations de gestion des inscriptions. Cette fonctionnalité est utilisée de manière transparente dans les Kits de développement logiciel (SDK) Microsoft, lesquels lèvent une exception si une mise à jour est refusée en raison d’accès concurrentiel. Le serveur principal d’application est chargé de gérer ces exceptions et, si nécessaire, de tenter une nouvelle mise à jour.
