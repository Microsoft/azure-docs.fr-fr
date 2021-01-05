---
title: Support pour les conteneurs
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 2647c604bebc2885f5c8516eb997792b65751e88
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80876074"
---
## <a name="create-an-computer-vision-resource"></a>Créer une ressource Vision par ordinateur

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Cliquez sur la ressource [Créer **Vision par ordinateur**](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision).
1. Entrez tous les paramètres obligatoires :

    |Paramètre|Valeur|
    |--|--|
    |Name|Nom de votre choix (2-64 caractères)|
    |Subscription|Sélectionner l’abonnement approprié|
    |Location|Sélectionnez n’importe quel emplacement disponible et proche|
    |Niveau de tarification|`F0` - le niveau tarifaire minimal|
    |Groupe de ressources|Sélectionner un groupe de ressources disponible|

1. Cliquez sur **Créer** et attendez que la ressource soit créée. Après sa création, accédez à la page de ressources.
1. Collectez les `{ENDPOINT_URI}` et `{API_KEY}` configurés, consultez [collecte des paramètres requis](../computer-vision-how-to-install-containers.md#gathering-required-parameters) pour obtenir plus d’informations.
