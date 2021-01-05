---
title: Étiqueter les images plus rapidement avec Smart Labeler
titleSuffix: Azure Cognitive Services
description: Dans ce guide, vous découvrirez comment utiliser Smart Labeler pour générer des balises suggérées pour les images. Vous pourrez ainsi étiqueter un grand nombre d’images plus rapidement lorsque vous formez un modèle Custom Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 10/29/2019
ms.author: pafarley
ms.openlocfilehash: 94ca47e6114e4f8c3485f6072facd07c25e4b96a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73647754"
---
# <a name="label-images-faster-with-smart-labeler"></a>Étiqueter les images plus rapidement avec Smart Labeler

Dans ce guide, vous découvrirez comment utiliser Smart Labeler pour générer des balises suggérées pour les images. Vous pourrez ainsi étiqueter un grand nombre d’images plus rapidement lorsque vous formez un modèle Custom Vision.

Quand vous étiquetez des images pour un modèle Custom Vision, le service utilise la dernière itération entraînée du modèle pour prédire les étiquettes d’images non balisées. Il présente ensuite ces prédictions sous forme de suggestions d’étiquettes en fonction du seuil de confiance sélectionné et du niveau d’incertitude de prédiction. Vous pouvez alors confirmer ou modifier les suggestions, ce qui a pour effet d’accélérer l’étiquetage manuel des images pour l’entraînement.

## <a name="when-to-use-smart-labeler"></a>Quand utiliser Smart Labeler ?

Gardez à l’esprit les limitations suivantes :

* Vous n’avez qu’à demander des étiquettes suggérées pour les images dont le contenu a déjà subi un apprentissage une fois. Ne sollicitez pas de suggestions pour une nouvelle étiquette dont vous commencez à peine l’entraînement.

> [!IMPORTANT]
> La fonctionnalité Smart Labeler s’appuie sur le même [modèle de prix](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/) que les prédictions normales. La première fois que vous déclenchez les suggestions d’étiquettes pour un ensemble d’images, vous êtes facturé comme si vous appeliez des prédictions. Après quoi, le service stocke les résultats pour les images sélectionnées dans une base de données pendant 30 jours, et vous pouvez y accéder gratuitement à tout moment au cours de cette période. Passé ce délai de 30 jours, vous êtes facturé si vous demandez à nouveau ces suggestions d’étiquettes.

## <a name="smart-labeler-workflow"></a>Workflow de Smart Labeler

Les étapes suivantes montrent comment utiliser Smart Labeler :

1. Chargez toutes vos images d’entraînement dans votre projet Custom Vision.
1. Étiquetez une partie de votre jeu de données en choisissant un nombre égale d’images pour chaque étiquette.
    > [!TIP]
    > Veillez à utiliser toutes les étiquettes pour lesquelles vous souhaitez obtenir des suggestions par la suite.
1. Commencez le processus d’entraînement.
1. Une fois l’entraînement terminé, accédez à la vue **Untagged** (Non étiqueté), puis sélectionnez le bouton **Get suggested tags** (Obtenir des suggestions d’étiquettes) dans le volet gauche.
    > [!div class="mx-imgBorder"]
    > ![Le bouton de suggestion d’étiquettes figure sous l’onglet des images non étiquetées.](./media/suggested-tags/suggested-tags-button.png)
1. Dans la fenêtre contextuelle qui s’affiche, définissez le nombre d’images pour lesquelles vous souhaitez obtenir des suggestions. Dans un premier temps, vous obtenez des suggestions d’étiquettes seulement pour une partie des images non étiquetées. Des suggestions plus abouties vous sont ensuite proposées à mesure que vous répétez ce processus.
1. Confirmez les étiquettes suggérées tout en prenant soin de corriger celles qui sont incorrectes.
    > [!TIP]
    > Les images assorties d’étiquettes suggérées sont triées selon le niveau d’incertitude de leur prédiction (des valeurs faibles indiquant un haut niveau de confiance). Vous pouvez changer l’ordre de tri à l’aide de l’option **Sort by uncertainty** (Trier par niveau d’incertitude). Si vous définissez cet ordre sur **high to low** (élevé à faible), vous pouvez corriger d’abord les prédictions à fort niveau d’incertitude, puis confirmer rapidement celles dont le niveau d’incertitude est faible.
    * Dans les projets de classification d’images, vous pouvez sélectionner et confirmer les étiquettes par lots (batch). Filtrez la vue en fonction d’une balise suggérée déterminée, désélectionnez les images dont l’étiquetage est incorrect, puis confirmez les autres en une seule opération.
        > [!div class="mx-imgBorder"]
        > ![Les suggestions d’étiquettes s’affichent en mode batch pour la classification d’images avec des filtres.](./media/suggested-tags/ic-batch-mode.png)

        Vous pouvez aussi utiliser les suggestions d’étiquettes en mode image individuelle en sélectionnant une image dans la galerie.

        ![Les suggestions d’étiquettes s’affichent en mode image individuelle pour la classification d’images.](./media/suggested-tags/ic-individual-image-mode.png)
    * Dans les projets de détection d’objets, les confirmations par lots ne sont pas prises en charge. Il vous reste cependant la possibilité de filtrer et trier l’affichage en fonction d’étiquettes suggérées pour une expérience d’étiquetage mieux organisée. Les miniatures de vos images étiquetées présentent un cadre englobant superposé qui indique l’emplacement de l’étiquette suggérée. Si vous ne sélectionnez pas de filtre d’étiquette suggérée, toutes vos images non étiquetées s’affichent sans cadre englobant superposé.
        > [!div class="mx-imgBorder"]
        > ![Les suggestions d’étiquettes s’affichent en mode batch pour la détection d’objets avec des filtres.](./media/suggested-tags/od-batch-mode.png)

        Pour confirmer les étiquettes de détection d’objets, vous devez les appliquer individuellement à chaque image de la galerie.

        ![Les étiquettes suggérées s’affichent en mode image individuelle dans le cas de la détection d’objets.](./media/suggested-tags/od-individual-image-mode.png)
1. Recommencez le processus d’entraînement.
1. Répétez les étapes précédentes jusqu’à ce que la qualité de suggestion vous convienne.

## <a name="next-steps"></a>Étapes suivantes

Suivez un guide de démarrage rapide pour commencer à créer et entraîner un projet Custom Vision.

* [Créer un classifieur](getting-started-build-a-classifier.md)
* [Créer un détecteur d’objets](get-started-build-detector.md)