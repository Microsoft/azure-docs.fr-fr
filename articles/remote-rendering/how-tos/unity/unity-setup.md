---
title: Configurer Remote Rendering pour Unity
description: Comment initialiser Azure Remote Rendering dans un projet Unity
author: jakrams
ms.author: jakras
ms.date: 02/27/2020
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 4a0be44d8709726e159e17e703566c6c576bc18f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89018975"
---
# <a name="set-up-remote-rendering-for-unity"></a>Configurer Remote Rendering pour Unity

Pour activer Azure Remote Rendering (ARR) dans Unity, nous fournissons des méthodes dédiées qui prennent en charge certains aspects propres à Unity.

## <a name="startup-and-shutdown"></a>Démarrage et arrêt

Pour initialiser Remote Rendering, utilisez `RemoteManagerUnity`. Cette classe effectue des appels dans le `RemoteManager` générique, mais implémente déjà des détails spécifiques à Unity pour vous. Par exemple, Unity utilise un système de coordonnées spécifique. Lors d’un appel à `RemoteManagerUnity.Initialize`, la convention appropriée est configurée. L’appel nécessite également que vous fournissiez l’appareil photo Unity qui doit être utilisé pour afficher le contenu rendu à distance.

```cs
// initialize Azure Remote Rendering for use in Unity:
// it needs to know which camera is used for rendering the scene
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);
```

Pour arrêter Remote Rendering, appelez `RemoteManagerStatic.ShutdownRemoteRendering()`.

Quand une `AzureSession` a été créée et choisie comme session de rendu principale, elle doit être inscrite auprès de `RemoteManagerUnity` :

```cs
RemoteManagerUnity.CurrentSession = ...
```

### <a name="full-example-code"></a>Exemple de code complet

Le code ci-dessous illustre toutes les étapes nécessaires à l’initialisation d’Azure Remote Rendering dans Unity :

```cs
// initialize Remote Rendering
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);

// create a frontend
AzureFrontendAccountInfo accountInfo = new AzureFrontendAccountInfo();
// ... fill out accountInfo ...
AzureFrontend frontend = new AzureFrontend(accountInfo);

// start a session
AzureSession session = await frontend.CreateNewRenderingSessionAsync(new RenderingSessionCreationParams(RenderingSessionVmSize.Standard, 0, 30)).AsTask();

// let RemoteManagerUnity know about the session we want to use
RemoteManagerUnity.CurrentSession = session;

session.ConnectToRuntime(new ConnectToRuntimeParams());

/// When connected, load and modify content

RemoteManagerStatic.ShutdownRemoteRendering();
```

## <a name="convenience-functions"></a>Fonctions pratiques

### <a name="session-state-events"></a>Événements d’état de session

`RemoteManagerUnity.OnSessionUpdate` émet des événements quand son état de session change. Consultez la documentation sur le code pour plus d’informations.

### <a name="arrserviceunity"></a>ARRServiceUnity

`ARRServiceUnity` est un composant facultatif permettant de simplifier la configuration et la gestion des sessions. Il inclut des options permettant d’arrêter automatiquement la session à la fermeture de l’application ou du mode lecture de l’éditeur. Par ailleurs, il renouvelle automatiquement le bail de session si nécessaire. Il met en cache des données telles que les propriétés de session (reportez-vous à sa variable `LastProperties`) et expose des événements pour les changements d’état de session et les erreurs de session.

Il ne peut pas y avoir plus d’une instance de `ARRServiceUnity` à la fois. Ce composant vous permet de commencer plus rapidement grâce à l’implémentation de fonctionnalités courantes. Pour une application plus grande, il peut vous être plus avantageux d’effectuer ces tâches vous-même.

Pour obtenir un exemple de configuration et d’utilisation de `ARRServiceUnity`, consultez le [Tutoriel : Affichage de modèles rendus à distance](../../tutorials/unity/view-remote-models/view-remote-models.md).

## <a name="next-steps"></a>Étapes suivantes

* [Installer le package Remote Rendering pour Unity](install-remote-rendering-unity-package.md)
* [Tutoriel : Affichage de modèles rendus à distance](../../tutorials/unity/view-remote-models/view-remote-models.md)
