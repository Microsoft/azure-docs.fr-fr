---
title: Assistants vocaux sur Windows - Lignes directrices de l’implémentation au-dessus du verrou
titleSuffix: Azure Cognitive Services
description: Les instructions permettant d’implémenter l’activation vocale et les fonctionnalités au-dessus du verrou pour une application de l’agent vocal.
services: cognitive-services
author: cfogg6
manager: trrwilson
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: travisw
ms.custom: devx-track-csharp
ms.openlocfilehash: 0503e0bf2fe152296ca6890e14503d05bd3bbeef
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024770"
---
# <a name="implementing-voice-assistants-on-windows"></a>Implémentation des assistants vocaux sur Windows

Ce guide présente des détails d’implémentation importants pour la création d’un assistant vocal sur Windows.

## <a name="implementing-voice-activation"></a>Implémentation de l’activation vocale

Une fois [votre environnement configuré](how-to-windows-voice-assistants-get-started.md) et lorsque vous avec découvert le [fonctionnement de l’activation vocale](windows-voice-assistants-overview.md#how-does-voice-activation-work), vous pouvez commencer à implémenter l’activation vocale pour votre propre application de l’assistant vocal.

### <a name="registration"></a>Inscription

#### <a name="ensure-that-the-microphone-is-available-and-accessible-then-monitor-its-state"></a>Assurez-vous que le microphone est disponible et accessible, puis analyser son état

MVA a besoin qu’un microphone présent et accessible pour pouvoir détecter une activation vocale. Utilisez les classes [AppCapability](/uwp/api/windows.security.authorization.appcapabilityaccess.appcapability?view=winrt-18362), [DeviceWatcher](/uwp/api/windows.devices.enumeration.devicewatcher?view=winrt-18362) et [MediaCapture](/uwp/api/windows.media.capture.mediacapture?view=winrt-18362) pour vérifier l’accès à la confidentialité du microphone, la présence d’appareils et l’état des appareils (par exemple, volume et muet), respectivement.

### <a name="register-the-application-with-the-background-service"></a>Enregistrer l'application auprès du service d'arrière-plan

Pour que MVA lance l’application en arrière-plan, l’application doit être enregistrée auprès du service d’arrière-plan. Consultez un guide complet pour l’enregistrement des services en arrière-plan [ici](/windows/uwp/launch-resume/register-a-background-task).

### <a name="unlock-the-limited-access-feature"></a>Déverrouiller la fonctionnalité d’accès limité

Utilisez votre clé de fonctionnalité d’accès limité fournie par Microsoft pour déverrouiller la fonctionnalité de l’assistant vocal. Pour ce faire, utilisez la classe [LimitedAccessFeature](/uwp/api/windows.applicationmodel.limitedaccessfeatures?view=winrt-18362) du SDK Windows.

### <a name="register-the-keyword-for-the-application"></a>Enregistrer le mot clé pour l’application

L’application doit s’enregistrer ainsi que son modèle de mot clé et sa langue auprès de Windows.

Commencez par récupérer le détecteur de mot clé. Dans cet exemple de code, nous récupérons le premier détecteur, mais vous pouvez sélectionner un détecteur en particulier en le sélectionnant dans `configurableDetectors`.

```csharp
private static async Task<ActivationSignalDetector> GetFirstEligibleDetectorAsync()
{
    var detectorManager = ConversationalAgentDetectorManager.Default;
    var allDetectors = await detectorManager.GetAllActivationSignalDetectorsAsync();
    var configurableDetectors = allDetectors.Where(candidate => candidate.CanCreateConfigurations
        && candidate.Kind == ActivationSignalDetectorKind.AudioPattern
        && (candidate.SupportedModelDataTypes.Contains("MICROSOFT_KWSGRAPH_V1")));

    if (configurableDetectors.Count() != 1)
    {
        throw new NotSupportedException($"System expects one eligible configurable keyword spotter; actual is {configurableDetectors.Count()}.");
    }

    var detector = configurableDetectors.First();

    return detector;
}
```

Après avoir récupéré l’objet ActivationSignalDetector, appelez sa méthode `ActivationSignalDetector.CreateConfigurationAsync` avec l’ID de signal, l’ID de modèle et le nom d’affichage pour enregistrer votre mot clé et récupérer les `ActivationSignalDetectionConfiguration` de votre application. Les ID de signal et de modèle doivent être des GUID décidés par le développeur et rester cohérents pour le même mot clé.

### <a name="verify-that-the-voice-activation-setting-is-enabled"></a>Vérifier que le paramètre d’activation vocale est activé

Pour utiliser l’activation vocale, un utilisateur doit l’activer pour son système et pour son application. Vous pouvez trouver le paramètre sous « Paramètres de confidentialité de l’activation vocale » dans les paramètres Windows. Pour vérifier l’état du paramètre d’activation vocale dans votre application, utilisez l’instance de la `ActivationSignalDetectionConfiguration` à partir de l’enregistrement du mot clé. Le champ [AvailabilityInfo](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-uwp/UWPVoiceAssistantSample/UIAudioStatus.cs#L128) de `ActivationSignalDetectionConfiguration` contient une valeur enum qui décrit l’état du paramètre d’activation vocale.

### <a name="retrieve-a-conversationalagentsession-to-register-the-app-with-the-mva-system"></a>Récupérer un ConversationalAgentSession pour enregistrer l’application auprès du système MVA

La `ConversationalAgentSession` est une classe du SDK Windows qui permet à votre application de mettre à jour Windows avec les événements d’état (inactif, détection, écoute, fonctionnement, parler) et de réception de l’application, telle que la détection d’activation et les modifications de l’état du système, tel que le verrouillage d’écran. La récupération d’une instance de l’AgentSession sert également à enregistrer l’application auprès de Windows comme étant activable par la voix. La meilleure pratique consiste à maintenir une référence sur la `ConversationalAgentSession`. Pour récupérer la session, utilisez l’API `ConversationalAgentSession.GetCurrentSessionAsync`.

### <a name="listen-to-the-two-activation-signals-the-onbackgroundactivated-and-onsignaldetected"></a>Écouter les deux signaux d’activation : OnBackgroundActivated et OnSignalDetected

Windows signale votre application lorsqu’elle détecte un mot clé de l’une des deux façons suivantes. Si l’application n’est pas active (autrement dit, si vous n’avez pas de référence à une instance non supprimée de `ConversationalAgentSession`), elle lance votre application et appelle la méthode OnBackgroundActivated dans le fichier App.xaml.cs de votre application. Si le champ`BackgroundActivatedEventArgs.TaskInstance.Task.Name` des arguments d’événements correspond à la chaîne "AgentBackgroundTrigger", le démarrage de l’application a été déclenché par l’activation vocale. L’application doit remplacer cette méthode et récupérer une instance de ConversationalAgentSession pour signaler à Windows qu’elle est maintenant active. Lorsque l’application est active, Windows signale l’occurrence d’une activation vocale à l’aide de l’événement `ConversationalAgentSession.OnSignalDetected`. Ajoutez un gestionnaire d’événements à cet événement dès que vous récupérez le `ConversationalAgentSession`.

## <a name="keyword-verification"></a>Vérification du mot clé

Une fois qu’une application d’agent vocal est activée par la voix, l’étape suivante consiste à vérifier que la détection du mot clé était valide. Windows ne fournit pas de solution de vérification par mot clé, mais il permet aux assistants vocaux d’accéder à l’audio à partir de l’activation hypothétique et termine la vérification par lui-même.

### <a name="retrieve-activation-audio"></a>Récupérer l’audio d’activation

Créez un [AudioGraph](/uwp/api/windows.media.audio.audiograph) et transmettez-le à la `CreateAudioDeviceInputNodeAsync` du `ConversationalAgentSession`. Cela chargera la mémoire tampon audio du graphique avec l’audio *démarrant approximativement 3 secondes avant que le mot clé ne soit détecté*. Cet autre audio de début est inclus pour tenir compte d’un large éventail de longueurs de mots clés et de vitesses de débit de parole de l’orateur. Gérez ensuite l’événement [QuantumStarted](/uwp/api/windows.media.audio.audiograph.quantumstarted?view=winrt-18362) à partir du graphique audio pour récupérer les données audio.

```csharp
var inputNode = await agentSession.CreateAudioDeviceInputNodeAsync(audioGraph);
var outputNode = inputGraph.CreateFrameOutputNode();
inputNode.AddOutgoingConnection(outputNode);
audioGraph.QuantumStarted += OnQuantumStarted;
```

Remarque : L’audio de début inclus dans la mémoire tampon audio peut provoquer l’échec de la vérification par mot clé. Pour résoudre ce problème, découpez le début de la mémoire tampon audio avant d’envoyer l’audio pour la vérification par mot clé. Ce découpage initial doit être adapté à chaque assistant.

### <a name="launch-in-the-foreground"></a>Lancer au premier plan

Lorsque la vérification par mot clé s’effectue correctement, l’application doit passer au premier plan pour afficher l’interface utilisateur. Appelez l’API `ConversationalAgentSession.RequestForegroundActivationAsync` pour déplacer votre application au premier plan.

### <a name="transition-from-compact-view-to-full-view"></a>Transition de la vue compacte vers la vue complète

La première fois que vous activez votre application par la voix, celle-ci se lance dans un affichage compact. Pour obtenir de l'aide sur les différents affichages et sur le passage de l'un à l'autre pour les assistants vocaux Windows, consultez les [Conseils de conception relatifs l'activation vocale (préversion)](windows-voice-assistants-best-practices.md#design-guidance-for-voice-activation-preview).

Pour passer de l'affichage compact à l'affichage complet de l'application, utilisez l'API applicationView `TryEnterViewModeAsync` :

```csharp
var appView = ApplicationView.GetForCurrentView();
await appView.TryEnterViewModeAsync(ApplicationViewMode.Default);
```

## <a name="implementing-above-lock-activation"></a>Implémentation de l’activation au-dessus du verrou

Les étapes suivantes décrivent la configuration requise pour permettre à un assistant vocal Windows de s’exécuter au-dessus du verrou, avec des références à un exemple de code et des instructions pour la gestion du cycle de vie de l’application.

Pour obtenir des conseils sur la conception des expériences au-dessus du verrou, consultez le [guide des meilleures pratiques](windows-voice-assistants-best-practices.md).

Quand une application affiche une vue au-dessus du verrou, elle est considérée comme étant en mode plein écran. Pour plus d’informations sur l’implémentation d’une application qui utilise le mode plein écran, consultez la [documentation en mode plein écran](/windows-hardware/drivers/partnerapps/create-a-kiosk-app-for-assigned-access).

### <a name="transitioning-above-lock"></a>Transition au-dessus du verrou

Une activation au-dessus du verrou est semblable à une activation en-dessous du verrou. S’il n’y a aucune instance active de l’application, une nouvelle instance est démarrée en arrière-plan et `OnBackgroundActivated` dans App.xaml.cs est appelé. S’il existe une instance de l’application, cette instance recevra une notification par le biais de l’événement `ConversationalAgentSession.SignalDetected`.

Si l’application n’est pas déjà affichée au-dessus du verrou, elle doit appeler `ConversationalAgentSession.RequestForegroundActivationAsync`. Cela déclenche la méthode `OnLaunched` dans App.xaml.cs qui doit accéder à la vue qui sera affichée au-dessus du verrou.

### <a name="detecting-lock-screen-transitions"></a>Détection des transitions de l’écran de verrouillage

La bibliothèque ConversationalAgent du SDK Windows fournit une API pour rendre l’état de l’écran de verrouillage et les modifications de l’état de l’écran de verrouillage facilement accessibles. Pour détecter l’état actuel de l’écran de verrouillage, examinez le champ `ConversationalAgentSession.IsUserAuthenticated`. Pour détecter les modifications de l’état de verrouillage, ajoutez un gestionnaire d’événements à l’événement `SystemStateChanged` de l’objet `ConversationalAgentSession`. Il se déclenche chaque fois que l’écran passe de déverrouillé à verrouillé, ou vice versa. Si la valeur des arguments d’événement est `ConversationalAgentSystemStateChangeType.UserAuthentication`, l’état de l’écran de verrouillage a changé.

```csharp
conversationalAgentSession.SystemStateChanged += (s, e) =>
{
    if (e.SystemStateChangeType == ConversationalAgentSystemStateChangeType.UserAuthentication)
    {
        // Handle lock state change
    }
};
```

### <a name="detecting-above-lock-activation-user-preference"></a>Détection de la préférence utilisateur d’activation au-dessus du verrou

L’entrée de l’application dans la page Paramètres de confidentialité de l’activation vocale offre un bouton bascule pour la fonctionnalité au-dessus du verrou. Pour que votre application puisse être lancée au-dessus du verrou, l’utilisateur doit activer ce paramètre. L’état des autorisations au-dessus du verrou est également stocké dans l’objet ActivationSignalDetectionConfiguration. L’État AvailabilityInfo.HasLockScreenPermission indique si l’utilisateur s’est vu accorder l’autorisation au-dessus du verrou. Si ce paramètre est désactivé, une application vocale peut inviter l’utilisateur à accéder à la page des paramètres appropriée au lien « ms-settings:privacy-voiceactivation », avec des instructions sur le mode d’activation au-dessus du verrou pour l’application.

## <a name="closing-the-application"></a>Fermeture de l'application

Pour fermer correctement l’application par programme alors qu’elle se trouve au-dessus ou au-dessous du verrou, utilisez l’API `WindowService.CloseWindow()`. Cela déclenche toutes les méthodes de cycle de vie UWP, y compris OnSuspend, permettant à l’application de supprimer son instance `ConversationalAgentSession` avant de se fermer.

> [!NOTE]
> L’application peut se fermer sans fermer l’[instance en-dessous du verrou](/windows-hardware/drivers/partnerapps/create-a-kiosk-app-for-assigned-access#add-a-way-out-of-assigned-access-). Dans ce cas, l’affichage au-dessus du verrou doit être « nettoyé », ce qui garantit qu’une fois l’écran déverrouillé, aucun gestionnaire d’événements ou aucune tâche ne tente de manipuler l’affichage au-dessus du verrou.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Visitez l’exemple d’application de l’Assistant vocal UWP pour obtenir des exemples et des procédures de code](windows-voice-assistants-faq.md#the-uwp-voice-assistant-sample)