---
title: Affinage des matériaux, de l’éclairage et des effets
description: Modifiez les matériaux et l’éclairage des modèles. Ajoutez des effets supplémentaires comme des contours et des plans de coupe.
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 04cb48a3ff84a67995c1a920a323fa568a67cdf3
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92203243"
---
# <a name="tutorial-refining-materials-lighting-and-effects"></a>Tutoriel : Affinage des matériaux, de l’éclairage et des effets

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Mettre en évidence les modèles et les composants de modèle et leur ajouter des contours
> * Appliquer différents matériaux aux modèles
> * Découper les modèles avec des plans de coupe
> * Ajouter des animations simples pour des objets rendus à distance

## <a name="prerequisites"></a>Prérequis

* Ce tutoriel s’appuie sur le [Tutoriel : Manipulation de modèles](..\manipulate-models\manipulate-models.md).

## <a name="highlighting-and-outlining"></a>Mise en évidence et ajout de contours

Il est important d’offrir un retour visuel à l’utilisateur d’une application au titre de l’expérience utilisateur. Azure Remote Rendering propose des mécanismes de retour visuel à travers les [remplacements d’état hiérarchiques](../../../overview/features/override-hierarchical-state.md). Les remplacements d’état hiérarchiques sont implémentés à partir des composants attachés aux instances locales de modèles. Nous avons vu comment créer ces instances locales dans la section [Synchronisation du graphe d’objets distants dans la hiérarchie Unity](../manipulate-models/manipulate-models.md#synchronizing-the-remote-object-graph-into-the-unity-hierarchy).

Pour commencer, nous allons créer un wrapper autour du composant [**HierarchicalStateOverrideComponent**](/dotnet/api/microsoft.azure.remoterendering.hierarchicalstateoverridecomponent). **HierarchicalStateOverrideComponent** est le script local qui contrôle les remplacements au niveau de l’entité distante. Les [**ressources du tutoriel**](../custom-models/custom-models.md#import-assets-used-by-this-tutorial) comportent une classe de base abstraite appelée **BaseEntityOverrideController**, que nous allons développer pour créer le wrapper.

1. Créez un script sous le nom **EntityOverrideController** et remplacez son contenu par le code suivant :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    public class EntityOverrideController : BaseEntityOverrideController
    {
        public override event Action<HierarchicalStates> FeatureOverrideChange;

        private ARRHierarchicalStateOverrideComponent localOverride;
        public override ARRHierarchicalStateOverrideComponent LocalOverride
        {
            get
            {
                if (localOverride == null)
                {
                    localOverride = gameObject.GetComponent<ARRHierarchicalStateOverrideComponent>();
                    if (localOverride == null)
                    {
                        localOverride = gameObject.AddComponent<ARRHierarchicalStateOverrideComponent>();
                    }

                    var remoteStateOverride = TargetEntity.Entity.FindComponentOfType<HierarchicalStateOverrideComponent>();

                    if (remoteStateOverride == null)
                    {
                        // if there is no HierarchicalStateOverrideComponent on the remote side yet, create one
                        localOverride.Create(RemoteManagerUnity.CurrentSession);
                    }
                    else
                    {
                        // otherwise, bind our local stateOverride component to the remote component
                        localOverride.Bind(remoteStateOverride);

                    }
                }
                return localOverride;
            }
        }

        private RemoteEntitySyncObject targetEntity;
        public override RemoteEntitySyncObject TargetEntity
        {
            get
            {
                if (targetEntity == null)
                    targetEntity = gameObject.GetComponent<RemoteEntitySyncObject>();
                return targetEntity;
            }
        }

        private HierarchicalEnableState ToggleState(HierarchicalStates feature)
        {
            HierarchicalEnableState setToState = HierarchicalEnableState.InheritFromParent;
            switch (LocalOverride.RemoteComponent.GetState(feature))
            {
                case HierarchicalEnableState.ForceOff:
                case HierarchicalEnableState.InheritFromParent:
                    setToState = HierarchicalEnableState.ForceOn;
                    break;
                case HierarchicalEnableState.ForceOn:
                    setToState = HierarchicalEnableState.InheritFromParent;
                    break;
            }

            return SetState(feature, setToState);
        }

        private HierarchicalEnableState SetState(HierarchicalStates feature, HierarchicalEnableState enableState)
        {
            if (GetState(feature) != enableState) //if this is actually different from the current state, act on it
            {
                LocalOverride.RemoteComponent.SetState(feature, enableState);
                FeatureOverrideChange?.Invoke(feature);
            }

            return enableState;
        }

        public override HierarchicalEnableState GetState(HierarchicalStates feature) => LocalOverride.RemoteComponent.GetState(feature);

        public override void ToggleHidden() => ToggleState(HierarchicalStates.Hidden);

        public override void ToggleSelect() => ToggleState(HierarchicalStates.Selected);

        public override void ToggleSeeThrough() => ToggleState(HierarchicalStates.SeeThrough);

        public override void ToggleTint(Color tintColor = default)
        {
            if (tintColor != default) LocalOverride.RemoteComponent.TintColor = tintColor.toRemote();
            ToggleState(HierarchicalStates.UseTintColor);
        }

        public override void ToggleDisabledCollision() => ToggleState(HierarchicalStates.DisableCollision);

        public override void RemoveOverride()
        {
            var remoteStateOverride = TargetEntity.Entity.FindComponentOfType<HierarchicalStateOverrideComponent>();
            if (remoteStateOverride != null)
            {
                remoteStateOverride.Destroy();
            }

            if (localOverride == null)
                localOverride = gameObject.GetComponent<ARRHierarchicalStateOverrideComponent>();

            if (localOverride != null)
            {
                Destroy(localOverride);
            }
        }
    }
    ```

La fonction principale de **LocalOverride** est de créer un lien entre lui-même et son `RemoteComponent`. **LocalOverride** nous permet ensuite de définir des indicateurs d’état au niveau du composant local, qui sont liés à l’entité distante. Les remplacements et leurs états sont décrits dans la page [Remplacements d’état hiérarchiques](../../../overview/features/override-hierarchical-state.md). 

Cette implémentation ne fait basculer qu’un seul état à la fois. Cependant, il est tout à fait possible de combiner plusieurs remplacements pour des entités uniques et de créer des combinaisons à différents niveaux de la hiérarchie. Par exemple, si vous combinez `Selected` et `SeeThrough` sur un composant unique, vous le doterez d’un contour tout en le rendant transparent. Ou bien, si vous définissez le remplacement `Hidden` de l’entité racine sur `ForceOn` et que vous attribuez au remplacement `Hidden` d’une entité enfant la valeur `ForceOff`, tout sera masqué sauf l’enfant associé au remplacement.

Pour appliquer les états aux entités, nous pouvons modifier le **RemoteEntityHelper** créé précédemment.

1. Modifiez la classe **RemoteEntityHelper** de façon à implémenter la classe abstraite **BaseRemoteEntityHelper**. Cette modification permettra l’utilisation d’un contrôleur d’affichage fourni dans les **ressources du tutoriel**. Une fois modifiée, la classe doit se présenter comme suit :

    ```csharp
    public class RemoteEntityHelper : BaseRemoteEntityHelper
    ```

2. Remplacez les méthodes abstraites à l’aide du code suivant :

    ```csharp
    public override BaseEntityOverrideController EnsureOverrideComponent(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var overrideComponent = entityGameObject.GetComponent<EntityOverrideController>();
        if (overrideComponent == null)
            overrideComponent = entityGameObject.AddComponent<EntityOverrideController>();
        return overrideComponent;
    }

    public override HierarchicalEnableState GetState(Entity entity, HierarchicalStates feature)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        return overrideComponent.GetState(feature);
    }

    public override void ToggleHidden(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleHidden();
    }

    public override void ToggleSelect(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleSelect();
    }

    public override void ToggleSeeThrough(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleSeeThrough();
    }

    public Color TintColor = new Color(0.0f, 1.0f, 0.0f, 0.1f);
    public override void ToggleTint(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleTint(TintColor);
    }

    public override void ToggleDisableCollision(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleHidden();
    }

    public override void RemoveOverrides(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var overrideComponent = entityGameObject.GetComponent<EntityOverrideController>();
        if (overrideComponent != null)
        {
            overrideComponent.RemoveOverride();
            Destroy(overrideComponent);
        }
    }
    ```

Ce code fait en sorte qu’un composant **EntityOverrideController** soit ajouté à l’entité cible, puis que l’une des méthodes bascules soit appelée. Si vous le souhaitez, au niveau du GameObject **TestModel**, il est possible d’appeler ces méthodes d’assistance en ajoutant le **RemoteEntityHelper** en tant que rappel à l’événement `OnRemoteEntityClicked` du composant **RemoteRayCastPointerHandler**.

![Rappels de pointeur](./media/pointer-event-callbacks.png)

Maintenant que ces scripts ont été ajoutés au modèle, une fois connecté au runtime, le contrôleur d’affichage **AppMenu** doit disposer d’interfaces supplémentaires (activées) pour interagir avec le script **EntityOverrideController**. Vérifiez dans le menu **Model Tools** (Outils de modèle) que les contrôleurs d’affichage sont déverrouillés.

À ce stade, les composants de votre GameObject **TestModel** doivent se présenter comme suit :

![Modèle de test avec des scripts supplémentaires](./media/test-model-updated.png)

Voici un exemple d’empilement de remplacements au niveau d’une même entité. Nous avons utilisé `Select` et `Tint` pour fournir à la fois un contour et une coloration :

![Sélection de teinte pour le modèle de test](./media/select-tint-test-model.png)

## <a name="cut-planes"></a>Plans de coupe

Les [plans de coupe](../../../overview/features/cut-planes.md) sont une fonctionnalité qui peut être ajoutée à n’importe quelle entité distante. En règle générale, une entité distante qui est créée ne doit pas être associée à des données de maillage pour pouvoir contenir le composant de plan de coupe. La position et l’orientation du plan de coupe sont déterminées par la position et l’orientation de l’entité distante à laquelle il est attaché.

Nous allons créer un script qui crée automatiquement une entité distante, ajoute un composant de plan de coupe et synchronise la transformation d’un objet local avec l’entité de plan de coupe. Ensuite, nous pourrons utiliser le **CutPlaneViewController** pour encapsuler le plan de coupe dans une interface afin de pouvoir le manipuler.

1. Créez un script sous le nom **RemoteCutPlane** et remplacez son code par le code ci-dessous :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    public class RemoteCutPlane : BaseRemoteCutPlane
    {
        public Color SliceColor = new Color(0.5f, 0f, 0f, .5f);
        public float FadeLength = 0.01f;
        public Axis SliceNormal = Axis.Y_Neg;

        public bool AutomaticallyCreate = true;

        private CutPlaneComponent remoteCutPlaneComponent;
        private bool cutPlaneReady = false;

        public override bool CutPlaneReady 
        { 
            get => cutPlaneReady;
            set 
            { 
                cutPlaneReady = value;
                CutPlaneReadyChanged?.Invoke(cutPlaneReady);
            }
        }

        public override event Action<bool> CutPlaneReadyChanged;

        public UnityBoolEvent OnCutPlaneReadyChanged = new UnityBoolEvent();

        public void Start()
        {
            // Hook up the event to the Unity event
            CutPlaneReadyChanged += (ready) => OnCutPlaneReadyChanged?.Invoke(ready);

            RemoteRenderingCoordinator.CoordinatorStateChange += RemoteRenderingCoordinator_CoordinatorStateChange;
            RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        private void RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    if (AutomaticallyCreate)
                        CreateCutPlane();
                    break;
                default:
                    DestroyCutPlane();
                    break;
            }
        }

        public override void CreateCutPlane()
        {
            //Implement me
        }

        public override void DestroyCutPlane()
        {
            //Implement me
        }
    }
    ```

    Ce code étend la classe **BaseRemoteCutPlane** incluse dans les **ressources du tutoriel**. Comme pour le modèle rendu à distance, ce script s’attache et écoute les modifications de `RemoteRenderingState` du coordinateur distant. Quand le coordinateur atteint l’état `RuntimeConnected`, il tente de se connecter automatiquement s’il est censé le faire. Il existe aussi une variable `CutPlaneComponent` que nous allons suivre. Il s’agit du composant Azure Remote Rendering qui se synchronise avec le plan de coupe de la session à distance. Voyons ce que nous devons faire pour créer le plan de coupe.

2. Remplacez la méthode `CreateCutPlane()` par la version complète ci-dessous :

    ```csharp
    public override void CreateCutPlane()
    {
        if (remoteCutPlaneComponent != null)
            return; //Nothing to do!

        //Create a root object for the cut plane
        var cutEntity = RemoteRenderingCoordinator.CurrentSession.Actions.CreateEntity();

        //Bind the remote entity to this game object
        cutEntity.BindToUnityGameObject(this.gameObject);

        //Sync the transform of this object so we can move the cut plane
        var syncComponent = this.gameObject.GetComponent<RemoteEntitySyncObject>();
        syncComponent.SyncEveryFrame = true;

        //Add a cut plane to the entity
        remoteCutPlaneComponent = RemoteRenderingCoordinator.CurrentSession.Actions.CreateComponent(ObjectType.CutPlaneComponent, cutEntity) as CutPlaneComponent;

        //Configure the cut plane
        remoteCutPlaneComponent.Normal = SliceNormal;
        remoteCutPlaneComponent.FadeColor = SliceColor.toRemote();
        remoteCutPlaneComponent.FadeLength = FadeLength;
        CutPlaneReady = true;
    }
    ```

    Ici, nous créons une entité distante et la lions à un GameObject local. Nous faisons en sorte que la transformation de l’entité distante se synchronise avec la transformation locale en définissant `SyncEveryFrame` sur `true`. Ensuite, nous utilisons l’appel de `CreateComponent` pour ajouter un `CutPlaneComponent` à l’objet distant. Enfin, nous configurons le plan de coupe avec les paramètres définis dans la partie supérieure du MonoBehaviour. Voyons comment nettoyer un plan de coupe en implémentant la méthode `DestroyCutPlane()`.

3. Remplacez la méthode `DestroyCutPlane()` par la version complète ci-dessous :

    ```csharp
    public override void DestroyCutPlane()
    {
        if (remoteCutPlaneComponent == null)
            return; //Nothing to do!

        remoteCutPlaneComponent.Owner.Destroy();
        remoteCutPlaneComponent = null;
        CutPlaneReady = false;
    }
    ```

Sachant que l’objet distant est relativement simple et que nous nettoyons uniquement l’extrémité distante (et que nous conservons notre objet local), il est facile d’appeler simplement `Destroy` au niveau de l’objet distant et d’effacer notre référence à celui-ci.

**AppMenu** comporte un contrôleur d’affichage qui est automatiquement attaché à votre plan de coupe et vous permet d’interagir avec lui. Il n’est pas indispensable d’utiliser **AppMenu** ou l’un des contrôleurs d’affichage, mais ils assurent une meilleure expérience. À présent, testez le plan de coupe et son contrôleur d’affichage.

1. Créez un GameObject vide dans la scène et nommez-le **CutPlane**.
1. Ajoutez le composant **RemoteCutPlane** au GameObject **CutPlane**.

   ![Configuration du composant Plan de coupe](./media/cut-plane-config.png)

1. Appuyez sur Play (Lecture) dans l’éditeur Unity pour charger et vous connecter à une session à distance.
1. En utilisant la simulation de main de MRTK, saisissez le plan de coupe et faites le pivoter (en maintenant la touche Ctrl enfoncée) pour le déplacer dans la scène. **TestModel** le découpe et en dévoile les composants internes.

![Exemple de plan de coupe](./media/cut-plane-example-engine.png)

## <a name="configuring-the-remote-lighting"></a>Configuration de l’éclairage à distance

La session de rendu à distance prend en charge tout un éventail d’[options d’éclairage](../../../overview/features/lights.md). Nous allons créer des scripts pour la [texture du ciel](../../../overview/features/sky.md) et une carte simple pour deux types de lumière Unity à utiliser avec le rendu à distance.

### <a name="sky-texture"></a>Texture du ciel

Vous pouvez effectuer un choix parmi les différents cubemaps intégrés au moment de modifier la texture du ciel. Ils sont chargés dans la session et appliqués à la texture du ciel. Il est également possible de [charger vos propres textures](../../../concepts/textures.md) et de les utiliser comme lumière du ciel.

Nous allons créer un script **RemoteSky** qui contient la liste des cubemaps intégrés disponibles sous forme de paramètres de chargement. Ensuite, nous offrirons la possibilité à l’utilisateur de sélectionner et charger une des options.

1. Créez un script sous le nom **RemoteSky** et remplacez tout son contenu par le code ci-dessous :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using System;
    using System.Collections.Generic;
    using UnityEngine;

    public class RemoteSky : BaseRemoteSky
    {
        public override Dictionary<string, LoadTextureFromSASParams> AvailableCubemaps => builtInTextures;

        private bool canSetSky;
        public override bool CanSetSky
        {
            get => canSetSky;
            set
            {
                canSetSky = value;
                CanSetSkyChanged?.Invoke(canSetSky);
            }
        }

        private string currentSky = "DefaultSky";
        public override string CurrentSky
        {
            get => currentSky;
            protected set
            {
                currentSky = value;
                SkyChanged?.Invoke(value);
            }
        }

        private Dictionary<string, LoadTextureFromSASParams> builtInTextures = new Dictionary<string, LoadTextureFromSASParams>()
        {
            {"Autoshop",new LoadTextureFromSASParams("builtin://Autoshop", TextureType.CubeMap)},
            {"BoilerRoom",new LoadTextureFromSASParams("builtin://BoilerRoom", TextureType.CubeMap)},
            {"ColorfulStudio",new LoadTextureFromSASParams("builtin://ColorfulStudio", TextureType.CubeMap)},
            {"Hangar",new LoadTextureFromSASParams("builtin://Hangar", TextureType.CubeMap)},
            {"IndustrialPipeAndValve",new LoadTextureFromSASParams("builtin://IndustrialPipeAndValve", TextureType.CubeMap)},
            {"Lebombo",new LoadTextureFromSASParams("builtin://Lebombo", TextureType.CubeMap)},
            {"SataraNight",new LoadTextureFromSASParams("builtin://SataraNight", TextureType.CubeMap)},
            {"SunnyVondelpark",new LoadTextureFromSASParams("builtin://SunnyVondelpark", TextureType.CubeMap)},
            {"Syferfontein",new LoadTextureFromSASParams("builtin://Syferfontein", TextureType.CubeMap)},
            {"TearsOfSteelBridge",new LoadTextureFromSASParams("builtin://TearsOfSteelBridge", TextureType.CubeMap)},
            {"VeniceSunset",new LoadTextureFromSASParams("builtin://VeniceSunset", TextureType.CubeMap)},
            {"WhippleCreekRegionalPark",new LoadTextureFromSASParams("builtin://WhippleCreekRegionalPark", TextureType.CubeMap)},
            {"WinterRiver",new LoadTextureFromSASParams("builtin://WinterRiver", TextureType.CubeMap)},
            {"DefaultSky",new LoadTextureFromSASParams("builtin://DefaultSky", TextureType.CubeMap)}
        };

        public UnityBoolEvent OnCanSetSkyChanged;
        public override event Action<bool> CanSetSkyChanged;

        public UnityStringEvent OnSkyChanged;
        public override event Action<string> SkyChanged;

        public void Start()
        {
            // Hook up the event to the Unity event
            CanSetSkyChanged += (canSet) => OnCanSetSkyChanged?.Invoke(canSet);
            SkyChanged += (key) => OnSkyChanged?.Invoke(key);

            RemoteRenderingCoordinator.CoordinatorStateChange += ApplyStateToView;
            ApplyStateToView(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        private void ApplyStateToView(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    CanSetSky = true;
                    break;
                default:
                    CanSetSky = false;
                    break;
            }
        }

        public override async void SetSky(string skyKey)
        {
            if (!CanSetSky)
            {
                Debug.Log("Unable to set sky right now");
                return;
            }

            if (AvailableCubemaps.ContainsKey(skyKey))
            {
                Debug.Log("Setting sky to " + skyKey);
                //Load the texture into the session
                var texture = await RemoteRenderingCoordinator.CurrentSession.Actions.LoadTextureFromSASAsync(AvailableCubemaps[skyKey]).AsTask();

                //Apply the texture to the SkyReflectionSettings
                RemoteRenderingCoordinator.CurrentSession.Actions.SkyReflectionSettings.SkyReflectionTexture = texture;
                SkyChanged?.Invoke(skyKey);
            }
            else
            {
                Debug.Log("Invalid sky key");
            }
        }
    }
    ```

    La partie la plus importante de ce code tient sur quelques lignes :

    ```csharp
    //Load the texture into the session
    var texture = await RemoteRenderingCoordinator.CurrentSession.Actions.LoadTextureFromSASAsync(AvailableCubemaps[skyKey]).AsTask();

    //Apply the texture to the SkyReflectionSettings
    RemoteRenderingCoordinator.CurrentSession.Actions.SkyReflectionSettings.SkyReflectionTexture = texture;
    ```

    Ici, nous obtenons une référence à la texture à utiliser en la chargeant dans la session à partir du stockage d’objets blob intégré. Ensuite, il nous suffit d’affecter cette texture à l’élément `SkyReflectionTexture` de la session pour l’appliquer.

1. Créez un GameObject vide dans votre scène et nommez-le **SkyLight**.

1. Ajoutez le script **RemoteSky** à votre GameObject **SkyLight**.

    Pour passer d’une lumière de ciel à une autre, vous devez appeler `SetSky` avec l’une des clés de chaîne définies dans `AvailableCubemaps`. Le contrôleur d’affichage intégré à **AppMenu** crée automatiquement des boutons et connecte leurs événements pour appeler `SetSky` avec leur clé respective.
1. Appuyez sur Play (Lecture) dans l’éditeur Unity et autorisez une connexion.
1. Après avoir connecté le runtime local à une session à distance, accédez à **AppMenu -> Session Tools -> Remote Sky** (AppMenu -> Outils de session Tools -> Ciel distant) pour explorer les différentes options de ciel et voir comment elles affectent **TestModel**.

### <a name="scene-lights"></a>Lumières de scène

Les lumières de scène distantes peuvent être ponctuelles, projetées ou directionnelles. Comme pour le plan de coupe que nous avons créé précédemment, ces lumières de scène sont des entités distantes auxquels des composants sont attachés. Lorsque vous éclairez votre scène distante, veillez à harmoniser l’éclairage avec celui de votre scène locale. Cette stratégie n’est pas toujours applicable, car nombreuses sont les applications Unity pour HoloLens 2 qui n’utilisent pas de rendu physique pour les objets rendus localement. Cependant, à un certain niveau, il est possible de simuler l’éclairage par défaut plus simple de Unity.

1. Créez un script sous le nom **RemoteLight** et remplacez son code par le code ci-dessous :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    [RequireComponent(typeof(Light))]
    public class RemoteLight : BaseRemoteLight
    {
        public bool AutomaticallyCreate = true;

        private bool lightReady = false;
        public override bool LightReady 
        {
            get => lightReady;
            set
            {
                lightReady = value;
                LightReadyChanged?.Invoke(lightReady);
            }
        }

        private ObjectType remoteLightType = ObjectType.Invalid;
        public override ObjectType RemoteLightType => remoteLightType;

        public UnityBoolEvent OnLightReadyChanged;

        public override event Action<bool> LightReadyChanged;

        private Light localLight; //Unity Light

        private Entity lightEntity;
        private LightComponentBase remoteLightComponent; //Remote Rendering Light

        private void Awake()
        {
            localLight = GetComponent<Light>();
            switch (localLight.type)
            {
                case LightType.Directional:
                    remoteLightType = ObjectType.DirectionalLightComponent;
                    break;
                case LightType.Point:
                    remoteLightType = ObjectType.PointLightComponent;
                    break;
                case LightType.Spot:
                case LightType.Area:
                    //Not supported in tutorial
                case LightType.Disc:
                    // No direct analog in remote rendering
                    remoteLightType = ObjectType.Invalid;
                    break;
            }
        }

        public void Start()
        {
            // Hook up the event to the Unity event
            LightReadyChanged += (ready) => OnLightReadyChanged?.Invoke(ready);

            RemoteRenderingCoordinator.CoordinatorStateChange += RemoteRenderingCoordinator_CoordinatorStateChange;
            RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        public void OnDestroy()
        {
            lightEntity?.Destroy();
        }

        private void RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    if (AutomaticallyCreate)
                        CreateLight();
                    break;
                default:
                    DestroyLight();
                    break;
            }
        }

        public override void CreateLight()
        {
            if (remoteLightComponent != null)
                return; //Nothing to do!

            //Create a root object for the light
            if(lightEntity == null)
                lightEntity = RemoteRenderingCoordinator.CurrentSession.Actions.CreateEntity();

            //Bind the remote entity to this game object
            lightEntity.BindToUnityGameObject(this.gameObject);

            //Sync the transform of this object so we can move the light
            var syncComponent = this.gameObject.GetComponent<RemoteEntitySyncObject>();
            syncComponent.SyncEveryFrame = true;

            //Add a light to the entity
            switch (RemoteLightType)
            {
                case ObjectType.DirectionalLightComponent:
                    var remoteDirectional = RemoteRenderingCoordinator.CurrentSession.Actions.CreateComponent(ObjectType.DirectionalLightComponent, lightEntity) as DirectionalLightComponent;
                    //No additional properties
                    remoteLightComponent = remoteDirectional;
                    break;

                case ObjectType.PointLightComponent:
                    var remotePoint = RemoteRenderingCoordinator.CurrentSession.Actions.CreateComponent(ObjectType.PointLightComponent, lightEntity) as PointLightComponent;
                    remotePoint.Radius = 0;
                    remotePoint.Length = localLight.range;
                    //remotePoint.AttenuationCutoff = //No direct analog in Unity legacy lights
                    //remotePoint.ProjectedCubeMap = //No direct analog in Unity legacy lights

                    remoteLightComponent = remotePoint;
                    break;
                default:
                    LightReady = false;
                    return;
            }

            // Set the common values for all light types
            UpdateRemoteLightSettings();

            LightReady = true;
        }

        public override void UpdateRemoteLightSettings()
        {
            remoteLightComponent.Color = localLight.color.toRemote();
            remoteLightComponent.Intensity = localLight.intensity;
        }

        public override void DestroyLight()
        {
            if (remoteLightComponent == null)
                return; //Nothing to do!

            remoteLightComponent.Destroy();
            remoteLightComponent = null;
            LightReady = false;
        }

        [ContextMenu("Sync Remote Light Configuration")]
        public override void RecreateLight()
        {
            DestroyLight();
            CreateLight();
        }

        public override void SetIntensity(float intensity)
        {
            localLight.intensity = Mathf.Clamp(intensity, 0, 1);
            UpdateRemoteLightSettings();
        }

        public override void SetColor(Color color)
        {
            localLight.color = color;
            UpdateRemoteLightSettings();
        }
    }
    ```

    Ce script crée différents types de lumières distantes en fonction du type de lumière Unity locale à laquelle le script est attaché. La lumière distante duplique alors la lumière locale selon sa position, sa rotation, sa couleur et son intensité. Quand cela est possible, la lumière distante définit aussi une configuration supplémentaire. Il ne s’agit pas d’une correspondance parfaite, car les lumières Unity ne sont pas des lumières PBR.

1. Recherchez le GameObject **DirectionalLight** dans votre scène. Si vous avez retiré le **DirectionalLight** par défaut de votre scène : dans la barre de menus du haut, sélectionnez *GameObject -> Light -> DirectionalLight* pour créer une nouvelle lumière dans votre scène.

1. Sélectionnez le GameObject **DirectionalLight** puis, à l’aide du bouton **Add Component** (Ajouter un composant), ajoutez le script **RemoteLight**.

1. Comme ce script implémente la classe de base `BaseRemoteLight`, vous pouvez utiliser le contrôleur d’affichage **AppMenu** fourni pour interagir avec la lumière distante. Accédez à **AppMenu -> Session Tools -> Directional Light** (AppMenu -> Outils de session -> Lumière directionnelle).

    > [!NOTE]
    > Pour plus de simplicité, l’interface utilisateur de **AppMenu** a été limitée à une seule lumière directionnelle. Cependant, il est toujours possible et recommandé d’ajouter des lumières ponctuelles et d’attacher le script **RemoteLight** à celles-ci. Ces lumières supplémentaires peuvent être modifiées dans l’éditeur via les propriétés de la lumière Unity. Vous devrez synchroniser manuellement les modifications locales apportées à la lumière distante à partir du menu contextuel **RemoteLight** de l’inspecteur :
    >
    > ![Synchronisation manuelle de la lumière distante](./media/sync-remote-light.png)

1. Appuyez sur Play (Lecture) dans l’éditeur Unity et autorisez une connexion.

1. Après avoir connecté votre runtime à une session à distance, positionnez et dirigez votre caméra (utilisez les touches WASD et le clic droit + déplacement de la souris) de façon à avoir le contrôleur d’affichage de la lumière directionnelle dans la vue. 
1. Utilisez le contrôleur d’affichage de la lumière distante pour modifier les propriétés de la lumière. En utilisant la simulation de main de MRTK, saisissez la lumière directionnelle et faites la pivoter (en maintenant la touche Ctrl enfoncée) pour voir l’effet sur l’éclairage de la scène.

    ![Lumière directionnelle](./media/directional-light-test.png)

## <a name="editing-materials"></a>Modification des matériaux

Les [matériaux](../../../concepts/materials.md) rendus à distance peuvent être modifiés de façon à offrir des effets visuels supplémentaires, à ajuster les objets visuels des modèles rendus ou à fournir un retour supplémentaire aux utilisateurs. Il existe de nombreuses façons et raisons de modifier un matériau. Ici, nous allons vous montrer comment modifier la couleur albédo d’un matériau et modifier le caractère rugueux (« roughness ») et métallique (« metalness ») d’un matériau PBR.

> [!NOTE]
> Dans de nombreux cas, si une fonctionnalité ou un effet peut être implémenté à l’aide d’un **HierarchicalStateOverrideComponent**, il est préférable de l’utiliser plutôt que de modifier le matériau.

Nous allons créer un script qui accepte une entité cible et configure quelques objets `OverrideMaterialProperty` de façon à modifier les propriétés du matériau de l’entité cible. Nous commençons par obtenir le [**MeshComponent**](../../../concepts/meshes.md#meshcomponent) de l’entité cible, qui contient la liste des matériaux utilisés sur le maillage. Pour plus de simplicité, nous allons simplement utiliser le premier matériau trouvé. Selon la façon dont le contenu a été créé, cette stratégie naïve peut très facilement capoter. Il est donc préférable d’adopter une approche plus complexe pour sélectionner le matériau adéquat.

À partir du matériau, nous pouvons accéder à des valeurs courantes comme « Albedo ». Tout d’abord, les matériaux doivent être castés dans leur type approprié, `PbrMaterial` ou `ColorMaterial`, pour récupérer leurs valeurs, comme nous l’avons vu pour la méthode **GetMaterialColor**. Dès lors que nous avons une référence au matériau souhaité, il suffit de définir les valeurs. ARR gère alors la synchronisation entre les propriétés du matériau local et le matériau distant.

1. Créez un script sous le nom **EntityMaterialController** et remplacez son contenu par le code suivant :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using System.Linq;
    using UnityEngine;
    // to prevent namespace conflicts
    using ARRMaterial = Microsoft.Azure.RemoteRendering.Material;

    public class EntityMaterialController : BaseEntityMaterialController
    {
        public override bool RevertOnEntityChange { get; set; } = true;

        public override OverrideMaterialProperty<Color> ColorOverride { get; set; }
        public override OverrideMaterialProperty<float> RoughnessOverride { get; set; }
        public override OverrideMaterialProperty<float> MetalnessOverride { get; set; }

        private Entity targetEntity;
        public override Entity TargetEntity
        {
            get => targetEntity;
            set
            {
                if (targetEntity != value)
                {
                    if (targetEntity != null && RevertOnEntityChange)
                    {
                        Revert();
                    }

                    targetEntity = value;
                    ConfigureTargetEntity();
                    TargetEntityChanged?.Invoke(value);
                }
            }
        }

        private ARRMaterial targetMaterial;
        private ARRMeshComponent meshComponent;

        public override event Action<Entity> TargetEntityChanged;
        public UnityRemoteEntityEvent OnTargetEntityChanged;

        public void Start()
        {
            // Forward events to Unity events
            TargetEntityChanged += (entity) => OnTargetEntityChanged?.Invoke(entity);

            // If there happens to be a remote RayCaster on this object, assume we should listen for events from it
            if (GetComponent<BaseRemoteRayCastPointerHandler>() != null)
                GetComponent<BaseRemoteRayCastPointerHandler>().RemoteEntityClicked += (entity) => TargetEntity = entity;
        }

        protected override void ConfigureTargetEntity()
        {
            //Get the Unity object, to get the sync object, to get the mesh component, to get the material.
            var targetEntityGameObject = TargetEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);

            var localSyncObject = targetEntityGameObject.GetComponent<RemoteEntitySyncObject>();
            meshComponent = targetEntityGameObject.GetComponent<ARRMeshComponent>();
            if (meshComponent == null)
            {
                var mesh = localSyncObject.Entity.FindComponentOfType<MeshComponent>();
                if (mesh != null)
                {
                    targetEntityGameObject.BindArrComponent<ARRMeshComponent>(mesh);
                    meshComponent = targetEntityGameObject.GetComponent<ARRMeshComponent>();
                }
            }

            meshComponent.enabled = true;

            targetMaterial = meshComponent.RemoteComponent.Mesh.Materials.FirstOrDefault();
            if (targetMaterial == default)
            {
                return;
            }

            ColorOverride = new OverrideMaterialProperty<Color>(
                GetMaterialColor(targetMaterial), //The original value
                targetMaterial, //The target material
                ApplyMaterialColor); //The action to take to apply the override

            //If the material is a PBR material, we can override some additional values
            if (targetMaterial.MaterialSubType == MaterialType.Pbr)
            {
                var firstPBRMaterial = (PbrMaterial)targetMaterial;

                RoughnessOverride = new OverrideMaterialProperty<float>(
                    firstPBRMaterial.Roughness, //The original value
                    targetMaterial, //The target material
                    ApplyRoughnessValue); //The action to take to apply the override

                MetalnessOverride = new OverrideMaterialProperty<float>(
                    firstPBRMaterial.Metalness, //The original value
                    targetMaterial, //The target material
                    ApplyMetalnessValue); //The action to take to apply the override
            }
            else //otherwise, ensure the overrides are cleared out from any previous entity
            {
                RoughnessOverride = null;
                MetalnessOverride = null;
            }
        }

        public override void Revert()
        {
            if (ColorOverride != null)
                ColorOverride.OverrideActive = false;

            if (RoughnessOverride != null)
                RoughnessOverride.OverrideActive = false;

            if (MetalnessOverride != null)
                MetalnessOverride.OverrideActive = false;
        }

        private Color GetMaterialColor(ARRMaterial material)
        {
            if (material == null)
                return default;

            if (material.MaterialSubType == MaterialType.Color)
                return ((ColorMaterial)material).AlbedoColor.toUnity();
            else
                return ((PbrMaterial)material).AlbedoColor.toUnity();
        }

        private void ApplyMaterialColor(ARRMaterial material, Color color)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Color)
                ((ColorMaterial)material).AlbedoColor = color.toRemoteColor4();
            else
                ((PbrMaterial)material).AlbedoColor = color.toRemoteColor4();
        }

        private void ApplyRoughnessValue(ARRMaterial material, float value)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Pbr) //Only PBR has Roughness
                ((PbrMaterial)material).Roughness = value;
        }

        private void ApplyMetalnessValue(ARRMaterial material, float value)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Pbr) //Only PBR has Metalness
                ((PbrMaterial)material).Metalness = value;
        }
    }
    ```

Le type `OverrideMaterialProperty` doit être suffisamment flexible pour permettre éventuellement la modification d’autres valeurs de matériau. Le type `OverrideMaterialProperty` suit l’état d’un remplacement, conserve l’ancienne et la nouvelle valeur et utilise un délégué pour définir le remplacement. À titre d’exemple, examinez le remplacement `ColorOverride` :

```csharp
ColorOverride = new OverrideMaterialProperty<Color>(
    GetMaterialColor(targetMaterial), //The original value
    targetMaterial, //The target material
    ApplyMaterialColor); //The action to take to apply the override
```

Ce code crée un élément `OverrideMaterialProperty` dans lequel le remplacement encapsule le type `Color`. Nous indiquons la couleur actuelle ou d’origine au moment où le remplacement est créé. Nous lui communiquons aussi le matériau ARR sur lequel agir. Enfin, le délégué qui va appliquer le remplacement est indiqué. Le délégué est une méthode qui accepte un matériau ARR ainsi que le type encapsulé par le remplacement. Cette méthode est l’élément le plus important pour comprendre comment ARR ajuste les valeurs des matériaux.

Le remplacement `ColorOverride` utilise la méthode `ApplyMaterialColor` pour accomplir sa tâche :

```csharp
private void ApplyMaterialColor(ARRMaterial material, Color color)
{
    if (material.MaterialSubType == MaterialType.Color)
        ((ColorMaterial)material).AlbedoColor = color.toRemoteColor4();
    else
        ((PbrMaterial)material).AlbedoColor = color.toRemoteColor4();
}
```

Ce code accepte un matériau et une couleur. Après avoir identifié le type du matériau, il effectue un cast du matériau pour appliquer la couleur.

Les remplacements `RoughnessOverride` et `MetalnessOverride` fonctionnent de la même façon et utilisent les méthodes `ApplyRoughnessValue` et `ApplyMetalnessValue` pour accomplir leur tâche.

À présent, testons le contrôleur de matériaux.

1. Ajoutez le script **EntityMaterialController** à votre GameObject **TestModel**.
1. Appuyez sur Play (Lecture) dans Unity pour démarrer la scène et vous connecter à ARR.
1. Une fois que vous avez connecté votre runtime à une session à distance puis chargé le modèle, accédez à **AppMenu -> Model Tools -> Edit Material** (AppMenu -> Outils de modèle -> Modifier un matériau).
1. Sélectionnez une entité dans le modèle à l’aide des mains simulées pour cliquer sur **TestModel**.
1. Vérifiez que le contrôleur d’affichage de matériaux (**AppMenu -> Model Tools -> Edit Material**) a été mis à jour en fonction de l’entité ciblée.
1. Utilisez le contrôleur d’affichage de matériaux pour ajuster le matériau de l’entité ciblée.

Sachant que nous modifions uniquement le premier matériau du maillage, il est possible que vous ne perceviez pas de changement. Utilisez le remplacement hiérarchique **SeeThrough** pour voir si le matériau que vous modifiez se trouve à l’intérieur du maillage.

![Exemple de modification de matériau](./media/material-edit-example.png)

## <a name="next-steps"></a>Étapes suivantes

Félicitations ! Vous avez maintenant implémenté toutes les fonctionnalités de base d’Azure Remote Rendering. Dans le prochain chapitre, nous allons découvrir comment sécuriser Azure Remote Rendering et le stockage d’objets blob. Il s’agira des premières étapes de la publication d’une application commerciale utilisant Azure Remote Rendering.

> [!div class="nextstepaction"]
> [Étape suivante : Sécurisation d’Azure Remote Rendering et du stockage de modèles](../security/security.md)