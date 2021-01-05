---
title: Envoyer des notifications Push avec Azure Notification Hubs et les données spatiales Bing | Microsoft Docs
description: Ce didacticiel vous présente comment envoyer des notifications Push en fonction du lieu avec Azure Notification Hubs et les données spatiales Bing.
services: notification-hubs
documentationcenter: windows
keywords: Notifications Push, notifications Push
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: f41beea1-0d62-4418-9ffc-c9d70607a1b7
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows-phone
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: 51ad23e67a77c28d0ad8a147168a0094f5de1796
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578000"
---
# <a name="tutorial-send-location-based-push-notifications-with-notification-hubs-and-bing-spatial-data"></a>Tutoriel : Envoyer des notifications Push en fonction du lieu avec Notification Hubs et les données spatiales Bing

Ce didacticiel vous présente comment envoyer des notifications Push en fonction du lieu avec Azure Notification Hubs et les données spatiales Bing.

Dans ce tutoriel, vous effectuez les étapes suivantes :

> [!div class="checklist"]
> * Configurer la source de données
> * Configurer l’application UWP
> * Configurer le serveur principal
> * Tester l’envoi de notifications Push dans l’application de la plate-forme Windows universelle (UWP)

## <a name="prerequisites"></a>Prérequis

* **Abonnement Azure**. Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.
* [Visual Studio 2015 Update 1](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx) ou ultérieur ([Community Edition](https://go.microsoft.com/fwlink/?LinkId=691978&clcid=0x409)).
* [Kit de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/)dans sa version la plus récente.
* [Compte Centre de développement Bing Cartes](https://www.bingmapsportal.com/) (vous pouvez en créer un gratuitement et l’associer à votre compte Microsoft).

## <a name="set-up-the-data-source"></a>Configurer la source de données

1. Connectez-vous au [Centre de développement Bing Cartes](https://www.bingmapsportal.com/).
2. Dans la barre de navigation supérieure, sélectionnez **Sources de données**, puis sélectionnez **Gérer les sources de données**.

    ![Capture d’écran du centre de développement Bing Maps dans la page Gérer les sources de données, avec l’option Charger les données en tant que source de données encadrée en rouge.](./media/notification-hubs-geofence/bing-maps-manage-data.png)
3. Si vous n’avez pas une source de données existante, vous voyez un lien pour créer une source de données. Sélectionnez **Télécharger des données sous la forme d’une source de données**. Vous pouvez également utiliser le menu **Sources de données** > **Télécharger des données**.

    ![Capture d’écran de la boîte de dialogue Charger une source de données.](./media/notification-hubs-geofence/bing-maps-create-data.png)
4. Créez un fichier `NotificationHubsGeofence.pipe` sur votre disque dur avec le contenu suivant : Dans ce tutoriel, vous utilisez un exemple de fichier au format pipe qui délimite une partie des quais de San Francisco :

    ```text
    Bing Spatial Data Services, 1.0, TestBoundaries
    EntityID(Edm.String,primaryKey)|Name(Edm.String)|Longitude(Edm.Double)|Latitude(Edm.Double)|Boundary(Edm.Geography)
    1|SanFranciscoPier|||POLYGON ((-122.389825 37.776598,-122.389438 37.773087,-122.381885 37.771849,-122.382186 37.777022,-122.389825 37.776598))
    ```

    Ce fichier au format pipe représente l’entité suivante :

    ![Capture d’écran d’une carte du front de mer de San Francisco, avec un polygone rouge délimitant la zone des jetées.](./media/notification-hubs-geofence/bing-maps-geofence.png)
5. Sur la page **Télécharger une source de données**, effectuez les actions suivantes :
   1. Sélectionnez **pipe** dans **format de données**.
   2. Recherchez et sélectionnez le fichier `NotificationHubGeofence.pipe` créé à l’étape précédente.
   3. Sélectionnez le bouton **Télécharger**.

      > [!NOTE]
      > Vous serez peut-être invité à spécifier une nouvelle clé pour la **clé principale** qui diffère de la **clé de requête**. Créez simplement une clé dans le tableau de bord et actualisez la page de téléchargement de la source de données.
6. Une fois que vous avez téléchargé le fichier de données, vous devez publier la source de données. Sélectionnez **Sources de données** -> **Gérer les sources de données** comme vous le faisiez avant.
7. Sélectionnez votre source de données dans la liste, puis choisissez **Publier** dans la colonne **Actions**.

    ![Capture d’écran du centre de développement Bing Maps dans la page Gérer les sources de données, avec l’onglet Données géocodées sélectionné et l’option Publier encadrée en rouge.](./media/notification-hubs-geofence/publish-button.png)
8. Basculez vers l’onglet **Sources de données publiées** et vérifiez que vous voyez la source de données dans la liste.

    ![Capture d’écran du centre de développement Bing Maps dans la page Gérer les sources de données, avec l’onglet Sources de données publiées sélectionné.](./media/notification-hubs-geofence/bing-maps-published-data.png)
9. Sélectionnez **Modifier**. Vous consultez (en un coup d’œil) les emplacements que vous avez introduits dans les données.

    ![Capture d’écran de la page Modifier les données d’entité montrant une carte de la côte ouest des États-Unis et un point rouge sur les quais de San Francisco.](./media/notification-hubs-geofence/bing-maps-data-details.png)

    À ce stade, le portail n’indique pas les limites de la clôture virtuelle que vous avez créée. Vous devez seulement avoir la confirmation que l’emplacement spécifié se trouve dans la bonne zone.
10. Toutes les conditions nécessaires sont désormais remplies pour la source de données. Pour obtenir plus d’informations sur l’URL de la demande pour l’appel d’API, sélectionnez **Sources de données** dans le Centre de développement Bing Cartes, puis **Informations sur la source de données**.

    ![Capture d’écran du centre de développement Bing Maps dans la page Informations sur la source de données.](./media/notification-hubs-geofence/bing-maps-data-info.png)

    L’**URL de requête** est le point de terminaison sur lequel vous pouvez exécuter des requêtes pour vérifier si l’appareil se trouve dans les limites d’un emplacement ou non. Pour effectuer cette vérification, il vous suffit d’exécuter un appel GET sur l’URL de requête, en ajoutant les paramètres suivants :

    ```text
    ?spatialFilter=intersects(%27POINT%20LONGITUDE%20LATITUDE)%27)&$format=json&key=QUERY_KEY
    ```

    Bing Cartes effectue automatiquement les calculs pour voir si l’appareil se situe dans la clôture virtuelle. Lorsque vous exécutez la requête via un navigateur (ou cURL), vous obtenez une réponse JSON standard :

    ![Capture d’écran de la réponse JSON standard.](./media/notification-hubs-geofence/bing-maps-json.png)

    Cette réponse se produit uniquement lorsque le point se trouve bien dans les limites désignées. Si ce n’est pas le cas, vous obtenez un compartiment **results** vide :

    ![Capture d’écran d’une réponse JSON avec un compartiment de résultats vide.](./media/notification-hubs-geofence/bing-maps-nores.png)

## <a name="set-up-the-uwp-application"></a>Configurer l’application UWP

1. Dans Visual Studio, démarrez un nouveau projet de type **Application vide (application Windows universelle)** .

    ![Capture d’écran de la boîte de dialogue Nouveau projet de Visual Studio, avec l’option Application vide (Windows universel) Visual C# mise en évidence.](./media/notification-hubs-geofence/notification-hubs-create-blank-app.png)

    Une fois le projet créé, vous pouvez commencer à exploiter l’application. Nous allons à présent définir l’infrastructure de clôture virtuelle. Pour ce faire, vous allez utiliser les services Bing pour cette solution, vous pouvez ainsi interroger des infrastructures de localisation spécifiques grâce à un point de terminaison d’API REST public :

    ```text
    http://spatial.virtualearth.net/REST/v1/data/
    ```
    Pour que cela fonctionne, spécifiez les paramètres suivants :

   * **ID de source de données** et **nom de source de données** : dans l’API Bing Cartes, les sources de données contiennent diverses métadonnées compartimentées, telles que les emplacements et les heures d’ouverture de l’opération.  
   * **Nom d’entité** : l’entité que vous souhaitez utiliser comme point de référence pour la notification.
   * **Clé d’API Bing Cartes** : la clé que vous avez obtenue précédemment lorsque vous avez créé le compte Centre de développement Bing.

     Maintenant que la source de données est prête, vous pouvez commencer à travailler sur l’application UWP.
2. Activez les services de localisation pour votre application. Ouvrez le fichier `Package.appxmanifest` dans **l’Explorateur de solution**.

    ![Capture d’écran de l’Explorateur de solutions avec le fichier Package.appxmanifest mis en évidence.](./media/notification-hubs-geofence/vs-package-manifest.png)
3. Dans l’onglet de propriétés du package que vous venez d’ouvrir, basculez vers l’onglet **Capacités**, et sélectionnez **Localisation**.

    ![Capture d’écran de la boîte de dialogue Propriétés du package, montrant l’onglet Capacités avec l’option Localisation mise en évidence.](./media/notification-hubs-geofence/vs-package-location.png)
4. Créez un dossier dans votre solution nommé `Core`, puis ajoutez-y un nouveau fichier, nommé `LocationHelper.cs` :

    ![Capture d’écran de l’Explorateur de solutions avec le nouveau dossier Core mis en évidence.](./media/notification-hubs-geofence/vs-location-helper.png)

    La classe `LocationHelper` a un code pour obtenir l’emplacement de l’utilisateur via l’API système :

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Windows.Devices.Geolocation;

    namespace NotificationHubs.Geofence.Core
    {
        public class LocationHelper
        {
            private static readonly uint AppDesiredAccuracyInMeters = 10;

            public async static Task<Geoposition> GetCurrentLocation()
            {
                var accessStatus = await Geolocator.RequestAccessAsync();
                switch (accessStatus)
                {
                    case GeolocationAccessStatus.Allowed:
                        {
                            Geolocator geolocator = new Geolocator { DesiredAccuracyInMeters = AppDesiredAccuracyInMeters };

                            return await geolocator.GetGeopositionAsync();
                        }
                    default:
                        {
                            return null;
                        }
                }
            }

        }
    }
    ```

    Pour en savoir plus sur l’obtention de l’emplacement de l’utilisateur dans les applications UWP, consultez [Obtenir l’emplacement de l’utilisateur](/windows/uwp/maps-and-location/get-location).
5. Pour vérifier que l’obtention de l’emplacement fonctionne correctement, accédez au code de votre page d’accueil (`MainPage.xaml.cs`). Créez un gestionnaire d’événements pour l’événement `Loaded` dans le constructeur `MainPage`.

    ```csharp
    public MainPage()
    {
        this.InitializeComponent();
        this.Loaded += MainPage_Loaded;
    }
    ```

    L’implémentation du gestionnaire d’événements se fait de la manière suivante :

    ```csharp
    private async void MainPage_Loaded(object sender, RoutedEventArgs e)
    {
        var location = await LocationHelper.GetCurrentLocation();

        if (location != null)
        {
            Debug.WriteLine(string.Concat(location.Coordinate.Longitude,
                " ", location.Coordinate.Latitude));
        }
    }
    ```
6. Exécutez l’application et permettez-lui d’accéder à votre emplacement.

    ![Capture d’écran de la boîte de dialogue Let Notification Hubs Geo Fence access your location.](./media/notification-hubs-geofence/notification-hubs-location-access.png)
7. Lorsque l’application démarre, vous devez être en mesure de voir les coordonnées dans la fenêtre **Sortie** :

    ![Capture d’écran de la fenêtre de sortie affichant les coordonnées.](./media/notification-hubs-geofence/notification-hubs-location-output.png)

    Maintenant que vous savez que l’obtention de l’emplacement fonctionne, vous pouvez supprimer le gestionnaire d’événements de test Loaded si vous le désirez car vous ne l’utiliserez plus.
8. L’étape suivante consiste à capturer le changement d’emplacement. Dans la classe `LocationHelper`, ajoutez le gestionnaire d’événements pour `PositionChanged` :

    ```csharp
    geolocator.PositionChanged += Geolocator_PositionChanged;
    ```

    L’implémentation affiche les coordonnées de l’emplacement dans la fenêtre **Sortie** :

    ```csharp
    private static async void Geolocator_PositionChanged(Geolocator sender, PositionChangedEventArgs args)
    {
        await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
        {
            Debug.WriteLine(string.Concat(args.Position.Coordinate.Longitude, " ", args.Position.Coordinate.Latitude));
        });
    }
    ```

## <a name="set-up-the-backend"></a>Configurer le serveur principal

1. Téléchargez [l’exemple de backend .NET sur GitHub](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/NotifyUsers).
2. Une fois téléchargé, ouvrez le dossier `NotifyUsers`, puis le fichier `NotifyUsers.sln` dans Visual Studio.
3. Définissez le projet `AppBackend` comme **Projet de démarrage** , puis exécutez-le.

    ![Capture d’écran du menu contextuel Solution, avec l’option Définir comme projet de démarrage mise en évidence.](./media/notification-hubs-geofence/vs-startup-project.png)

    Le projet est déjà configuré pour envoyer des notifications Push aux appareils cibles. Vous aurez seulement deux actions à effectuer : spécifier la bonne chaîne de connexion pour le hub de notification et ajouter l’identification des limites afin d’envoyer une notification uniquement lorsque l’utilisateur se trouve dans les limites de la clôture virtuelle.

4. Pour configurer la chaîne de connexion dans le dossier `Models`, ouvrez `Notifications.cs`. La fonction `NotificationHubClient.CreateClientFromConnectionString` doit contenir les informations sur votre hub de notification que vous pouvez obtenir dans le [portail Azure](https://portal.azure.com) (consultez la page **Stratégies d’accès** dans **Paramètres**). Enregistrez le fichier de configuration mis à jour.
5. Créez un modèle pour le résultat de l’API Bing Cartes. Pour ce faire, la méthode la plus simple consiste à ouvrir le dossier `Models`, puis à sélectionner **Ajouter** > **Classe**. Nommez-le `GeofenceBoundary.cs`. Une fois terminé, copiez le fichier JSON à partir de la réponse de l’API que vous avez obtenue dans la première section. Dans Visual Studio, utilisez **Modifier** > **Collage spécial** > **Coller le code JSON en tant que classes**.

    De cette façon, l’objet sera désérialisé (ce qui était l’objectif). La classe obtenue doit ressembler à la classe suivante :

    ```csharp
    namespace AppBackend.Models
    {
        public class Rootobject
        {
            public D d { get; set; }
        }

        public class D
        {
            public string __copyright { get; set; }
            public Result[] results { get; set; }
        }

        public class Result
        {
            public __Metadata __metadata { get; set; }
            public string EntityID { get; set; }
            public string Name { get; set; }
            public float Longitude { get; set; }
            public float Latitude { get; set; }
            public string Boundary { get; set; }
            public string Confidence { get; set; }
            public string Locality { get; set; }
            public string AddressLine { get; set; }
            public string AdminDistrict { get; set; }
            public string CountryRegion { get; set; }
            public string PostalCode { get; set; }
        }

        public class __Metadata
        {
            public string uri { get; set; }
        }
    }
    ```
6. Ensuite, ouvrez `Controllers` > `NotificationsController.cs`. Mettez à jour l’appel POST afin de prendre en compte la longitude et la latitude cibles. Pour cela, ajoutez deux chaînes à la signature de fonction : `latitude` et `longitude`.

    ```csharp
    public async Task<HttpResponseMessage> Post(string pns, [FromBody]string message, string to_tag, string latitude, string longitude)
    ```
7. Dans le projet, créez une classe appelée `ApiHelper.cs`. Celle-ci vous permet de vous connecter à Bing afin de vérifier les intersections des limites des points. Implémentez une fonction `IsPointWithinBounds` comme indiqué dans le code suivant :

    ```csharp
    public class ApiHelper
    {
        public static readonly string ApiEndpoint = "{YOUR_QUERY_ENDPOINT}?spatialFilter=intersects(%27POINT%20({0}%20{1})%27)&$format=json&key={2}";
        public static readonly string ApiKey = "{YOUR_API_KEY}";

        public static bool IsPointWithinBounds(string longitude,string latitude)
        {
            var json = new WebClient().DownloadString(string.Format(ApiEndpoint, longitude, latitude, ApiKey));
            var result = JsonConvert.DeserializeObject<Rootobject>(json);
            if (result.d.results != null && result.d.results.Count() > 0)
            {
                return true;
            }
            else
            {
                return false;
            }
        }
    }
    ```

    > [!IMPORTANT]
    > Veillez à remplacer le point de terminaison d’API avec l’URL de requête obtenue précédemment dans le Centre de développement Bing (cela s’applique également à la clé d’API).

    Si la requête renvoie des résultats, cela signifie que le point spécifié se trouve dans les limites de la clôture virtuelle. La fonction retourne donc `true`. Si Bing ne renvoie pas de résultats, cela signifie que le point se trouve en dehors des limites de la recherche. La fonction retourne donc `false`.
8. Dans `NotificationsController.cs`, créez une vérification juste avant l’instruction switch :

    ```csharp
    if (ApiHelper.IsPointWithinBounds(longitude, latitude))
    {
        switch (pns.ToLower())
        {
            case "wns":
                //// Windows 8.1 / Windows Phone 8.1
                var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
                            "From " + user + ": " + message + "</text></binding></visual></toast>";
                outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);

                // Windows 10 specific Action Center support
                toast = @"<toast><visual><binding template=""ToastGeneric""><text id=""1"">" +
                            "From " + user + ": " + message + "</text></binding></visual></toast>";
                outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);

                break;
        }
    }
    ```

## <a name="test-push-notifications-in-the-uwp-app"></a>Test des notifications Push dans l’application UWP

1. Dans l’application UWP, vous devriez à présent être en mesure de tester les notifications. Dans la classe `LocationHelper`, créez une fonction `SendLocationToBackend` :

    ```csharp
    public static async Task SendLocationToBackend(string pns, string userTag, string message, string latitude, string longitude)
    {
        var POST_URL = "http://localhost:8741/api/notifications?pns=" +
            pns + "&to_tag=" + userTag + "&latitude=" + latitude + "&longitude=" + longitude;

        using (var httpClient = new HttpClient())
        {
            try
            {
                await httpClient.PostAsync(POST_URL, new StringContent("\"" + message + "\"",
                    System.Text.Encoding.UTF8, "application/json"));
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex.Message);
            }
        }
    }
    ```

    > [!NOTE]
    > Définissez le `POST_URL` à l’emplacement de votre application web déployée. Il est possible de l’exécuter localement pour le moment, mais quand vous travaillez sur le déploiement d’une version publique, vous devez l’héberger avec un fournisseur externe.
2. Inscrivez l’application UWP pour les notifications Push. Dans Visual Studio, sélectionnez **Projet** > **Store** > **Associer l’application au Store**.

    ![Capture d’écran du menu contextuel Solution, avec les options Store et Associer l’application au Store mises en évidence.](./media/notification-hubs-geofence/vs-associate-with-store.png)
3. Une fois que vous êtes connecté à votre compte de développeur, sélectionnez une application existante ou créez une application et associez le package à celle-ci.
4. Accédez au Centre de développement et ouvrez l’application que vous avez créée. Sélectionnez **Services** > **Notifications push** > **Services Microsoft Live**.

    ![Capture d’écran du centre de développement Windows affichant la page des notifications Push avec le site Services Live mis en évidence.](./media/notification-hubs-geofence/ms-live-services.png)
5. Sur le site, prenez note du **Secret d’application** et du **Package SID**. Vous avez besoin de ces deux éléments dans le portail Azure. Ouvrez votre hub de notification, sélectionnez **Paramètres** > **Services de notification** > **Windows (WNS)** et entrez les informations dans les champs requis.

    ![Capture d’écran montrant la page Paramètres avec les options Notification Services et Windows (WNS) mises en évidence, ainsi que des valeurs renseignées pour SID du package et Clé de sécurité.](./media/notification-hubs-geofence/notification-hubs-wns.png)
6. Choisissez **Enregistrer**.
7. Dans l’**Explorateur de solutions**, ouvrez **Références**, puis sélectionnez **Gérer les packages NuGet**. Ajoutez une référence à la **bibliothèque managée Microsoft Azure Service Bus**; recherchez simplement `WindowsAzure.Messaging.Managed` et ajoutez-le à votre projet.

    ![Capture d’écran de la boîte de dialogue Gérer les packages NuGet avec le package WindowsAzure.Messaging.Managed mis en évidence.](./media/notification-hubs-geofence/vs-nuget.png)
8. À des fins de test, recréez le gestionnaire d’événements `MainPage_Loaded` , puis ajoutez-lui cet extrait de code :

    ```csharp
    var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

    var hub = new NotificationHub("HUB_NAME", "HUB_LISTEN_CONNECTION_STRING");
    var result = await hub.RegisterNativeAsync(channel.Uri);

    // Displays the registration ID so you know it was successful
    if (result.RegistrationId != null)
    {
        Debug.WriteLine("Reg successful.");
    }
    ```

    Le code inscrit l’application auprès du hub de notification. Vous avez terminé !
9. Dans `LocationHelper`, à l’intérieur du gestionnaire `Geolocator_PositionChanged`, vous pouvez ajouter un extrait de code de test qui force l’emplacement à l’intérieur des limites de la clôture virtuelle :

    ```csharp
    await LocationHelper.SendLocationToBackend("wns", "TEST_USER", "TEST", "37.7746", "-122.3858");
    ```

10. Étant donné que vous ne transmettez pas les coordonnées réelles (qui ne se trouvent peut-être pas dans les limites actuellement) et que vous utilisez des valeurs de test prédéfinies, vous verrez une notification s’afficher lors de la mise à jour :

    ![Capture d’écran d’un poste de travail Windows affichant le message TEST.](./media/notification-hubs-geofence/notification-hubs-test-notification.png)

## <a name="next-steps"></a>Étapes suivantes

Vous devrez peut-être suivre quelques étapes supplémentaires pour vous assurer que la solution est prête pour la production.

1. Tout d’abord, assurez-vous que la clôture virtuelle est dynamique. Cette opération nécessite des actions supplémentaires avec l’API Bing pour pouvoir télécharger les nouvelles limites au sein de la source de données existante. Pour plus d’informations, consultez la [documentation de l’API des services de données spatiales Bing](/bingmaps/spatial-data-services/).
2. Ensuite, vous pouvez cibler les participants appropriés par [balisage](notification-hubs-tags-segment-push-message.md)pour garantir une remise correcte.

La solution indiquée dans ce didacticiel décrit un scénario dans lequel vous disposez d’un large choix de plateformes cibles. La clôture virtuelle n’a donc pas été limitée à des fonctionnalités spécifiques du système. Ceci dit, la plateforme Windows universelle offre des fonctionnalités [prêtes à l’emploi pouvant détecter les clôtures virtuelles](/windows/uwp/maps-and-location/set-up-a-geofence).
