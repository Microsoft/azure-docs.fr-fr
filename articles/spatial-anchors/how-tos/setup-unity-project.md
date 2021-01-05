---
title: Installer des Spatial Anchors Azure pour Unity
description: Configurer un projet Unity pour utiliser des Spatial Anchors Azure
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: how-to
ms.service: azure-spatial-anchors
ms.openlocfilehash: 2330310b0bf4e165af71208477db128650a787cf
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95496939"
---
# <a name="configuring-azure-spatial-anchors-in-a-unity-project"></a>Configurer des Spatial Anchors Azure dans un projet Unity

Ce guide vous montre comment prendre en main le kit SDK Azure Spatial Anchors dans votre projet Unity.

## <a name="requirements"></a>Spécifications

Les Spatial Anchors Azure prennent actuellement en charge Unity 2019.4 (LTS) avec les configurations suivantes.

* Unity 2019.4 avec AR Foundation 3.1 est pris en charge dans les Spatial Anchors Azure 2.4.0+.

## <a name="configuring-a-project"></a>Configuration d’un projet

### <a name="add-the-unity-package-manager-packages-to-your-project"></a>[Ajouter les packages Unity Package Manager à votre projet](#tab/UPMPackage)

Azure Spatial Anchors pour Unity est actuellement distribué à l’aide de packages UPM (Unity Package Manager). Ces packages se trouvent dans notre [registre NPM](https://bintray.com/microsoft/AzureMixedReality-NPM). Pour en savoir plus sur l’utilisation des registres de packages délimités dans un projet Unity, consultez la documentation Unity officielle [ici](https://docs.unity3d.com/Manual/upm-scoped.html).

#### <a name="add-the-registry-to-your-unity-project"></a>Ajouter le registre à votre projet Unity

1. Dans un explorateur de fichiers, accédez au dossier `Packages` de votre projet Unity. Ouvrez le fichier de manifeste du projet, `manifest.json`, dans un éditeur de texte.
2. En haut du fichier, au même niveau que la section `dependencies`, ajoutez l’entrée suivante pour inclure le registre Azure Spatial Anchors à votre projet. L’entrée `scopedRegistries` indique à Unity où rechercher les packages du SDK Azure Spatial Anchors.

    [!code-json[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-unity-scoped-registry-setup.md?range=9-19&highlight=2-10)]

#### <a name="add-the-sdk-packages-to-your-unity-project"></a>Ajouter le ou les packages du SDK à votre projet Unity

| Plateforme | Nom du package                                    |
|----------|-------------------------------------------------|
| Android  | com.microsoft.azure.spatial-anchors-sdk.android |
| iOS      | com.microsoft.azure.spatial-anchors-sdk.ios     |
| HoloLens | com.microsoft.azure.spatial-anchors-sdk.windows |

1. Pour chaque plateforme (Android/iOS/HoloLens) que vous souhaitez prendre en charge dans votre projet, ajoutez une entrée avec le nom et la version du package à la section `dependencies` dans le manifeste de votre projet. Voir l’exemple ci-dessous.

    [!code-json[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-unity-scoped-registry-setup.md?range=9-22&highlight=12-14)]

2. Enregistrez et fermez le fichier `manifest.json`. Quand vous revenez à Unity, Unity doit détecter automatiquement le changement de manifeste du projet et récupérer les packages spécifiés. Vous pouvez développer le dossier `Packages` dans votre vue du projet pour vérifier que les packages appropriés ont été importés.

#### <a name="android-only-configure-the-maintemplategradle-file"></a>Android uniquement : Configurer le fichier mainTemplate.gradle

1. Accédez à **Edit** > **Project Settings** > **Player**.
2. Dans le **panneau Inspector** (Inspecteur) pour **Player Settings** (Paramètres du lecteur), sélectionnez l’icône **Android**.
3. Sous la section **Build**, cochez la case **Custom Main Gradle Template** (Principal modèle Gradle personnalisé) pour générer un modèle Gradle personnalisé sur `Assets\Plugins\Android\mainTemplate.gradle`.
4. Ouvrez votre fichier `mainTemplate.gradle` dans un éditeur de texte.
5. Dans la section `dependencies`, collez les dépendances suivantes :

    ```gradle
    implementation('com.squareup.okhttp3:okhttp:[3.11.0]')
    implementation('com.microsoft.appcenter:appcenter-analytics:[1.10.0]')
    ```

Quand vous avez terminé, votre section `dependencies` doit ressembler à ce qui suit :

[!code-gradle[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-unity-android-gradle-setup.md?range=9-13&highlight=3-4)]

### <a name="import-the-asset-package"></a>[Importer le package de ressources](#tab/UnityAssetPackage)

> [!WARNING]
> La distribution du package de ressources Unity du Kit de développement logiciel (SDK) Azure Spatial Anchors a été déconseillée dans la version 2.5.0, et n’est plus disponible depuis la version 2.6.0.

1. Téléchargez le fichier `AzureSpatialAnchors.unitypackage` pour la version que vous souhaitez cibler à partir des [Mises en production GitHub](https://github.com/Azure/azure-spatial-anchors-samples/releases).
2. Suivez les instructions [ici](https://docs.unity3d.com/Manual/AssetPackagesImport.html) pour importer le package de ressources Unity dans votre projet.

---

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Guide pratique pour Créer et localiser des ancres dans Unity](./create-locate-anchors-unity.md)
