---
title: 'Tutoriel : Modérer les images de produits e-commerce - Content Moderator'
titleSuffix: Azure Cognitive Services
description: Ce tutoriel montre comment configurer une application pour analyser et classifier des images de produits avec des étiquettes spécifiques (à l’aide des API Azure Vision par ordinateur et Custom Vision). Étiquetez les images litigieuses pour les analyser de manière plus approfondie (à l’aide d’Azure Content Moderator).
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: tutorial
ms.date: 10/23/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 6d105528404c99f7273687fcdea6972b4212fcf1
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913685"
---
# <a name="tutorial-moderate-e-commerce-product-images-with-azure-content-moderator"></a>Tutoriel : Modérer les images de produits e-commerce avec Azure Content Moderator

Dans ce tutoriel, vous allez apprendre à utiliser Azure Cognitive Services, et notamment Content Moderator, pour classer et modérer des images de produits dans le cadre d’un scénario d’e-commerce. Vous allez utiliser Vision par ordinateur et Custom Vision pour appliquer des étiquettes aux images, puis vous allez créer une révision d’équipe, qui associe les technologies de Content Moderator basées sur le machine learning à des équipes de révision humaines pour fournir un système de modération intelligent.

Ce didacticiel vous explique les procédures suivantes :

> [!div class="checklist"]
> * Inscription à Content Moderator et création d’une équipe de révision.
> * Utilisation de l’API image Content Moderator pour analyser du contenu pour adultes ou osé.
> * Utilisation du service Vision par ordinateur pour rechercher du contenu sur les célébrités (ou d’autres étiquettes détectables par Vision par ordinateur).
> * Utilisation du service Vision personnalisée pour rechercher la présence de drapeaux, jouets et stylos (ou autres étiquettes personnalisées).
> * Présentation des résultats d’analyses combinées pour révision par un opérateur humain et prise de décision finale.

L’exemple de code complet est disponible dans le dépôt [Samples eCommerce Catalog Moderation](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration) sur GitHub.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

- Une clé d’abonnement Content Moderator. Suivez les instructions dans [Créer un compte Cognitive Services](../cognitive-services-apis-create-account.md) pour vous abonner au service Content Moderator et obtenir votre clé.
- Une clé d’abonnement Vision par ordinateur (mêmes instructions que ci-dessus).
- N’importe quelle édition de [Visual Studio 2015 ou 2017](https://www.visualstudio.com/downloads/).
- Un ensemble d’images pour chaque étiquette que le classifieur Vision personnalisée utilisera (en l’occurrence, jouets, stylos et drapeaux américains).

## <a name="create-a-review-team"></a>Créer une équipe de révision

Reportez-vous au guide de démarrage rapide [Essayer Content Moderator sur le web](quick-start.md) pour obtenir des instructions sur la façon de vous inscrire à l’[Outil de révision Content Moderator](https://contentmoderator.cognitive.microsoft.com/) et de créer une équipe de révision. Prenez note de la valeur de l’ **ID de l’équipe** dans la page **Informations d’identification** .

## <a name="create-custom-moderation-tags"></a>Créer des étiquettes de modération personnalisées

Créez ensuite des étiquettes personnalisées dans l’outil de révision (consultez l’article [Étiquettes](./review-tool-user-guide/configure.md#tags) si vous avez besoin d’aide pour ce processus). En l’occurrence, nous allons ajouter les étiquettes suivantes : **célébrité** , **USA** , **drapeau** , **jouet** et **stylo** . Toutes les étiquettes ne doivent pas nécessairement être des catégories détectables dans Vision par ordinateur (par exemple **célébrité** ). Vous pouvez ajouter vos propres étiquettes personnalisées à condition d’entraîner le classifieur Custom Vision à les détecter par la suite.

![Configurer les balises personnalisées](images/tutorial-ecommerce-tags2.PNG)

## <a name="create-visual-studio-project"></a>Création d’un projet Visual Studio

1. Dans Visual Studio, ouvrez la boîte de dialogue Nouveau projet. Développez **Installé** et **Visual C#** , puis sélectionnez **Application console (.NET Framework)** .
1. Nommez l’application **EcommerceModeration** , puis cliquez sur **OK** .
1. Si vous ajoutez ce projet à une solution existante, sélectionnez-le en tant que projet de démarrage unique.

Ce tutoriel met en évidence le code qui se trouve au cœur du projet, mais il ne couvre pas toutes les lignes de code. Copiez le contenu complet de _Program.cs_ à partir de l’exemple de projet ( [Samples eCommerce Catalog Moderation](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration)) dans le fichier _Program.cs_ de votre nouveau projet. Ensuite, parcourez les sections suivantes pour en savoir plus sur le fonctionnement du projet et son utilisation.

## <a name="define-api-keys-and-endpoints"></a>Définir les clés et points de terminaison API

Ce tutoriel utilise trois services cognitifs. Il nécessite donc trois clés et points de terminaison d’API correspondants. Observez les champs suivants dans la classe **Program**  :

[!code-csharp[define API keys and endpoint URIs](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=21-29)]

Vous devez mettre à jour les champs de `___Key` avec les valeurs de vos clés d’abonnement et modifier les champs de `___Uri` avec les URL de point de terminaison appropriées (vous obtiendrez la clé et le point de terminaison Custom Vision plus tard). Vous trouverez ces valeurs sous les onglets de **démarrage rapide** de chaque ressource Azure. Renseignez la partie `YOURTEAMID` du champ `ReviewUri` avec l’ID de l’équipe de révision que vous avez créée précédemment. Vous renseignerez la dernière partie du champ `CustomVisionUri` plus tard.

[!INCLUDE [subdomains note](../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="primary-method-calls"></a>Appels de la méthode principale

Observez le code suivant dans la méthode **Main** , qui effectue une itération sur une liste d’URL d’images. Il analyse chaque image avec les trois services différents, enregistre les étiquettes appliquées dans le tableau **ReviewTags** , puis crée une révision pour les modérateurs humains en envoyant les images à l’outil de révision Content Moderator. Vous allez explorer ces méthodes dans les sections suivantes. Si vous le souhaitez, vous pouvez contrôler le choix des images envoyées pour révision, à l’aide du tableau **ReviewTags** dans une instruction conditionnelle pour vérifier quelles sont les étiquettes qui ont été appliquées.

[!code-csharp[Main: evaluate each image and create review](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=53-70)]

## <a name="evaluateadultracy-method"></a>Méthode EvaluateAdultRacy

Observez la méthode **EvaluateAdultRacy** dans la classe **Program** . Cette méthode prend l’URL d’une image et un tableau de paires clé/valeur comme paramètres. Elle appelle l’API Image Content Moderator (en utilisant REST) pour obtenir les scores de l’image pour Adultes et Osé. Si le score de l’une de ces catégories est supérieur à 0,4 (la plage est comprise entre 0 et 1), cela entraîne la définition de la valeur correspondante dans le tableau **ReviewTags** à **True** .

[!code-csharp[define EvaluateAdultRacy method](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=73-113)]

## <a name="evaluatecomputervisiontags-method"></a>Méthode EvaluateComputerVisionTags

La méthode suivante prend une URL d’image et vos informations d’abonnement Vision par ordinateur et recherche la présence de célébrités dans l’image. Si une ou plusieurs célébrités sont trouvées, elle définit la valeur correspondante dans le tableau **ReviewTags** sur **True** .

[!code-csharp[define EvaluateCustomVisionTags method](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=115-146)]

## <a name="evaluatecustomvisiontags-method"></a>Méthode EvaluateCustomVisionTags

Ensuite, observez la méthode **EvaluateCustomVisionTags** , qui classifie les produits proprement dits&mdash;en l’occurrence, les drapeaux, jouets et stylos. Suivez les instructions du [Guide pratique pour créer un classifieur](../custom-vision-service/getting-started-build-a-classifier.md) et créez votre propre classifieur d’images personnalisé afin de détecter la présence de drapeaux, jouets et stylos (ou de ce que vous avez choisi comme étiquettes personnalisées) dans les images. Vous pouvez utiliser les images du dossier **sample-images** sur le [dépôt GitHub](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration) pour entraîner rapidement certaines catégories de cet exemple.

![Page web Vision personnalisée avec images d’entraînement de stylos, jouets et drapeaux](images/tutorial-ecommerce-custom-vision.PNG)

Une fois que vous avez entraîné votre classifieur, obtenez la clé de prédiction et l’URL de point de terminaison de prédiction (consultez [Obtenir l’URL et la clé de prédiction](../custom-vision-service/use-prediction-api.md#get-the-url-and-prediction-key), si vous avez besoin d’aide pour les récupérer), puis affectez ces valeurs à vos champs `CustomVisionKey` et `CustomVisionUri`, respectivement. La méthode utilise ces valeurs pour interroger le classifieur. Si le classifieur détecte une ou plusieurs étiquettes personnalisées dans l’image, cette méthode définit la ou les valeurs correspondantes dans le tableau **ReviewTags** sur **True** .

[!code-csharp[define EvaluateCustomVisionTags method](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=148-171)]

## <a name="create-reviews-for-review-tool"></a>Créer des révisions pour l’outil de révision

Les sections précédentes vous ont montré comment l’application analyse les images entrantes à la recherche de contenus pour adultes et de contenus osés (Content Moderator), de célébrités (Vision par ordinateur) et de divers autres objets (Custom Vision). À présent, observez la méthode **CreateReview** , qui charge les images ainsi que toutes leurs étiquettes appliquées (passées en tant que _Métadonnées_ ) à l’Outil de révision Content Moderator.

[!code-csharp[define CreateReview method](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=173-196)]

Les images apparaissent sous l’onglet Review de l’[outil de révision Content Moderator](https://contentmoderator.cognitive.microsoft.com/).

![Capture d’écran de l’outil de révision Content Moderator avec plusieurs images et étiquettes correspondantes en surbrillance](images/tutorial-ecommerce-content-moderator.PNG)

## <a name="submit-a-list-of-test-images"></a>Envoyer une liste d’images de test

Comme vous pouvez le voir dans la méthode **Main** , ce programme recherche un répertoire « C:Test » avec un fichier _Urls.txt_ qui contient une liste d’URL d’images. Créez ce fichier et ce répertoire, ou changez le chemin pour qu’il pointe vers votre fichier texte. Remplissez ensuite ce fichier avec les URL des images à tester.

[!code-csharp[Main: set up test directory, read lines](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=38-51)]

## <a name="run-the-program"></a>Exécuter le programme

Si vous avez suivi toutes les étapes ci-dessus, le programme doit traiter chaque image (en interrogeant les trois services en fonction des étiquettes appropriées), puis charger les images avec les informations d’étiquettes dans l’outil de révision Content Moderator.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré un programme pour analyser des images de produits, les étiqueter par type de produit et permettre à une équipe de révision de prendre des décisions informées dans le cadre d’une modération du contenu. À présent, découvrez les détails de la modération des images.

> [!div class="nextstepaction"]
> [Réviser les images modérées](./review-tool-user-guide/review-moderated-images.md)