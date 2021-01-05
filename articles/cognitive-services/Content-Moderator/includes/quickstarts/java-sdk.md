---
title: 'Démarrage rapide : Bibliothèque de client Java Content Moderator'
titleSuffix: Azure Cognitive Services
description: Dans ce guide de démarrage rapide, apprenez à utiliser la bibliothèque cliente Azure Content Moderator pour Java. Intégrez un logiciel de filtrage de contenu dans votre application afin de vous conformer aux réglementations ou de maintenir l’environnement souhaité pour vos utilisateurs.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 10/16/2020
ms.custom: devx-track-java, cog-serv-seo-aug-2020
ms.author: pafarley
ms.openlocfilehash: 7713765a36207f0d9da05c4c11629e4a7f1164d9
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561509"
---
Commencez à utiliser la bibliothèque cliente Azure Content Moderator pour Java. Suivez les étapes suivantes pour installer le package Maven et essayer l’exemple de code pour les tâches de base. 

Content Moderator est un service d’IA qui vous permet de gérer le contenu potentiellement offensant, risqué ou indésirable. Utilisez le service de modération de contenu alimenté par l’IA pour analyser du texte, des images et des vidéos et appliquer automatiquement des indicateurs de contenu. Intégrez un logiciel de filtrage de contenu dans votre application afin de vous conformer aux réglementations ou de maintenir l’environnement souhaité pour vos utilisateurs.

Utilisez la bibliothèque cliente Content Moderator pour Java aux fins suivantes :

* Modérer du texte
* Modérer des images

[Documentation de référence](/java/api/overview/azure/cognitiveservices/client/contentmoderator?view=azure-java-stable) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-contentmoderator) |[Artefact (Maven)](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-contentmoderator) | [Exemples](/samples/browse/?products=azure&term=content-moderator)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* La version actuelle du [JDK (Java Development Kit)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* L’[outil de génération Gradle](https://gradle.org/install/) ou un autre gestionnaire de dépendances.
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator"  title="Créer une ressource Content Moderator"  target="_blank">créez une ressource Content Moderator<span class="docon docon-navigate-external x-hidden-focus"></span></a> sur le portail Azure pour obtenir votre clé et votre point de terminaison. Attendez qu’elle se déploie, puis cliquez sur le bouton **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à Content Moderator. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-gradle-project"></a>Créer un projet Gradle

Dans une fenêtre de console (telle que cmd, PowerShell ou bash), créez un répertoire pour votre application et accédez-y. 

```console
mkdir myapp && cd myapp
```

Exécutez la commande `gradle init` à partir de votre répertoire de travail. Cette commande crée des fichiers de build essentiels pour Gradle, notamment *build.gradle.kts*, qui est utilisé au moment de l’exécution pour créer et configurer votre application.

```console
gradle init --type basic
```

Quand vous êtes invité à choisir un **DSL**, sélectionnez **Kotlin**.

## <a name="install-the-client-library"></a>Installer la bibliothèque de client

Recherchez *build.gradle.kts* et ouvrez-le avec votre IDE ou éditeur de texte préféré. Copiez-y ensuite la configuration de build suivante. Cette configuration définit le projet en tant qu’application Java dont le point d’entrée est la classe **ContentModeratorQuickstart**. Elle importe la bibliothèque de client Content Moderator ainsi que le SDK GSON pour la sérialisation JSON.

```kotlin
plugins {
    java
    application
}

application{ 
    mainClassName = "ContentModeratorQuickstart"
}

repositories{
    mavenCentral()
}

dependencies{
    compile(group = "com.microsoft.azure.cognitiveservices", name = "azure-cognitiveservices-contentmoderator", version = "1.0.2-beta")
    compile(group = "com.google.code.gson", name = "gson", version = "2.8.5")
}
```

### <a name="create-a-java-file"></a>Créer un fichier Java


Dans votre répertoire de travail, exécutez la commande suivante pour créer un dossier de projet source :

```console
mkdir -p src/main/java
```

Accédez au nouveau dossier et créez un fichier sous le nom *ContentModeratorQuickstart.java*. Ouvrez-le dans votre éditeur ou IDE habituel et ajoutez les instructions `import` suivantes :

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imports)]

> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.

Dans la classe **ContentModeratorQuickstart** de l’application, créez des variables pour la clé et le point de terminaison de votre ressource.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_creds)]

> [!IMPORTANT]
> Accédez au portail Azure. Si la ressource [nom du produit] que vous avez créée dans la section **Prérequis** a été déployée, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. La clé et le point de terminaison se trouvent dans la page **Clé et point de terminaison** de la ressource, sous **Gestion des ressources**. 
>
> N’oubliez pas de supprimer la clé de votre code une fois que vous avez terminé, et ne la postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Pour plus d’informations, consultez l’article sur la [sécurité](../../../cognitive-services-security.md) de Cognitive Services.

Dans la méthode **main** de l’application, ajoutez des appels pour les méthodes utilisées dans ce guide de démarrage rapide. Vous les définirez plus tard.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_maincalls)]


## <a name="object-model"></a>Modèle objet

Les classes suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client Java Content Moderator.

|Nom|Description|
|---|---|
|[ContentModeratorClient](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.contentmoderatorclient?view=azure-java-stable)|Cette classe est nécessaire pour toutes les fonctionnalités Content Moderator. Vous pouvez l’instancier avec vos informations d’abonnement et l’utiliser pour produire des instances d’autres classes.|
|[ImageModeration](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.imagemoderations?view=azure-java-stable)|Cette classe fournit les fonctionnalités permettant d’analyser des images pour y rechercher du contenu pour adultes, des informations personnelles ou des visages.|
|[TextModerations](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.textmoderations?view=azure-java-stable)|Cette classe fournit les fonctionnalités d’analyse de texte pour le langage, les blasphèmes, les erreurs et les informations personnelles.|
|[Révisions](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.reviews?view=azure-java-stable)|Cette classe fournit les fonctionnalités des API de révision, notamment les méthodes de création de travaux, les workflows personnalisés et les révisions humaines.|


## <a name="code-examples"></a>Exemples de code

Ces extraits de code montrent comment effectuer les tâches suivantes avec la bibliothèque cliente Content Moderator pour Java :

* [Authentifier le client](#authenticate-the-client)
* [Modérer du texte](#moderate-text)
* [Modérer les images](#moderate-images)


## <a name="authenticate-the-client"></a>Authentifier le client

Dans la méthode `main` de l’application, créez un objet [ContentModeratorClient](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.contentmoderatorclient?view=azure-java-stable) à l’aide de la valeur de votre point de terminaison d’abonnement et de la clé d’abonnement.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_client)]



## <a name="moderate-text"></a>Modérer du texte

### <a name="set-up-sample-text"></a>Configurer un exemple de texte

En haut de votre classe **ContentModeratorQuickstart**, définissez une référence à un fichier texte local. Ajoutez un fichier .txt au répertoire de votre projet et entrez le texte que vous souhaitez analyser.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_var)]

### <a name="analyze-text"></a>Analyser le texte

Créez une méthode qui lit le fichier .txt et appelle la méthode **screenText** sur chaque ligne.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod)]

### <a name="print-text-moderation-results"></a>Afficher les résultats de la modération du texte

Ajoutez le code suivant pour afficher les résultats de la modération dans un fichier .json enregistré dans le répertoire de votre projet.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_print)]

Fermez l’instruction `try` et `catch` pour terminer la méthode.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_catch)]


## <a name="moderate-images"></a>Modérer des images

### <a name="set-up-sample-image"></a>Configurer un exemple d’image

Dans une nouvelle méthode, construisez un objet **[BodyModelModel](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.models.bodymodelmodel?view=azure-java-stable)** avec une chaîne d’URL donnée qui pointe vers une image.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod)]


### <a name="define-helper-class"></a>Définir une classe d’assistance

Ensuite, dans votre fichier *ContentModeratorQuickstart.java*, ajoutez la définition de classe suivante à l’intérieur de la classe **ContentModeratorQuickstart**. Cette classe interne est utilisé dans le processus de modération d’image.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_evaluationdata)]


### <a name="analyze-content"></a>Analyser le contenu
Cette ligne de code vérifie l’image à l’URL donnée pour y rechercher du contenu pour adultes ou osé. Pour plus d’informations sur ces termes, consultez le guide conceptuel sur la modération d’images.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_ar)]

### <a name="check-for-text"></a>Rechercher du texte
Cette ligne de code recherche le texte visible dans l’image.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_text)]

### <a name="check-for-faces"></a>Rechercher des visages
Cette ligne de code recherche les visages humains dans l’image.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_faces)]

Enfin, stockez les informations renvoyées dans la liste `EvaluationData`.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_storedata)]

### <a name="print-results"></a>Imprimer les résultats

Après la boucle `while`, ajoutez le code suivant, qui imprime les résultats dans la console et dans un fichier de sortie nommé *SRC/main/Resources/ModerationOutput.json*.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_printdata)]

Fermez l’instruction `try` et ajoutez une instruction `catch` pour terminer la méthode.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_catch)]

## <a name="run-the-application"></a>Exécution de l'application

Vous pouvez générer l’application avec :

```console
gradle build
```

Exécutez l’application avec la commande `gradle run` :

```console
gradle run
```

Accédez ensuite au fichier *src/main/resources/ModerationOutput.json* et affichez les résultats de la modération du contenu.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à utiliser la bibliothèque Java Content Moderator pour effectuer des tâches de modération. Pour plus d’informations sur la modération des images ou d’autres éléments multimédias, consultez le guide conceptuel.

> [!div class="nextstepaction"]
> [Concepts liés à la modération d’image](../../image-moderation-api.md)

* [Qu’est-ce qu’Azure Content Moderator ?](../../overview.md)
* Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java).