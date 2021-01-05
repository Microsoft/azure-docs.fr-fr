---
title: 'Tutoriel : Créer une application HoloLens Unity'
description: Dans ce tutoriel, vous allez apprendre à créer une application HoloLens Unity avec Azure Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: tutorial
ms.service: azure-spatial-anchors
ms.openlocfilehash: ee0bf9b4ce009f37dd1931d4ed030defa24e7d38
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95996256"
---
# <a name="tutorial-step-by-step-instructions-to-create-a-new-hololens-unity-app-using-azure-spatial-anchors"></a>Tutoriel : Instructions pas à pas pour créer une application HoloLens Unity avec Azure Spatial Anchors

Ce tutoriel vous montre comment créer une application HoloLens Unity avec Azure Spatial Anchors.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, veillez à disposer des éléments suivants :

1. Une machine Windows dotée de <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017+</a> avec la charge de travail de **développement pour la plateforme Windows universelle** et le composant **SDK Windows 10 (version 10.0.18362.0 ou plus récente)** , et <a href="https://git-scm.com/download/win" target="_blank">Git for Windows</a>.
2. L’[extension Visual Studio (VSIX) C++/WinRT](https://aka.ms/cppwinrt/vsix) pour Visual Studio doit être installée à partir de [Visual Studio Marketplace](https://marketplace.visualstudio.com/).
3. Un appareil HoloLens avec le [mode développeur](/windows/mixed-reality/using-visual-studio) activé. L’appareil utilisé dans cet article est un HoloLens doté de la [mise à jour Windows 10 de mai 2020](/windows/mixed-reality/whats-new/release-notes-may-2020). Pour mettre à jour votre appareil HoloLens vers la dernière version, ouvrez l’application **Paramètres**, accédez à **Mise et sécurité**, puis sélectionnez le bouton **Vérifier les mises à jour**.

## <a name="getting-started"></a>Prise en main

Nous allons tout d’abord configurer notre projet et la scène Unity :
1. Démarrez Unity.
2. Sélectionnez **Nouveau**.
4. Vérifiez que l’option **3D** est sélectionnée.
5. Nommez votre projet et renseignez **Location** (Emplacement) pour enregistrer le projet.
6. Sélectionnez **Créer un projet**.
7. Enregistrez la scène par défaut vide dans un nouveau fichier via : **File** > **Save As**.
8. Nommez la nouvelle scène **Main** et appuyez sur le bouton **Save**.

**Configurer les paramètres du projet**

Nous allons maintenant définir les paramètres du projet Unity qui nous permettent de cibler le kit SDK Windows Holographique pour le développement.

Tout d’abord, définissons les paramètres de qualité de notre application.
1. Sélectionnez **Edit** > **Project Settings** > **Quality**
2. Dans la colonne située sous le logo **Windows Store**, cliquez sur la flèche de la ligne **Default** et sélectionnez **Very low**. Vous savez que le paramètre est correctement appliqué lorsque la zone dans la colonne **Windows Store** et la ligne **Very Low** est verte.

Nous devons configurer notre application Unity avec une vue immersive, et non pas une vue 2D. Nous pouvons créer une vue immersive en activant la prise en charge de la réalité virtuelle sur Unity ciblant le SDK Windows 10.
1. Accédez à **Edit** > **Project Settings** > **Player**.
2. Dans le panneau **Inspector** (Inspecteur) pour **Player Settings** (Paramètres du lecteur), sélectionnez l’icône **Windows**.
3. Développez le groupe **XR Settings**.
4. Dans la section **Rendering** (Rendu), cochez la case **Virtual Reality Supported** (Réalité virtuelle prise en charge) pour ajouter une nouvelle liste **Virtual Reality SDKs** (SDK de réalité virtuelle).
5. Vérifiez que **Windows Mixed Reality** apparaît dans la liste. Dans le cas contraire, sélectionnez le bouton **+** en bas de la liste et choisissez **Windows Mixed Reality**.

> [!NOTE]
> Si vous ne voyez pas l’icône Windows, vérifiez que vous avez bien sélectionné Windows .NET Scripting Backend avant l’installation. Si ce n’est pas le cas, vous devrez peut-être réinstaller Unity avec l’installation de Windows appropriée.

**Vérifier la configuration du serveur principal de script**
1. Accédez à **Edit** > **Project Settings** > **Player** (**Player** est peut-être resté ouvert depuis l’étape précédente).
2. Dans **Inspector Panel** pour **Player Settings**, sélectionnez l’icône **Windows Store**.
3. Dans la section Configuration **Autres paramètres**, assurez-vous que l’option **Serveur principal de script** est définie sur **IL2CPP**.

**Définir les fonctionnalités**
1. Accédez à **Edit** > **Project Settings** > **Player** (**Player** est peut-être resté ouvert depuis l’étape précédente).
2. Dans **Inspector Panel** pour **Player Settings**, sélectionnez l’icône **Windows Store**.
3. Dans **Publishing Settings** de la section Configuration, cochez les options **InternetClientServer** et **SpatialPerception**.

**Configurer la caméra virtuelle principale**
1. Dans **Hierarchy Panel**, sélectionnez **Main Camera**.
2. Dans **Inspector**, définissez sa position de transformation sur **0,0,0**.
3. Recherchez la propriété **Clear Flags** et remplacez **Skybox** par **Solid Color** dans la liste déroulante.
4. Cliquez sur le champ **Background** pour ouvrir un sélecteur de couleurs.
5. Définissez **R, G, B et A** sur **0**.
6. Sélectionnez **Add Component**, puis recherchez et ajoutez **Spatial Mapping Collider**.

**Créer notre script**
1. Dans le volet **Project**, créez un dossier, **Scripts**, sous le dossier **Assets**.
2. Cliquez avec le bouton droit sur le dossier, puis sélectionnez **Créer >** , **Script C#** . Intitulez-le **AzureSpatialAnchorsScript**.
3. Accédez à **GameObject** -> **Create Empty**.
4. Sélectionnez-le, puis dans **Inspector** renommez-le depuis **GameObject** pour **MixedRealityCloud**. Sélectionnez **Add Component**, puis recherchez et ajoutez le script **AzureSpatialAnchorsScript**.

**Créer le Prefab de la sphère**
1. Accédez à **GameObject** -> **3D Object** -> **Sphere**.
2. Dans **Inspector**, définissez son échelle à **0.25, 0.25, 0.25**.
3. Recherchez l’objet **Sphere** dans le volet **Hierarchy**. Cliquez dessus et faites-le glisser dans le dossier **Assets** du volet **Project**.
4. Cliquez avec le bouton droit et **supprimez** la sphère d’origine que vous avez créée dans le volet **Hierarchy**.

Vous devez désormais avoir un Prefab de sphère dans votre volet **Project**.

## <a name="trying-it-out"></a>Essai
Pour vérifier que tout fonctionne correctement, générez votre application **Unity** et déployez-la à partir de **Visual Studio**. Suivez le chapitre 6 dans [**Réalité mixte - Principes fondamentaux - Cours 100 : Bien démarrer avec Unity**](/windows/mixed-reality/holograms-100#chapter-6---build-and-deploy-to-device-from-visual-studio) pour cela. Vous devez voir l’écran de démarrage Unity, puis un écran clair.

## <a name="place-an-object-in-the-real-world"></a>Placer un objet dans le monde réel
Vous allez maintenant créer et placer un objet à l’aide de votre application. Ouvrez la solution Visual Studio que nous avons créée lorsque nous [avons déployé notre application](#trying-it-out).

Pour commencer, ajoutez les importations suivantes à `Assembly-CSharp (Universal Windows)\Scripts\AzureSpatialAnchorsScript.cs` :

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=19-24)]

Ajoutez ensuite les variables membres suivantes à votre classe `AzureSpatialAnchorsScript` :

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=26-47,53-57,65-84)]

Avant de continuer, nous devons définir le Prefab de sphère que nous avons créé dans notre variable membre spherePrefab. Revenez à **Unity**.
1. Dans **Unity**, sélectionnez l’objet **MixedRealityCloud** dans le volet **Hierarchy**.
2. Cliquez sur le Prefab **Sphere** que vous avez enregistré dans le volet **Project**. Faites glisser l’objet **Sphere** sur lequel vous avez cliqué dans la zone **Sphere Prefab** sous **Azure Spatial Anchors Script (Script)** dans le volet **Inspector**.

L’objet **Sphere** doit maintenant être défini en tant que Prefab dans votre script. Effectuez une build à partir de **Unity**, puis rouvrez la solution **Visual Studio** résultante, comme vous venez de le faire dans [Essai](#trying-it-out).

Dans **Visual Studio**, rouvrez `AzureSpatialAnchorsScript.cs`. Ajoutez le code suivant à votre méthode `Start()`. Ce code raccorde `GestureRecognizer`, qui appelle `HandleTap` quand il détecte un clic aérien (air tap).

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=86-95,98&highlight=4-10)]

Nous devons à présent ajouter la méthode `HandleTap()` suivante sous `Update()`. Un raycast va être effectué pour obtenir un point d’impact sur lequel placer une sphère.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=273-283,305-306,310-318)]

Nous devons maintenant créer la sphère. La sphère est blanche au départ, mais cette valeur est ajustée par la suite. Ajoutez la méthode `CreateAndSaveSphere()` suivante :

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=320-331,396)]

Exécutez votre application à partir de **Visual Studio** pour la valider à nouveau. Cette fois-ci, appuyez sur l’écran pour créer et placer votre sphère blanche sur la surface de votre choix.

## <a name="set-up-the-dispatcher-pattern"></a>Configurer le modèle de répartiteur

Quand vous travaillez avec Unity, toutes les API Unity (comme celles que vous utilisez pour effectuer des mises à jour de l’interface utilisateur), doivent se produire sur le thread principal. Dans le code que nous allons écrire, par contre, nous obtiendrons des rappels sur d’autres threads. Comme nous voulons mettre à jour l’interface utilisateur dans ces rappels, nous avons besoin d’un moyen pour passer d’un thread secondaire au thread principal. Pour exécuter du code sur le thread principal à partir d’un thread secondaire, nous allons utiliser le modèle de répartiteur.

Ajoutons une variable membre, `dispatchQueue`, qui est une File d’attente d’Actions. Nous allons envoyer (push) des Actions sur la file d’attente, puis les retirer de la file d’attente pour les exécuter sur le thread principal.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=43-56&highlight=6-9)]

Adjoignons ensuite un moyen permettant d’ajouter une Action à la File d’attente. Ajoutons `QueueOnUpdate()` juste après `Update()` :

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=112-122)]

Nous pouvons utiliser la boucle Update() pour vérifier l’existence d’une Action en file d’attente. Si tel est le cas, nous la retirerons de la file d’attente et nous l’exécuterons.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=100-110&highlight=4-10)]

## <a name="get-the-azure-spatial-anchors-sdk"></a>Obtenir le kit SDK Azure Spatial Anchors

## <a name="via-unity-package-manager-upm-package"></a>[À l’aide du package UPM (Unity Package Manager)](#tab/UPMPackage)

Cette méthode est compatible avec Unity versions 2019.1 et ultérieures.

### <a name="add-the-registry-to-your-unity-project"></a>Ajouter le registre à votre projet Unity

1. Dans un explorateur de fichiers, accédez au dossier `Packages` de votre projet Unity. Ouvrez le fichier de manifeste du projet, `manifest.json`, dans un éditeur de texte.
2. En haut du fichier, au même niveau que la section `dependencies`, ajoutez l’entrée suivante pour inclure le registre Azure Spatial Anchors à votre projet. L’entrée `scopedRegistries` indique à Unity où rechercher les packages du SDK Azure Spatial Anchors.

    [!code-json[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-unity-scoped-registry-setup.md?range=9-19&highlight=2-10)]

### <a name="add-the-sdk-package-to-your-unity-project"></a>Ajouter le package du SDK à votre projet Unity

1. Ajoutez une entrée avec le nom du package du SDK Windows Azure Spatial Anchors (`com.microsoft.azure.spatial-anchors-sdk.windows`) et la version du package à la section `dependencies` du manifeste de votre projet. Voir l’exemple ci-dessous.

    [!code-json[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-unity-scoped-registry-setup.md?range=9-20&highlight=12)]

2. Enregistrez et fermez le fichier `manifest.json`. Quand vous revenez à Unity, Unity doit détecter automatiquement le changement de manifeste du projet et récupérer les packages spécifiés. Vous pouvez développer le dossier `Packages` dans votre vue du projet pour vérifier que les packages appropriés ont été importés.

## <a name="via-unity-asset-package"></a>[À l’aide du package de ressources Unity](#tab/UnityAssetPackage)

> [!WARNING]
> La distribution du package de ressources Unity du SDK Azure Spatial Anchors sera dépréciée après le SDK version 2.5.0.

Téléchargeons maintenant le SDK Azure Spatial Anchors. Accédez à la [page des versions Azure Spatial Anchors de GitHub](https://github.com/Azure/azure-spatial-anchors-samples/releases). Sous **Assets** (Ressources), téléchargez le fichier **AzureSpatialAnchors.unitypackage**. Dans Unity, accédez à **Assets** (Ressources), sélectionnez **Import Package** (Importer un package) > **Custom Package...** (Package personnalisé...). Accédez au package et sélectionnez **Open**.

Dans la nouvelle fenêtre **Import Unity Package** (Importer le package Unity) qui s’affiche, désélectionnez **Plug-ins**, puis sélectionnez **Import** (Importer) en bas à droite.

---

Dans votre solution **Visual Studio**, ajoutez l’importation suivante à votre `<ProjectName>\Assets\Scripts\AzureSpatialAnchorsScript.cs` :

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=18-21&highlight=1)]

Ajoutez ensuite les variables de membre suivantes dans votre classe `AzureSpatialAnchorsScript` :

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=53-68&highlight=6-11)]

## <a name="attach-a-local-azure-spatial-anchor-to-the-local-anchor"></a>Attacher une ancre spatiale Azure locale à l’ancre locale

Configurons CloudSpatialAnchorSession d’Azure Spatial Anchor. Nous allons commencer par ajouter la méthode `InitializeSession()` suivante à l’intérieur de votre classe `AzureSpatialAnchorsScript`. Une fois appelée, elle assurera la création et l’initialisation d’une session Azure Spatial Anchors pendant le démarrage de votre application.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=179-208,211-215)]

Nous devons maintenant écrire du code pour gérer les appels de délégué. Nous leur en ajouterons d’autres au fur et à mesure.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=217-232)]

À présent, raccordez la méthode `initializeSession()` dans la méthode `Start()`.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=86-98&highlight=12)]

Enfin, ajoutez le code suivant à votre méthode `CreateAndSaveSphere()`. Ce code a pour effet d’attacher une ancre spatiale Azure locale à la sphère que vous placez dans le monde réel.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=320-344,396&highlight=14-25)]

Avant de poursuivre, vous devez créer un compte Azure Spatial Anchors pour obtenir l’identificateur, la clé et le domaine du compte. Si vous n’avez pas encore ces valeurs, suivez la prochaine section pour les obtenir.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="upload-your-local-anchor-into-the-cloud"></a>Charger votre ancre locale dans le cloud

Une fois que vous avez l’identificateur, la clé et le domaine de votre compte Azure Spatial Anchors, affichez puis collez `Account Id` dans `SpatialAnchorsAccountId`, `Account Key` dans `SpatialAnchorsAccountKey` et `Account Domain` dans `SpatialAnchorsAccountDomain`.

Enfin, raccordez tout ensemble. Dans votre méthode `CreateAndSaveSphere()`, ajoutez le code suivant. Il appelle la méthode `CreateAnchorAsync()` aussitôt que la sphère est créée. Une fois que la méthode retourne un résultat, le code met à jour votre sphère une dernière fois, en lui donnant la couleur bleue.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=320-397&highlight=26-77)]

Exécutez une nouvelle fois votre application à partir de **Visual Studio**. Bougez votre tête, puis faites un geste d’appui (air tap) pour placer votre sphère. Une fois que le nombre d’images est suffisant, la sphère devient jaune et le chargement cloud débute. À l’issue du chargement, la sphère passe au bleu. Si vous le souhaitez, vous pouvez aussi utiliser la [fenêtre Sortie](/visualstudio/ide/reference/output-window) pendant le débogage dans **Visual Studio** pour superviser les messages de journal envoyés par votre application. Veillez à déployer la configuration `Debug` de votre application à partir de Visual Studio pour afficher les messages de journal. Vous pouvez regarder `RecommendedForCreateProgress` et, une fois le chargement terminé, vous pouvez voir l’identificateur d’ancre retourné à partir du cloud.

> [!NOTE]
> Si vous obtenez « DllNotFoundException : Impossible de charger la DLL AzureSpatialAnchors : le module spécifié est introuvable. », vous devez **Supprimer** et **Générer** à nouveau votre solution.

## <a name="locate-your-cloud-spatial-anchor"></a>Localiser l’ancre spatiale cloud

Une fois que votre ancre est chargée dans le cloud, vous pouvez essayer de la localiser à nouveau. Ajoutons le code suivant à votre méthode `HandleTap()`. Ce code a pour effet de :

* Appelez `ResetSession()`, ce qui arrête `CloudSpatialAnchorSession` et supprime la sphère bleue existante de l’écran.
* Initialisez `CloudSpatialAnchorSession` de nouveau. Nous procédons ainsi pour nous assurer que l’ancre à localiser provient du cloud, et qu’il ne s’agit pas de l’ancre locale que vous avez créée.
* Créez un **Watcher** qui recherche l’ancre chargée sur Azure Spatial Anchors.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=273-311&highlight=13-31,35-36)]

Ajoutons maintenant nos méthodes `ResetSession()` et `CleanupObjects()`. Vous pouvez les mettre en dessous de `QueueOnUpdate()`

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=124-177)]

Nous devons maintenant raccorder le code qui sera appelé dès que l’ancre recherchée sera localisée. Dans `InitializeSession()`, ajoutez les rappels suivants :

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=206-212&highlight=4-5)]


À présent, ajoutons le code qui crée et place une sphère verte une fois la localisation de CloudSpatialAnchor effectuée. Il réactive également les appuis sur l’écran, ce qui vous permet de répéter une nouvelle fois le scénario dans son intégralité : création, chargement et relocalisation d’une nouvelle ancre.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=234-271)]

Et voilà ! Exécutez une dernière fois votre application depuis **Visual Studio** pour essayer l’ensemble du scénario de bout en bout. Déplacez votre appareil et placez votre sphère blanche. Ensuite, bougez votre tête pour capturer les données de l’environnement jusqu’à ce que la sphère devienne jaune. Votre ancre locale est alors chargée et la sphère devient bleue. Enfin, appuyez une nouvelle fois sur l’écran pour supprimer votre ancre locale et commencer à rechercher son équivalent cloud. Continuez de déplacer votre appareil jusqu’à ce que l’ancre spatiale cloud soit localisée. Une sphère verte doit alors apparaître à l’emplacement approprié, et vous pouvez répéter une nouvelle fois l’ensemble du scénario.

[!INCLUDE [AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md)]