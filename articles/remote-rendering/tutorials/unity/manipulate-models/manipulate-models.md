---
title: Manipulation de modèles
description: Manipuler des modèles rendus à distance en déplaçant, faisant pivoter, mettant à l’échelle, etc.
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 48c835070329b5cb0892b10760d37708e46bfa1d
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96559131"
---
# <a name="tutorial-manipulating-models"></a>Tutoriel : Manipulation de modèles

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Ajouter des limites visuelles et de manipulation autour des modèles rendus à distance
> * Déplacer, faire pivoter et mettre à l’échelle
> * Raycast avec requêtes spatiales
> * Ajouter des animations simples pour les objets rendus à distance

## <a name="prerequisites"></a>Prérequis

* Ce tutoriel s’appuie sur le [Tutoriel : Interfaces et modèles personnalisés](../custom-models/custom-models.md).

## <a name="query-remote-object-bounds-and-apply-to-local-bounds"></a>Interroger des limites d’objets distants et les appliquer à des limites locales

Pour interagir avec des objets distants, nous avons d’abord besoin d’une représentation locale avec laquelle interagir. Les [limites d’objets](../../../concepts/object-bounds.md) sont utiles pour manipuler rapidement un objet distant. Les limites distantes peuvent être interrogées à partir d’ARR, en utilisant l’entité locale comme référence. Les limites sont interrogées après que le modèle a été chargé dans la session à distance.

Les limites d’un modèle sont définies par la zone qui contient le modèle entier, à l’instar de [**BoxCollider**](https://docs.unity3d.com/Manual/class-BoxCollider.html) de Unity, dont le centre et la taille sont définis pour les axes x, y et z. En fait, nous allons utiliser **BoxCollider** de Unity pour représenter les limites du modèle distant.

1. Créez un script dans le même répertoire que **RemoteRenderedModel** et nommez-le **RemoteBounds**.
1. Remplacez le contenu du script par le code suivant :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    [RequireComponent(typeof(BaseRemoteRenderedModel))]
    public class RemoteBounds : BaseRemoteBounds
    {
        //Remote bounds works with a specific remotely rendered model
        private BaseRemoteRenderedModel targetModel = null;

        private BoundsQueryAsync remoteBoundsQuery = null;

        private RemoteBoundsState currentBoundsState = RemoteBoundsState.NotReady;

        public override RemoteBoundsState CurrentBoundsState
        {
            get => currentBoundsState;
            protected set
            {
                if (currentBoundsState != value)
                {
                    currentBoundsState = value;
                    BoundsStateChange?.Invoke(value);
                }
            }
        }

        public override event Action<RemoteBoundsState> BoundsStateChange;

        public void Awake()
        {
            BoundsStateChange += HandleUnityEvents;
            targetModel = GetComponent<BaseRemoteRenderedModel>();

            targetModel.ModelStateChange += TargetModel_OnModelStateChange;
            TargetModel_OnModelStateChange(targetModel.CurrentModelState);
        }

        private void TargetModel_OnModelStateChange(ModelState state)
        {
            switch (state)
            {
                case ModelState.Loaded:
                    QueryBounds();
                    break;
                default:
                    BoundsBoxCollider.enabled = false;
                    CurrentBoundsState = RemoteBoundsState.NotReady;
                    break;
            }
        }

        // Create a query using the model entity
        private void QueryBounds()
        {
            //Implement me
        }

        // Check the result and apply it to the local Unity bounding box if it was successful
        private void ProcessQueryResult(BoundsQueryAsync remoteBounds)
        {
            //Implement me
        }
    }
    ```

    > [!NOTE]
    > Si vous voyez une erreur dans Visual Studio qui affirme que *La fonctionnalité « X » n’est pas disponible en C# 6. Utilisez la version de langage 7.0 ou une version supérieure*, vous pouvez ignorer cette erreur sans risque. Elle est liée à la génération de solution et de projet Unity.

    Ce script doit être ajouté au même GameObject que le script qui implémente **BaseRemoteRenderedModel**. Dans ce cas, il s’agit de **RemoteRenderedModel**. Comme dans les scripts précédents, ce code initial gère tous les changements d’état, événements et autres données en rapport avec les limites distantes.

    Il reste deux méthodes à implémenter : **QueryBounds** et **ProcessQueryResult**. **QueryBounds** récupère (fetch) les limites, tandis que **ProcessQueryResult** applique le résultat de la requête au **BoxCollider** local.

    La méthode **QueryBounds** est simple : il s’agit d’envoyer une requête à la session de rendu à distance et d’écouter l’événement `Completed`.

1. Remplacez la méthode **QueryBounds** par la méthode complète suivante :

    ```csharp
    // Create a query using the model entity
    private void QueryBounds()
    {
        remoteBoundsQuery = targetModel.ModelEntity.QueryLocalBoundsAsync();
        CurrentBoundsState = RemoteBoundsState.Updating;
        remoteBoundsQuery.Completed += ProcessQueryResult;
    }
    ```

    **ProcessQueryResult** s’avère tout aussi simple. Nous vérifierons le résultat pour voir si elle a abouti. Si c’est le cas, nous convertirons et appliquerons les limites retournées dans un format que le **BoxCollider** peut accepter.    

1. Remplacez la méthode **ProcessQueryResult** par la méthode complète suivante :

    ```csharp
    // Check the result and apply it to the local Unity bounding box if it was successful
    private void ProcessQueryResult(BoundsQueryAsync remoteBounds)
    {
        if (remoteBounds.IsRanToCompletion)
        {
            var newBounds = remoteBounds.Result.toUnity();
            BoundsBoxCollider.center = newBounds.center;
            BoundsBoxCollider.size = newBounds.size;
            BoundsBoxCollider.enabled = true;
            CurrentBoundsState = RemoteBoundsState.Ready;
        }
        else
        {
            CurrentBoundsState = RemoteBoundsState.Error;
        }
    }
    ```

Désormais, quand le script **RemoteBounds** est ajouté au même objet de jeu que **RemoteRenderedModel**, un **BoxCollider** est ajouté si nécessaire et quand le modèle atteint son état `Loaded`, les limites sont automatiquement interrogées et appliquées au **BoxCollider**.

1. En utilisant le GameObject **TestModel** créé précédemment, ajoutez le composant **RemoteBounds**.
1. Vérifiez que le script est ajouté.

     ![Ajouter un composant RemoteBounds](./media/remote-bounds-script.png)

1. Exécutez de nouveau l'application. Peu de temps après le chargement du modèle, les limites de l’objet distant s’affichent. Vous obtenez quelque chose de semblable aux valeurs ci-dessous :

     ![Capture d’écran montrant l’exemple de limites de l’objet distant.](./media/updated-bounds.png)

Nous disposons à présent d’un **BoxCollider** local configuré avec des limites précises au niveau de l’objet Unity. Les limites autorisent la visualisation et l’interaction avec les mêmes stratégies que nous utiliserions pour un objet rendu localement. Tel est le cas, par exemple, des scripts qui modifient la transformation, la physique, etc.

## <a name="move-rotate-and-scale"></a>Déplacer, faire pivoter et mettre à l’échelle  

Le déplacement, la rotation et la mise à l’échelle d’objets rendus à distance fonctionnent de la même façon que pour n’importe quel autre objet Unity. Le **RemoteRenderingCoordinator**, dans sa méthode `LateUpdate`, appelle `Update` dans la session active du moment. La fonction de `Update` est notamment de synchroniser les transformations d’entités de modèle local avec leurs homologues distants. Pour déplacer, faire pivoter ou mettre à l’échelle un modèle rendu à distance, il vous suffit de déplacer, faire pivoter ou mettre à l’échelle la transformation du GameObject représentant le modèle distant. Ici, nous allons modifier la transformation du GameObject parent auquel le script **RemoteRenderedModel** est attaché.

Ce tutoriel utilise MRTK pour l’interaction des objets. La majeure partie de l’implémentation spécifique de MRTK pour le déplacement, la rotation et la mise à l’échelle d’un objet dépasse le cadre de ce tutoriel. Il existe un contrôleur d’affichage de modèle qui est préconfiguré à l’intérieur de **AppMenu**, dans le menu **Model Tools** (Outils de modèle).

1. Vérifiez que le GameObject **TestModel** créé précédemment se trouve dans la scène.
1. Vérifiez que le prefab **AppMenu** se trouve dans la scène.
1. Appuyez sur le bouton Play de Unity pour lire la scène, puis ouvrez **Model Tools** (Outils de modèle) à l’intérieur de **AppMenu**.
![Contrôleur d’affichage](./media/model-with-view-controller.png)

**AppMenu** contient un sous-menu **Model Tools** (Outils de modèle) qui implémente un contrôleur d’affichage pour établir une liaison avec le modèle. Quand le GameObject contient un composant **RemoteBounds**, le contrôleur d’affichage ajoute un composant [**BoundingBox**](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/README_BoundingBox.html), qui est un composant MRTK qui affiche un cadre englobant autour d’un objet assorti d’un **BoxCollider**. Un [**ObjectManipulator**](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.5.1/api/Microsoft.MixedReality.Toolkit.UI.ObjectManipulator.html), qui est responsable des interactions avec la main. Ces scripts combinés nous permettront de déplacer, faire pivoter et mettre à l’échelle le modèle rendu à distance.

1. Déplacez votre souris jusqu’au panneau Game (Jeu), puis cliquez à l’intérieur de celui-ci pour lui attribuer le focus.
1. En utilisant la [simulation de main de MRTK](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/InputSimulation/InputSimulationService.html#hand-simulation), maintenez la touche Maj gauche enfoncée.
1. Orientez la main simulée de telle sorte que son rayon pointe vers le modèle de test.

    ![Rayon de main pointé](./media/handray-engine.png)

1. Maintenez le clic gauche et faites glisser le modèle pour le déplacer.

Le contenu rendu à distance doit alors se déplacer avec le cadre englobant. Vous remarquerez peut-être un léger contretemps ou décalage entre le cadre englobant et le contenu distant. Ce contretemps est lié à la latence et la bande passante de votre connexion Internet.

## <a name="ray-cast-and-spatial-queries-of-remote-models"></a>Raycast et requêtes spatiales de modèles distants

L’utilisation d’un BoxCollider autour des modèles convient pour interagir avec le modèle entier, mais il n’est pas suffisamment détaillé pour interagir avec les différentes parties d’un modèle. Pour résoudre ce problème, nous pouvons utiliser le [raycasting à distance](../../../overview/features/spatial-queries.md#ray-casts). Le raycasting à distance est une API fournie par Azure Remote Rendering destinée à lancer des rayons dans la scène distante et à retourner les résultats de hits localement. Cette technique peut être utilisée pour sélectionner les entités enfant d’un modèle volumineux ou pour obtenir des informations sur les résultats de hits comme la position, la normale de surface et la distance.

Le modèle de test présente un certain nombre de sous-entités qui peuvent être interrogées et sélectionnées. Pour le moment, la sélection génère le nom de l’entité sélectionnée dans la console Unity. Consultez le chapitre [Matériaux, éclairage et effets](../materials-lighting-effects/materials-lighting-effects.md#highlighting-and-outlining) pour mettre en évidence l’entité sélectionnée.

Pour commencer, créons un wrapper statique autour des requêtes de raycast distant. Ce script accepte une position et une direction dans l’espace Unity, le convertit en types de données acceptés par le raycast distant et retourne les résultats. Le script utilise l’API `RayCastQueryAsync`.

1. Créez un script sous le nom **RemoteRayCaster** et remplacez son contenu par le code suivant :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System.Linq;
    using System.Threading.Tasks;
    using UnityEngine;

    /// <summary>
    /// Wraps the Azure Remote Rendering RayCast queries to easily send requests using Unity data types
    /// </summary>
    public class RemoteRayCaster
    {
        public static double maxDistance = 30.0;

        public static async Task<RayCastHit[]> RemoteRayCast(Vector3 origin, Vector3 dir, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            if(RemoteRenderingCoordinator.instance.CurrentCoordinatorState == RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected)
            {
                var rayCast = new RayCast(origin.toRemotePos(), dir.toRemoteDir(), maxDistance, hitPolicy);
                return await RemoteRenderingCoordinator.CurrentSession.Actions.RayCastQueryAsync(rayCast).AsTask();
            }
            else
            {
                return new RayCastHit[0];
            }
        }

        public static async Task<Entity[]> RemoteRayCastEntities(Vector3 origin, Vector3 dir, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            var hits = await RemoteRayCast(origin, dir, hitPolicy);
            return hits.Select(hit => hit.HitEntity).Where(entity => entity != null).ToArray();
        }
    }
    ```

    > [!NOTE]
    > Unity dispose d’une classe nommée [**RaycastHit**](https://docs.unity3d.com/ScriptReference/RaycastHit.html) et Azure Remote Rendering d’une classe nommée [**RayCastHit**](/dotnet/api/microsoft.azure.remoterendering.raycasthit). Le **C** majuscule est une différence importante destinées à éviter les erreurs de compilation.

    **RemoteRayCaster** offre un point d’accès commun pour lancer des rayons distants dans la session active. Pour être plus précis, nous implémenterons ensuite un gestionnaire de pointeur MRTK. Le script implémentera l’interface `IMixedRealityPointerHandler`, qui indiquera à MRTK que ce script doit écouter les événements de [pointeur de réalité mixte](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/Input/Pointers.html).

1. Créez un script sous le nom **RemoteRayCasterPointerHandler** et remplacez le code par le code suivant :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.MixedReality.Toolkit.Input;
    using System;
    using System.Linq;
    using System.Threading.Tasks;
    using UnityEngine;

    public class RemoteRayCastPointerHandler : BaseRemoteRayCastPointerHandler, IMixedRealityPointerHandler
    {
        public UnityRemoteEntityEvent OnRemoteEntityClicked = new UnityRemoteEntityEvent();

        public override event Action<Entity> RemoteEntityClicked;

        public void Awake()
        {
            // Forward events to Unity events
            RemoteEntityClicked += (entity) => OnRemoteEntityClicked?.Invoke(entity);
        }

        public async void OnPointerClicked(MixedRealityPointerEventData eventData)
        {
            if (RemoteEntityClicked != null) //Ensure someone is listening before we do the work
            {
                var firstHit = await PointerDataToRemoteRayCast(eventData.Pointer);
                if (firstHit.success)
                    RemoteEntityClicked.Invoke(firstHit.hit.HitEntity);
            }
        }

        public void OnPointerDown(MixedRealityPointerEventData eventData) { }

        public void OnPointerDragged(MixedRealityPointerEventData eventData) { }

        public void OnPointerUp(MixedRealityPointerEventData eventData) { }

        private async Task<(bool success, RayCastHit hit)> PointerDataToRemoteRayCast(IMixedRealityPointer pointer, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            RayCastHit hit;
            var result = pointer.Result;
            if (result != null)
            {
                var endPoint = result.Details.Point;
                var direction = pointer.Rays[pointer.Result.RayStepIndex].Direction;
                Debug.DrawRay(endPoint, direction, Color.green, 0);
                hit = (await RemoteRayCaster.RemoteRayCast(endPoint, direction, hitPolicy)).FirstOrDefault();
            }
            else
            {
                hit = new RayCastHit();
            }
            return (hit.HitEntity != null, hit);
        }
    }
    ```

La méthode `OnPointerClicked` de **RemoteRayCastPointerHandler** est appelée par MRTK quand un pointeur « clique » sur un collider, comme notre BoxCollider. Après cela, `PointerDataToRemoteRayCast` est appelé pour convertir le résultat du pointeur en un point et une direction. Le point et la direction en question sont ensuite utilisés pour lancer un rayon distant dans la session à distance.

![Limites mises à jour](./media/raycast-local-remote.png)

L’envoi de demandes de raycasting sur clic est une stratégie efficace pour interroger les objets distants. Or, il ne s’agit pas d’une expérience utilisateur idéale, car le curseur entre en collision avec le BoxCollider, et non avec le modèle lui-même.

Vous pouvez aussi créer un nouveau pointeur MRTK qui lance ses rayons dans la session à distance de manière plus fréquente. Plus complexe, cette approche est néanmoins de nature à améliorer l’expérience utilisateur. Même si cette stratégie sort du cadre de ce tutoriel, vous trouverez un exemple de cette approche dans l’application Showcase, qui se trouve dans le [dépôt d’exemples ARR](https://github.com/Azure/azure-remote-rendering/tree/master/Unity/Showcase).

Quand un raycast aboutit dans le **RemoteRayCastPointerHandler**, l’élément `Entity` atteint est émis à partir de l’événement Unity `OnRemoteEntityClicked`. Pour répondre à cet événement, nous allons créer un script d’assistance qui accepte l’élément `Entity` et effectue une action sur celui-ci. Commençons par obtenir le script pour imprimer le nom de l’élément `Entity` dans le journal de débogage.

1. Créez un script sous le nom **RemoteEntityHelper** et remplacez son contenu par le suivant :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.
    
    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using UnityEngine;
    
    public class RemoteEntityHelper : MonoBehaviour
    {
        public void EntityToDebugLog(Entity entity)
        {
            Debug.Log(entity.Name);
        }
    }
    ```

2. Dans le GameObject **TestModel** créé précédemment, ajoutez le composant **RemoteRayCastPointerHandler** et le composant **RemoteEntityHelper**.
1. Affectez la méthode `EntityToDebugLog` à l’événement `OnRemoteEntityClicked`. Quand le type de sortie de l’événement et le type d’entrée de la méthode correspondent, nous pouvons utiliser la connexion d’événements dynamique de Unity, qui passera automatiquement la valeur de l’événement dans la méthode.
    1. Créez un champ de rappel. ![Ajouter le rappel](./media/add-callback-remote-entity-clicked.png)
    1. Faites glisser le composant **Remote Entity Helper** vers le champ Object pour référencer le GameObject parent. ![Attribuer l’objet](./media/assign-object.png)
    1. Attribuez `EntityToDebugLog` comme rappel. ![Attribuer le rappel](./media/remote-entity-event.png)
1. Appuyez sur Play (Lecture) dans l’éditeur Unity pour démarrer la scène, connectez-vous à une session à distance et chargez le modèle de test.
1. En utilisant la simulation de main de MRTK, maintenez la touche Maj gauche enfoncée.
1. Orientez la main simulée de telle sorte que son rayon pointe vers le modèle de test.
1. Cliquez longuement pour simuler un clic aérien (« air-tap »), en exécutant l’événement `OnPointerClicked`.
1. Recherchez dans la console Unity un message de journal contenant le nom de l’entité enfant sélectionnée. Par exemple : ![Exemple d’entité enfant](./media/child-entity-example.png)

## <a name="synchronizing-the-remote-object-graph-into-the-unity-hierarchy"></a>Synchronisation du graphe d’objets distants dans la hiérarchie Unity

Jusqu’à présent, nous ne nous sommes intéressés qu’à un GameObject local unique représentant le modèle entier. Il est utile pour le rendu et la manipulation du modèle entier. En revanche, si nous voulons appliquer des effets ou manipuler des sous-entités spécifiques, nous devrons créer des GameObjects locaux pour représenter ces entités. Tout d’abord, nous pouvons explorer manuellement le modèle de test.

1. Démarrez la scène et chargez le modèle de test.
1. Développez les enfants du GameObject **TestModel** dans la hiérarchie de Unity et sélectionnez le GameObject **TestModel_Entity**.
1. Dans l’inspecteur, cliquez sur le bouton *Show Children* (Afficher les enfants).
![Show Children](./media/show-remote-children.png) (Afficher les enfants)
1. Continuez de développer les enfants dans la hiérarchie en cliquant sur *Show Children* jusqu’à obtenir une longue liste d’enfants.
![Tous les enfants](./media/test-model-children.png)

Une liste constituée de plusieurs dizaines d’entités peuple maintenant la hiérarchie. Si vous sélectionnez l’une d’elles, les composants `Transform` et `RemoteEntitySyncObject` s’affichent dans l’inspecteur. Par défaut, chaque entité n’est pas automatiquement synchronisée avec chaque frame. De ce fait, les modifications locales apportées à `Transform` ne sont pas synchronisées avec le serveur. Vous pouvez cocher *Sync Every Frame* (Synchroniser chaque frame) pour ensuite déplacer, mettre à l’échelle ou faire pivoter la transformation dans la vue Scene ; le modèle rendu ne s’affiche pas dans la vue Scene, mais dans la vue Game, où la position et la rotation du modèle se mettent à jour visuellement.

Le même processus peut être effectué par programmation et correspond à la première étape de la procédure de modification d’entités distantes spécifiques.

1. Modifiez le script **RemoteEntityHelper** pour qu’il contienne aussi la méthode suivante :

    ```csharp
    public void MakeSyncedGameObject(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var sync = entityGameObject.GetComponent<RemoteEntitySyncObject>();
        sync.SyncEveryFrame = true;
    }
    ```

1. Ajoutez un rappel supplémentaire à l’événement **RemoteRayCastPointerHandler** `OnRemoteEntityClicked` en le définissant sur `MakeSyncedGameObject`.
![Rappel supplémentaire](./media/additional-callback.png)
1. En utilisant la simulation de main de MRTK, maintenez la touche Maj gauche enfoncée.
1. Orientez la main simulée de telle sorte que son rayon pointe vers le modèle de test.
1. Cliquez longuement pour simuler un clic aérien (« air-tap »), en exécutant l’événement `OnPointerClicked`.
1. Cochez et développez la hiérarchie pour afficher un nouvel objet enfant représentant l’entité sur laquelle vous avez cliqué.
![Représentation du GameObject](./media/gameobject-representing-entity.png)
1. Après le test, supprimez le rappel pour `MakeSyncedGameObject`, car nous allons l’incorporer par la suite dans d’autres effets.

> [!NOTE]
> La synchronisation de chaque frame est nécessaire uniquement quand vous avez besoin de synchroniser les données de transformation. La synchronisation de transformations entraînant une certaine surcharge, il est conseillé de l’utiliser avec parcimonie.

Créer une instance locale et faire en sorte qu’elle se synchronise automatiquement est la première chose à faire pour manipuler des sous-entités. Les techniques que nous avons employées pour manipuler le modèle dans son ensemble valent aussi pour les sous-entités. Par exemple, après avoir créé une instance locale synchronisée d’une entité, vous pouvez interroger ses limites et ajouter des gestionnaires de manipulation pour permettre son déplacement par les rayons de main de l’utilisateur.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez désormais manipuler vos modèles rendus à distance et interagir avec eux ! Dans le prochain tutoriel, nous allons aborder la modification des matériaux, la modification de l’éclairage et l’application d’effets aux modèles rendus à distance.

> [!div class="nextstepaction"]
> [Étape suivante : Affinage des matériaux, de l’éclairage et des effets](../materials-lighting-effects/materials-lighting-effects.md)