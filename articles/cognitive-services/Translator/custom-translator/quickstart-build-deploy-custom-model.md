---
title: 'Démarrage rapide : Créer, déployer et utiliser un modèle personnalisé - Custom Translator'
titleSuffix: Azure Cognitive Services
description: Dans ce guide de démarrage rapide, vous allez être guidé pas à pas dans le processus de création d’un système de traduction avec Custom Translator.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 12/09/2019
ms.author: swmachan
ms.topic: quickstart
ms.openlocfilehash: 4fbac2c2d77d838ac7fc4f3caaa73983633c3e03
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94366041"
---
# <a name="quickstart-build-deploy-and-use-a-custom-model-for-translation"></a>Démarrage rapide : Créer, déployer et utiliser un modèle personnalisé

Cet article fournit des instructions détaillées permettant de créer un système de traduction avec Custom Translator.

## <a name="prerequisites"></a>Prérequis

1. Pour utiliser le portail [Custom Translator](https://portal.customtranslator.azure.ai), vous devez avoir un [compte Microsoft](https://signup.live.com) ou un [compte Azure AD](../../../active-directory/fundamentals/active-directory-whatis.md) (compte d’organisation hébergé sur Azure) pour vous connecter.

2. Un abonnement à l’API de traduction de texte Translator Text via le portail Azure. Vous devez avoir la clé d’abonnement de l’API de traduction de texte Translator Text à associer à votre espace de travail dans Custom Translator. Consultez [Guide pratique pour s’inscrire à l’API de traduction de texte Translator Text](../translator-how-to-signup.md).

3. Quand vous disposez des deux éléments ci-dessus, connectez-vous à [Custom Translator](https://portal.customtranslator.azure.ai) pour créer des espaces de travail, des projets, charger des fichiers et créer/déployer des modèles.

>[!Note]
>Custom Translator ne prend pas en charge la création d’espace de travail pour la ressource d’API Translator créée dans un [réseau virtuel activé](../../../api-management/api-management-using-with-vnet.md).

## <a name="create-a-workspace"></a>Créer un espace de travail

Si vous êtes ici pour la première fois, il vous est demandé d’accepter les conditions d’utilisation du service, de créer un espace de travail et d’associer votre espace de travail à l’abonnement à l’API Microsoft Traduction de texte Translator Text.

![Créer un espace de travail](media/quickstart/terms-of-service.png)
![Image Créer un espace de travail 1](media/quickstart/create-workspace-1.png)
![Image Créer un espace de travail 2](media/quickstart/create-workspace-2.png)
![Image Créer un espace de travail 3](media/quickstart/create-workspace-3.png)
![Image Créer un espace de travail 4](media/quickstart/create-workspace-4.png)
![Image Créer un espace de travail 5](media/quickstart/create-workspace-5.png)
![Image Créer un espace de travail 6](media/quickstart/create-workspace-6.png)

Lors de vos prochaines visites sur le portail Custom Translator, accédez à la page des paramètres pour gérer votre espace de travail, créer d’autres espaces de travail, associer votre clé d’abonnement à l’API Microsoft Traduction de texte Translator Text à vos espaces de travail, ajouter des copropriétaires et modifier une clé d’abonnement.

## <a name="create-a-project"></a>Création d’un projet

Dans la page d’accueil du portail Custom Translator, cliquez sur New project (Nouveau projet). Dans la boîte de dialogue, vous pouvez entrer le nom de votre choix pour votre projet, la paire de langues, la catégorie, ainsi que d’autres champs appropriés. Ensuite, enregistrez votre projet. Pour plus d’informations, visitez [Créer un projet](how-to-create-project.md).

![Créer un projet](media/quickstart/ct-how-to-create-project.png)


## <a name="upload-documents"></a>Chargement de documents

Ensuite, chargez les jeux de documents d’[entraînement](training-and-model.md#training-document-type-for-custom-translator), de [paramétrage](training-and-model.md#tuning-document-type-for-custom-translator) et de [test](training-and-model.md#testing-dataset-for-custom-translator). Vous pouvez charger à la fois des documents [parallèles](what-are-parallel-documents.md) et variés. Vous pouvez également charger un [dictionnaire](what-is-dictionary.md).

Vous pouvez charger des documents à partir de l’onglet des documents ou de la page propre à un projet.

![Chargement de documents](media/quickstart/ct-how-to-upload.png)

Lors du chargement des documents, choisissez le type de document (entraînement, paramétrage ou test) et la paire de langues. Lors du chargement des documents parallèles, vous devez également spécifier un nom de document. Pour plus d’informations, visitez [Charger un document](how-to-upload-document.md).

## <a name="create-a-model"></a>Créer un modèle

Lorsque vous avez chargé tous les documents qu’il vous faut, l’étape suivante consiste à créer votre modèle.

Sélectionnez le projet que vous avez créé. Vous voyez tous les documents que vous avez chargés qui partagent une paire de langues avec ce projet. Sélectionnez les documents que vous souhaitez inclure dans votre modèle. Vous pouvez sélectionner des données d’[entraînement](training-and-model.md#training-document-type-for-custom-translator), de [paramétrage](training-and-model.md#tuning-document-type-for-custom-translator) et de [test](training-and-model.md#testing-dataset-for-custom-translator), ou sélectionner juste les données d’entraînement et laisser Custom Translator générer automatiquement les jeux de paramétrage et de test pour votre modèle.

![Créer un modèle](media/quickstart/ct-how-to-train.png)

Quand vous avez fini de sélectionner les documents de votre choix, cliquez sur le bouton Créer un modèle pour créer votre modèle et commencer à l’entraîner. Vous pouvez voir l’état de votre entraînement et les détails de tous les modèles que vous avez entraînés sous l’onglet Modèles.

Pour plus d’informations, consultez [Créer un modèle](how-to-train-model.md).

## <a name="analyze-your-model"></a>Analyser votre modèle

Une fois que vous avez fini votre entraînement, examinez les résultats. Le score BLEU est une métrique qui indique la qualité de votre traduction. Vous pouvez aussi comparer manuellement les traductions effectuées à l’aide de votre modèle personnalisé avec les traductions fournies dans votre série de tests en accédant à l’onglet « Test » et en cliquant sur « System Results » (Résultats système). En inspectant manuellement quelques-unes de ces traductions, vous aurez une bonne idée de la qualité de traduction produite par votre système. Pour plus d’informations, consultez [Résultats des tests système](how-to-view-system-test-results.md).

## <a name="deploy-a-trained-model"></a>Déployer un modèle entraîné

Lorsque vous êtes prêt à déployer votre modèle entraîné, cliquez sur le bouton « Deploy » (Déployer). Vous pouvez avoir un modèle déployé par projet et voir l’état de votre déploiement dans la colonne Status (État). Pour plus d’informations, consultez [Modèle de déploiement](how-to-view-system-test-results.md#deploy-a-model)

![Déployer un modèle entraîné](media/quickstart/ct-how-to-deploy.png)

## <a name="swap-deployed-model"></a>Échanger un modèle déployé

Pour échanger un modèle déployé avec un autre au sein d’un projet, cliquez sur le bouton « Échanger » situé en regard du modèle souhaité. Pendant le processus d’échange, le modèle déployé continue d’être disponible pour traiter les demandes de traduction. 

![Échanger un modèle déployé](media/quickstart/ct-how-to-swap-model.png)

## <a name="use-a-deployed-model"></a>Utiliser un modèle déployé

Les modèles déployés sont accessibles via l’[API de traduction de texte Translator Text V3 de Microsoft en spécifiant le CategoryID](../reference/v3-0-translate.md?tabs=curl). Vous trouverez plus d’informations sur l’API de traduction de texte Translator Text dans la page web [Informations de référence sur les API](../reference/v3-0-reference.md).

## <a name="next-steps"></a>Étapes suivantes

- Apprenez à naviguer dans l’[espace de travail Custom Translator et gérer vos projets](workspace-and-project.md).