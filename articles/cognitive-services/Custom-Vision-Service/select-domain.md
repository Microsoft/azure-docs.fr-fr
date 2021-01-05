---
title: Sélectionner un domaine pour un projet Custom Vision - Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Cet article vous explique comment sélectionner un domaine pour votre projet dans le service Custom Vision.
services: cognitive-services
author: shonohs
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 03/06/2020
ms.author: shono
ms.openlocfilehash: 87b9e4a3ca7151b3666928b00add175eddeea050
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94409380"
---
# <a name="select-a-domain-for-a-custom-vision-project"></a>Sélectionner un domaine pour un projet Custom Vision

Le panneau Paramètres de votre projet Custom Vision vous permet de sélectionner un domaine. Choisissez le domaine qui se rapproche le plus de votre scénario.

## <a name="image-classification"></a>Classification d’image

|Domain|Objectif|
|---|---|
|__Générique__| Optimisé pour un large éventail de tâches de classification d’images. Si aucun autre domaine n’est approprié, ou si vous hésitez sur le choix du domaine, sélectionnez le domaine Générique.|
|__Food__ (Nourriture)|Optimisé pour les photographies de plats, tels que vous pouvez les voir dans un menu de restaurant. Si vous souhaitez classer des photographies de fruits ou de légumes distincts, utilisez le domaine Food (Nourriture).|
|__Landmarks__ (Monuments et sites)|Optimisé pour les monuments et sites reconnaissables, naturels et artificiels. Ce domaine fonctionne mieux lorsque le monument ou le site est clairement visible dans la photographie. Ce domaine fonctionne même si le monument ou le site est légèrement masqué par des personnes placées devant lui.|
|__Retail__ (Commerce)|Optimisé pour les images qui se trouvent dans des catalogues de vente ou sur des site Web commerciaux. Si vous souhaitez un classement de grande précision pour des robes, des pantalons et des chemises, utilisez ce domaine.|
|__Compact Domains__ (Domaines compacts)| Optimisés en fonction des contraintes liées à la classification en temps réel sur les périphériques.|

## <a name="object-detection"></a>Détection d’objets

|Domain|Objectif|
|---|---|
|__Généralités__| Optimisé pour un large éventail de tâches de détection d’objets. Si aucun autre domaine n’est approprié, ou si vous hésitez sur le choix du domaine, sélectionnez le domaine Generic (Général).|
|__Logo__|Optimisé pour rechercher des logos de marque dans les images.|
|__Products on shelves__ (Produits en rayon)|Optimisé pour la détection et la classification des produits en rayon.|
|__Compact Domains__ (Domaines compacts)| Optimisés en fonction des contraintes liées à la détection d'objets en temps réel sur les périphériques.|

## <a name="compact-domains"></a>Domaines compacts

Les modèles générés par les domaines compacts sont exportables pour s’exécuter localement. Dans l’API Custom Vision 3.4 en préversion publique, vous pouvez obtenir la liste des plateformes exportables pour les domaines compacts en appelant l’API GetDomains.

Les performances du modèle varient selon le domaine sélectionné. Le tableau ci-dessous spécifie la taille du modèle et le temps d'inférence sur processeur Intel Desktop et GPU NVidia \[1\]. Ces chiffres n'incluent pas les délais de prétraitement et de post-traitement.

|Tâche|Domain|Taille des modèles|Temps d'inférence sur processeur|Temps d'inférence sur GPU|
|---|---|---|---|---|
|classification ;|General (compact) (Général (compact))|5 Mo|13 ms|5 ms|
|Détection d’objets|General (compact) (Général (compact))|45 Mo|35 ms|5 ms|
|Détection d’objets|General (compact) [S1] (Général (compact) [S1])|14 Mo|27 ms|7 ms|

>[!NOTE]
>Le domaine __General (compact)__ (Général (compact)) disponible pour la détection d'objets requiert une logique de post-traitement particulière. Pour plus d'informations, un exemple de script est disponible dans le package zip exporté. Si vous avez besoin d'un modèle sans logique de post-traitement, utilisez __General (compact) [S1]__ (Général (compact) [S1]).

>[!IMPORTANT]
>Rien ne garantit que les modèles exportés donneront exactement les mêmes résultats que l'API de prédiction du cloud. Une légère différence en termes de plateforme exécutée ou d'implémentation du prétraitement peut donner des résultats très différents avec les modèles. Pour plus d'informations sur la logique de prétraitement, consultez [ce document](quickstarts/image-classification.md).

\[1\] Processeur Intel Xeon E5-2690 et GPU NVIDIA Tesla M60
