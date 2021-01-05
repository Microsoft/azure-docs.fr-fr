---
author: PatrickFarley
ms.service: cognitive-services
ms.topic: include
ms.date: 02/07/2019
ms.author: pafarley
ms.openlocfilehash: 874d76bebdfd3bd0daba1f83cb1f06c093f192ec
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021270"
---
## <a name="get-the-training-and-prediction-keys"></a>Obtenir les clés d’entraînement et de prédiction

Le projet doit avoir un jeu de clés d’abonnement valide afin d’interagir avec le service. Vous pouvez trouver les éléments sur le [site web de Custom Vision](https://customvision.ai). Connectez-vous avec le compte associé au compte Azure que vous avez utilisé pour créer vos ressources Custom Vision. Dans la page d’accueil (la page avec l’option permettant d’ajouter un nouveau projet), sélectionnez l’__icône d’engrenage__ en haut à droite. Recherchez vos ressources d’entraînement et de prédiction dans la liste et développez-les. Vous trouverez ici les valeurs de la clé d’entraînement, de la clé de prédiction et de l’ID de ressource de prédiction. Enregistrez ces valeurs dans un emplacement temporaire.

> [!NOTE]
> Si vous utilisez une clé tout-en-un Cognitive Services pour accéder à Custom Vision, vous ne verrez qu’une seule clé sur l’écran des paramètres. Dans ce cas, vous allez utiliser la même clé pour les opérations d’entraînement et de prédiction.

![Image de l’interface utilisateur des clés](../media/csharp-tutorial/training-prediction-keys.png)

Sinon, vous pouvez obtenir ces clés et l’ID sur le [portail Azure](https://www.portal.azure.com), en consultant vos ressources d’entraînement et de prédiction Custom Vision et en accédant à l’onglet __Clés__. C’est là que vous trouverez votre clé d’entraînement et votre clé de prédiction. Accédez à l’onglet __Propriétés__ de votre ressource de prédiction pour obtenir votre ID de ressource de prédiction.

