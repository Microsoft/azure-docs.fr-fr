---
title: Envoyer une activité Commandes personnalisées à l’application cliente
titleSuffix: Azure Cognitive Services
description: Dans cet article, vous allez apprendre à envoyer une activité à partir d’une application Commandes personnalisées vers une application cliente exécutant le kit de développement logiciel (SDK) Speech.
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: 52a4dbc4ff01515af8cd7d2503877184a09f7e64
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566093"
---
# <a name="send-custom-commands-activity-to-client-application"></a>Envoyer une activité Commandes personnalisées à l’application cliente

Dans cet article, vous allez apprendre à envoyer une activité à partir d’une application Commandes personnalisées vers une application cliente exécutant le kit de développement logiciel (SDK) Speech.

Effectuez les tâches suivantes :

- Définir et envoyer une charge utile JSON personnalisée à partir de votre application de commandes personnalisées
- Recevoir et visualiser le contenu de la charge utile JSON personnalisée d’une application cliente du Kit de développement logiciel (SDK) Speech UWP C#

## <a name="prerequisites"></a>Prérequis
> [!div class = "checklist"]
> * [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) ou version ultérieure. Ce guide utilise Visual Studio 2019
> * Clé d'abonnement Azure pour le service Speech : [Obtenez-en une gratuitement](overview.md#try-the-speech-service-for-free) ou créez-la sur le [portail Azure](https://portal.azure.com)
> * Application [Commandes personnalisées](quickstart-custom-commands-application.md) créée précédemment
> * Application cliente compatible avec le kit de développement logiciel (SDK) Speech : [Guide pratique pour Intégration à une application cliente à l’aide du SDK Speech](./how-to-custom-commands-setup-speech-sdk.md)

## <a name="setup-send-activity-to-client"></a>Configurer Envoyer une activité au client 
1. Ouvrez l'application Commandes personnalisées que vous avez précédemment créée
1. Sélectionnez la commande **TurnOnOff**, **ConfirmationResponse** dans la règle d’exécution, puis **Add an action**
1. Sous **New Action-Type**, sélectionnez **Send activity to client**
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
1. Cliquez sur **Enregistrer** pour créer une nouvelle règle avec une action Envoyer l’activité, puis **Former** et **Publier** la modification

   > [!div class="mx-imgBorder"]
   > ![Règle d’exécution Send Activity (Activité d’envoi)](media/custom-commands/send-activity-to-client-completion-rules.png)

## <a name="integrate-with-client-application"></a>Intégrer à l’application cliente

Dans [Guide pratique : Configurer une application cliente avec le Kit de développement logiciel (SDK) Speech (préversion)](./how-to-custom-commands-setup-speech-sdk.md), vous avez créé une application cliente UWP avec le Kit de développement logiciel (SDK) Speech qui gérait des commandes telles que `turn on the tv`, `turn off the fan`. Avec l'ajout de quelques éléments visuels, vous pouvez voir le résultat de ces commandes.

Pour ajouter des zones étiquetées avec du texte qui indique **Activé** ou **Désactivé**, ajoutez le bloc XML suivant à `MainPage.xaml`.

```xml
<StackPanel Orientation="Vertical" H......>
......
</StackPanel>
<StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Margin="20">
    <Grid x:Name="Grid_TV" Margin="50, 0" Width="100" Height="100" Background="LightBlue">
        <StackPanel>
            <TextBlock Text="TV" Margin="0, 10" TextAlignment="Center"/>
            <TextBlock x:Name="State_TV" Text="off" TextAlignment="Center"/>
        </StackPanel>
    </Grid>
    <Grid x:Name="Grid_Fan" Margin="50, 0" Width="100" Height="100" Background="LightBlue">
        <StackPanel>
            <TextBlock Text="Fan" Margin="0, 10" TextAlignment="Center"/>
            <TextBlock x:Name="State_Fan" Text="off" TextAlignment="Center"/>
        </StackPanel>
    </Grid>
</StackPanel>
<MediaElement ....../>
```

### <a name="add-reference-libraries"></a>Ajouter des bibliothèques de référence

Comme vous avez créé une charge utile JSON, vous devez ajouter une référence à la bibliothèque [JSON.NET](https://www.newtonsoft.com/json) pour gérer la désérialisation.

1. Cliquez avec le bouton droit sur votre solution.
1. Sélectionnez **Gérer les packages NuGet de la solution**, puis **Parcourir** 
1. Si vous avez déjà installé **Newtonsoft.json**, assurez-vous que sa version est au moins 12.0.3. Si ce n’est pas le cas, accédez à **Gérer les packages NuGet pour la solution - Mises à jour** et recherchez **Newtonsoft.json** pour mettre la version à jour. Ce guide utilise la version 12.0.3.

    > [!div class="mx-imgBorder"]
    > ![Charge utile d’activité d’envoi](media/custom-commands/send-activity-to-client-json-nuget.png)

1. Veillez également à ce que le package NuGet **Microsoft.NETCore.UniversalWindowsPlatform** soit au moins à la version 6.2.10. Ce guide utilise la version 6.2.10.

Dans « MainPage.xaml.cs », ajoutez

```C#
using Newtonsoft.Json; 
using Windows.ApplicationModel.Core;
using Windows.UI.Core;
```

### <a name="handle-the-received-payload"></a>Gérer la charge utile reçue

Dans `InitializeDialogServiceConnector`, remplacez le gestionnaire d'événements `ActivityReceived` par le code suivant. Le gestionnaire d'événements `ActivityReceived` modifié extraira la charge utile de l'activité et modifiera l'état visuel du téléviseur ou du ventilateur, respectivement.

```C#
connector.ActivityReceived += async (sender, activityReceivedEventArgs) =>
{
    NotifyUser($"Activity received, hasAudio={activityReceivedEventArgs.HasAudio} activity={activityReceivedEventArgs.Activity}");

    dynamic activity = JsonConvert.DeserializeObject(activityReceivedEventArgs.Activity);
    var name = activity?.name != null ? activity.name.ToString() : string.Empty;

    if (name.Equals("UpdateDeviceState"))
    {
        Debug.WriteLine("Here");
        var state = activity?.value?.state != null ? activity.value.state.ToString() : string.Empty;
        var device = activity?.value?.device != null ? activity.value.device.ToString() : string.Empty;

        if (state.Equals("on") || state.Equals("off"))
        {
            switch (device)
            {
                case "tv":
                    await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(
                        CoreDispatcherPriority.Normal, () => { State_TV.Text = state; });
                    break;
                case "fan":
                    await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(
                        CoreDispatcherPriority.Normal, () => { State_Fan.Text = state; });
                    break;
                default:
                    NotifyUser($"Received request to set unsupported device {device} to {state}");
                    break;
            }
        }
        else { 
            NotifyUser($"Received request to set unsupported state {state}");
        }
    }

    if (activityReceivedEventArgs.HasAudio)
    {
        SynchronouslyPlayActivityAudio(activityReceivedEventArgs.Audio);
    }
};
```

## <a name="try-it-out"></a>Faites un essai

1. Lancer l’application
1. Sélectionner Activer le microphone
1. Cliquer sur le bouton Parler
1. Dire `turn on the tv`
1. L’état visuel du téléviseur doit passer à « Activé »
   > [!div class="mx-imgBorder"]
   > ![Screenshot that shows that the visual state of the T V is now on.](media/custom-commands/send-activity-to-client-turn-on-tv.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Guide pratique : Configurer des points de terminaison Web (préversion)](./how-to-custom-commands-setup-web-endpoints.md)
