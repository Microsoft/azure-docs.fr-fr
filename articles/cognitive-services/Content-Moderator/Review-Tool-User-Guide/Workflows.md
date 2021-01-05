---
title: Définir et utiliser des flux de travail de contenu via l’outil de révision - Content Moderator
titleSuffix: Azure Cognitive Services
description: Vous pouvez utiliser le concepteur de flux de travail Azure Content Moderator pour définir des flux de travail personnalisés et des seuils en fonction de vos stratégies de contenu.
services: cognitive-services
author: PatrickFarley
manager: mikemcca
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: pafarley
ms.openlocfilehash: 6eb2a2d2762b60a12bb9a24b92e2edae4b846cd1
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904140"
---
# <a name="define-and-use-moderation-workflows"></a>Définir et utiliser des flux de travail de modération

Dans ce guide, vous apprendrez à configurer et à utiliser des [flux de travail](../review-api.md#workflows) sur le site web de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com). Les flux de travail sont des filtres personnalisés basés sur le cloud qui vous permettent de gérer plus efficacement le contenu. Flux de travail peut se connecter à une variété de services pour filtrer le contenu de différentes façons et puis exécutez l’action appropriée. Ce guide vous montre comment utiliser le connecteur Content Moderator (inclus par défaut) pour filtrer le contenu et configurer les évaluations humaines dans un scénario de modération typique.

## <a name="create-a-new-workflow"></a>Créer un flux de travail

Accédez à l’[outil de révision Content Moderator](https://contentmoderator.cognitive.microsoft.com/) et inscrivez-vous. Dans l’onglet **Paramètres**, sélectionnez **Flux de travail**.

![Paramètres des flux de travail](images/2-workflows-0.png)

Sur l’écran suivant, sélectionnez **Ajouter un flux de travail**.

![Modifier un flux de travail](images/2-workflows-1.png)

### <a name="assign-a-name-and-description"></a>Assigner un nom et une description

Nommez votre flux de travail, entrez une description et choisissez si le flux de travail va gérer les images ou du texte.

![Nom et description du flux de travail](images/image-workflow-create.PNG)

### <a name="define-evaluation-criteria"></a>Définir les critères d’évaluation

Sur l’écran suivant, accédez à la section **If**. Dans le menu déroulant supérieur, choisissez **Condition**. Cela vous permettra de configurer la condition dans laquelle le flux de travail va opérer. Si vous voulez utiliser plusieurs conditions, choisissez plutôt **Combination**. 

Sélectionnez ensuite un connecteur. Cet exemple utilise **Content Moderator**. Selon le connecteur que vous choisissez, vous obtiendrez différentes options pour la sortie des données. Consultez la section [Connecteurs](./configure.md#connectors) du guide de configuration de l'outil de vérification pour savoir comment configurer d'autres connecteurs.

![Sélectionner le connecteur de flux de travail](images/image-workflow-connect-to.PNG)

Choisissez la sortie désirée à utiliser, puis définissez les conditions de vérification.

![Définir la condition du flux de travail](images/image-workflow-condition.PNG)

### <a name="define-the-action"></a>Définir l’action

Accédez à la section **Then**, où vous sélectionnez une action. L’exemple suivant crée une révision de l’image et lui attribue une balise. Vous pouvez éventuellement ajouter un chemin alternatif (Else) et définir une action.

![Définir l’action du flux de travail](images/image-workflow-action.PNG)

### <a name="save-the-workflow"></a>Enregistrer le flux de travail

Notez le nom du flux de travail car vous aurez besoin pour lancer un travail de modération avec l'API de flux de travail (voir ci-dessous). Pour finir, enregistrez le flux de travail à l'aide du bouton **Enregistrer** en haut de la page.

## <a name="test-the-workflow"></a>Tester le flux de travail

Maintenant que vous avez défini un flux de travail personnalisé, testez-le avec un exemple de contenu. Accédez à **Flux de travail**, puis sélectionnez le bouton **Exécuter le flux de travail** correspondant.

![Test du flux de travail](images/image-workflow-execute.PNG)

Enregistrez cet [exemple d’image](https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg) sur votre disque local. Sélectionnez ensuite **Choisir le(s) fichier(s)** et chargez l’image dans le flux de travail.

![Un coureur avec une citation superposée à l'image](images/sample-text.jpg)

### <a name="track-progress"></a>Suivre la progression

Vous pouvez suivre la progression du flux de travail dans la fenêtre contextuelle suivante.

![Suivre l’exécution du flux de travail](images/image-workflow-job.PNG)

### <a name="verify-workflow-action"></a>Vérifier l’action du flux de travail

Ouvrez l'onglet **Image** sous **Révision**, puis vérifiez qu’il contient une nouvelle révision d'image.

![Réviser les images](images/image-workflow-review.PNG)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à configurer et à utiliser des flux de travail de modération fournis par l'[outil de révision](https://contentmoderator.cognitive.microsoft.com) Content Moderator. Consultez maintenant le [guide sur la console d’API](../try-review-api-workflow.md) pour apprendre à créer des flux de travail par programmation.
