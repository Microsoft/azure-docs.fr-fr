---
title: 'Démarrage rapide : Bibliothèque de client .NET Visage'
description: Utilisez la bibliothèque de client Visage pour .NET afin de détecter des visages, rechercher des visages semblables (recherche faciale par image), identifier des visages (recherche avec reconnaissance faciale) et migrer vos données de visage.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 10/26/2020
ms.author: pafarley
ms.openlocfilehash: 70235001ad28901abd3c47872a6a241422e6f916
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763648"
---
Commencez à utiliser la reconnaissance faciale avec la bibliothèque de client Visage pour .NET. Suivez les étapes suivantes pour installer le package et essayer l’exemple de code pour les tâches de base. Le service Visage vous donne accès à des algorithmes avancés pour la détection et la reconnaissance des visages dans des images.

Utilisez la bibliothèque de client Visage pour .NET afin d’effectuer les opérations suivantes :

* [Détecter des visages sur une image](#detect-faces-in-an-image)
* [Rechercher des visages semblables](#find-similar-faces)
* [Créer un groupe de personnes](#create-a-person-group)
* [Identifier un visage](#identify-a-face)

[Documentation de référence](/dotnet/api/overview/azure/cognitiveservices/client/faceapi?view=azure-dotnet) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.Face) | [Package (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.6.0-preview.1) | [Exemples](/samples/browse/?products=azure&term=face)

## <a name="prerequisites"></a>Prérequis


* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* L’[IDE Visual Studio](https://visualstudio.microsoft.com/vs/) ou la version actuelle de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="créez une ressource Visage"  target="_blank">créer une ressource Visage <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le Portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Visage. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-c-application"></a>Créer une application C#

#### <a name="visual-studio-ide"></a>[IDE Visual Studio](#tab/visual-studio)

À l’aide de Visual Studio, créez une application .NET Core. 

### <a name="install-the-client-library"></a>Installer la bibliothèque de client 

Une fois que vous avez créé un projet, installez la bibliothèque de client en cliquant avec le bouton droit sur la solution de projet dans l’**Explorateur de solutions** et en sélectionnant **Gérer les packages NuGet**. Dans le gestionnaire de package qui s’ouvre, sélectionnez **Parcourir**, cochez **Inclure la préversion** et recherchez `Microsoft.Azure.CognitiveServices.Vision.Face`. Sélectionnez la version `2.6.0-preview.1`, puis **Installer**. 

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/cli)

Dans une fenêtre de console (par exemple cmd, PowerShell ou Bash), utilisez la commande `dotnet new` pour créer une application console avec le nom `face-quickstart`. Cette commande crée un projet C# simple nommé « Hello World » avec un seul fichier source : *program.cs*. 

```console
dotnet new console -n face-quickstart
```

Déplacez vos répertoires vers le dossier d’application nouvellement créé. Vous pouvez générer l’application avec :

```console
dotnet build
```

La sortie de génération ne doit contenir aucun avertissement ni erreur. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Installer la bibliothèque de client 

Dans le répertoire de l’application, installez la bibliothèque de client Visage pour .NET avec la commande suivante :

```console
dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
```

---

> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/Face/FaceQuickstart.cs), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.


À partir du répertoire du projet, ouvrez le fichier *program.cs* et ajoutez ce qui suit en utilisant les directives `using` suivantes :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_using)]

Dans la classe **Program** de l’application, créez des variables pour la clé et le point de terminaison de votre ressource.


> [!IMPORTANT]
> Accédez au portail Azure. Si la ressource [nom du produit] que vous avez créée dans la section **Prérequis** a été déployée, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. La clé et le point de terminaison se trouvent dans la page **Clé et point de terminaison** de la ressource, sous **Gestion des ressources**. 
>
> N’oubliez pas de supprimer la clé de votre code une fois que vous avez terminé, et ne la postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Pour plus d’informations, consultez l’article sur la [sécurité](../../../cognitive-services-security.md) de Cognitive Services.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_creds)]

Dans la méthode **Main** de l’application, ajoutez des appels pour les méthodes utilisées dans ce guide de démarrage rapide. Vous les implémenterez ultérieurement.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_maincalls)]

## <a name="object-model"></a>Modèle objet

Les classes et interfaces suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client .NET Visage :

|Nom|Description|
|---|---|
|[FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient?view=azure-dotnet) | Cette classe représente votre autorisation d’utiliser le service Visage. Vous en avez besoin pour toutes les fonctionnalités de Visage. Vous pouvez l’instancier avec vos informations d’abonnement et l’utiliser pour produire des instances d’autres classes. |
|[FaceOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperations?view=azure-dotnet)|Cette classe gère les tâches de détection et de reconnaissance de base que vous pouvez effectuer avec les visages. |
|[DetectedFace](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface?view=azure-dotnet)|Cette classe représente toutes les données détectées à partir d’un visage unique dans une image. Vous pouvez l’utiliser pour récupérer des informations détaillées sur le visage.|
|[FaceListOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.facelistoperations?view=azure-dotnet)|Cette classe gère les constructions **FaceList** stockées dans le cloud, comprenant un ensemble de visages assortis. |
|[PersonGroupPersonExtensions](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongrouppersonextensions?view=azure-dotnet)| Cette classe gère les constructions **Person** stockées dans le cloud, comprenant un ensemble de visages appartenant à une même personne.|
|[PersonGroupOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperations?view=azure-dotnet)| Cette classe gère les constructions **PersonGroup** stockées dans le cloud, comprenant un ensemble d’objets **Person** assortis. |

## <a name="code-examples"></a>Exemples de code

Les extraits de code ci-dessous vous montrent comment effectuer les tâches suivantes avec la bibliothèque de client Visage pour .NET :

* [Authentifier le client](#authenticate-the-client)
* [Détecter des visages sur une image](#detect-faces-in-an-image)
* [Rechercher des visages semblables](#find-similar-faces)
* [Créer un groupe de personnes](#create-a-person-group)
* [Identifier un visage](#identify-a-face)

## <a name="authenticate-the-client"></a>Authentifier le client

Dans une nouvelle méthode, instanciez un client avec votre point de terminaison et la clé. Créez un objet **[ApiKeyServiceClientCredentials](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.apikeyserviceclientcredentials?view=azure-dotnet)** avec votre clé et utilisez-le avec votre point de terminaison pour créer un objet **[FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient?view=azure-dotnet)** .

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_auth)]

### <a name="declare-helper-fields"></a>Déclarer des champs d’assistance

Les champs suivants sont nécessaires pour plusieurs des opérations de Visage que vous ajouterez ultérieurement. À la racine de votre classe **Program**, définissez la chaîne d’URL suivante. Cette URL pointe vers un dossier d’exemples d’images.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_image_url)]

Dans votre méthode **Main**, définissez des chaînes pointant vers les différents types de modèles de reconnaissance. Plus tard, vous pourrez spécifier le modèle de reconnaissance que vous souhaitez utiliser pour la détection des visages. Pour plus d’informations sur ces options, consultez [Spécifier un modèle de reconnaissance](../../Face-API-How-to-Topics/specify-recognition-model.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect_models)]

## <a name="detect-faces-in-an-image"></a>Détecter des visages dans une image

### <a name="get-detected-face-objects"></a>Recevoir les objets de visage détectés

Créez une méthode pour détecter les visages. La méthode `DetectFaceExtract` traite trois des images à l’URL donnée, puis crée une liste d’objets **[DetectedFace](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface?view=azure-dotnet)** dans la mémoire du programme. La liste des valeurs **[FaceAttributeType](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype?view=azure-dotnet)** spécifie les fonctionnalités à extraire. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect)]

> [!TIP]
> Vous pouvez également détecter les visages dans une image locale. Consultez les méthodes [IFaceOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.ifaceoperations?view=azure-dotnet) comme **DetectWithStreamAsync**.

### <a name="display-detected-face-data"></a>Afficher les données de visage détectées

Le reste de la méthode `DetectFaceExtract` analyse et imprime les données d’attribut pour chaque visage détecté. Chaque attribut doit être spécifié séparément dans l’appel initial de l’API de détection des visages (dans la liste **[FaceAttributeType](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype?view=azure-dotnet)** ). Le code suivant traite chaque attribut, mais vous n’aurez probablement besoin d’en utiliser qu’un ou plusieurs.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect_parse)]

## <a name="find-similar-faces"></a>Rechercher des visages semblables

Le code suivant utilise un visage unique détecté (la source) et recherche un ensemble d’autres visages (la cible) pour trouver des correspondances (recherche faciale par image). Quand il trouve une correspondance, il affiche l’ID du visage correspondant sur la console.

### <a name="detect-faces-for-comparison"></a>Détecter des visages pour les comparer

Tout d’abord, définissez une deuxième méthode de détection de visage. Vous devez détecter des visages dans des images avant de pouvoir les comparer. Cette méthode de détection est optimisée pour les opérations de comparaison. Elle n’extrait pas les attributs de visage détaillés comme dans la section ci-dessus, et utilise un modèle de reconnaissance différent.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_face_detect_recognize)]

### <a name="find-matches"></a>Rechercher des correspondances

La méthode suivante détecte des visages dans un ensemble d’images cibles et dans une image source unique. Elle les compare ensuite et recherche toutes les images cibles semblables à l’image source.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_find_similar)]

### <a name="print-matches"></a>Afficher les correspondances

Le code suivant affiche les détails relatifs aux correspondances sur la console :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_find_similar_print)]

## <a name="identify-a-face"></a>Identifier un visage

L’opération d’identification prend une image d’une personne (ou de plusieurs personnes) et recherche l’identité de chaque visage dans l’image (recherche avec reconnaissance faciale). Il compare chaque visage détecté à un **PersonGroup**, une base de données comprenant différents objets **Person** dont les caractéristiques du visage sont connues. Pour effectuer l’opération d’identification, vous devez d’abord créer et entraîner un **PersonGroup**.

### <a name="create-a-person-group"></a>Créer un groupe de personnes

Le code suivant crée un **PersonGroup** avec six objets **Person** différents. Il associe chaque objet **Person** à un ensemble d’exemples d’images, puis s’entraîne à reconnaître chaque personne par leurs caractéristiques faciales. Les objets **Person** et **PersonGroup** sont utilisés dans les opérations de vérification, d’identification et de groupe.

Déclarez une variable de chaîne à la racine de votre classe pour représenter l’ID du **PersonGroup** que vous allez créer.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_declare)]

Dans une nouvelle méthode, ajoutez le code suivant. Cette méthode exécutera l’opération d’identification. Le premier bloc de code associe les noms des personnes à leurs exemples d’images.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_files)]

Notez que ce code définit une variable `sourceImageFileName`. Cette variable correspond à l’image source (l’image qui contient les personnes à identifier).

Ajoutez ensuite le code suivant pour créer un objet **Person** pour chaque personne du dictionnaire et ajouter les données de visage à partir des images appropriées. Chaque objet **Person** est associé au même **PersonGroup** par le biais de sa chaîne d’ID unique. N’oubliez pas de passer les variables `client`, `url` et `RECOGNITION_MODEL1` dans cette méthode.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_create)]

> [!TIP]
> Vous pouvez également créer un **PersonGroup** à partir d’images locales. Consultez les méthodes [IPersonGroupPerson](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.ipersongroupperson?view=azure-dotnet) comme **AddFaceFromStreamAsync**.

### <a name="train-the-persongroup"></a>Entraîner le PersonGroup

Une fois que vous avez extrait les données de visage de vos images et que vous les avez triées dans des objets **Person** distincts, vous devez entraîner le **PersonGroup** à identifier les caractéristiques visuelles associées à chacun de ses objets **Person**. Le code suivant appelle la méthode **train** asynchrone et interroge les résultats, en affichant l’état sur la console.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_train)]

> [!TIP]
> L’API Visage s’exécute sur un ensemble de modèles prédéfinis qui sont statiques par nature (les performances du modèle ne se dégradent pas ou ne s’améliorent pas quand le service est exécuté). Les résultats générés par le modèle risquent de changer si Microsoft met à jour le back-end du modèle sans migrer vers une version entièrement nouvelle du modèle. Pour bénéficier d’une version plus récente d’un modèle, vous pouvez réentraîner votre **PersonGroup**, en spécifiant le modèle plus récent en tant que paramètre avec les mêmes images d’inscription.

Ce groupe **Person** et ses objets **Person** associés sont maintenant prêts à être utilisés dans les opérations de vérification, d’identification ou de groupe.

### <a name="identify-faces"></a>Identifier des visages

Le code suivant prend l’image source et crée une liste de tous les visages détectés qu’elle contient. Il s’agit des visages qui seront identifiés par rapport au **PersonGroup**.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_identify_sources)]

L’extrait de code suivant appelle l’opération **IdentifyAsync** et affiche les résultats sur la console. Ici, le service tente de faire correspondre chaque visage de l’image source à un **Person** dans le **PersonGroup** donné. Cela ferme votre méthode d’identification.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_identify)]

## <a name="run-the-application"></a>Exécuter l’application

#### <a name="visual-studio-ide"></a>[IDE Visual Studio](#tab/visual-studio)

Exécutez l’application en cliquant sur le bouton **Déboguer** en haut de la fenêtre de l’IDE.

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/cli)

Exécutez l’application à partir de votre répertoire d’application avec la commande `dotnet run`.

```dotnet
dotnet run
```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

Si vous avez créé un **PersonGroup** dans le cadre de ce guide de démarrage rapide et que vous voulez le supprimer, exécutez le code suivant dans votre programme :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_delete)]

Définissez la méthode de suppression avec le code suivant :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_deletepersongroup)]

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez vu comment utiliser la bibliothèque de client Visage pour .NET afin d’effectuer des tâches de reconnaissance faciale basiques. Pour plus d’informations sur la bibliothèque, reportez-vous à la documentation de référence.

> [!div class="nextstepaction"]
> [Informations de référence sur l’API Visage (.NET)](/dotnet/api/overview/azure/cognitiveservices/client/faceapi?view=azure-dotnet)

* [Qu’est ce que le service Visage ?](../../overview.md)
* Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/Face/FaceQuickstart.cs).