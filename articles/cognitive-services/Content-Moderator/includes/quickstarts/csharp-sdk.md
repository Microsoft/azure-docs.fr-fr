---
title: 'Démarrage rapide : Bibliothèque de client .NET Content Moderator'
titleSuffix: Azure Cognitive Services
description: Dans ce guide de démarrage rapide, apprenez à utiliser la bibliothèque cliente Azure Content Moderator pour .NET. Intégrez un logiciel de filtrage de contenu dans votre application afin de vous conformer aux réglementations ou de maintenir l’environnement souhaité pour vos utilisateurs.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 09/15/2020
ms.author: pafarley
ms.custom: devx-track-dotnet, cog-serv-seo-aug-2020
ms.openlocfilehash: 8a715c23900ac0d5a16ff4a0a8ade5ea9458cfed
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561492"
---
Commencez à utiliser la bibliothèque cliente Azure Content Moderator pour .NET. Suivez les étapes suivantes pour installer le package NuGet et essayer l’exemple de code pour les tâches de base. 

Content Moderator est un service d’IA qui vous permet de gérer le contenu potentiellement offensant, risqué ou indésirable. Utilisez le service de modération de contenu alimenté par l’IA pour analyser du texte, des images et des vidéos et appliquer automatiquement des indicateurs de contenu. Ensuite, intégrez votre application à l’outil Review, un environnement de modérateur en ligne pour une équipe de réviseurs humains. Intégrez un logiciel de filtrage de contenu dans votre application afin de vous conformer aux réglementations ou de maintenir l’environnement souhaité pour vos utilisateurs.

Utilisez la bibliothèque de client Content Moderator pour .NET aux fins suivantes :

* Modérer du texte
* Modérer des images
* Créer une révision

[Documentation de référence](/dotnet/api/overview/azure/cognitiveservices/client/contentmoderator?view=azure-dotnet) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.ContentModerator) | [Package (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) | [Exemples](../../samples-dotnet.md)

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* L’[IDE Visual Studio](https://visualstudio.microsoft.com/vs/) ou la version actuelle de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator"  title="Créer une ressource Content Moderator"  target="_blank">créez une ressource Content Moderator<span class="docon docon-navigate-external x-hidden-focus"></span></a> sur le portail Azure pour obtenir votre clé et votre point de terminaison. Attendez qu’elle se déploie, puis cliquez sur le bouton **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à Content Moderator. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-c-application"></a>Créer une application C#

#### <a name="visual-studio-ide"></a>[IDE Visual Studio](#tab/visual-studio)

À l’aide de Visual Studio, créez une application .NET Core. 

### <a name="install-the-client-library"></a>Installer la bibliothèque de client 

Une fois que vous avez créé un projet, installez la bibliothèque de client en cliquant avec le bouton droit sur la solution de projet dans l’**Explorateur de solutions** et en sélectionnant **Gérer les packages NuGet**. Dans le gestionnaire de package qui s’ouvre, sélectionnez **Parcourir**, cochez **Inclure la préversion** et recherchez `Microsoft.Azure.CognitiveServices.ContentModerator`. Sélectionnez la version `2.0.0`, puis **Installer**. 

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/cli)

Dans une fenêtre de console (par exemple cmd, PowerShell ou Bash), utilisez la commande `dotnet new` pour créer une application console avec le nom `content-moderator-quickstart`. Cette commande crée un projet C# « Hello World » simple avec un seul fichier source : *Program.cs*.

```console
dotnet new console -n content-moderator-quickstart
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

Dans le répertoire de l’application, installez la bibliothèque de client Content Moderator pour .NET avec la commande suivante :

```console
dotnet add package Microsoft.Azure.CognitiveServices.ContentModerator --version 2.0.0
```

---

> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ContentModerator/Program.cs), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.

À partir du répertoire de projet, ouvrez le fichier *Program.cs* dans votre éditeur ou votre IDE favori. Ajoutez les instructions `using` suivantes :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_using)]

Dans la classe **Program**, créez des variables pour la clé et le point de terminaison de votre ressource.

> [!IMPORTANT]
> Accédez au portail Azure. Si la ressource Content Moderator que vous avez créée dans la section **Prérequis** a été déployée, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. La clé et le point de terminaison se trouvent dans la page **Clé et point de terminaison** de la ressource, sous **Gestion des ressources**. 
>
> N’oubliez pas de supprimer la clé de votre code une fois que vous avez terminé, et ne la postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Pour plus d’informations, consultez l’article sur la [sécurité](../../../cognitive-services-security.md) de Cognitive Services.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_creds)]


Dans la méthode `main()` de l’application, ajoutez des appels pour les méthodes utilisées dans ce guide de démarrage rapide. Vous les créerez ultérieurement.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_client)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_textmod_call)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_call)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_call)]


## <a name="object-model"></a>Modèle objet

Les classes suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client .NET Content Moderator.

|Nom|Description|
|---|---|
|[ContentModeratorClient](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.contentmoderatorclient?view=azure-dotnet)|Cette classe est nécessaire pour toutes les fonctionnalités Content Moderator. Vous pouvez l’instancier avec vos informations d’abonnement et l’utiliser pour produire des instances d’autres classes.|
|[ImageModeration](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation?view=azure-dotnet)|Cette classe fournit les fonctionnalités permettant d’analyser des images pour y rechercher du contenu pour adultes, des informations personnelles ou des visages.|
|[TextModeration](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.textmoderation?view=azure-dotnet)|Cette classe fournit les fonctionnalités d’analyse de texte pour le langage, les blasphèmes, les erreurs et les informations personnelles.|
|[Révisions](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.reviews?view=azure-dotnet)|Cette classe fournit les fonctionnalités des API de révision, notamment les méthodes de création de travaux, les workflows personnalisés et les révisions humaines.|

## <a name="code-examples"></a>Exemples de code

Ces extraits de code montrent comment effectuer les tâches suivantes avec la bibliothèque de client Content Moderator pour .NET :

* [Authentifier le client](#authenticate-the-client)
* [Modérer du texte](#moderate-text)
* [Modérer les images](#moderate-images)
* [Créer une révision](#create-a-review)

## <a name="authenticate-the-client"></a>Authentifier le client

Dans une nouvelle méthode, instanciez les objets clients avec le point de terminaison et la clé.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_auth)]

## <a name="moderate-text"></a>Modérer du texte

Le code suivant utilise un client Content Moderator pour analyser un corps de texte et imprimer les résultats dans la console. À la racine de la classe **Program**, définissez les fichiers d’entrée et de sortie :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_text_vars)]

Ensuite, à la racine de votre projet, ajoutez un fichier *TextFile.txt*. Ajoutez votre propre texte à ce fichier, ou utilisez l’exemple de texte suivant :

```
Is this a grabage email abcdef@abcd.com, phone: 4255550111, IP: 255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555.
Crap is the profanity here. Is this information PII? phone 4255550111
```


Définissez ensuite la méthode de modération du texte dans votre classe **Program** :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_textmod)]

## <a name="moderate-images"></a>Modérer des images

Le code suivant utilise un client Content Moderator, ainsi qu’un objet [ImageModeration](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation?view=azure-dotnet), pour analyser les images distantes et y rechercher du contenu pour adultes.

> [!NOTE]
> Vous pouvez également analyser le contenu d’une image locale. Pour connaître les méthodes et les opérations qui fonctionnent avec les images locales, consultez la [documentation de référence](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation.evaluatefileinputwithhttpmessagesasync?view=azure-dotnet#Microsoft_Azure_CognitiveServices_ContentModerator_ImageModeration_EvaluateFileInputWithHttpMessagesAsync_System_IO_Stream_System_Nullable_System_Boolean__System_Collections_Generic_Dictionary_System_String_System_Collections_Generic_List_System_String___System_Threading_CancellationToken_).

### <a name="get-sample-images"></a>Obtenir des exemples d’images

Définissez vos fichiers d’entrée et de sortie à la racine de votre classe **Program** :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_image_vars)]

Créez ensuite le fichier d’entrée (*ImageFiles.txt*) à la racine de votre projet. Dans ce fichier, ajoutez les URL des images pour analyser&mdash;une URL sur chaque ligne. Vous pouvez utiliser les exemples d’image suivants :

```
https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg
https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png
```

### <a name="define-helper-class"></a>Définir une classe d’assistance

Ajoutez la définition de classe suivante dans la classe **Program**. Cette classe interne gère les résultats de la modération d’images.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_dataclass)]

### <a name="define-the-image-moderation-method"></a>Définir la méthode de modération des images

La méthode suivante parcourt les URL d’image d’un fichier texte, crée une instance **EvaluationData** et analyse l’image à la recherche de contenu pour adultes, de texte et de visages humains. Ensuite, elle ajoute la dernière instance **EvaluationData** à une liste et affiche la liste complète des données retournées dans la console.

#### <a name="iterate-through-images"></a>Parcourir les images

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_iterate)]

#### <a name="analyze-content"></a>Analyser le contenu

Pour plus d’informations sur les attributs d’image que Content Moderator peut filtrer, consultez le guide [Concepts relatifs à la modération d’image](../../image-moderation-api.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_analyze)]

#### <a name="write-moderation-results-to-file"></a>Écrire les résultats de la modération dans un fichier

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_save)]

## <a name="create-a-review"></a>Créer une révision

Vous pouvez utiliser la bibliothèque de client .NET Content Moderator pour alimenter le contenu dans l’[outil de révision](https://contentmoderator.cognitive.microsoft.com) et permettre à des modérateurs humains de l’examiner. Pour en savoir plus sur l’outil de révision, consultez le [Guide conceptuel de l’outil de révision du code](../../review-tool-user-guide/human-in-the-loop.md).

La méthode de cette section utilise la classe [Reviews](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.reviews?view=azure-dotnet) pour créer une révision, récupérer son ID et vérifier ses détails après avoir reçu une entrée humaine via le portail web de l’outil de révision. Elle journalise toutes ces informations dans un fichier texte de sortie. 

### <a name="get-sample-images"></a>Obtenir des exemples d’images

Déclarez le tableau suivant à la racine de votre classe **Program**. Cette variable référence l’exemple d’image à utiliser pour créer la révision.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_urls)]

### <a name="get-review-credentials"></a>Obtenir les informations d’identification de révision

Connectez-vous à l’[outil de révision](https://contentmoderator.cognitive.microsoft.com) et récupérez le nom de votre équipe. Ensuite, affectez-le à la variable appropriée dans la classe **Program**. Si vous le souhaitez, vous pouvez configurer un point de terminaison de rappel pour recevoir des mises à jour sur l’activité de la révision.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_vars)]

### <a name="define-helper-class"></a>Définir une classe d’assistance

Ajoutez la définition de classe suivante dans la classe **Program**. Cette classe sera utilisée pour représenter une instance de révision unique soumise à l’outil de révision.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_item)]

### <a name="define-helper-method"></a>Définir une méthode d’assistance

Ajoutez la méthode suivante à la classe **Program**. Cette méthode écrit les résultats des requêtes de révision dans le fichier texte de sortie.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_writeline)]

### <a name="define-the-review-creation-method"></a>Définir la méthode de création de la révision

Vous êtes maintenant prêt à définir la méthode qui va gérer la création et l’interrogation de la révision. Ajoutez une méthode (**CreateReviews**) et définissez les variables locales suivantes.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_fields)]

#### <a name="post-reviews-to-the-review-tool"></a>Publier des révisions sur l’outil de révision

Ensuite, ajoutez le code suivant pour parcourir les exemples d’images donnés, ajouter des métadonnées et les envoyer à l’outil de révision dans un même lot. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_create)]

L’objet retourné par l’appel de l’API contient des valeurs d’ID uniques pour chaque image chargée. Le code suivant analyse ces ID, puis les utilise pour interroger Content Moderator afin d’obtenir l’état de chaque image du lot.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_ids)]

### <a name="get-review-details"></a>Obtenir les informations d’une révision

Le code suivant demande au programme d’attendre une entrée utilisateur. Lorsque vous arrivez à cette étape de l’exécution, vous pouvez accéder à l’[outil de révision](https://contentmoderator.cognitive.microsoft.com), vérifier que l’exemple d’image a été chargé et interagir avec celui-ci. Pour plus d’informations sur la façon d’interagir avec une révision, consultez le [Guide pratique sur les révisions](../../review-tool-user-guide/review-moderated-images.md). Lorsque vous avez terminé, vous pouvez appuyer sur n’importe quelle touche pour continuer le programme et récupérer les résultats du processus de révision.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_results)]

Si vous avez utilisé un point de terminaison de rappel dans ce scénario, il doit recevoir un événement au format suivant :

```console
{'callback_endpoint': 'https://requestb.in/qmsakwqm',
 'content': '',
 'content_id': '3ebe16cb-31ed-4292-8b71-1dfe9b0e821f',
 'created_by': 'cspythonsdk',
 'metadata': [{'key': 'sc', 'value': 'True'}],
 'review_id': '201901i14682e2afe624fee95ebb248643139e7',
 'reviewer_result_tags': [{'key': 'a', 'value': 'True'},
                          {'key': 'r', 'value': 'True'}],
 'status': 'Complete',
 'sub_team': 'public',
 'type': 'Image'}
```

## <a name="run-the-application"></a>Exécution de l'application

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

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à utiliser la bibliothèque .NET Content Moderator pour effectuer des tâches de modération. Pour plus d’informations sur la modération des images ou d’autres éléments multimédias, consultez le guide conceptuel.

> [!div class="nextstepaction"]
> [Concepts liés à la modération d’image](../../image-moderation-api.md)

* [Qu’est-ce qu’Azure Content Moderator ?](../../overview.md)
* Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ContentModerator/Program.cs).