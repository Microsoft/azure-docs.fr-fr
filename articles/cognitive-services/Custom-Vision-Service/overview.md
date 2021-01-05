---
title: Qu’est-ce que Custom Vision ?
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser le service Azure Custom Vision pour créer des détecteurs d’images et des classifieurs d’images IA personnalisés dans le cloud Azure.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: overview
ms.date: 12/15/2020
ms.author: pafarley
ms.custom: cog-serv-seo-aug-2020
keywords: reconnaissance d’image, identificateur d’image, application de reconnaissance d’image, vision personnalisée
ms.openlocfilehash: 7274fed961c4bbac8841e1c741f54031614cea5c
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562988"
---
# <a name="what-is-custom-vision"></a>Qu’est-ce que Custom Vision ?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Azure Custom Vision est un service de reconnaissance d’image qui vous permet de créer, de déployer et d’améliorer vos propres identificateurs d’images. Un identificateur d’images applique des étiquettes (représentant des classes ou des objets) à des images en fonction de leurs caractéristiques visuelles. Contrairement au service [Vision par ordinateur](../computer-vision/overview.md), le service Custom Vision vous permet de spécifier les étiquettes et d’entraîner des modèles personnalisés pour les détecter.

## <a name="what-it-does"></a>Résultat

Le service Custom Vision utilise un algorithme de Machine Learning pour analyser les images. En tant que développeur, vous envoyez des groupes d’images présentant les caractéristiques en question ainsi que des groupes d’images n’en disposant pas. Vous étiquetez les images vous-même au moment de la soumission. L’algorithme s’entraîne ensuite avec ces données et calcule sa propre justesse en se testant lui-même à l’aide de ces mêmes images. Une fois l’algorithme entraîné, vous pouvez le tester, le réentraîner, puis l’utiliser dans votre application de reconnaissance d’images pour classifier de nouvelles images. Vous pouvez également exporter le modèle en question pour l’utiliser en mode hors connexion.

### <a name="classification-and-object-detection"></a>Classification et détection d’objet

Les fonctionnalités du service Vision personnalisée peuvent être divisées en deux fonctionnalités. La **classification d’images** applique une ou plusieurs étiquettes à une image. La **détection d’objet** est similaire, mais elle retourne également les coordonnées dans l’image où se trouvent la ou les étiquettes appliquées.

### <a name="optimization"></a>Optimization

Le service Custom Vision est optimisé pour reconnaître rapidement les principales différences entre les images. Vous pouvez ainsi commencer à prototyper votre modèle avec une petite quantité de données. Il est généralement conseillé de commencer avec 50 images par étiquette. Toutefois, le service n’est pas optimal pour la détection de différences subtiles au sein des images (par exemple, la détection de petites fissures ou de petits trous dans des scénarios d’assurance qualité).

En outre, vous pouvez choisir parmi plusieurs types d’algorithme de Custom Vision qui sont optimisés pour certains sujets (par exemple, des points de repère ou des articles vendus au détail). Pour plus d’informations, consultez les guides [Créer un classifieur](getting-started-build-a-classifier.md) et [Créer un détecteur d’objet](get-started-build-detector.md).

## <a name="what-it-includes"></a>Contenu

Le service Custom Vision est disponible sous la forme d’un ensemble de kits SDK natifs ainsi que via une interface web sur le [site web du service Custom Vision](https://customvision.ai/). Vous pouvez créer, tester et entraîner un modèle avec une interface ou les deux.

![Site web du service Custom Vision dans une fenêtre du navigateur Chrome](media/browser-home.png)

## <a name="data-privacy-and-security"></a>Sécurité et confidentialité des données

Comme avec tous les services Cognitive Services, les développeurs utilisant le service Vision personnalisée doivent connaître les politiques de Microsoft relatives aux données client. Pour en savoir plus, consultez la [page Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) dans le Centre de gestion de la confidentialité Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Suivez le guide [Créer un classifieur](getting-started-build-a-classifier.md) pour découvrir comment utiliser le service Custom Vision sur le portail web. Sinon, vous pouvez suivre un [guide de démarrage rapide consacré à une bibliothèque de client](quickstarts/image-classification.md) pour implémenter les scénarios de base dans le code.