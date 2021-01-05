---
title: Documentation Swagger - Service Speech
titleSuffix: Azure Cognitive Services
description: La documentation Swagger peut être utilisée pour générer automatiquement des SDK pour plusieurs langages de programmation. Toutes les opérations de notre service sont prises en charge par Swagger
services: cognitive-services
author: PanosPeriorellis
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: reference
ms.date: 07/05/2019
ms.author: erhopf
ms.openlocfilehash: 6bb50e427fa85a170c5ad23a63d67c01e898a17d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91665682"
---
# <a name="swagger-documentation"></a>Documentation Swagger

Le service Speech offre une spécification Swagger pour interagir avec quelques API REST utilisées pour importer des données, créer des modèles, tester le niveau de justesse des modèles, créer des points de terminaison personnalisés, mettre en file d’attente des transcriptions par lots et gérer des abonnements. La plupart des opérations disponibles par le biais du portail Custom Speech peuvent être effectuées par programmation à l’aide de ces API.

> [!NOTE]
> Les opérations de reconnaissance vocale et de synthèse vocale sont prises en charge et disponibles en tant qu’API REST, à leur tour décrites dans la spécification Swagger.

## <a name="generating-code-from-the-swagger-specification"></a>Génération de code à partir de la spécification Swagger

La [spécification Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) comporte des options qui vous permettent de tester rapidement divers parcours. En revanche, il est parfois souhaitable de générer du code pour tous les parcours, en créant une bibliothèque unique d’appels sur laquelle vous pouvez baser les futures solutions. Intéressons-nous au processus permettant de générer une bibliothèque Python.

Vous avez besoin de définir Swagger sur la même région que celle de votre abonnement au service Speech. Vous pouvez vérifier votre région dans le portail Azure, sous votre ressource Speech. Pour obtenir la liste complète des régions prises en charge, consultez [Régions](regions.md).

1. Dans un navigateur, accédez à la spécification Swagger de votre région :  
       `https://<your-region>.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0`
1. Sur cette page, cliquez sur **Définition de l'API**, puis sur **Swagger**. Copiez l'URL de la page qui s'affiche.
1. Dans un nouveau navigateur, accédez à https://editor.swagger.io
1. Cliquez sur **Fichier**, sur **Importer l'URL**, collez l'URL, puis cliquez sur **OK**.
1. Cliquez sur **Générer un client** et sélectionnez **python**. La bibliothèque de client est téléchargée sur votre ordinateur sous forme de fichier `.zip`.
1. Extrayez l'ensemble du téléchargement. Vous pouvez utiliser `tar -xf` pour tout extraire.
1. Installez le module extrait dans votre environnement Python :  
       `pip install path/to/package/python-client`
1. Le package installé porte le nom suivant : `swagger_client`. Vérifiez que l’installation a fonctionné :  
       `python -c "import swagger_client"`

Vous pouvez utiliser la bibliothèque Python que vous avez générée avec les [exemples du service Speech sur GitHub](https://aka.ms/csspeech/samples).

## <a name="reference-docs"></a>Documents de référence

* [REST (Swagger) : Transcription et personnalisation par lot](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)
* [API REST : Reconnaissance vocale](rest-speech-to-text.md)
* [API REST : Synthèse vocale](rest-text-to-speech.md)

## <a name="next-steps"></a>Étapes suivantes

* [Exemples Speech sur GitHub](https://aka.ms/csspeech/samples)
* [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
