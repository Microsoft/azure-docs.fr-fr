---
title: Comment mettre à jour votre projet vers l’API 3.0
titleSuffix: Azure Cognitive Services
description: Découvrez comment mettre à jour des projets Custom Vision à partir de la version précédente de l’API vers l’API 3.0.
services: cognitive-services
author: areddish
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: areddish
ms.openlocfilehash: c134f30b124113a23df0e73cd1bbc8209e335183
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73647500"
---
# <a name="update-to-the-30-api"></a>Mettre à jour vers l’API 3.0

Custom Vision est désormais en disponibilité générale et a subi une mise à jour de l’API.
Cette mise à jour inclut plusieurs nouvelles fonctionnalités et, plus important encore, quelques changements cassants :

* L’API Prediction est désormais divisée en deux en fonction du type de projet.
* L’option d’exportation du Kit Vision AI Developer (VAIDK) nécessite la création d’un projet de manière spécifique.
* Les itérations par défaut ont été supprimées en faveur de la publication / dépublication d’une itération nommée.

Ce guide vous explique comment mettre à jour vos projets pour travailler avec la nouvelle version d’API. Consultez les [notes de publication](release-notes.md) pour obtenir la liste complète des modifications.

## <a name="use-the-updated-prediction-api"></a>Utiliser l’API Prediction mise à jour

Les API 2.x utilisaient le même appel de prédiction pour les projets de classifieurs d’images et de détecteurs d’objets. Les deux types de projets étaient acceptables pour les appels **PredictImage** et **PredictImageUrl**. À compter de la version 3.0, nous avons divisé cette API et vous devez faire correspondre le type de projet à l’appel :

* Utilisez **[ClassifyImage](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c15)** et **[ClassifyImageUrl](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c14)** afin d’obtenir des prédictions pour les projets de classification d’image.
* Utilisez **[DetectImage](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c19)** et **[DetectImageUrl](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c18)** afin d’obtenir des prédictions pour les projets de détection d’objet.

## <a name="use-the-new-iteration-publishing-workflow"></a>Utiliser le nouveau workflow de publication d’itération

Les API 2.x utilisaient l’itération par défaut ou un ID d’itération spécifié afin de choisir l’itération à utiliser pour la prédiction. À compter de la version 3.0, nous avons adopté un flux de publication dans lequel vous publiez tout d’abord une itération sous un nom spécifié à partir de l’API de formation. Ensuite, vous passez le nom aux méthodes de prédiction pour spécifier l’itération à utiliser.

> [!IMPORTANT]
> Les API 3.0 n’utilisent pas la fonctionnalité d’itération par défaut. Jusqu’à la dépréciation des anciennes API, vous pouvez continuer à utiliser les API 2.x pour activer/désactiver une itération par défaut. Ces API seront conservées pendant un certain temps, et vous pouvez appeler la méthode **[UpdateIteration](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.0/operations/5c771cdcbf6a2b18a0c3b818)** pour marquer une itération comme valeur par défaut.

### <a name="publish-an-iteration"></a>Publier une itération

Une fois qu’une itération est formée, vous pouvez la rendre disponible pour la prédiction à l’aide de la méthode **[PublishIteration](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.0/operations/5c82db28bf6a2b11a8247bbc)** . Pour publier une itération, vous aurez besoin de l’ID de ressource de prédiction, qui est disponible sur la page des paramètres du site web Custom Vision.

![Page des paramètres du site web Custom Vision avec l’ID de ressource de prédiction surlignée.](./media/update-application-to-3.0-sdk/prediction-id.png)

> [!TIP]
> Vous pouvez également obtenir cette information à partir du [portail Azure](https://portal.azure.com) en accédant à la ressource de prédiction Custom Vision et en sélectionnant **Propriétés**.

Une fois que votre itération est publiée, les applications peuvent l’utiliser pour la prédiction en spécifiant le nom dans leur appel de l’API de prédiction. Pour rendre une itération non disponible pour les appels de prédiction, utilisez l’API **[UnpublishIteration](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.0/operations/5c771cdcbf6a2b18a0c3b81a)** .

## <a name="next-steps"></a>Étapes suivantes

* [Documentation de référence sur l’API de formation (REST)](https://go.microsoft.com/fwlink/?linkid=865446)
* [Documentation de référence sur l’API de prédiction (REST)](https://go.microsoft.com/fwlink/?linkid=865445)