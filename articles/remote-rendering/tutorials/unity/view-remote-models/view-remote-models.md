---
title: Affichage d’un modèle rendu à distance
description: Le tutoriel « Hello World » d’Azure Remote Rendering montre comment afficher un modèle rendu à distance par Azure
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 834df29597abaaadad98b232ce75b32a6431cfc2
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574732"
---
# <a name="tutorial-viewing-a-remotely-rendered-model"></a>Tutoriel : Affichage d’un modèle rendu à distance

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Provisionner une instance Azure Remote Rendering (ARR)
> * Créer et arrêter une session de rendu
> * Réutiliser une session de rendu existante
> * Se connecter et se déconnecter de sessions
> * Charger des modèles dans une session de rendu

## <a name="prerequisites"></a>Prérequis

Pour ce tutoriel, vous avez besoin des éléments suivants :

* Un abonnement Azure actif avec paiement à l’utilisation [Créer un compte](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/)
* Le SDK Windows 10.0.18362.0 [(télécharger)](https://developer.microsoft.com/windows/downloads/windows-10-sdk)
* Installez la dernière version de Visual Studio 2019 [(télécharger)](https://visualstudio.microsoft.com/vs/older-downloads/)
* Git [(télécharger)](https://git-scm.com/downloads)
* Unity dans sa version la plus récente (2019.3) ; nous vous recommandons d’utiliser Unity Hub [(télécharger)](https://unity3d.com/get-unity/download)
  * Installez les modules ci-dessous dans Unity :
    * **UWP** : prise en charge de la build de la plateforme Windows universelle
    * **IL2CPP** : prise en charge de la build Windows (IL2CPP)
* Connaissance intermédiaire de Unity et du langage C# (par exemple : création de scripts et d’objets, utilisation de prefabs, configuration des événements Unity, etc.)

## <a name="provision-an-azure-remote-rendering-arr-instance"></a>Provisionner une instance Azure Remote Rendering (ARR)

Pour pouvoir accéder au service Azure Remote Rendering, vous devez d’abord [créer un compte](../../../how-tos/create-an-account.md#create-an-account).

## <a name="create-a-new-unity-project"></a>Créer un projet Unity

> [!TIP]
> Le [dépôt d’exemples ARR](https://github.com/Azure/azure-remote-rendering) contient un projet dans lequel tous les tutoriels ont été effectués ; il peut servir de référence. Le projet Unity complet se trouve dans *Unity\Tutorial-Complete*.

Dans Unity Hub, créez un projet.
Dans cet exemple, nous partons du principe que le projet est créé dans un dossier nommé **RemoteRendering**.

:::image type="content" source="./media/unity-new-project.PNG" alt-text="Nouveau projet Unity":::

## <a name="include-the-azure-remote-rendering-package"></a>Inclure le package Azure Remote Rendering

Vous devez modifier le fichier `Packages/manifest.json` qui se trouve dans le dossier de votre projet Unity. Ouvrez le fichier dans un éditeur de texte et ajoutez les lignes suivantes dans la partie supérieure de votre manifeste :

```json
{
    "scopedRegistries": [
    {
        "name": "Azure Mixed Reality Services",
        "url": "https://api.bintray.com/npm/microsoft/AzureMixedReality-NPM/",
        "scopes": ["com.microsoft.azure"]
    }
    ],
    "dependencies": {
        "com.unity.render-pipelines.universal": "7.3.1",
        "com.microsoft.azure.remote-rendering": "0.1.31",
        ...existing dependencies...
    }
}
```

Après avoir modifié et enregistré le manifeste, Unity s’actualise automatiquement. Vérifiez que les packages ont été chargés dans la fenêtre *Project* (Projet) :

:::image type="content" source="./media/confirm-packages.png" alt-text="vérifier les importations de packages":::

Si vos packages ne se chargent pas, vérifiez s’il y a des erreurs dans votre console Unity. Si ce n’est pas le cas mais que les packages ne figurent toujours pas dans le dossier **Packages**, cochez le bouton bascule de visibilité des packages.\
![Capture d’écran avec une flèche pointant vers le bouton bascule de visibilité des packages.](./media/unity-package-visibility.png)

## <a name="ensure-you-have-the-latest-version-of-the-package"></a>Vérifiez que vous avez la version la plus récente du package.

Les étapes suivantes permettent de vérifier que votre projet utilise la version la plus récente du package Remote Rendering.

1. Dans le menu supérieur de l’éditeur Unity, ouvrez *Window -> Package Manager* (Fenêtre -> Gestionnaire de packages).
1. Sélectionnez le package **Microsoft Azure Remote Rendering**.
1. Dans la page Package Manager (Gestionnaire de packages), vérifiez que le bouton **Update** (Mettre à jour) est disponible pour le package **Microsoft Azure Remote Rendering**. Si c’est le cas, cliquez dessus pour mettre à jour le package vers la dernière version disponible :\
![Le package ARR dans le Gestionnaire de package](./media/package-manager.png)
1. La mise à jour du package peut parfois occasionner des erreurs dans la console. Si cela se produit, essayez de fermer, puis de rouvrir le projet.
1. Quand le package est à jour, le Gestionnaire de packages affiche **Up to date** (À jour) à la place du bouton Update (Mettre à jour).\
![Package à jour](./media/package-up-to-date.png)
## <a name="configure-the-camera"></a>Configurer la caméra

1. Sélectionner le nœud **Main Camera** (Caméra principale).

1. Ouvrez le menu contextuel en cliquant avec le bouton droit sur le composant *Transform* (Transformation), puis sélectionnez l’option **Reset** (Réinitialiser) :

    ![Réinitialisation de la transformation de la caméra](./media/camera-reset-transform.png)

1. Définissez **Clear flags** (Effacer les indicateurs) sur *Solid Color* (Couleur unie).

1. Définissez **Background** (Arrière-plan) sur *Black* (Noir) (#000000) en sélectionnant la valeur d’alpha (A) complètement transparente (0).

    ![Roue chromatique](./media/color-wheel-black.png)

1. Définissez **Clipping Planes** (Plans de découpage) sur *Near = 0.3* (Proche = 0,3) et *Far = 20* (Lointain = 20). Cela signifie que le rendu découpera la géométrie qui est plus proche que 30 cm ou plus lointaine que 20 mètres.

    ![Propriétés de la caméra Unity](./media/camera-properties.png)

## <a name="adjust-the-project-settings"></a>Ajuster les paramètres du projet

1. Ouvrez *Edit >Project Settings...* (Modifier > Paramètres du projet).
1. Sélectionnez **Quality** (Qualité) dans le menu de liste de gauche.
1. Définissez **Default Quality Level** (Niveau de qualité par défaut) de toutes les plateformes sur *Low* (Bas). Ce paramètre assure un rendu plus efficace du contenu local et n’affecte pas la qualité du contenu rendu à distance.

    ![Modifier les paramètres de qualité du projet](./media/settings-quality.png)

1. Sélectionnez **Graphics** (Graphiques) dans le menu de liste de gauche.
1. Définissez le paramètre **Scriptable Rendering Pipeline** (Pipeline de rendu pouvant contenir des scripts) sur *HybridRenderingPipeline*.\
    ![Capture d’écran qui pointe vers le paramètre Scriptable Rendering Pipeline (Pipeline de rendu scriptable) à définir sur HybridRenderingPipeline.](./media/settings-graphics-render-pipeline.png)\
    Parfois, l’interface utilisateur ne remplit pas la liste des types de pipeline disponibles à partir des packages. Si cela se produit, vous devez faire glisser la ressource *HybridRenderingPipeline* manuellement vers le champ :\
    ![Modification des paramètres graphiques du projet](./media/hybrid-rendering-pipeline.png)

    > [!NOTE]
    > Si vous ne parvenez pas à glisser-déplacer la ressource *HybridRenderingPipeline* vers le champ Render Pipeline Asset (peut-être parce que le champ n’existe pas !), vérifiez que votre configuration de package contient le package `com.unity.render-pipelines.universal`.

1. Sélectionnez **Player** (Lecteur) dans le menu de liste de gauche.
1. Sélectionnez l’onglet **Universal Windows Platform settings** (Paramètres de la plateforme Windows universelle), représenté par une icône Windows.
1. Modifiez les paramètres **XR Settings** pour prendre en charge Windows Mixed Reality, comme indiqué ci-dessous :
    1. Activez **Virtual Reality Supported** (Réalité virtuelle prise en charge).
    1. Appuyez sur le bouton « + » et ajoutez **Windows Mixed Reality**.
    1. Définissez **Depth Format**(Format de profondeur) sur *16-Bit Depth* (Profondeur de 16 bits).
    1. Vérifiez que **Depth Buffer Sharing** (Partage de la mémoire tampon de profondeur) est activé.
    1. Définissez **Stereo Rendering Mode** (Mode de rendu stéréo) sur *Single Pass Instanced* (Instance à passe unique).

    ![Paramètres du lecteur](./media/xr-player-settings.png)

1. Dans la même fenêtre, au-dessus de **XR Settings**, développez **Publishing Settings** (Paramètres de publication).
1. Faites défiler jusqu’à **Capabilities** (Fonctionnalités), puis sélectionnez :
    * **InternetClient**
    * **InternetClientServer**
    * **SpatialPerception**
    * **PrivateNetworkClientServer** (*facultatif*). Sélectionnez cette option si vous souhaitez connecter le débogueur distant Unity à votre appareil.

1. Sous **Supported Device Families** (Familles d’appareils prises en charge), activez **Holographic** (Holographique) et **Desktop** (Bureau).
1. Fermez ou ancrez le panneau **Project Settings** (Paramètres du projet).
1. Ouvrez *File->Build Settings* (Fichier->Paramètres de génération).
1. Sélectionnez **Universal Windows Platform** (Plateforme Windows universelle).
1. Configurez vos paramètres tels qu’ils sont définis ci-dessous.
1. Appuyez sur le bouton **Switch Platform** (Changer de plateforme).\
![Paramètres de génération](./media/build-settings.png)
1. Une fois que Unity a changé de plateforme, fermez le panneau de génération.

## <a name="validate-project-setup"></a>Valider la configuration du projet

Effectuez les étapes suivantes pour vérifier que les paramètres du projet sont corrects.

1. Choisissez l’entrée **ValidateProject** dans le menu **RemoteRendering** de la barre d’outils de l’éditeur Unity.
1. Vérifiez s’il y a des erreurs dans la fenêtre **ValidateProject** et corrigez les paramètres du projet, si nécessaire.

    ![Validation du projet de l’éditeur Unity](./media/remote-render-unity-validation.png)

## <a name="create-a-script-to-coordinate-azure-remote-rendering-connection-and-state"></a>Créer un script pour coordonner la connexion et l’état d’Azure Remote Rendering

L’affichage des modèles rendus à distance se fait en quatre phases, comme le montre le diagramme de flux ci-dessous. Chaque phase doit être effectuée dans l’ordre. L’étape suivante consiste à créer un script qui gérera l’état de l’application et passera par chaque phase obligatoire.

![Pile 0 ARR](./media/remote-render-stack-0.png)

1. Dans le volet *Project* (Projet), sous **Assets** (Ressources), créez un dossier sous le nom *RemoteRenderingCore*. Ensuite, à l’intérieur de *RemoteRenderingCore*, créez un autre dossier sous le nom *Scripts*.

1. Créez un [nouveau script C#](https://docs.unity3d.com/Manual/CreatingAndUsingScripts.html) sous le nom **RemoteRenderingCoordinator**.
Votre programme doit se présenter comme suit :

    ![Hiérarchie du projet](./media/project-structure.png)

    Ce script coordinateur suit et gère l’état du rendu à distance. Il est à noter qu’une partie de ce code sert à gérer l’état, à exposer les fonctionnalités à d’autres composants, à déclencher des événements et à stocker des données propres à une application qui ne sont pas *directement* liées à Azure Remote Rendering. Utilisez le code ci-dessous comme point de départ et nous aborderons et implémenterons le code propre à Azure Remote Rendering plus loin dans ce tutoriel.

1. Ouvrez **RemoteRenderingCoordinator** dans votre éditeur de code et remplacez tout son contenu par le code ci-dessous :

```csharp
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License. See LICENSE in the project root for license information.

using Microsoft.Azure.RemoteRendering;
using Microsoft.Azure.RemoteRendering.Unity;
using System;
using System.Linq;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;

#if UNITY_WSA
using UnityEngine.XR.WSA;
#endif

/// <summary>
/// Remote Rendering Coordinator is the controller for all Remote Rendering operations.
/// </summary>

// Require the GameObject with a RemoteRenderingCoordinator to also have the ARRServiceUnity component
[RequireComponent(typeof(ARRServiceUnity))]
public class RemoteRenderingCoordinator : MonoBehaviour
{
    public enum RemoteRenderingState
    {
        NotSet,
        NotInitialized,
        NotAuthorized,
        NoSession,
        ConnectingToExistingRemoteSession,
        ConnectingToNewRemoteSession,
        RemoteSessionReady,
        ConnectingToRuntime,
        RuntimeConnected
    }

    public static RemoteRenderingCoordinator instance;

    // AccountDomain must be '<region>.mixedreality.azure.com' - if no '<region>' is specified, connections will fail
    // The list of regions is available at https://docs.microsoft.com/azure/remote-rendering/reference/regions
    [SerializeField]
    private string accountDomain = "westus2.mixedreality.azure.com";
    public string AccountDomain
    {
        get => accountDomain.Trim();
        set => accountDomain = value;
    }

    [Header("Development Account Credentials")]
    [SerializeField]
    private string accountId = "<enter your account id here>";
    public string AccountId {
        get => accountId.Trim();
        set => accountId = value;
    }

    [SerializeField]
    private string accountKey = "<enter your account key here>";
    public string AccountKey {
        get => accountKey.Trim();
        set => accountKey = value;
    }

    // These settings are important. All three should be set as low as possible, while maintaining a good user experience
    // See the documentation around session management and the technical differences in session VM size
    [Header("New Session Defaults")]

    public RenderingSessionVmSize renderingSessionVmSize = RenderingSessionVmSize.Standard;
    public uint maxLeaseHours = 0;
    public uint maxLeaseMinutes = 20;

    [Header("Other Configuration")]

    [Tooltip("If you have a known active SessionID, you can fill it in here before connecting")]
    [SerializeField]
    private string sessionIDOverride;
    public string SessionIDOverride {
        get => sessionIDOverride.Trim();
        set => sessionIDOverride = value;
    }

    // When Automatic Mode is true, the coordinator will attempt to automatically proceed through the process of connecting and loading a model
    public bool automaticMode = true;

    public event Action RequestingAuthorization;
    public UnityEvent OnRequestingAuthorization = new UnityEvent();

    public event Action AuthorizedChanged;
    public UnityEvent OnAuthorizationChanged = new UnityEvent();
    private bool authorized;
    public bool Authorized
    {
        get => authorized;
        set
        {
            if (value == true) //This is a one-way value, once we're authorized it lasts until the app is shutdown.
            {
                authorized = value;
                AuthorizedChanged?.Invoke();
            }
        }
    }

    public delegate Task<AzureFrontendAccountInfo> AccountInfoGetter();

    public static AccountInfoGetter ARRCredentialGetter
    {
        private get;
        set;
    }

    private RemoteRenderingState currentCoordinatorState = RemoteRenderingState.NotSet;
    public RemoteRenderingState CurrentCoordinatorState
    {
        get => currentCoordinatorState;
        private set
        {
            if (currentCoordinatorState != value)
            {
                currentCoordinatorState = value;
                Debug.Log($"State changed to: {currentCoordinatorState}");
                CoordinatorStateChange?.Invoke(currentCoordinatorState);
            }
        }
    }

    public static event Action<RemoteRenderingState> CoordinatorStateChange;

    public static AzureSession CurrentSession => instance?.ARRSessionService?.CurrentActiveSession;

    private ARRServiceUnity arrSessionService;

    private ARRServiceUnity ARRSessionService
    {
        get
        {
            if (arrSessionService == null)
                arrSessionService = GetComponent<ARRServiceUnity>();
            return arrSessionService;
        }
    }

    private async Task<AzureFrontendAccountInfo> GetDevelopmentCredentials()
    {
        Debug.LogWarning("Using development credentials! Not recommended for production.");
        return await Task.FromResult(new AzureFrontendAccountInfo(AccountDomain, AccountId, AccountKey));
    }

    /// <summary>
    /// Keep the last used SessionID, when launching, connect to this session if its available
    /// </summary>
    private string LastUsedSessionID
    {
        get
        {
            if (!string.IsNullOrEmpty(SessionIDOverride))
                return SessionIDOverride;

            if (PlayerPrefs.HasKey("LastUsedSessionID"))
                return PlayerPrefs.GetString("LastUsedSessionID");
            else
                return null;
        }
        set
        {
            PlayerPrefs.SetString("LastUsedSessionID", value);
        }
    }

    public void Awake()
    {
        //Forward events to Unity events
        RequestingAuthorization += () => OnRequestingAuthorization?.Invoke();
        AuthorizedChanged += () => OnAuthorizationChanged?.Invoke();

        //Attach to event
        AuthorizedChanged += RemoteRenderingCoordinator_AuthorizedChanged;

        if (instance == null)
            instance = this;
        else
            Destroy(this);

        CoordinatorStateChange += AutomaticMode;

        CurrentCoordinatorState = RemoteRenderingState.NotInitialized;
    }

    private void RemoteRenderingCoordinator_AuthorizedChanged()
    {
        if (CurrentCoordinatorState != RemoteRenderingState.NotAuthorized)
            return; //This isn't valid from any other state than NotAuthorized


        //We just became authorized to connect to Azure
        InitializeSessionService();
    }

    /// <summary>
    /// Automatic mode attempts to automatically progress through the connection and loading steps. Doesn't handle error states.
    /// </summary>
    /// <param name="currentState">The current state</param>
    private async void AutomaticMode(RemoteRenderingState currentState)
    {
        if (!automaticMode)
            return;

        //Add a small delay for visual effect
        await Task.Delay(1500);
        switch (currentState)
        {
            case RemoteRenderingState.NotInitialized:
                InitializeARR();
                break;
            case RemoteRenderingState.NotAuthorized:
                RequestAuthorization();
                break;
            case RemoteRenderingState.NoSession:
                JoinRemoteSession();
                break;
            case RemoteRenderingState.RemoteSessionReady:
                ConnectRuntimeToRemoteSession();
                break;
        }
    }

    /// <summary>
    /// Initializes ARR, associating the main camera
    /// Note: This must be called on the main Unity thread
    /// </summary>
    public void InitializeARR()
    {
        //Implement me
    }

    /// <summary>
    /// Create a new remote session manager
    /// If the ARRCredentialGetter is set, use it as it, otherwise use the development credentials 
    /// </summary>
    public async void InitializeSessionService()
    {
        //Implement me
    }

    /// <summary>
    /// Trigger the event for checking authorization, respond to this event by prompting the user for authentication
    /// If authorized, set Authorized = true
    /// </summary>
    public void RequestAuthorization()
    {
        RequestingAuthorization?.Invoke();
    }

    public void BypassAuthorization()
    {
        Authorized = true;
    }

    /// <summary>
    /// Attempts to join an existing session or start a new session
    /// </summary>
    public async void JoinRemoteSession()
    {
        //Implement me
    }

    public void StopRemoteSession()
    {
        //Implement me
    }

    private async Task<bool> IsSessionAvailable(string sessionID)
    {
        var allSessions = await ARRSessionService.Frontend.GetCurrentRenderingSessionsAsync().AsTask();
        return allSessions.Any(x => x.Id == sessionID && (x.Status == RenderingSessionStatus.Ready || x.Status == RenderingSessionStatus.Starting));
    }

    /// <summary>
    /// Connects the local runtime to the current active session, if there's a session available
    /// </summary>
    public void ConnectRuntimeToRemoteSession()
    {
        //Implement me
    }

    public void DisconnectRuntimeFromRemoteSession()
    {
        //Implement me
    }

    /// <summary>
    /// The session must have its runtime pump updated.
    /// The Actions.Update() will push messages to the server, receive messages, and update the frame-buffer with the remotely rendered content.
    /// </summary>
    private void LateUpdate()
    {
        ARRSessionService?.CurrentActiveSession?.Actions?.Update();
    }

    /// <summary>
    /// Loads a model into the remote session for rendering
    /// </summary>
    /// <param name="modelPath">The model's path</param>
    /// <param name="progress">A call back method that accepts a float progress value [0->1]</param>
    /// <param name="parent">The parent Transform for this remote entity</param>
    /// <returns>An awaitable Remote Rendering Entity</returns>
    public async Task<Entity> LoadModel(string modelPath, Transform parent = null, ProgressHandler progress = null)
    {
        //Implement me
        return null;
    }

    private async void OnRemoteSessionStatusChanged(ARRServiceUnity caller, AzureSession session)
    {
        var properties = await session.GetPropertiesAsync().AsTask();

        switch (properties.Status)
        {
            case RenderingSessionStatus.Error:
            case RenderingSessionStatus.Expired:
            case RenderingSessionStatus.Stopped:
            case RenderingSessionStatus.Unknown:
                CurrentCoordinatorState = RemoteRenderingState.NoSession;
                break;
            case RenderingSessionStatus.Starting:
                CurrentCoordinatorState = RemoteRenderingState.ConnectingToNewRemoteSession;
                break;
            case RenderingSessionStatus.Ready:
                CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
                break;
        }
    }

    private void OnLocalRuntimeStatusChanged(ConnectionStatus status, Result error)
    {
        switch (status)
        {
            case ConnectionStatus.Connected:
                CurrentCoordinatorState = RemoteRenderingState.RuntimeConnected;
                break;
            case ConnectionStatus.Connecting:
                CurrentCoordinatorState = RemoteRenderingState.ConnectingToRuntime;
                break;
            case ConnectionStatus.Disconnected:
                CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
                break;
        }
    }
}
```

## <a name="create-the-azure-remote-rendering-gameobject"></a>Créer le GameObject Azure Remote Rendering

Le coordinateur de rendu à distance et son script obligatoire (*ARRServiceUnity*) sont tous deux des MonoBehaviours qui doivent être attachés à un GameObject de la scène. Le script *ARRServiceUnity* est fourni par ARR pour exposer la plupart des fonctionnalités d’ARR permettant de se connecter aux sessions à distance et de les gérer.

1. Créez un nouveau GameObject dans la scène [Ctrl+Maj+N ou *GameObject->Create Empty* (GameObject->Créer vide)] et nommez-le **RemoteRenderingCoordinator**.
1. Ajoutez le script *RemoteRenderingCoordinator* au GameObject **RemoteRenderingCoordinator**.\
![Ajouter le composant RemoteRenderingCoordinator](./media/add-coordinator-script.png)
1. Vérifiez que le script *ARRServiceUnity*, qui apparaît sous le nom *Service* dans l’inspecteur, est automatiquement ajouté au GameObject. Au cas où vous vous poseriez la question, il faut avoir `[RequireComponent(typeof(ARRServiceUnity))]` en haut du script **RemoteRenderingCoordinator**.
1. Ajoutez vos informations d’identification Azure Remote Rendering et le domaine de votre compte au script du coordinateur :\
![Ajouter vos informations d'identification](./media/configure-coordinator-script.png)

## <a name="initialize-azure-remote-rendering"></a>Initialiser Azure Remote Rendering

Maintenant que nous disposons de l’infrastructure pour notre coordinateur, nous allons implémenter chacune des quatre phases en commençant par **Initialiser Remote Rendering**.

![Pile 1 ARR](./media/remote-render-stack-1.png)

L’**initialisation** indique à Azure Remote Rendering quel objet camera utiliser pour le rendu et fait passer la machine à états à **NotAuthorized**. Cela signifie qu’il est initialisé mais pas encore autorisé à se connecter à une session. Sachant que le démarrage d’une session ARR induit des coûts, nous devons vérifier que l’utilisateur souhaite continuer.

Quand l’état passe à **NotAuthorized**, **CheckAuthorization**  est appelé, lequel invoque l’événement **RequestingAuthorization** et détermine quelles informations d’identification de compte utiliser (**AccountInfo** est défini en haut de la classe et utilise les informations d’identification que vous avez définies dans l’inspecteur Unity à l’étape précédente).

   > [!NOTE]
   > La recompilation du runtime n’est pas prise en charge par ARR. Si vous modifiez puis enregistrez le script pendant que le mode lecture est actif, Unity peut se figer et vous forcer à l’arrêter par le biais du gestionnaire des tâches. Veillez à toujours arrêter le mode lecture avant de modifier vos scripts.

1. Remplacez le contenu de **InitializeARR** et de **InitializeSessionService** par le code complet ci-dessous :

 ```csharp
/// <summary>
/// Initializes ARR, associating the main camera
/// Note: This must be called on the main Unity thread
/// </summary>
public void InitializeARR()
{
    RemoteManagerUnity.InitializeManager(new RemoteUnityClientInit(Camera.main));

    CurrentCoordinatorState = RemoteRenderingState.NotAuthorized;
}

/// <summary>
/// Create a new remote session manager
/// If the ARRCredentialGetter is set, use it as it, otherwise use the development credentials 
/// </summary>
public async void InitializeSessionService()
{
    if (ARRCredentialGetter == null)
        ARRCredentialGetter = GetDevelopmentCredentials;

    var accountInfo = await ARRCredentialGetter.Invoke();

    ARRSessionService.OnSessionStatusChanged += OnRemoteSessionStatusChanged;

    ARRSessionService.Initialize(accountInfo);

    CurrentCoordinatorState = RemoteRenderingState.NoSession;
}
```

Pour passer de **NotAuthorized** à **NoSession**, nous présentons généralement une boîte de dialogue modale à l’utilisateur pour lui permettre de choisir (nous ferons cela dans un autre chapitre). Pour le moment, nous allons contourner automatiquement la vérification d’autorisation en appelant **ByPassAuthentication** dès que l’événement **RequestingAuthorization** sera déclenché.

1. Sélectionnez le GameObject **RemoteRenderingCoordinator** et recherchez l’événement Unity **OnRequestingAuthorization** exposé dans l’inspecteur du composant **RemoteRenderingCoordinator**.

1. Ajoutez un nouvel événement en appuyant sur le signe « + » en bas à droite.
1. Faites glisser le composant jusqu’à son propre événement pour qu’il fasse référence à lui-même.\
![Contourner l’authentification](./media/bypass-authorization-add-event.png)\
1. Dans la liste déroulante, sélectionnez **RemoteRenderingCoordinator -> BypassAuthorization**.\
![Capture d’écran montrant l’option RemoteRenderingCoordinator.BypassAuthorization sélectionnée.](./media/bypass-authorization-event.png)

## <a name="create-or-join-a-remote-session"></a>Créer ou rejoindre une session à distance

La deuxième phase consiste à créer ou rejoindre une session Remote Rendering (pour plus d’informations, consultez [Sessions Remote Rendering](../../../concepts/sessions.md)).

![Pile 2 ARR](./media/remote-render-stack-2.png)

C’est dans la session à distance que le rendu des modèles se produit. La méthode **JoinRemoteSession( )** tente de rejoindre une session existante, suivie avec la propriété **LastUsedSessionID** ou s’il y a un ID de session active affecté sur **SessionIDOverride**. **SessionIDOverride** est destiné uniquement au débogage. Vous ne devez l’utiliser que si vous savez que la session existe et que vous voulez vous y connecter explicitement.

Si aucune session n’est disponible, une nouvelle session est créée. Cependant, la création d’une session est une opération fastidieuse. Par conséquent, vous ne devez essayer de créer des sessions qu’en cas d’absolue nécessité et les réutiliser dans la mesure du possible (consultez [Prêt pour la commercialisation : Regroupement de sessions, planification et bonnes pratiques](../commercial-ready/commercial-ready.md#fast-startup-time-strategies) pour plus d’informations sur la gestion des sessions).

> [!TIP]
> **StopRemoteSession ()** met fin à la session active. Pour éviter des frais inutiles, veillez à toujours arrêter les sessions dès que vous n’en n’avez plus besoin.

La machine à états passe maintenant à **ConnectingToNewRemoteSession** ou **ConnectingToExistingRemoteSession**, en fonction des sessions disponibles. L’ouverture d’une session existante ou la création d’une nouvelle session déclenchent tous deux l’événement **ARRSessionService.OnSessionStatusChanged**, en exécutant notre méthode **OnRemoteSessionStatusChanged**. Dans l’idéal, cela a pour effet de faire passer la machine à états à **RemoteSessionReady**.

1. Pour rejoindre une nouvelle session, modifiez le code en remplaçant les méthodes **JoinRemoteSession( )** et **StopRemoteSession( )** par les exemples complets ci-dessous :

```csharp
/// <summary>
/// Attempts to join an existing session or start a new session
/// </summary>
public async void JoinRemoteSession()
{
    //If there's a session available that previously belonged to us, and it's ready, use it. Otherwise start a new session.
    RenderingSessionProperties joinResult;
    if (await IsSessionAvailable(LastUsedSessionID))
    {
        CurrentCoordinatorState = RemoteRenderingState.ConnectingToExistingRemoteSession;
        joinResult = await ARRSessionService.OpenSession(LastUsedSessionID);
    }
    else
    {
        CurrentCoordinatorState = RemoteRenderingState.ConnectingToNewRemoteSession;
        joinResult = await ARRSessionService.StartSession(new RenderingSessionCreationParams(renderingSessionVmSize, maxLeaseHours, maxLeaseMinutes));
    }

    if (joinResult.Status == RenderingSessionStatus.Ready || joinResult.Status == RenderingSessionStatus.Starting)
    {
        LastUsedSessionID = joinResult.Id;
    }
    else
    {
        //The session should be ready or starting, if it's not, something went wrong
        await ARRSessionService.StopSession();
        if(LastUsedSessionID == SessionIDOverride)
            SessionIDOverride = "";
        CurrentCoordinatorState = RemoteRenderingState.NoSession;
    }
}

public void StopRemoteSession()
{
    if (ARRSessionService.CurrentActiveSession != null)
    {
        ARRSessionService.CurrentActiveSession.StopAsync();
    }
}
```

Si vous souhaitez gagner du temps en réutilisant des sessions, veillez à désactiver l’option **Auto-Stop Session** (Arrêter automatiquement la session) dans le composant *ARRServiceUnity*. Ne perdez pas de vue que les sessions resteront actives, même si personne n’y est connecté. Votre session peut s’exécuter pendant la durée définie dans *MaxLeaseTime* jusqu’à ce que le serveur l’arrête [la valeur de *MaxLeaseTime* peut être modifiée dans le composantRemote Rendering Coordinator , en dessous de *New Session Defaults* (Nouvelles valeurs par défaut de session)]. En revanche, si vous arrêtez automatiquement chaque session lors de la déconnexion, vous devrez à chaque fois attendre qu’une nouvelle session soit démarrée, ce qui peut prendre du temps.

> [!NOTE]
> L’arrêt d’une session prend effet immédiatement et ne peut pas être annulé. Une fois la session arrêtée, vous devez en créer une nouvelle, entraînant les mêmes frais de démarrage.

## <a name="connect-the-local-runtime-to-the-remote-session"></a>Connecter le runtime local à la session à distance

L’application doit ensuite connecter son runtime local à la session à distance.

![Pile 3 ARR](./media/remote-render-stack-3.png)

L’application doit aussi écouter les événements relatifs à la connexion entre le runtime et la session active ; ces changements d’état sont gérées dans **OnLocalRuntimeStatusChanged**. Ce code fait passer l’état à **ConnectingToRuntime**. Une fois la connexion établie, dans **OnLocalRuntimeStatusChanged**, l’état passe à **RuntimeConnected**. La connexion au runtime est le dernier état dont se soucie le coordinateur, ce qui signifie que la configuration commune de l’application est terminée et que celle-ci est prête à commencer le travail de session spécifique, à savoir charger les modèles et en assurer le rendu.

 1. Remplacez les méthodes **ConnectRuntimeToRemoteSession( )** et **DisconnectRuntimeFromRemoteSession( )** par les versions complètes ci-dessous.
 1. Il est important de constater la présence de la méthode Unity **LateUpdate** et qu’elle met à jour la session active du moment. Cela permet à la session actuelle d’envoyer/recevoir des messages et de mettre à jour la mémoire tampon de frames avec les frames reçues en provenance de la session à distance. Cela est essentiel au bon fonctionnement d’ARR.

```csharp
/// <summary>
/// Connects the local runtime to the current active session, if there's a session available
/// </summary>
public void ConnectRuntimeToRemoteSession()
{
    if (ARRSessionService == null || ARRSessionService.CurrentActiveSession == null)
    {
        Debug.LogError("Not ready to connect runtime");
        return;
    }

    //Connect the local runtime to the currently connected session
    //This session is set when connecting to a new or existing session

    ARRSessionService.CurrentActiveSession.ConnectionStatusChanged += OnLocalRuntimeStatusChanged;
    ARRSessionService.CurrentActiveSession.ConnectToRuntime(new ConnectToRuntimeParams());
    CurrentCoordinatorState = RemoteRenderingState.ConnectingToRuntime;
}

public void DisconnectRuntimeFromRemoteSession()
{
    if (ARRSessionService == null || ARRSessionService.CurrentActiveSession == null || ARRSessionService.CurrentActiveSession.ConnectionStatus != ConnectionStatus.Connected)
    {
        Debug.LogError("Runtime not connected!");
        return;
    }

    ARRSessionService.CurrentActiveSession.DisconnectFromRuntime();
    ARRSessionService.CurrentActiveSession.ConnectionStatusChanged -= OnLocalRuntimeStatusChanged;
    CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
}

/// <summary>
/// The session must have its runtime pump updated.
/// The Actions.Update() will push messages to the server, receive messages, and update the frame-buffer with the remotely rendered content.
/// </summary>
private void LateUpdate()
{
    ARRSessionService?.CurrentActiveSession?.Actions?.Update();
}
```

> [!NOTE]
> La connexion du runtime local à une session à distance est tributaire de l’appel de **Update** dans la session active du moment. Si vous constatez que l’état de votre application ne va jamais au-delà de **ConnectingToRuntime**, veillez à appeler régulièrement **Update** au niveau de la session active.

## <a name="load-a-model"></a>Charger un modèle

Dès lors que la fondation nécessaire est en place, vous êtes prêt à charger un modèle dans la session à distance et à commencer à recevoir des frames.

![Diagramme montrant le flux de processus de préparation du chargement et de l’affichage d’un modèle.](./media/remote-render-stack-4.png)

La méthode **LoadModel** est conçue pour accepter un chemin de modèle, un gestionnaire de progression et une transformation parente. Ces arguments serviront à charger un modèle dans la session à distance, à informer l’utilisateur de la progression du chargement et à orienter le modèle rendu à distance en fonction de la transformation parente.

1. Remplacez entièrement la méthode **LoadModel** par le code ci-dessous :

    ```csharp
    /// <summary>
    /// Loads a model into the remote session for rendering
    /// </summary>
    /// <param name="modelName">The model's path</param>
    /// <param name="parent">The parent Transform for this remote entity</param>
    /// <param name="progress">A call back method that accepts a float progress value [0->1]</param>
    /// <returns>An awaitable Remote Rendering Entity</returns>
    public async Task<Entity> LoadModel(string modelPath, Transform parent = null, ProgressHandler progress = null)
    {
        //Create a root object to parent a loaded model to
        var modelEntity = ARRSessionService.CurrentActiveSession.Actions.CreateEntity();

        //Get the game object representation of this entity
        var modelGameObject = modelEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);

        //Ensure the entity will sync its transform with the server
        var sync = modelGameObject.GetComponent<RemoteEntitySyncObject>();
        sync.SyncEveryFrame = true;

        //Parent the new object under the defined parent
        if (parent != null)
        {
            modelGameObject.transform.SetParent(parent, false);
            modelGameObject.name = parent.name + "_Entity";
        }

    #if UNITY_WSA
        //Anchor the model in the world, prefer anchoring parent if there is one
        if (parent != null)
        {
            parent.gameObject.AddComponent<WorldAnchor>();
        }
        else
        {
            modelGameObject.AddComponent<WorldAnchor>();
        }
    #endif

        //Load a model that will be parented to the entity
        var loadModelParams = new LoadModelFromSASParams(modelPath, modelEntity);
        var loadModelAsync = ARRSessionService.CurrentActiveSession.Actions.LoadModelFromSASAsync(loadModelParams);
        if(progress != null)
            loadModelAsync.ProgressUpdated += progress;
        var result = await loadModelAsync.AsTask();
        return modelEntity;
    }
    ```

Le code ci-dessus effectue les étapes suivantes :

1. Création d’une [entité distante](../../../concepts/entities.md).
1. Création d’un GameObject local pour représenter l’entité distante.
1. Configuration du GameObject local pour synchroniser son état (c.-à-d. Transformation) avec l’entité distante à chaque frame.
1. Définition d’un nom et ajout d’un [**WorldAnchor**](https://docs.unity3d.com/ScriptReference/XR.WSA.WorldAnchor.html) pour faciliter la stabilisation.
1. Chargement de données de modèle de Stockage Blob vers l’entité distante.
1. Retour de l’entité parente, pour référence ultérieure.

## <a name="view-the-test-model"></a>Afficher le modèle de test

Nous avons maintenant tout le code nécessaire pour afficher un modèle rendu à distance, les quatre phases nécessaires au rendu à distance étant terminées. Il nous faut à présent ajouter un peu de code pour lancer le processus de chargement de modèle.

![Pile 4 ARR](./media/remote-render-stack-5.png)

1. Ajoutez le code suivant à la classe **RemoteRenderingCoordinator**, par exemple en dessous de la méthode **LoadModel** :

    ```csharp
    private bool loadingTestModel = false;
    [ContextMenu("Load Test Model")]
    public async void LoadTestModel()
    {
        if(CurrentCoordinatorState != RemoteRenderingState.RuntimeConnected)
        {
            Debug.LogError("Please wait for the runtime to connect before loading the test model. Try again later.");
            return;
        }
        if(loadingTestModel)
        {
            Debug.Log("Test model already loading or loaded!");
            return;
        }
        loadingTestModel = true;
    
        // Create a parent object to use for positioning
        GameObject testParent = new GameObject("TestModelParent");
        testParent.transform.position = new Vector3(0f, 0f, 3f);
    
        // The 'built in engine path' is a special path that references a test model built into Azure Remote Rendering.
        await LoadModel("builtin://Engine", testParent.transform, (progressValue) => Debug.Log($"Loading Test Model progress: {Math.Round(progressValue * 100, 2)}%"));
    }
    ```
    
    Ce code crée un GameObject pour faire office de parent du modèle de test. Il appelle ensuite la méthode **LoadModel** pour charger le modèle « builtin://Engine », qui est une ressource intégrée à Azure Remote Rendering destinée à tester le rendu.

1. Enregistrez votre code.
1. Appuyez sur le bouton Play (Lecture) dans l’éditeur Unity pour lancer la connexion à Azure Remote Rendering et la création d’une nouvelle session.
1. Peu de choses seront visibles dans la vue Game (Jeu), mais celle-ci présentera l’état changeant de l’application. Il passera probablement à `ConnectingToNewRemoteSession` et n’évoluera peut-être plus pendant cinq minutes.
1. Sélectionnez le GameObject **RemoteRenderingCoordinator** pour voir les scripts qui lui sont attachés dans l’inspecteur. Observez le composant **Service** se mettre à jour à mesure qu’il progresse à travers ses étapes d’initialisation et de connexion.
1. Supervisez la sortie de la console, en attendant que l’état passe à **RuntimeConnected**.
1. Une fois le runtime connecté, cliquez avec le bouton droit sur le **RemoteRenderingCoordinator** dans l’inspecteur pour afficher le menu contextuel. Cliquez ensuite sur l’option **Load Test Model** (Charger le modèle de test) dans le menu contextuel, qui a été ajoutée par la partie `[ContextMenu("Load Test Model")]` de notre code ci-dessus.

    ![Charger à partir du menu contextuel](./media/load-test-model.png)

1. Recherchez dans la console la sortie de **ProgressHandler** que nous avons passée dans la méthode **LoadModel**.
1. Examinez le modèle rendu à distance !

> [!NOTE]
> Le modèle distant ne sera jamais visible dans la vue Scene, seulement dans la vue Game. Cela est dû au fait que ARR assure le rendu des frames à distance précisément pour la perspective de la caméra de la vue Game et ne détecte pas la caméra de l’éditeur (utilisée pour le rendu de la vue Scene).

## <a name="next-steps"></a>Étapes suivantes

![Modèle chargé](./media/test-model-rendered.png)

Félicitations ! Vous avez créé une application de base capable d’afficher les modèles rendus à distance utilisant Azure Remote Rendering. Dans le prochain tutoriel, nous allons intégrer MRTK et importer nos propres modèles.

> [!div class="nextstepaction"]
> [Étape suivante : Interfaces et modèles personnalisés](../custom-models/custom-models.md)
