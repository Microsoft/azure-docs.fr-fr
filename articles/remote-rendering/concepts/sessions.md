---
title: Sessions Remote Rendering
description: Explique ce qu’est une session Remote Rendering
author: jakrams
ms.author: jakras
ms.date: 02/21/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 8f2adc846247c4f06c9356f482501fd01c5463bf
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202682"
---
# <a name="remote-rendering-sessions"></a>Sessions Remote Rendering

Dans Azure Remote Rendering (ARR), une *session* est un concept clé. Cet article explique précisément ce qu’est une session.

## <a name="overview"></a>Vue d’ensemble

Azure Remote Rendering fonctionne en déplaçant les tâches de rendu complexes dans le cloud. Ces tâches de rendu ne peuvent pas être exécutées par n’importe quel serveur, car la plupart des serveurs cloud n’ont pas de GPU. Compte tenu de la quantité de données impliquées et de la nécessité incontournable de produire des résultats à des fréquences d’images interactives, la responsabilité du choix des serveurs devant traiter les différentes demandes des utilisateurs ne peut non plus être confiée à une autre machine à la volée, ce qui est possible pour le trafic web plus courant.

Ainsi, quand vous utilisez Azure Remote Rendering, un serveur cloud possédant les capacités matérielles nécessaires doit être exclusivement réservé au traitement de vos demandes de rendu. Une *session* fait référence à tout ce qu’implique l’interaction avec ce serveur. Elle commence par la demande initiale de réservation (*bail*) de la machine à utiliser, se poursuit avec toutes les commandes de chargement et de manipulation de modèles et se termine par la libération du bail sur le serveur cloud.

## <a name="managing-sessions"></a>Gestion des sessions

Il existe plusieurs façons de gérer les sessions et d’interagir avec celles-ci. L’[API REST de gestion de session](../how-tos/session-rest-api.md) offre un moyen indépendant du langage de créer, de mettre à jour et d’arrêter des sessions. En C# et C++, ces opérations sont exposées par le biais des classes `AzureFrontend` et `AzureSession`. Pour les applications Unity, d’autres fonctions utilitaires sont fournies par le composant `ARRServiceUnity`.

Quand vous êtes *connecté* à une session active, les opérations telles que le [chargement de modèles](models.md) et l’interaction avec la scène sont exposées par le biais de la classe `AzureSession`.

### <a name="managing-multiple-sessions-simultaneously"></a>Gestion simultanée de plusieurs sessions

Vous ne pouvez pas vous *connecter* complètement à plusieurs sessions à partir d’un même appareil. Toutefois, vous pouvez créer, observer et arrêter autant de sessions que vous le souhaitez à partir d’une même application. Tant que l’application n’est pas censée se connecter à une session, il n’est pas non plus nécessaire qu’elle s’exécute sur un appareil comme HoloLens 2. Le contrôle de sessions à partir d’un mécanisme central peut représenter un cas d’usage pour une telle implémentation. Par exemple, il est possible de créer une application web à laquelle plusieurs tablettes et appareils HoloLens peuvent se connecter. L’application peut ensuite afficher des options sur les tablettes comme le modèle de CAO à afficher. Si un utilisateur effectue une sélection, ces informations sont communiquées à tous les appareils HoloLens pour créer une expérience partagée.

## <a name="session-phases"></a>Phases d’une session

Chaque session se compose de plusieurs phases.

### <a name="session-startup"></a>Démarrage de la session

Quand vous demandez à ARR de [créer une session](../how-tos/session-rest-api.md#create-a-session), il commence par retourner un [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) de session. Cet UUID vous permet de demander des informations sur la session. L’UUID est conservé pendant 30 jours de même que certaines informations de base sur la session. Vous pouvez donc demander ces informations même après l’arrêt de la session. À ce stade, l’**état de la session** indiqué est **Démarrage**.

Ensuite, Azure Remote Rendering tente de trouver un serveur qui peut héberger votre session. Deux paramètres sont pris en compte pour cette recherche. Tout d’abord, ARR ne réserve que des serveurs de votre [région](../reference/regions.md). En effet, la latence du réseau entre régions peut être trop élevée pour garantir une expérience correcte. Le deuxième facteur est la *taille* souhaitée que vous avez spécifiée. Chaque région inclut un nombre limité de serveurs pouvant répondre à une demande de taille [*Standard*](../reference/vm-sizes.md) ou [*Premium*](../reference/vm-sizes.md). Ainsi, si tous les serveurs de la taille demandée sont en cours d’utilisation dans votre région, la création de la session échoue. Vous pouvez [demander](../how-tos/session-rest-api.md#get-sessions-properties) la raison de l’échec.

> [!IMPORTANT]
> Si vous demandez une taille de serveur *Standard* et que la demande échoue en raison d’une forte sollicitation, cela ne signifie pas qu’une demande de serveur *Premium* échouera également. Si vous disposez de cette option, vous pouvez donc essayer de revenir à un serveur *Premium*.

Quand le service trouve un serveur adéquat, il doit copier la machine virtuelle appropriée sur celui-ci pour en faire un hôte Azure Remote Rendering. Cette opération prend plusieurs minutes. Ensuite, la machine virtuelle est démarrée et l’**état de la session** devient **Prêt**.

À ce stade, le serveur attend votre entrée uniquement. C’est également à partir de ce moment que vous êtes facturé pour le service.

### <a name="connecting-to-a-session"></a>Connexion à une session

Quand la session est à l’état *Prêt*, vous pouvez vous y *connecter*. Quand vous êtes connecté, l’appareil peut envoyer des commandes pour charger et modifier des modèles. Chaque hôte ARR ne sert qu’un appareil client à la fois. Ainsi, quand un client se connecte à une session, il dispose d’un contrôle exclusif sur le contenu rendu. Ceci signifie également que les performances de rendu ne varient jamais pour des raisons échappant à votre contrôle.

> [!IMPORTANT]
> Même si un seul client peut *se connecter* à une session, les informations de base sur les sessions (comme leur état actuel) peuvent être demandées sans connexion.

Quand un appareil est connecté à une session, les tentatives de connexion des autres appareils échouent. Toutefois, dès lors que l’appareil connecté se déconnecte (volontairement ou à la suite d’une défaillance quelconque), la session accepte une autre demande de connexion. L’état précédent (modèles chargés, etc.) est supprimé, et le prochain appareil qui se connecte bénéficie d’un environnement propre. Les sessions peuvent donc être réutilisées plusieurs fois par différents appareils, et, dans la plupart des cas, il est possible de rendre la surcharge de démarrage de session imperceptible pour l’utilisateur final.

> [!IMPORTANT]
> Le serveur distant ne modifie jamais l’état des données côté client. Toutes les mutations de données (mises à jour de transformation et demandes de chargement, par exemple) doivent être effectuées par l’application cliente. Toutes les actions mettent immédiatement à jour l’état du client.

### <a name="session-end"></a>Fin de la session

Quand vous demandez une nouvelle session, vous spécifiez une *durée de bail maximale*, en général dans une plage d’une à huit heures. Il s’agit de la durée pendant laquelle l’hôte acceptera votre entrée.

Il existe deux raisons courantes pour lesquelles une session peut prendre fin : vous demandez manuellement l’arrêt de la session ou la durée de bail maximale a expiré. Dans les deux cas, toute connexion active à l’hôte est immédiatement fermée et le service est arrêté sur ce serveur. Le serveur est alors réintégré au pool Azure et peut être réservé à d’autres fins. L’arrêt d’une session ne peut pas être annulé. Quand vous demandez l’**état de la session** sur une session arrêtée, la demande retourne **Arrêté** ou **Expiré** selon que la session a été arrêtée manuellement ou que la durée de bail maximale a été atteinte.

Une session peut également être arrêtée en raison d’une défaillance.

Dans tous les cas, la facturation prend fin dès lors qu’une session est arrêtée.

> [!WARNING]
> Le fait que vous soyez connecté ou non à une session (et pendant combien de temps) n’affecte pas la facturation. Ce que vous payez pour le service dépend de la *durée de la session* (c’est-à-dire de la durée pendant laquelle un serveur vous est exclusivement réservé) et des capacités matérielles demandées ([taille allouée](../reference/vm-sizes.md)). Si vous démarrez une session, que vous vous connectez pendant cinq minutes et que vous n’arrêtez pas la session (qui continue de s’exécuter jusqu’à ce que son bail expire), vous serez facturé pour toute la durée de bail de la session. La *durée de bail maximale* peut, quant à elle, être considérée comme un filet de sécurité. Vous pouvez demander une session avec une durée de bail de huit heures, puis l’utiliser seulement pendant cinq minutes. Ceci n’a pas d’importance si vous arrêtez la session manuellement par la suite.

#### <a name="extend-a-sessions-lease-time"></a>Prolonger la durée du bail d’une session

Vous pouvez [prolonger la durée du bail](../how-tos/session-rest-api.md#modify-and-query-session-properties) d’une session active si vous en avez besoin plus longtemps.

## <a name="example-code"></a>Exemple de code

Le code ci-dessous illustre une implémentation simple incluant le démarrage d’une session, l’attente de l’état *Prêt*, la connexion, la déconnexion et l’arrêt.

```cs
RemoteRenderingInitialization init = new RemoteRenderingInitialization();
// fill out RemoteRenderingInitialization parameters...

RemoteManagerStatic.StartupRemoteRendering(init);

AzureFrontendAccountInfo accountInfo = new AzureFrontendAccountInfo();
// fill out accountInfo details...

AzureFrontend frontend = new AzureFrontend(accountInfo);

RenderingSessionCreationParams sessionCreationParams = new RenderingSessionCreationParams();
// fill out sessionCreationParams...

AzureSession session = await frontend.CreateNewRenderingSessionAsync(sessionCreationParams).AsTask();

RenderingSessionProperties sessionProperties;
while (true)
{
    sessionProperties = await session.GetPropertiesAsync().AsTask();
    if (sessionProperties.Status != RenderingSessionStatus.Starting &&
        sessionProperties.Status != RenderingSessionStatus.Unknown)
    {
        break;
    }
    // REST calls must not be issued too frequently, otherwise the server returns failure code 429 ("too many requests"). So we insert the recommended delay of 10s
    await Task.Delay(TimeSpan.FromSeconds(10));
}

if (sessionProperties.Status != RenderingSessionStatus.Ready)
{
    // Do some error handling and either terminate or retry.
}

// Connect to server
Result connectResult = await session.ConnectToRuntime(new ConnectToRuntimeParams()).AsTask();

// Connected!

while(...)
{
    // per frame update

    session.Actions.Update();
}

// Disconnect
session.DisconnectFromRuntime();

// stop the session
await session.StopAsync().AsTask();

// shut down the remote rendering SDK
RemoteManagerStatic.ShutdownRemoteRendering();
```

Vous pouvez gérer, manipuler et interroger plusieurs instances `AzureFrontend` et `AzureSession` à partir du code. Toutefois, un seul appareil peut se connecter à une instance `AzureSession` à la fois.

La durée de vie d’une machine virtuelle n’est pas liée à l’instance `AzureFrontend` ou à l’instance `AzureSession`. `AzureSession.StopAsync` doit être appelé pour arrêter une session.

L’ID de session persistant peut être demandé avec `AzureSession.SessionUUID()` et mis en cache localement. Avec cet ID, une application peut appeler `AzureFrontend.OpenSession` pour se lier à cette session.

Quand `AzureSession.IsConnected` a la valeur true, `AzureSession.Actions` retourne une instance de `RemoteManager`, qui contient les fonctions permettant de [charger des modèles](models.md), de manipuler des [entités](entities.md) et de [demander des informations](../overview/features/spatial-queries.md) sur la scène rendue.

## <a name="api-documentation"></a>Documentation de l’API

* [AzureSession, classe C#](/dotnet/api/microsoft.azure.remoterendering.azuresession)
* [AzureFrontend.CreateNewRenderingSessionAsync(), C#](/dotnet/api/microsoft.azure.remoterendering.azurefrontend.createnewrenderingsessionasync)
* [AzureFrontend.OpenRenderingSession(), C#](/dotnet/api/microsoft.azure.remoterendering.azurefrontend.openrenderingsession)
* [AzureSession, classe C++](/cpp/api/remote-rendering/azuresession)
* [AzureFrontend::CreateNewRenderingSessionAsync, C++](/cpp/api/remote-rendering/azurefrontend#createnewrenderingsessionasync)
* [AzureFrontend::OpenRenderingSession, C++](/cpp/api/remote-rendering/azurefrontend#openrenderingsession)

## <a name="next-steps"></a>Étapes suivantes

* [Entités](entities.md)
* [Modèles](models.md)