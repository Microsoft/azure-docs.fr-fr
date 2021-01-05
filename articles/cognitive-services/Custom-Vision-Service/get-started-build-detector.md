---
title: 'Démarrage rapide : Créer un détecteur d’objets avec le site web Custom Vision'
titleSuffix: Azure Cognitive Services
description: Dans ce guide de démarrage rapide, vous allez découvrir comment utiliser le site web Custom Vision pour créer, former et tester un modèle de détecteur d’objets.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: quickstart
ms.date: 09/30/2020
ms.author: pafarley
ms.custom: cog-serv-seo-aug-2020
keywords: reconnaissance d’image, application de reconnaissance d’image, vision personnalisée
ms.openlocfilehash: ab747ad5e7b9362e9c587741198a0191c032b124
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018357"
---
# <a name="quickstart-build-an-object-detector-with-the-custom-vision-website"></a>Démarrage rapide : Créer un détecteur d’objets avec le site web Custom Vision

Dans ce guide de démarrage rapide, vous allez découvrir comment utiliser le site web Custom Vision pour créer un modèle de détecteur d’objets. Une fois que vous avez créé un modèle, vous pouvez le tester avec de nouvelles images et l’intégrer finalement à votre propre application de reconnaissance d’images.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

- Un ensemble d’images avec lequel entraîner votre modèle de détecteur. Vous pouvez utiliser l’ensemble d’[exemples d’images](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/tree/master/samples/vision/images) disponible sur GitHub, ou choisir vos propres images à l’aide des conseils ci-dessous.

## <a name="create-custom-vision-resources"></a>Créer des ressources Custom Vision

[!INCLUDE [create-resources](includes/create-resources.md)]

## <a name="create-a-new-project"></a>Création d'un projet

Dans votre navigateur web, accédez à la [page web Custom Vision](https://customvision.ai) et sélectionnez __Sign in__ (Se connecter). Connectez-vous avec le même compte que celui utilisé pour vous connecter au portail Azure.

![Image de la page de connexion](./media/browser-home.png)


1. Pour créer votre premier projet, sélectionnez **New Project** (Nouveau projet). La boîte de dialogue **Create new project** (Créer un projet) s’affiche.

    ![La boîte de dialogue du nouveau projet comporte des champs permettant de renseigner le nom, la description et les domaines.](./media/get-started-build-detector/new-project.png)

1. Entrez un nom et une description pour le projet. Sélectionnez ensuite un groupe de ressources. Si votre compte de connexion est associé à un compte Azure, la liste déroulante Resource Group (Groupe de ressources) affiche tous vos groupes de ressources Azure qui incluent une ressource du service Vision personnalisée. 

   > [!NOTE]
   > Si aucun groupe de ressources n’est disponible, vérifiez que vous vous êtes connecté à [customvision.ai](https://customvision.ai) avec le même compte que celui utilisé pour vous connecter au [portail Azure](https://portal.azure.com/). Vérifiez aussi que vous avez sélectionné le même « Répertoire » dans le site web Custom Vision que celui du portail Azure où se trouvent vos ressources Custom Vision. Dans les deux sites, vous pouvez sélectionner votre répertoire à partir du menu déroulant de compte situé en haut à droite de l’écran. 

1. Sélectionnez __Object Detection__ (Détection d’objet) sous __Project Types__ (Types de projets).

1. Sélectionnez ensuite un des domaines disponibles. Chaque domaine optimise le détecteur pour des types spécifiques d’images, comme décrit dans le tableau suivant. Vous pourrez changer ultérieurement le domaine si vous le souhaitez.

    |Domain|Objectif|
    |---|---|
    |__Généralités__| Optimisé pour un large éventail de tâches de détection d’objets. Si aucun autre domaine n’est approprié, ou si vous hésitez sur le choix du domaine, sélectionnez le domaine Generic (Général). |
    |__Logo__|Optimisé pour rechercher des logos de marque dans les images.|
    |__Products on shelves__ (Produits en rayon)|Optimisé pour la détection et la classification des produits en rayon.|
    |__Compact Domains__ (Domaines compacts)| Optimisé en fonction des contraintes de détection d’objets en temps réel sur les appareils mobiles. Les modèles générés par les domaines compacts sont exportables pour s’exécuter localement.|

1. Enfin, sélectionnez __Create project__ (Créer le projet).

## <a name="choose-training-images"></a>Choisir les images d’entraînement

[!INCLUDE [choose training images](includes/choose-training-images.md)]

## <a name="upload-and-tag-images"></a>Charger et étiqueter des images

Dans cette section, vous allez charger et étiqueter manuellement des images pour faciliter l’entraînement du détecteur. 

1. Pour ajouter des images, cliquez sur le bouton __Add images__ (Ajouter des images), puis sélectionnez __Browse local files__ (Parcourir les fichiers locaux). Sélectionnez __Open__ (Ouvrir) pour charger les images.

    ![Le contrôle d’ajout d’images est affiché dans le coin supérieur gauche, et sous la forme d’un bouton dans le bas de la page, au centre.](./media/get-started-build-detector/add-images.png)

1. Vous verrez vos images chargées dans la section **Untagged** (Sans étiquette) de l’interface utilisateur. L’étape suivante consiste à étiqueter manuellement les objets que vous souhaitez que le détecteur apprenne à reconnaître. Cliquez sur la première image pour ouvrir la fenêtre de dialogue d’étiquetage. 

    ![Images chargées, dans la section Untagged](./media/get-started-build-detector/images-untagged.png)

1. Cliquez et faites glisser un rectangle autour de l’objet dans votre image. Ensuite, entrez un nouveau nom d’étiquette avec le bouton **+** , ou sélectionnez une étiquette existante dans la liste déroulante. Il est très important d’étiqueter chaque instance des objets que vous souhaitez détecter, car le détecteur utilise la zone d’arrière-plan sans étiquette comme exemple négatif d’entraînement. Quand vous avez terminé l’étiquetage, cliquez sur la flèche à droite pour enregistrer vos étiquettes et passer à l’image suivante.

    ![Étiquetage d’un objet avec une sélection rectangulaire](./media/get-started-build-detector/image-tagging.png)

Pour charger un autre ensemble d’images, revenez en haut de cette section et répétez les étapes.

## <a name="train-the-detector"></a>Entraîner le détecteur

Pour entraîner le modèle de détecteur, sélectionnez le bouton **Train** (Entraîner). Le détecteur utilise toutes les images actuelles et leurs étiquettes pour créer un modèle qui identifie chaque objet étiqueté.

![Le bouton Train (Entraîner) en haut à droite de la barre d’outils de l’en-tête de la page web](./media/getting-started-build-a-classifier/train01.png)

Le processus d’entraînement ne doit durer que quelques minutes. Pendant ce temps, des informations sur le processus d’entraînement s’affichent dans l’onglet **Performance**.

![Fenêtre du navigateur avec une boîte de dialogue d’entraînement dans la section principale](./media/get-started-build-detector/training.png)

## <a name="evaluate-the-detector"></a>Évaluer le détecteur

Une fois l’entraînement terminé, les performances du modèle sont calculées et affichées. Le service Custom Vision utilise les images que vous avez soumises pour l’entraînement afin de calculer la précision, le rappel et la précision moyenne. La précision et le rappel sont deux mesures différentes de l’efficacité d’un détecteur :

- La **précision** indique la proportion des classifications identifiées qui étaient correctes. Par exemple, si le modèle a identifié 100 images comme étant des chiens et que 99 d’entre elles étaient effectivement des chiens, la précision est de 99 %.
- Le **rappel** indique la proportion des classifications réelles qui ont été correctement identifiées. Par exemple, s’il y avait 100 images de pommes et que le modèle en a identifié 80 comme étant des pommes, le rappel est de 80 %.

![Les résultats de l’entraînement montrent la précision et le rappel globaux, ainsi que la précision moyenne.](./media/get-started-build-detector/trained-performance.png)

### <a name="probability-threshold"></a>Seuil de probabilité

[!INCLUDE [probability threshold](includes/probability-threshold.md)]

## <a name="manage-training-iterations"></a>Gérer les itérations d’entraînement

Chaque fois que vous entraînez votre détecteur, vous créez une nouvelle _itération_ avec ses propres métriques de performances mises à jour. Vous pouvez afficher toutes vos itérations dans le volet gauche de l’onglet **Performance**. Dans le volet gauche, vous trouvez également le bouton **Delete** (Supprimer), que vous pouvez utiliser pour supprimer une itération si elle est obsolète. Quand vous supprimez une itération, vous supprimez également toutes les images qui lui sont spécifiquement associées.

Consultez [Utiliser votre modèle avec l’API de prédiction](./use-prediction-api.md) pour apprendre à accéder à vos modèles entraînés par programmation.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez découvert comment créer et entraîner un modèle de détecteur d’objets avec le site web Custom Vision. Vous obtenez ensuite plus d’informations sur le processus itératif d’amélioration de votre modèle.

> [!div class="nextstepaction"]
> [Tester et réentraîner un modèle](test-your-model.md)

* [Qu’est-ce que Custom Vision ?](./overview.md)
