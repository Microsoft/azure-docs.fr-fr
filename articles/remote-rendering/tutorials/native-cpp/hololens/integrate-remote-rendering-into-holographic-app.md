---
title: Intégrer Remote Rendering à une application holographique C++/DirectX 11
description: Explique comment intégrer Remote Rendering à une application holographique C++/DirectX 11 ordinaire créée à l’aide de l’Assistant de projet de Visual Studio
author: florianborn71
ms.author: flborn
ms.date: 05/04/2020
ms.topic: tutorial
ms.openlocfilehash: 56e889778e3b598dc4ded5f64eef20101c542b6a
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207510"
---
# <a name="tutorial-integrate-remote-rendering-into-a-hololens-holographic-app"></a>Tutoriel : Intégrer Remote Rendering dans une application holographique HoloLens

Ce didacticiel vous apprendra à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Utilisation de Visual Studio pour créer une application holographique qui peut être déployée sur HoloLens
> * Ajouter les extraits de code et paramètres de projet nécessaires pour combiner du rendu local avec du contenu rendu à distance

Ce tutoriel se concentre sur l’ajout des éléments nécessaires à un exemple `Holographic App` native pour combiner du rendu local avec Azure Remote Rendering. Le seul type de commentaires d’état dans cette application est disponible par le biais du panneau de sortie de débogage dans Visual Studio. Il est donc recommandé de démarrer l’exemple à partir de Visual Studio. L’ajout de commentaires dans l’application appropriés n’entre pas dans le cadre de cet exemple, car la création d’un volet de texte dynamique à partir de zéro implique beaucoup de codage. La classe `StatusDisplay`, qui fait partie de l’[exemple de projet Remoting Player sur GitHub](https://github.com/microsoft/MixedReality-HolographicRemoting-Samples/tree/master/player/common/Content), constitue un bon point de départ. En fait, la version prédéfinie de ce tutoriel utilise une copie locale de cette classe.

> [!TIP]
> Le [dépôt d’exemples ARR](https://github.com/Azure/azure-remote-rendering) contient le résultat de ce tutoriel sous la forme d’un projet Visual Studio prêt à être utilisé. Il est également enrichi avec les rapports d’erreur et d’état appropriés par le biais de la classe d’interface utilisateur `StatusDisplay`. Dans ce tutoriel, tous les ajouts spécifiques à ARR sont signalés avec `#ifdef USE_REMOTE_RENDERING` / `#endif`, il est donc facile d’identifier les ajouts Remote Rendering.

## <a name="prerequisites"></a>Prérequis

Pour ce tutoriel, vous avez besoin des éléments suivants :

* Les informations de votre compte (ID du compte, clé du compte, ID de l’abonnement). Si vous n’avez pas de compte, [créez-en un](../../../how-tos/create-an-account.md).
* Le SDK Windows 10.0.18362.0 [(télécharger)](https://developer.microsoft.com/windows/downloads/windows-10-sdk).
* La dernière version de Visual Studio 2019 [(télécharger)](https://visualstudio.microsoft.com/vs/older-downloads/).
* [Outils Visual Studio pour Mixed Reality](/windows/mixed-reality/install-the-tools). Plus précisément, les installations de *charge de travail* suivantes sont obligatoires :
  * **Développement Desktop en C++**
  * **Développement de la plateforme Windows universelle (UWP)**
* Les modèles d’application Windows Mixed Reality pour Visual Studio [(téléchargement)](https://marketplace.visualstudio.com/items?itemName=WindowsMixedRealityteam.WindowsMixedRealityAppTemplatesVSIX).

## <a name="create-a-new-holographic-app-sample"></a>Créer un exemple d’application holographique

Dans un premier temps, nous allons créer un exemple de stock qui est la base de l’intégration Remote Rendering. Ouvrez Visual Studio, sélectionnez « Créer un projet », puis recherchez « Application DirectX 11 holographique (Windows universel) (C++/WinRT) »

![Création d’un projet](media/new-project-wizard.png)

Tapez le nom du projet de votre choix, choisissez un chemin, puis sélectionnez le bouton « Créer ».
Dans le nouveau projet, passez à la configuration **« Debug/ARM64 »** . Vous devez maintenant être en mesure de le compiler et de le déployer sur un appareil HoloLens 2 connecté. Si vous l’exécutez sur HoloLens, un cube en rotation doit normalement s’afficher devant vous.

## <a name="add-remote-rendering-dependencies-through-nuget"></a>Ajouter des dépendances Remote Rendering par le biais de NuGet

La première étape de l’ajout de fonctionnalités Remote Rendering consiste à ajouter les dépendances côté client. Les dépendances pertinentes sont disponibles sous la forme d’un package NuGet.
Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet, puis sélectionnez **« Gérer les packages NuGet… »** dans le menu contextuel.

Dans la boîte de dialogue d’invite, recherchez le package NuGet nommé **« Microsoft.Azure.RemoteRendering.Cpp »**  :

![Rechercher le package NuGet](media/add-nuget.png)

et ajoutez-le au projet en sélectionnant le package, puis en appuyant sur le bouton « Installer ».

Le package NuGet ajoute les dépendances Remote Rendering au projet. Plus précisément :
* Lien en regard de la bibliothèque cliente (RemoteRenderingClient.lib).
* Configurer les dépendances .dll.
* Définir le chemin correct du répertoire include.

## <a name="project-preparation"></a>Préparation du projet

Nous devons apporter de légères modifications au projet existant. Ces modifications sont subtiles, mais elles sont indispensables pour que Remote Rendering fonctionne.

### <a name="enable-multithread-protection-on-directx-device"></a>Activer la protection multithread sur un appareil DirectX
La protection multithread doit être activée sur l’appareil `DirectX11`. Pour cela, ouvrez le fichier DeviceResources.cpp situé dans le dossier « Common », puis insérez le code suivant à la fin de la fonction `DeviceResources::CreateDeviceResources()` :

```cpp
// Enable multi thread protection as now multiple threads use the immediate context.
Microsoft::WRL::ComPtr<ID3D11Multithread> contextMultithread;
if (context.As(&contextMultithread) == S_OK)
{
    contextMultithread->SetMultithreadProtected(true);
}
```

### <a name="enable-network-capabilities-in-the-app-manifest"></a>Activer les fonctionnalités réseau dans le manifeste de l’application
Les fonctionnalités réseau doivent être explicitement activées pour l’application déployée. Si cela n’est pas configuré, les requêtes de connexion sont susceptibles d’entraîner des délais d’attente. Pour les activer, double-cliquez sur l’élément `package.appxmanifest` dans l’Explorateur de solutions. Dans l’interface utilisateur suivante, accédez à l’onglet **Fonctionnalités**, puis sélectionnez :
* Internet (client et serveur)
* Internet (client)

![Fonctionnalités réseau](media/appx-manifest-caps.png)


## <a name="integrate-remote-rendering"></a>Intégrer Remote Rendering

Maintenant que le projet est préparé, nous pouvons commencer avec le code. La classe `HolographicAppMain` (fichier HolographicAppMain.h/cpp) constitue un bon point de départ dans l’application car elle a tous les raccordements nécessaires pour l’initialisation, la désinitialisation et le rendu.

### <a name="includes"></a>Includes

Nous commençons par ajouter les includes nécessaires. Ajoutez l’instruction include suivante au fichier HolographicAppMain.h :

```cpp
#include <AzureRemoteRendering.h>
```

... et ces directives `include` supplémentaires au fichier HolographicAppMain.cpp :

```cpp
#include <AzureRemoteRendering.inl>
#include <RemoteRenderingExtensions.h>
#include <windows.perception.spatial.h>
```

Pour simplifier le code, nous définissons le raccourci d’espace de noms suivant en haut du fichier HolographicAppMain.h, après les directives `include` :

```cpp
namespace RR = Microsoft::Azure::RemoteRendering;
```

Ce raccourci est utile pour nous éviter d’avoir à écrire l’espace de noms complet partout, tout en nous permettant de toujours reconnaître les structures de données spécifique à ARR. Bien entendu, nous pourrions également utiliser la directive `using namespace...`.

### <a name="remote-rendering-initialization"></a>Initialisation de Remote Rendering
 
Nous devons conserver quelques objets pour la session pendant la durée de vie de l’application. Cette durée de vie coïncide avec la durée de vie de l’objet `HolographicAppMain` de l’application. Nous ajoutons donc nos objets en tant que membres à la classe `HolographicAppMain`. L’étape suivante consiste à ajouter les membres de classe suivants au fichier HolographicAppMain.h :

```cpp
class HolographicAppMain
{
    ...
    // members:
    std::string m_sessionOverride;                // if we have a valid session ID, we specify it here. Otherwise a new one is created
    RR::ApiHandle<RR::AzureFrontend> m_frontEnd;  // the front end instance
    RR::ApiHandle<RR::AzureSession> m_session;    // the current remote rendering session
    RR::ApiHandle<RR::RemoteManager> m_api;       // the API instance, that is used to perform all the actions. This is just a shortcut to m_session->Actions()
    RR::ApiHandle<RR::GraphicsBindingWmrD3d11> m_graphicsBinding; // the graphics binding instance
}
```

Le constructeur de la classe `HolographicAppMain` constitue un bon emplacement pour effectuer l’implémentation réelle. Nous devons effectuer trois types d’initialisation ici :
1. Initialisation unique du système Remote Rendering
1. Création front-end
1. Création de session

Nous effectuons toutes ces opérations de façon séquentielle dans le constructeur. Toutefois, dans les cas d’usage réels, il peut être approprié d’effectuer ces étapes séparément.

Ajoutez le code suivant au début du corps du constructeur dans le fichier HolographicAppMain.cpp :

```cpp
HolographicAppMain::HolographicAppMain(std::shared_ptr<DX::DeviceResources> const& deviceResources) :
    m_deviceResources(deviceResources)
{
    // 1. One time initialization
    {
        RR::RemoteRenderingInitialization clientInit;
        clientInit.connectionType = RR::ConnectionType::General;
        clientInit.graphicsApi = RR::GraphicsApiType::WmrD3D11;
        clientInit.toolId = "<sample name goes here>"; // <put your sample name here>
        clientInit.unitsPerMeter = 1.0f;
        clientInit.forward = RR::Axis::Z_Neg;
        clientInit.right = RR::Axis::X;
        clientInit.up = RR::Axis::Y;
        if (RR::StartupRemoteRendering(clientInit) != RR::Result::Success)
        {
            // something fundamental went wrong with the initialization
            throw std::exception("Failed to start remote rendering. Invalid client init data.");
        }
    }


    // 2. Create front end
    {
        // Users need to fill out the following with their account data and model
        RR::AzureFrontendAccountInfo init;
        init.AccountId = "00000000-0000-0000-0000-000000000000";
        init.AccountKey = "<account key>";
        init.AccountDomain = "westus2.mixedreality.azure.com"; // <change to your region>
        m_modelURI = "builtin://Engine";
        m_sessionOverride = ""; // If there is a valid session ID to re-use, put it here. Otherwise a new one is created

        m_frontEnd = RR::ApiHandle(RR::AzureFrontend(init));
    }

    // 3. Open/create rendering session
    {
        bool createNewSession = true;

        // If we had an old (valid) session that we can recycle, we call synchronous function m_frontEnd->OpenRenderingSession
        if (!m_sessionOverride.empty())
        {
            auto openSessionRes = m_frontEnd->OpenRenderingSession(m_sessionOverride);
            if (openSessionRes->valid())
            {
                SetNewSession(*openSessionRes);
                createNewSession = false;
            }
        }

        if (createNewSession)
        {
            // create a new session
            RR::RenderingSessionCreationParams init;
            init.MaxLease.minute = 10; // session is leased for 10 minutes
            init.Size = RR::RenderingSessionVmSize::Standard;
            auto createSessionAsync = *m_frontEnd->CreateNewRenderingSessionAsync(init);
            createSessionAsync->Completed([&](auto handler)
                {
                    if (handler->GetStatus() == RR::Result::Success)
                    {
                        SetNewSession(handler->GetResult());
                    }
                    else
                    {
                        SetNewState(AppConnectionStatus::ConnectionFailed, "failed");
                    }
                });
            SetNewState(AppConnectionStatus::CreatingSession, nullptr);
        }
    }

    // Rest of constructor code:
    ...
}
```

Le code appelle des fonctions membres `SetNewSession` et `SetNewState`, que nous allons implémenter dans le paragraphe suivant avec le reste du code machine à états.

Notez que les informations d’identification sont codées en dur dans l’exemple et qu’elles doivent être renseignées sur place ([ID de compte, clé de compte](../../../how-tos/create-an-account.md#retrieve-the-account-information) et [domaine](../../../reference/regions.md)).

Nous procédons à la désinitialisation de manière symétrique et dans l’ordre inverse à la fin du corps du destructeur :

```cpp
HolographicAppMain::~HolographicAppMain()
{
    // Existing destructor code:
    ...
    
    // Destroy session:
    if (m_session != nullptr)
    {
        m_session->DisconnectFromRuntime();
        m_session = nullptr;
    }

    // Destroy front end:
    m_frontEnd = nullptr;

    // One-time de-initialization:
    RR::ShutdownRemoteRendering();
}
```

## <a name="state-machine"></a>Machine d’état

Dans Remote Rendering, les principales fonctions permettant de créer une session et charger un modèle sont des fonctions asynchrones. Pour tenir compte de ce point, nous avons besoin d’une machine d’état simple qui, essentiellement, passe automatiquement par les états suivants :

*Initialisation -> Création de session -> Démarrage de session -> Chargement du modèle (avec la progression)*

En conséquence, lors de la prochaine étape, nous ajoutons un peu de gestion de la machine d’état à la classe. Nous déclarons notre propre énumération d’états `AppConnectionStatus` pour les différents états dans lesquels notre application peut se trouver. Elle est similaire à `RR::ConnectionStatus`, mais elle comporte un état supplémentaire pour la connexion ayant échoué.

Ajoutez les membres et fonctions suivants à la déclaration de classe :

```cpp
namespace HolographicApp
{
    // Our application's possible states:
    enum class AppConnectionStatus
    {
        Disconnected,

        CreatingSession,
        StartingSession,
        Connecting,
        Connected,

        // error state:
        ConnectionFailed,
    };

    class HolographicAppMain
    {
        ...
        // Member functions for state transition handling
        void OnConnectionStatusChanged(RR::ConnectionStatus status, RR::Result error);
        void SetNewState(AppConnectionStatus state, const char* statusMsg);
        void SetNewSession(RR::ApiHandle<RR::AzureSession> newSession);
        void StartModelLoading();

        // Members for state handling:

        // Model loading:
        std::string m_modelURI;
        RR::ApiHandle<RR::LoadModelAsync> m_loadModelAsync;

        // Connection state machine:
        AppConnectionStatus m_currentStatus = AppConnectionStatus::Disconnected;
        std::string m_statusMsg;
        RR::Result m_connectionResult = RR::Result::Success;
        RR::Result m_modelLoadResult = RR::Result::Success;
        bool m_isConnected = false;
        bool m_sessionStarted = false;
        RR::ApiHandle<RR::SessionPropertiesAsync> m_sessionPropertiesAsync;
        bool m_modelLoadTriggered = false;
        float m_modelLoadingProgress = 0.f;
        bool m_modelLoadFinished = false;
        double m_timeAtLastRESTCall = 0;
        bool m_needsCoordinateSystemUpdate = true;
    }
```

Du côté implémentation dans le fichier .cpp, ajoutez les corps de fonction suivants :

```cpp
void HolographicAppMain::StartModelLoading()
{
    m_modelLoadingProgress = 0.f;

    RR::LoadModelFromSASParams params;
    params.ModelUrl = m_modelURI.c_str();
    params.Parent = nullptr;

    // Start the async model loading
    if (auto loadModel = m_api->LoadModelFromSASAsync(params))
    {
        m_loadModelAsync = *loadModel;
        m_loadModelAsync->Completed([this](const RR::ApiHandle<RR::LoadModelAsync>& async)
        {
            m_modelLoadResult = async->GetStatus();
            m_modelLoadFinished = true; // successful if m_modelLoadResult==RR::Result::Success
            m_loadModelAsync = nullptr;
            char buffer[1024];
            sprintf_s(buffer, "Remote Rendering: Model loading completed. Result: %s\n", RR::ResultToString(m_modelLoadResult));
            OutputDebugStringA(buffer);
            });
        m_loadModelAsync->ProgressUpdated([this](float progress)
        {
            // Progress callback
            m_modelLoadingProgress = progress;

            // Output progress percentage to VS output
            char buffer[1024];
            sprintf_s(buffer, "Remote Rendering: Model loading progress: %.1f\n", m_modelLoadingProgress * 100.f);
            OutputDebugStringA(buffer);
        });
    }
}



void HolographicAppMain::SetNewState(AppConnectionStatus state, const char* statusMsg)
{
    m_currentStatus = state;
    m_statusMsg = statusMsg ? statusMsg : "";

    // Some log for the VS output panel:
    const char* appStatus = nullptr;

    switch (state)
    {
        case AppConnectionStatus::Disconnected: appStatus = "Disconnected"; break;
        case AppConnectionStatus::CreatingSession: appStatus = "CreatingSession"; break;
        case AppConnectionStatus::StartingSession: appStatus = "StartingSession"; break;
        case AppConnectionStatus::Connecting: appStatus = "Connecting"; break;
        case AppConnectionStatus::Connected: appStatus = "Connected"; break;
        case AppConnectionStatus::ConnectionFailed: appStatus = "ConnectionFailed"; break;
    }

    char buffer[1024];
    sprintf_s(buffer, "Remote Rendering: New status: %s, result: %s\n", appStatus, m_statusMsg.c_str());
    OutputDebugStringA(buffer);
}

void HolographicAppMain::SetNewSession(RR::ApiHandle<RR::AzureSession> newSession)
{
    SetNewState(AppConnectionStatus::StartingSession, nullptr);

    m_timeAtLastRESTCall = m_timer.GetTotalSeconds();
    m_session = newSession;
    m_api = m_session->Actions();
    m_graphicsBinding = m_session->GetGraphicsBinding().as<RR::GraphicsBindingWmrD3d11>();
    m_session->ConnectionStatusChanged([this](auto status, auto error)
        {
            OnConnectionStatusChanged(status, error);
        });

};

void HolographicAppMain::OnConnectionStatusChanged(RR::ConnectionStatus status, RR::Result error)
{
    const char* asString = RR::ResultToString(error);
    m_connectionResult = error;

    switch (status)
    {
    case RR::ConnectionStatus::Connecting:
        SetNewState(AppConnectionStatus::Connecting, asString);
        break;
    case RR::ConnectionStatus::Connected:
        if (error == RR::Result::Success)
        {
            SetNewState(AppConnectionStatus::Connected, asString);
        }
        else
        {
            SetNewState(AppConnectionStatus::ConnectionFailed, asString);
        }
        m_modelLoadTriggered = m_modelLoadFinished = false;
        m_isConnected = error == RR::Result::Success;
        break;
    case RR::ConnectionStatus::Disconnected:
        if (error == RR::Result::Success)
        {
            SetNewState(AppConnectionStatus::Disconnected, asString);
        }
        else
        {
            SetNewState(AppConnectionStatus::ConnectionFailed, asString);
        }
        m_modelLoadTriggered = m_modelLoadFinished = false;
        m_isConnected = false;
        break;
    default:
        break;
    }
    
}
```

### <a name="per-frame-update"></a>Mise à jour image par image

Nous devons mettre à jour le client une fois par cycle de simulation et effectuer des mises à jour d’état supplémentaires. La fonction `HolographicAppMain::Update` fournit un bon raccordement pour les mises à jour par image.

#### <a name="state-machine-update"></a>Mise à jour de machine à états

Nous devons interroger l’état de la session et voir si elle est passée à l’état `Ready`. Si nous sommes parvenus à nous connecter, il ne nous restera plus qu’à charger le modèle via `StartModelLoading`.

Ajoutez le code suivant au corps de la fonction `HolographicAppMain::Update` :

```cpp
// Updates the application state once per frame.
HolographicFrame HolographicAppMain::Update()
{
    if (m_session != nullptr)
    {
        // Tick the client to receive messages
        m_api->Update();

        if (!m_sessionStarted)
        {
            // Important: To avoid server-side throttling of the requests, we should call GetPropertiesAsync very infrequently:
            const double delayBetweenRESTCalls = 10.0;

            // query session status periodically until we reach 'session started'
            if (m_sessionPropertiesAsync == nullptr && m_timer.GetTotalSeconds() - m_timeAtLastRESTCall > delayBetweenRESTCalls)
            {
                m_timeAtLastRESTCall = m_timer.GetTotalSeconds();
                if (auto propAsync = m_session->GetPropertiesAsync())
                {
                    m_sessionPropertiesAsync = *propAsync;
                    m_sessionPropertiesAsync->Completed([this](const RR::ApiHandle<RR::SessionPropertiesAsync>& async)
                        {
                            if (async->GetStatus() == RR::Result::Success)
                            {
                                auto res = async->GetResult();
                                switch (res.Status)
                                {
                                case RR::RenderingSessionStatus::Ready:
                                {
                                    // The following is async, but we'll get notifications via OnConnectionStatusChanged
                                    m_sessionStarted = true;
                                    SetNewState(AppConnectionStatus::Connecting, nullptr);
                                    RR::ConnectToRuntimeParams init;
                                    init.ignoreCertificateValidation = false;
                                    init.mode = RR::ServiceRenderMode::Default;
                                    m_session->ConnectToRuntime(init);
                                }
                                break;
                                case RR::RenderingSessionStatus::Error:
                                    SetNewState(AppConnectionStatus::ConnectionFailed, "Session error");
                                    break;
                                case RR::RenderingSessionStatus::Stopped:
                                    SetNewState(AppConnectionStatus::ConnectionFailed, "Session stopped");
                                    break;
                                case RR::RenderingSessionStatus::Expired:
                                    SetNewState(AppConnectionStatus::ConnectionFailed, "Session expired");
                                    break;
                                }
    
                            }
                            else
                            {
                                SetNewState(AppConnectionStatus::ConnectionFailed, "Failed to retrieve session status");
                            }
                            m_sessionPropertiesAsync = nullptr; // next try
                            m_needsStatusUpdate = true;
                        });
                }
            }
        }
        if (m_isConnected && !m_modelLoadTriggered)
        {
            m_modelLoadTriggered = true;
            StartModelLoading();
        }
    }

    if (m_needsCoordinateSystemUpdate && m_stationaryReferenceFrame && m_graphicsBinding)
    {
        // Set the coordinate system once. This must be called again whenever the coordinate system changes.
        winrt::com_ptr<ABI::Windows::Perception::Spatial::ISpatialCoordinateSystem> ptr{ m_stationaryReferenceFrame.CoordinateSystem().as<ABI::Windows::Perception::Spatial::ISpatialCoordinateSystem>() };
        m_graphicsBinding->UpdateUserCoordinateSystem(ptr.get());
        m_needsCoordinateSystemUpdate = false;
    }

    // Rest of the body:
    ...
}
```

#### <a name="coordinate-system-update"></a>Mise à jour du système de coordonnées

Nous devons convenir d’un système de coordonnées à utiliser avec le service de rendu. Pour accéder au système de coordonnées que nous souhaitons utiliser, nous avons besoin du `m_stationaryReferenceFrame` créé à la fin de la fonction `HolographicAppMain::OnHolographicDisplayIsAvailableChanged`.

Ce système de coordonnées ne change généralement pas ; il s’agit donc d’une initialisation ponctuelle. Elle doit être appelée à nouveau si votre application modifie le système de coordonnées.

Le code ci-dessus définit le système de coordonnées une fois dans la fonction `Update` dès que nous avons un système de coordonnées de référence et une session connectée.

#### <a name="camera-update"></a>Mise à jour de la caméra

Nous devons mettre à jour les plans de découpage de caméra afin que la caméra du serveur reste synchronisée avec la caméra locale. Nous pouvons le faire à la fin de la fonction `Update` :

```cpp
    ...
    if (m_isConnected)
    {
        // Any near/far plane values of your choosing.
        constexpr float fNear = 0.1f;
        constexpr float fFar = 10.0f;
        for (HolographicCameraPose const& cameraPose : prediction.CameraPoses())
        {
            // Set near and far to the holographic camera as normal
            cameraPose.HolographicCamera().SetNearPlaneDistance(fNear);
            cameraPose.HolographicCamera().SetFarPlaneDistance(fFar);
        }

        // The API to inform the server always requires near < far. Depth buffer data will be converted locally to match what is set on the HolographicCamera.
        auto settings = m_api->GetCameraSettings();
        settings->SetNearAndFarPlane(std::min(fNear, fFar), std::max(fNear, fFar));
        settings->SetEnableDepth(true);
    }

    // The holographic frame will be used to get up-to-date view and projection matrices and
    // to present the swap chain.
    return holographicFrame;
}

```

### <a name="rendering"></a>Rendu

La dernière chose à faire est d’appeler le rendu du contenu distant. Nous devons effectuer cet appel à la position exacte dans le pipeline de rendu, une fois la cible de rendu effacée et la fenêtre d’affichage définie. Insérez l’extrait de code suivant dans le verrou `UseHolographicCameraResources` à l’intérieur de la fonction `HolographicAppMain::Render` :

```cpp
        ...
        // Existing clear function:
        context->ClearDepthStencilView(depthStencilView, D3D11_CLEAR_DEPTH | D3D11_CLEAR_STENCIL, 1.0f, 0);
        
        // ...

        // Exiting check to test for valid camera:
        bool cameraActive = pCameraResources->AttachViewProjectionBuffer(m_deviceResources);


        // Inject remote rendering: as soon as we are connected, start blitting the remote frame.
        // We do the blitting after the Clear, and before cube rendering.
        if (m_isConnected && cameraActive)
        {
            m_graphicsBinding->BlitRemoteFrame();
        }

        ...
```

## <a name="run-the-sample"></a>Exécution de l'exemple

L’exemple doit maintenant être dans un état où il est compilé et s’exécute.

Quand l’exemple s’exécute correctement, il affiche le cube en rotation juste devant vous, et après une création de session et un chargement de modèle, il restitue le modèle de moteur situé à la position de début actuelle. La création de session et le chargement de modèle peuvent prendre quelques minutes. L’état actuel est écrit uniquement dans le volet de sortie de Visual Studio. Il est donc recommandé de démarrer l’exemple à partir de Visual Studio.

> [!CAUTION]
> Le client se déconnecte du serveur quand la fonction tick n’est pas appelée pendant quelques secondes. Par conséquent, le déclenchement de points d’arrêt peut très facilement entraîner la déconnexion de l’application.

Pour un affichage correct de l’état avec un volet texte, reportez-vous à la version prédéfinie de ce tutoriel sur GitHub.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris toutes les étapes nécessaires à l’ajout de Remote Rendering à un exemple d’**application holographique** de stockage C++/DirectX 11.
Pour convertir votre propre modèle, reportez-vous au guide de démarrage rapide suivant :

> [!div class="nextstepaction"]
> [Démarrage rapide : Convertir un modèle pour le rendu](../../../quickstarts/convert-model.md)