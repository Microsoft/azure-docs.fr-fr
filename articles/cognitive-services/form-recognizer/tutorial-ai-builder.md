---
title: 'Tutoriel : Créer une application de traitement de formulaires avec AI Builder - Form Recognizer'
titleSuffix: Azure Cognitive Services
description: Dans ce tutoriel, vous allez utiliser AI Builder pour créer et entraîner une application de traitement de formulaires.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: tutorial
ms.date: 11/23/2020
ms.author: pafarley
ms.openlocfilehash: 3ba08f11fb52fc0ffe3acda5e81e21b7e7629628
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95757418"
---
# <a name="tutorial-create-a-form-processing-app-with-ai-builder"></a>Tutoriel : Créer une application de traitement de formulaires avec AI Builder

[AI Builder](/ai-builder/overview) est une fonctionnalité de Power Platform qui vous permet d’automatiser les processus et de prédire des résultats pour améliorer les performances de l’entreprise. Vous pouvez utiliser le traitement de formulaires d’AI Builder pour créer des modèles IA qui identifient et extraient des paires clé-valeur et des données de table à partir de documents de formulaire.

> [!NOTE]
> Ce projet est également disponible sous la forme de [module Microsoft Learn](/learn/modules/get-started-with-form-processing/).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un modèle IA de traitement de formulaires
> * Entraîner votre modèle
> * Publier votre modèle pour une utilisation dans Azure Power Apps ou Power Automate

## <a name="prerequisites"></a>Prérequis

* Un ensemble d’au moins cinq formulaires de même type à utiliser pour les données d’entraînement/de test. Consultez [Créer un jeu de données d’entraînement](./build-training-data-set.md) pour obtenir des conseils et des options pour constituer votre jeu de données d’entraînement. Dans le cadre de ce guide de démarrage rapide, vous pouvez utiliser les fichiers disponibles dans le dossier **Train** de l’[exemple de jeu de données](https://go.microsoft.com/fwlink/?linkid=2128080).
* Une licence Power Apps ou Power Automate - consultez le [guide de gestion des licences](https://go.microsoft.com/fwlink/?linkid=2085130). La licence doit inclure [Common Data Service](https://powerplatform.microsoft.com/common-data-service/).
* Un [module complémentaire ou une version d’essai gratuit](https://go.microsoft.com/fwlink/?LinkId=2113956&clcid=0x409) d’AI Builder.


## <a name="create-a-form-processing-project"></a>Créer un projet de traitement de formulaires

1. Accédez à [Power Apps](https://make.powerapps.com/) ou [Power Automate](https://flow.microsoft.com/signin), puis connectez-vous avec votre compte d’entreprise.
1. Dans le volet gauche, sélectionnez **AI Builder** > **Générer**.
1. Sélectionnez la carte **Traitement de formulaires**.
1. Tapez un nom pour votre modèle.
1. Sélectionnez **Create** (Créer).

## <a name="upload-and-analyze-documents"></a>Charger et analyser des documents

Dans la page **Ajouter des documents**, vous devez ajouter des exemples de documents afin d’entraîner votre modèle pour le type de formulaire à partir duquel vous souhaitez extraire des informations. AI Builder analyse les documents chargés pour vérifier qu’ils sont suffisants pour entraîner le modèle.

> [!NOTE]
> Actuellement, AI Builder ne prend pas en charge les types de données d’entrée suivants pour le traitement de formulaires :
>
> - Tables complexes (tables imbriquées, cellules ou en-têtes fusionnés, etc.)
> - Cases à cocher ou cases d’option
> - Documents PDF de plus de 50 pages
> - PDF remplissables
>
> Pour plus d’informations sur les exigences relatives aux documents d’entrée, consultez [Critères des entrées](./overview.md#input-requirements).

### <a name="upload-your-documents"></a>Charger les documents

1. Sélectionnez **Ajouter des documents**, sélectionnez au moins cinq documents, puis sélectionnez **Charger**.
1. Quand le chargement est terminé, sélectionnez **Fermer**.
1. Sélectionnez ensuite **Analyser**.

> [!NOTE] 
> Après avoir chargé ces documents, vous pourrez toujours en charger d’autres ou en supprimer.

> [!div class="mx-imgBorder"]
> ![Capture d’écran mettant en évidence le bouton Ajouter des documents.](./media/tutorial-ai-builder/add-documents-page.png)

### <a name="analyze-your-documents"></a>Analyser les documents

À l’étape d’analyse, AI Builder examine les documents que vous avez chargés et détecte les champs et les tables. La durée de cette opération dépend du nombre de documents fournis, mais se limite généralement à quelques minutes.

Quand l’analyse est terminée, sélectionnez la miniature pour ouvrir l’expérience de sélection de champ.

> [!IMPORTANT]
> Si l’analyse a échoué, il est probable qu’AI Builder n’ait pas pu détecter le texte structuré dans vos documents. Vérifiez que les documents que vous avez mis à jour respectent les [critères des entrées](./overview.md#input-requirements).

## <a name="select-your-form-fields"></a>Sélectionner les champs de formulaire

Sur la page de sélection des champs, choisissez les champs qui vous intéressent :

1. Pour sélectionner un champ, cliquez sur un rectangle indiquant un champ détecté dans le document ou sélectionnez plusieurs champs en cliquant et en faisant glisser le curseur de la souris. Vous pouvez également sélectionner des champs dans le volet de droite.
1. Cliquez sur le nom du champ sélectionné si vous souhaitez le renommer en fonction de vos besoins ou pour standardiser les étiquettes extraites.

    Quand vous cliquez sur un champ détecté, les informations suivantes s’affichent :

    - **Nom du champ** : nom de l’étiquette du champ détecté
    - **Valeur de champ** : valeur du champ détecté

> [!div class="mx-imgBorder"]
> ![Page d’étiquetage](./media/tutorial-ai-builder/select-fields-page.png)

### <a name="label-undetected-fields"></a>Étiqueter les champs non détectés

Si le champ que vous souhaitez étiqueter n’a pas été détecté automatiquement par le modèle, vous pouvez tracer un rectangle autour de sa valeur et taper un nom d’étiquette dans la boîte de dialogue qui s’affiche.

## <a name="train-your-model"></a>Entraîner votre modèle

1. Sélectionnez **Suivant** pour vérifier les champs de formulaire sélectionnés. Si tout semble correct, sélectionnez **Entraîner** pour entraîner le modèle.

    > [!div class="mx-imgBorder"]
    > ![Page d’entraînement](./media/tutorial-ai-builder/summary-train-page.png)
1. Quand l’entraînement est terminé, sélectionnez **Aller à la page Détails** dans l’écran **Entraînement effectué**.
## <a name="quick-test-your-model"></a>Effectuer un test rapide du modèle

La page Détails vous permet de tester le modèle avant de le publier ou de l’utiliser :

1. Dans la page Détails, sélectionnez **Test rapide**.
2. Vous pouvez glisser-déposer un document ou sélectionner **Charger à partir de mon appareil** pour charger le fichier de test. L’affichage des résultats du test rapide ne doit prendre que quelques secondes.
3. Vous pouvez sélectionner **Recommencer** pour exécuter un autre test ou **Fermer** si vous avez terminé.

### <a name="troubleshooting-tips"></a>Conseils de dépannage

Si vous obtenez de mauvais résultats ou des scores de confiance faibles pour certains champs, appliquez les conseils suivants :

- Procédez de nouveau à l’entraînement en utilisant des formulaires avec des valeurs différentes dans chaque champ.
- Procédez de nouveau à l’entraînement en utilisant un plus grand ensemble de documents d’entraînement. Plus vous étiquetez de documents, plus AI Builder apprendra à mieux reconnaître les champs.
- Vous pouvez optimiser les fichiers PDF en sélectionnant uniquement certaines pages pour l’entraînement. Utilisez l’option **Imprimer** > **Imprimer dans un PDF** pour sélectionner des pages spécifiques de votre document.

## <a name="publish-your-model"></a>Publier le modèle

Si vous êtes satisfait de votre modèle, sélectionnez **Publier** pour le publier. Quand la publication est effectuée, votre modèle est promu comme **Publié** et est prêt à être utilisé.

> [!div class="mx-imgBorder"]
> ![Page de publication du modèle](./media/tutorial-ai-builder/model-page.png)

Après avoir publié votre modèle de traitement de formulaires, vous pouvez l’utiliser dans une [application de canevas dans Power Apps](/ai-builder/form-processor-component-in-powerapps) ou dans [Power Automate](/ai-builder/form-processing-model-in-flow).

## <a name="next-steps"></a>Étapes suivantes

Reportez-vous à la documentation d’AI Builder pour utiliser un modèle de traitement de formulaires.

* [Utiliser le composant de processeur de formulaire dans Power Apps](/ai-builder/form-processor-component-in-powerapps)
* [Utiliser un modèle de traitement de formulaires dans Power Automate](/ai-builder/form-processing-model-in-flow)