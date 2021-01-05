---
title: Configurer des points de terminaison Web (préversion)
titleSuffix: Azure Cognitive Services
description: configurer des points de terminaison Web pour des commandes personnalisées
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: e50d7aba5cc5b3d5d620d844cc9ad169ad8b3bf6
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025889"
---
# <a name="set-up-web-endpoints"></a>Configurer des points de terminaison Web

Dans cet article, vous apprendrez à configurer des points de terminaison web dans une application Commandes personnalisées vous permettant d’effectuer des requêtes HTTP à partir d’une application cliente. Vous allez effectuer les étapes suivantes :

- Configurer des points de terminaison Web dans une application Commandes personnalisées
- Appeler des points de terminaison Web dans une application Commandes personnalisées
- Recevoir la réponse des points de terminaison Web 
- Intégrer la réponse des points de terminaison Web dans une charge utile JSON personnalisée, l’envoyer, puis la visualiser dans une application cliente du Kit de développement logiciel (SDK) UWP C#

## <a name="prerequisites"></a>Prérequis
> [!div class = "checklist"]
> * [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)
> * Clé d'abonnement Azure pour le service Speech : [Obtenez-en une gratuitement](overview.md#try-the-speech-service-for-free) ou créez-la sur le [portail Azure](https://portal.azure.com)
> * Application [Commandes personnalisées](quickstart-custom-commands-application.md) créée précédemment
> * Application cliente compatible avec le kit de développement logiciel (SDK) Speech : [Guide pratique : Envoyer une activité à l’application cliente](./how-to-custom-commands-setup-speech-sdk.md)

## <a name="setup-web-endpoints"></a>Configurer des points de terminaison Web

1. Ouvrez l'application Commandes personnalisées que vous avez précédemment créée. 
1. Accédez à « Points de terminaison Web », puis cliquez sur « Nouveau point de terminaison Web ».

   > [!div class="mx-imgBorder"]
   > ![Nouveau point de terminaison Web](media/custom-commands/setup-web-endpoint-new-endpoint.png)

   | Paramètre | Valeur suggérée | Description |
   | ------- | --------------- | ----------- |
   | Nom | UpdateDeviceState | Nom du point de terminaison Web. |
   | URL | https://webendpointexample.azurewebsites.net/api/DeviceState | URL du point de terminaison avec laquelle votre application Commandes personnalisées doit communiquer. |
   | Méthode | POST | Interactions autorisées (par exemple GET, POST) avec votre point de terminaison.|
   | headers | Clé : application, Valeur : prendre les 8 premiers chiffres de votre applicationId | Paramètres d’en-tête à inclure dans l’en-tête de la demande.|

    > [!NOTE]
    > - Exemple de point de terminaison Web créé à l’aide d'[Azure Function](../../azure-functions/index.yml), qui se connecte à la base de données enregistrant l’état du téléviseur et du ventilateur
    > - L’en-tête suggéré n’est nécessaire que pour l’exemple de point de terminaison
    > - Pour vous assurer que la valeur de l’en-tête est unique dans notre exemple de point de terminaison, prenez les 8 premiers chiffres de votre applicationId
    > - Dans le monde réel, le point de terminaison Web peut être le point de terminaison du [hub IoT](../../iot-hub/about-iot-hub.md) qui gère vos appareils

1. Cliquez sur **Enregistrer**.

## <a name="call-web-endpoints"></a>Appeler des points de terminaison Web

1. Accédez à la commande **TurnOnOff**, sélectionnez **ConfirmationResponse** dans la règle d’exécution, puis **Add an action**.
1. Sous **New Action-Type**, sélectionnez **Call web endpoint**
1. Dans **Edit Action - Endpoints**, sélectionnez **UpdateDeviceState**, qui correspond au point de terminaison Web que nous avons créé.  
1. Dans **Configuration**, entrez les valeurs suivantes : 
   > [!div class="mx-imgBorder"]
   > ![Appeler les paramètres d’action des points de terminaison Web](media/custom-commands/setup-web-endpoint-edit-action-parameters.png)

   | Paramètre | Valeur suggérée | Description |
   | ------- | --------------- | ----------- |
   | Points de terminaison | UpdateDeviceState | Point de terminaison Web que vous souhaitez appeler dans cette action. |
   | Paramètres de requête | item={SubjectDevice}&&value={OnOff} | Paramètres de requête à ajouter à l’URL du point de terminaison Web.  |
   | Type de contenu | N/A | Contenu du corps de la demande. |

    > [!NOTE]
    > - Les paramètres de la demande suggérés sont uniquement nécessaires pour l’exemple de point de terminaison

1. Dans **On Success - Action to execute**, sélectionnez **Send speech response**.
    
    Dans **Éditeur simple**, entrez `{SubjectDevice} is {OnOff}`.
   
   > [!div class="mx-imgBorder"]
   > ![Capture d’écran montrant l’écran « On Success – Action to execute » (« Réussite – Action à effectuer »).](media/custom-commands/setup-web-endpoint-edit-action-on-success-send-response.png)

   | Paramètre | Valeur suggérée | Description |
   | ------- | --------------- | ----------- |
   | Action à exécuter | Envoyer une réponse vocale | Action à exécuter si la demande envoyée au point de terminaison Web a abouti |
   
   > [!NOTE]
   > - Vous pouvez également accéder directement aux champs de la réponse http en utilisant `{YourWebEndpointName.FieldName}`. Par exemple : `{UpdateDeviceState.TV}`

1. Dans **On Failure - Action to execute**, sélectionnez **Send speech response**

    Dans **Éditeur simple**, entrez `Sorry, {WebEndpointErrorMessage}`.

   > [!div class="mx-imgBorder"]
   > ![Action Appeler les points de terminaison Web en cas d’échec](media/custom-commands/setup-web-endpoint-edit-action-on-fail.png)

   | Paramètre | Valeur suggérée | Description |
   | ------- | --------------- | ----------- |
   | Action à exécuter | Envoyer une réponse vocale | Action à exécuter en cas d’échec de la demande sur le point de terminaison Web |

   > [!NOTE]
   > - `{WebEndpointErrorMessage}` est facultatif. Vous êtes libre de supprimer cet élément si vous ne souhaitez pas afficher de message d’erreur.
   > - Dans notre exemple de point de terminaison, nous renvoyons une réponse HTTP avec des messages d’erreur détaillés pour les erreurs courantes telles que des paramètres d’en-tête manquants. 

### <a name="try-it-out-in-test-portal"></a>Tester dans le portail test
- Réponse en cas de succès\
Enregistrer, effectuer l’apprentissage et tester
   > [!div class="mx-imgBorder"]
   > ![Capture d’écran montrant l’écran « On Success response » (« Réponse en cas de réussite »).](media/custom-commands/setup-web-endpoint-on-success-response.png)
- Réponse en cas d’échec\
Supprimer l’un des paramètres de la demande, enregistrer, effectuer un nouvel apprentissage et tester
   > [!div class="mx-imgBorder"]
   > ![Action Appeler les points de terminaison Web en cas de réussite](media/custom-commands/setup-web-endpoint-on-fail-response.png)

## <a name="integrate-with-client-application"></a>Intégrer à l’application cliente

Dans [Guide pratique : Envoyer l’activité à l’application cliente (préversion)](./how-to-custom-commands-send-activity-to-client.md), vous avez ajouté une action **Envoyer l’activité au client**. L’activité est envoyée à l’application cliente, que l’action **Appeler le point de terminaison Web** ait réussi ou non.
Toutefois, dans la plupart des cas, vous souhaitez uniquement envoyer l’activité à l’application cliente lorsque l’appel au point de terminaison Web réussit. Cet exemple illustre un scénario où l’état de l’appareil est correctement mis à jour.

1. Supprimez l’action **Envoyer l’activité au client** que vous avez ajoutée précédemment.
1. Modifiez le point de terminaison Web de l’appel : 
    1. Dans **Configuration**, vérifiez que le paramètre **Query Parameters** affiche `item={SubjectDevice}&&value={OnOff}`
    1. Dans **On Success**, définissez **Action to execute** sur **Send activity to client**
    1. Copiez le fichier JSON ci-dessous dans **Contenu de l'activité**
   ```json
   {
      "type": "event",
      "name": "UpdateDeviceState",
      "value": {
        "state": "{OnOff}",
        "device": "{SubjectDevice}"
      }
    }
   ```
   
À présent, vous envoyez l’activité au client uniquement lorsque la demande au point de terminaison Web est réussie.

### <a name="create-visuals-for-syncing-device-state"></a>Créer des éléments visuels pour la synchronisation de l’état de l’appareil
Ajoutez le code XML suivant à `MainPage.xaml` au-dessus du bloc `"EnableMicrophoneButton"`.

```xml
<Button x:Name="SyncDeviceStateButton" Content="Sync Device State"
        Margin="0,10,10,0" Click="SyncDeviceState_ButtonClicked"
        Height="35"/>
<Button x:Name="EnableMicrophoneButton" ......
        .........../>
```

### <a name="sync-device-state"></a>Synchroniser l’état de l’appareil 

Dans `MainPage.xaml.cs`, ajoutez la référence `using Windows.Web.Http;`. Ajoutez le code suivant à la classe `MainPage` . Cette méthode enverra une requête GET à l’exemple de point de terminaison, puis extraira l’état actuel de l’appareil pour votre application. Veillez à modifier `<your_app_name>` selon ce que vous avez utilisé dans l’**en-tête** du point de terminaison Web Commande personnalisée

```C#
private async void SyncDeviceState_ButtonClicked(object sender, RoutedEventArgs e)
{
    //Create an HTTP client object
    var httpClient = new HttpClient();

    //Add a user-agent header to the GET request. 
    var your_app_name = "<your-app-name>";

    Uri endpoint = new Uri("https://webendpointexample.azurewebsites.net/api/DeviceState");
    var requestMessage = new HttpRequestMessage(HttpMethod.Get, endpoint);
    requestMessage.Headers.Add("app", $"{your_app_name}");

    try
    {
        //Send the GET request
        var httpResponse = await httpClient.SendRequestAsync(requestMessage);
        httpResponse.EnsureSuccessStatusCode();
        var httpResponseBody = await httpResponse.Content.ReadAsStringAsync();
        dynamic deviceState = JsonConvert.DeserializeObject(httpResponseBody);
        var TVState = deviceState.TV.ToString();
        var FanState = deviceState.Fan.ToString();
        await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(
            CoreDispatcherPriority.Normal,
            () =>
            {
                State_TV.Text = TVState;
                State_Fan.Text = FanState;
            });
    }
    catch (Exception ex)
    {
        NotifyUser(
            $"Unable to sync device status: {ex.Message}");
    }
}
```

## <a name="try-it-out"></a>Faites un essai

1. Lancer l’application
1. Cliquez sur Synchroniser l’état de l’appareil. \
Si vous avez testé l’application avec `turn on tv` dans la section précédente, vous verrez que la télévision affiche l’état « on » (activée).
    > [!div class="mx-imgBorder"]
    > ![Synchroniser l’état de l’appareil](media/custom-commands/setup-web-endpoint-sync-device-state.png)
1. Sélectionner Activer le microphone
1. Cliquer sur le bouton Parler
1. Dire `turn on the fan`
1. L’état visuel du ventilateur doit passer à « on » (activé)
    > [!div class="mx-imgBorder"]
    > ![Activer le ventilateur](media/custom-commands/setup-web-endpoint-turn-on-fan.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Exporter une application Commandes personnalisées en tant que compétence distante](./how-to-custom-commands-integrate-remote-skills.md)

