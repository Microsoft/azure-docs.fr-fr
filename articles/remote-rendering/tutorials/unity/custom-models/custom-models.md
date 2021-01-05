---
title: Interfaces et modèles personnalisés
description: Ajouter des contrôleurs d’affichage et ingérer des modèles personnalisés pour qu’ils soient affichés par Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 5793e2958edce0a4c97660a75d0ecefa914c12d2
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96559080"
---
# <a name="tutorial-interfaces-and-custom-models"></a>Tutoriel : Interfaces et modèles personnalisés

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Ajouter Mixed Reality Toolkit au projet
> * Gérer l’état du modèle
> * Configurer le stockage Blob Azure pour l’ingestion de modèles
> * Charger et traiter des modèles pour l’affichage

## <a name="prerequisites"></a>Prérequis

* Ce tutoriel s’appuie sur le [Tutoriel : Affichage d’un modèle rendu à distance](../view-remote-models/view-remote-models.md).

## <a name="get-started-with-the-mixed-reality-toolkit-mrtk"></a>Bien démarrer avec MRTK (Mixed Reality Toolkit)

MRTK (Mixed Reality Toolkit) est un kit de ressources multiplateforme qui permet de générer des expériences de réalité mixte. Nous allons utiliser MRTK 2.5.1 pour ses fonctionnalités d’interaction et de visualisation.

Pour ajouter MRTK, suivez les [étapes nécessaires](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.5.1/Documentation/Installation.html#required) qui sont listées dans le [Guide d’installation de MRTK](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.5.1/Documentation/Installation.html).

à savoir :
 - [Se procurer les tout derniers packages MRTK Unity](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.5.1/Documentation/Installation.html#1-get-the-latest-mrtk-unity-packages)
     - Même si l’article parle des tout derniers packages, utilisez la version 2.5.1 qui est disponible dans la page des versions MRTK.
     - Nous utilisons uniquement le package *Foundation* dans ce tutoriel. Les packages *Extensions*, *Tools* et *Examples* ne sont pas nécessaires.
 - [Importer les packages MRTK dans votre projet Unity](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.5.1/Documentation/Installation.html#2-import-mrtk-packages-into-your-unity-project)
 - [Basculer votre projet Unity vers la plateforme cible](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.5.1/Documentation/Installation.html#3-switch-your-unity-project-to-the-target-platform)
     - Vous devez déjà avoir effectué cette étape dans le premier chapitre, mais c’est le moment de vous en assurer !
 - [Ajouter MRTK à une nouvelle scène ou à un nouveau projet](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.5.1/Documentation/Installation.html#4-add-and-configure-mrtk-with-a-new-scene)
     - Vous pouvez ajouter MRTK à une nouvelle scène et rajouter vos objets/scripts de coordinateur et de modèle, ou vous pouvez ajouter MRTK à votre scène existante au moyen de la commande de menu *Mixed Reality Toolkit -> Add to Scene and Configure* (Ajouter à la scène et configurer).

## <a name="import-assets-used-by-this-tutorial"></a>Importer les ressources utilisées par ce tutoriel

À partir de ce chapitre, nous allons implémenter un schéma [contrôleur-affichage-modèle](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) simple pour la plupart des éléments traités. La partie *modèle* du schéma est le code propre à Azure Remote Rendering, et la gestion des états liés à Azure Remote Rendering. Les parties *contrôleur* et *affichage* du schéma sont implémentées à l’aide de ressources MRTK et de quelques scripts personnalisés. Il est possible d’utiliser le *modèle* dans ce tutoriel sans que soient implémentées les parties *contrôleur-affichage* ici. Cette séparation vous permet d’intégrer facilement le code trouvé dans ce tutoriel à votre propre application, dans laquelle il prendra le relais de la partie *contrôleur-affichage* du schéma de conception.

Avec l’introduction de MRTK, il existe un certain nombre de scripts, d’éléments préfabriqués et de ressources qui peuvent désormais être ajoutés au projet pour prendre en charge les interactions et les retours visuels. Ces ressources, appelées **Tutorial Assets (Ressources du tutoriel)** , sont regroupées dans un [Package de ressources Unity](https://docs.unity3d.com/Manual/AssetPackages.html), qui est présent dans le [GitHub d’Azure Remote Rendering](https://github.com/Azure/azure-remote-rendering) à cet emplacement : \Unity\TutorialAssets\TutorialAssets.unitypackage.

1. Clonez ou téléchargez le dépôt Git [Azure Remote Rendering](https://github.com/Azure/azure-remote-rendering) ; si vous choisissez le téléchargement, extrayez le fichier zip à un emplacement connu.
1. Dans votre projet Unity, choisissez *Assets -> Import Package -> Custom Package (Ressources -> Importer un package -> Package personnalisé)* .
1. Dans l’Explorateur de fichiers, accédez au répertoire dans lequel vous avez cloné ou décompressé le dépôt Azure Remote Rendering, puis sélectionnez .unitypackage trouvé dans **Unity -> TutorialAssets -> TutorialAssets.unitypackage**
1. Sélectionnez le bouton **Import** pour importer le contenu du package dans votre projet.
1. Dans l’éditeur Unity, sélectionnez *Mixed Reality Toolkit -> Utilities (Utilitaires) -> Upgrade MRTK Standard Shader for Lightweight Render Pipeline (Mettre à niveau le nuanceur standard MRTK pour le pipeline de rendu léger)* dans la barre de menus supérieure, et suivez les invites pour mettre à niveau le nuanceur.

Une fois le kit MRTK et les ressources du tutoriel intégrés au projet, nous substituerons le profil MRTK pour un profil mieux adapté au tutoriel.

1. Sélectionnez le GameObject **MixedRealityToolkit** dans la hiérarchie de la scène.
1. Dans l’inspecteur, sous le composant **MixedRealityToolkit**, basculez le profil de configuration vers *ARRMixedRealityToolkitConfigurationProfile*.
1. Appuyez sur les touches *Ctrl+S* pour enregistrer vos modifications.

MRTK sera ainsi configuré, principalement, avec les profils HoloLens 2 par défaut. Les profils fournis sont préconfigurés selon les manières suivantes :
 - Désactivez le profileur (Appuyez sur 9 pour l’activer/le désactiver, ou dites « Show/Hide Profiler » (Afficher/masquer le profileur) sur l’appareil).
 - Désactivez le curseur en forme d’œil.
 - Activez les clics de souris Unity, afin de vous servir de la souris, au lieu de la main simulée, pour cliquer sur les éléments d’interface utilisateur MRTK.

## <a name="add-the-app-menu"></a>Ajouter le menu de l’application

La plupart des contrôleurs d’affichage de ce tutoriel fonctionnent sur des classes de base abstraites plutôt que sur des classes concrètes. Ce modèle offre plus de flexibilité et nous permet de vous fournir les contrôleurs d’affichage tout en continuant de vous aider dans votre apprentissage du code Azure Remote Rendering. Par souci de simplicité, la classe **RemoteRenderingCoordinator** n’a pas de classe abstraite fournie, et son contrôleur d’affichage fonctionne directement sur la classe concrète.

Vous pouvez maintenant ajouter l’élément préfabriqué **AppMenu** à la scène pour obtenir un retour visuel de l’état de la session en cours. Ce contrôleur d’affichage va « déverrouiller » d’autres contrôleurs d’affichage de sous-menu à mesure que nous implémenterons et intégrerons des fonctionnalités ARR supplémentaires dans la scène. Pour le moment, l’élément **AppMenu** donnera une indication visuelle de l’état ARR et présentera le panneau modal que l’utilisateur utilise pour autoriser l’application à se connecter à ARR.

1. Localisez l’élément préfabriqué **AppMenu** dans *Assets/RemoteRenderingTutorial/Prefabs/AppMenu*.
1. Faites glisser l’élément préfabriqué **AppMenu** dans la scène.
1. Vous verrez probablement une boîte de dialogue de l’**importateur TMP**, car c’est la première fois que vous intégrez des ressources *Text Mesh Pro* à la scène. Suivez les invites pour **importer TMP Essentials**. Fermez ensuite la boîte de dialogue de l’importateur, les exemples et les suppléments ne sont pas nécessaires.
1. **AppMenu** est configuré pour se raccorder automatiquement et fournir le modal de consentement pour la connexion à une session ; nous pouvons donc supprimer le contournement placé précédemment. Dans le GameObject **RemoteRenderingCoordinator**, supprimez le contournement pour l’autorisation que nous avons implémenté auparavant en sélectionnant le bouton « - » dans l’événement **On Requesting Authorization** (Demande d’autorisation).
 ![Supprimer le contournement](./media/remove-bypass-event.png).
1. Testez le contrôleur d’affichage en appuyant sur **Play** (Lecture) dans l’éditeur Unity.
1. Dans l’éditeur, à présent que MRTK est configuré, vous pouvez utiliser les touches WASD pour modifier la position de votre affichage, et maintenir le bouton droit de la souris enfoncé pendant le déplacement de la souris pour modifier la direction de l’affichage. Essayez « d’explorer » un peu la scène pour vous faire une idée des contrôles.
1. Sur l’appareil, vous pouvez lever la paume pour appeler **AppMenu** ; dans l’éditeur Unity, utilisez la touche d’accès rapide « M ».
1. Si vous ne voyez plus le menu, appuyez sur la touche « M » pour le faire apparaître. Le menu sera placé près de la caméra pour faciliter l’interaction.
1. L’autorisation s’affiche à présent sous la forme d’une requête dans la partie droite de **AppMenu** ; à partir de maintenant, vous l’utiliserez pour autoriser l’application à gérer les sessions de rendu à distance.
 ![Autoriser l’IU](./media/authorize-request-ui.png)
1. Arrêtez la lecture de Unity pour continuer le tutoriel.

## <a name="manage-model-state"></a>Gérer l’état du modèle

À présent, nous allons implémenter un nouveau script, **RemoteRenderedModel**, qui est utilisé pour le suivi de l’état, la réponse aux événements, le déclenchement des événements et la configuration. Pour l’essentiel, **RemoteRenderedModel** stocke le chemin distant des données du modèle dans `modelPath`. Il écoute les modifications d’état dans **RemoteRenderingCoordinator** pour voir s’il doit charger ou décharger automatiquement le modèle qu’il définit. Le GameObject auquel est attaché **RemoteRenderedModel** sera le parent local du contenu distant.

Notez que le script **RemoteRenderedModel** implémente **BaseRemoteRenderedModel**, inclus dans les **Tutorial Assets** (Ressources du tutoriel). Cela permettra au contrôleur d’affichage du modèle distant de se lier à votre script.

1. Créez un script nommé **RemoteRenderedModel** dans le même dossier que **RemoteRenderingCoordinator**. Remplacez l’intégralité du contenu par le code suivant :

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;
    using UnityEngine.Events;

    public class RemoteRenderedModel : BaseRemoteRenderedModel
    {
        public bool AutomaticallyLoad = true;

        private ModelState currentModelState = ModelState.NotReady;

        [SerializeField]
        [Tooltip("The friendly name for this model")]
        private string modelDisplayName;
        public override string ModelDisplayName { get => modelDisplayName; set => modelDisplayName = value; }

        [SerializeField]
        [Tooltip("The URI for this model")]
        private string modelPath;
        public override string ModelPath
        {
            get => modelPath.Trim();
            set => modelPath = value;
        }

        public override ModelState CurrentModelState
        {
            get => currentModelState;
            protected set
            {
                if (currentModelState != value)
                {
                    currentModelState = value;
                    ModelStateChange?.Invoke(value);
                }
            }
        }

        public override event Action<ModelState> ModelStateChange;
        public override event Action<float> LoadProgress;
        public override Entity ModelEntity { get; protected set; }

        public UnityEvent OnModelNotReady = new UnityEvent();
        public UnityEvent OnModelReady = new UnityEvent();
        public UnityEvent OnStartLoading = new UnityEvent();
        public UnityEvent OnModelLoaded = new UnityEvent();
        public UnityEvent OnModelUnloading = new UnityEvent();

        public UnityFloatEvent OnLoadProgress = new UnityFloatEvent();

        public void Awake()
        {
            // Hook up the event to the Unity event
            LoadProgress += (progress) => OnLoadProgress?.Invoke(progress);

            ModelStateChange += HandleUnityStateEvents;
        }

        private void HandleUnityStateEvents(ModelState modelState)
        {
            switch (modelState)
            {
                case ModelState.NotReady:  OnModelNotReady?.Invoke();  break;
                case ModelState.Ready:     OnModelReady?.Invoke();     break;
                case ModelState.Loading:   OnStartLoading?.Invoke();   break;
                case ModelState.Loaded:    OnModelLoaded?.Invoke();    break;
                case ModelState.Unloading: OnModelUnloading?.Invoke(); break;
            }
        }

        private void Start()
        {
            //Attach to and initialize current state (in case we're attaching late)
            RemoteRenderingCoordinator.CoordinatorStateChange += Instance_CoordinatorStateChange;
            Instance_CoordinatorStateChange(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        /// <summary>
        /// Listen for state changes on the coordinator, clean up this model's remote objects if we're no longer connected.
        /// Automatically load if required
        /// </summary>
        private void Instance_CoordinatorStateChange(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    CurrentModelState = ModelState.Ready;
                    if (AutomaticallyLoad)
                        LoadModel();
                    break;
                default:
                    UnloadModel();
                    break;
            }
        }

        private void OnDestroy()
        {
            RemoteRenderingCoordinator.CoordinatorStateChange -= Instance_CoordinatorStateChange;
            UnloadModel();
        }

        /// <summary>
        /// Asks the coordinator to create a model entity and listens for coordinator state changes
        /// </summary>
        [ContextMenu("Load Model")]
        public override async void LoadModel()
        {
            if (CurrentModelState != ModelState.Ready)
                return; //We're already loaded, currently loading, or not ready to load

            CurrentModelState = ModelState.Loading;

            ModelEntity = await RemoteRenderingCoordinator.instance?.LoadModel(ModelPath, this.transform, SetLoadingProgress);

            if (ModelEntity != null)
                CurrentModelState = ModelState.Loaded;
            else
                CurrentModelState = ModelState.Error;
        }

        /// <summary>
        /// Clean up the local model instances
        /// </summary>
        [ContextMenu("Unload Model")]
        public override void UnloadModel()
        {
            CurrentModelState = ModelState.Unloading;

            if (ModelEntity != null)
            {
                var modelGameObject = ModelEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
                Destroy(modelGameObject);
                ModelEntity.Destroy();
                ModelEntity = null;
            }

            if (RemoteRenderingCoordinator.instance.CurrentCoordinatorState == RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected)
                CurrentModelState = ModelState.Ready;
            else
                CurrentModelState = ModelState.NotReady;
        }

        /// <summary>
        /// Update the Unity progress event
        /// </summary>
        /// <param name="progressValue"></param>
        public override void SetLoadingProgress(float progressValue)
        {
            LoadProgress?.Invoke(progressValue);
        }
    }
    ```

Pour faire simple, **RemoteRenderedModel** contient les données nécessaires au chargement d’un modèle (dans le cas présent, l’URI SAS ou *builtin://* ) et effectue le suivi de l’état du modèle distant. Au moment du chargement, la méthode `LoadModel` est appelée sur **RemoteRenderingCoordinator** et l’Entité contenant le modèle est retournée pour référence et déchargement.

## <a name="load-the-test-model"></a>Charger le modèle de test

Testons le nouveau script en chargeant à nouveau le modèle de test. Nous allons créer un objet de jeu pour contenir le script et être un parent du modèle de test.

1. Créez un objet de jeu vide dans la scène et nommez-le **TestModel**.
1. Ajoutez le script *RemoteRenderedModel* à **TestModel**.
![Ajouter le composant RemoteRenderedModel](./media/add-remote-rendered-model-script.png)
1. Renseignez `Model Display Name` et `Model Path` en utilisant respectivement « *TestModel* » et « *builtin://Engine* ».
![Renseigner les détails du modèle](./media/add-model-script.png)
1. Placez l’objet **TestModel** devant la caméra, à la position **x = 0, y = 0, z = 3**.
![Positionner l’objet](./media/test-model-position.png)
1. Assurez-vous que  **AutomaticallyLoad** est activé.
1. Appuyez sur **Play** (Lecture) dans l’éditeur Unity pour tester l’application.
1. Accordez l’autorisation en cliquant sur le bouton *Connect* (Connexion) pour permettre à l’application de créer une session ; elle se connectera à une session et chargera automatiquement le modèle.

Regardez la console tandis que l’application progresse dans ses différents états. Gardez à l’esprit que certains états peuvent prendre un certain temps et ne pas afficher de progression. À la fin, vous verrez les journaux de chargement du modèle, puis le modèle de test affiché dans la scène.

Essayez de déplacer et de faire pivoter le GameObject **TestModel** à l’aide de l’outil de transformation dans l’inspecteur, ou dans l’affichage de la scène. Vous verrez le modèle se déplacer et pivoter dans l’affichage du jeu.

![Journal Unity](./media/unity-loading-log.png)

## <a name="provision-blob-storage-in-azure-and-custom-model-ingestion"></a>Provisionner le stockage Blob dans Azure et l’ingestion de modèle personnalisé

Nous pouvons à présent essayer de charger votre propre modèle. Pour ce faire, vous devez configurer le stockage d’objets Blob et, sur Azure, charger puis convertir un modèle pour que nous chargions ensuite ce modèle à l’aide du script **RemoteRenderedModel**. Les étapes de chargement du modèle personnalisé peuvent être ignorées sans risque si vous ne disposez pas de votre propre modèle à charger pour l’instant.

Suivez les étapes indiquées dans le [Guide de démarrage rapide : Convertir un modèle pour le rendu](../../../quickstarts/convert-model.md). Pour les besoins de ce tutoriel, ignorez la section **Insérer un nouveau modèle dans l’exemple d’application de démarrage rapide**. Lorsque vous disposez de l’URI *SAS (Shared Access Signature)* de votre modèle ingéré, passez à l’étape suivante ci-dessous.

## <a name="load-and-rendering-a-custom-model"></a>Charger et afficher un modèle personnalisé

1. Créez un GameObject vide dans la scène et nommez-le comme votre modèle personnalisé.
1. Ajoutez le script *RemoteRenderedModel* au GameObject nouvellement créé.
 ![Ajouter le composant RemoteRenderedModel](./media/add-remote-rendered-model-script.png)
1. Renseignez `Model Display Name` en utilisant un nom approprié pour votre modèle.
1. Renseignez `Model Path` à l’aide de l’URI *SAS (Shared Access Signature)* du modèle que vous avez créé dans les étapes d’ingestion ci-dessus.
1. Placez le GameObject devant la caméra, à la position **x = 0, y = 0, z = 3.**
1. Assurez-vous que  **AutomaticallyLoad** est activé.
1. Appuyez sur **Play** (Lecture) dans l’éditeur Unity pour tester l’application.

    Vous verrez la console commencer à se remplir avec l’état actuel, et à la fin, les messages de progression du chargement du modèle. Votre modèle personnalisé sera ensuite chargé dans la scène.

1. Supprimez de la scène l’objet de votre modèle personnalisé. Une expérience optimale de ce tutoriel est obtenue en utilisant le modèle de test. Bien que plusieurs modèles soient évidemment pris en charge dans ARR, ce tutoriel a été écrit pour optimiser la prise en charge d’un seul modèle distant à la fois.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez désormais charger vos propres modèles dans Azure Remote Rendering et les afficher dans votre application ! Ensuite, nous allons vous guider dans la manipulation de vos modèles.

> [!div class="nextstepaction"]
> [Étape suivante : Manipulation de modèles](../manipulate-models/manipulate-models.md)
