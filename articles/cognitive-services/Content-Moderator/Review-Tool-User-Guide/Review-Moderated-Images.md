---
title: Utiliser les révisions de contenu via l’outil de vérification - Content Moderator
titleSuffix: Azure Cognitive Services
description: Découvrez comment l'outil de révision permet aux modérateurs humains d'examiner des images sur un portail web.
services: cognitive-services
author: PatrickFarley
manager: mikemcca
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/15/2019
ms.author: pafarley
ms.openlocfilehash: 77d7b8a0bf4d7fe9a94a61ea3f2f4279246ffb69
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903885"
---
# <a name="create-human-reviews"></a>Créer des révisions humaines

Dans ce guide, vous apprendrez à configurer des [révisions](../review-api.md#reviews) sur le site web de l’outil de révision. Les révisions stockent et affichent le contenu que les modérateurs humains vont évaluer. Les modérateurs peuvent modifier les balises appliquées et appliquer leurs propres balises personnalisées selon leurs besoins. Lorsqu’un utilisateur termine une révision, les résultats sont envoyés à un point de terminaison de rappel spécifié, puis le contenu est retiré du site.

## <a name="prerequisites"></a>Conditions préalables requises

- Connectez-vous ou créez un compte sur le site de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.

## <a name="image-reviews"></a>Révisions d’images

1. Accédez à l’[outil de révision](https://contentmoderator.cognitive.microsoft.com/), sélectionnez l’onglet **Essai**, puis chargez des images à réviser.
1. Une fois le traitement des images échargées terminé, accédez à l'onglet **Révision**, puis sélectionnez **Image**.

    ![Navigateur Chrome montrant l’outil de révision avec l’option Réviser - Image en surbrillance](images/review-images-1.png)

    Les images sont affichées avec des étiquettes attribuées par le processus de modération automatique. Les images que vous avez soumises via l'outil de révision ne sont pas visibles pour les autres évaluateurs.

1. Vous pouvez également déplacer le curseur **Révisions à afficher** (1) pour ajuster le nombre d’images affichées à l’écran. Cliquez sur les boutons permettant de trier les images qui sont accompagnées de **balises** et les **autres** (2). Cliquez sur un panneau de balise (3) pour l’activer ou la désactiver.

    ![Navigateur Chrome montrant l’outil de révision avec des images marquées pour révision](images/review-images-2.png)

1. Pour plus d’informations sur une image, cliquez sur les points de suspension dans la miniature, puis sélectionnez **Afficher les détails**. Vous pouvez attribuer une image à une sous-équipe à l’aide de l'option **Déplacer vers** (voir la section [équipes](./configure.md#manage-team-and-subteams) pour en savoir plus sur les sous-équipes).

    ![Une image avec l’option Afficher les détails en surbrillance](images/review-images-3.png)

1. Parcourez les informations de modération de l’image sur la page de détails.

    ![Une image avec les détails de modération listés dans un volet distinct](images/review-images-4.png)

1. Une fois que vous avez révisé et mis à jour les balises attribuées en fonction de la situation, cliquez sur **Suivant** pour envoyer vos révisions. Une fois que vous avez envoyé vos commentaires, vous disposez d’un délai de cinq secondes pour cliquer sur le bouton **Prec** pour revenir à l’écran précédent et examiner à nouveau les images. Passé ce délai, les images ne sont plus dans la file d’attente d’envoi et le bouton **Prec** n’est plus disponible.

## <a name="text-reviews"></a>Révisions de texte

Les révisions de texte fonctionnent de la même manière que les révisions d'images. Au lieu de charger du contenu, il vous suffit d'écrire ou de coller du texte (jusqu'à 1 024 caractères). Ensuite, Content Moderator analyse le texte et applique des balises (en plus d'autres informations de modération, comme les grossièretés et les données personnelles). Dans les révisions de texte, vous pouvez activer/désactiver les balises appliquées et/ou appliquer des balises personnalisées avant de soumettre la révision.

![Capture d’écran de l’outil de vérification montrant du texte marqué d’indicateurs dans une fenêtre du navigateur Chrome](../images/reviewresults_text.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à configurer et à utiliser des révisions fournies par l'[outil de révision](https://contentmoderator.cognitive.microsoft.com) Content Moderator. Consultez maintenant le [Guide sur la console d’API](../try-review-api-review.md) ou le [Démarrage rapide du SDK .NET](../client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp) pour apprendre à créer des révisions par programmation.