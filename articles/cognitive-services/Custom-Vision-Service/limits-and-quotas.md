---
title: Limites et quotas – Service Vision personnalisée
titleSuffix: Azure Cognitive Services
description: Cet article présente les différents types de clés de licence, ainsi que les limites et quotas applicables au service Custom Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: pafarley
ms.openlocfilehash: b79bf5e8ead16bbdf9c69e8d7faae43fa778ab3e
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91316577"
---
# <a name="limits-and-quotas"></a>Limites et quotas

Il existe deux niveaux de clés pour le service Vision personnalisée. Vous pouvez souscrire à un abonnement F0 (gratuit) ou S0 (standard) via le portail Azure. Consultez la page [Tarification de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/) correspondante pour en savoir plus sur la tarification et les transactions.

Le nombre d’images d’apprentissage par projet et le nombre de balises par projet sont censés augmenter au fil du temps pour les projets S0.

|Factor|**F0**|**S0**|
|-----|-----|-----|
|Projets|2|100|
|Images d’apprentissage par projet |5 000|100 000|
|Prédictions/mois|10 000 |Illimité|
|Étiquettes/projet|50|500|
|Itérations |10|10|
|Nombre minimal d’images étiquetées par balise, classification (recommandation : plus de 50) |5|5|
|Nombre minimal d’images étiquetées par balise, détection d’objets (recommandation : plus de 50)|15|15|
|Durée de stockage des images de prédiction|30 jours|30 jours|
|Opérations de [prédiction](https://go.microsoft.com/fwlink/?linkid=865445) avec stockage (transactions par seconde)|2|10|
|Opérations de [prédiction](https://go.microsoft.com/fwlink/?linkid=865445) sans stockage (transactions par seconde)|2|20|
|[TrainProject](https://go.microsoft.com/fwlink/?linkid=865446) (appels d’API par seconde)|2|10|
|[Autres appels d’API](https://go.microsoft.com/fwlink/?linkid=865446) (transactions par seconde)|10|10|
|Types d’images acceptés|jpg, png, bmp, gif|jpg, png, bmp, gif|
|Hauteur/largeur d’image minimale en pixels|256 (voir la remarque)|256 (voir la remarque)|
|Hauteur/largeur d’image maximale en pixels|10 240|10 240|
|Taille maximale de l’image (chargement de l’image d’apprentissage) |6 Mo|6 Mo|
|Taille maximale de l’image (prédiction)|4 Mo|4 Mo|
|Nombre maximal de régions par image d’entraînement de détection d’objets|300|300|
|Nombre maximal de balises par image de classification|100|100|

> [!NOTE]
> Les images inférieures à 256 pixels seront acceptées mais mises à l’échelle.
> Les proportions de l’image ne doivent pas être supérieures à 25.
