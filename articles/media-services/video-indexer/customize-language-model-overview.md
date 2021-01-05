---
title: Personnaliser un modèle de langue dans Video Indexer - Azure
titleSuffix: Azure Media Services
description: Cet article donne une vue d’ensemble d’un modèle de langue dans Video Indexer et explique comment le personnaliser.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: kumud
ms.openlocfilehash: 29490e08748a37f7eb93fbb8804f55f74c53df35
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87047131"
---
# <a name="customize-a-language-model-with-video-indexer"></a>Personnaliser un modèle de langue avec Video Indexer

Video Indexer prend en charge la reconnaissance vocale automatique via l’intégration au service Microsoft [Custom Speech Service](https://azure.microsoft.com/services/cognitive-services/custom-speech-service/). Vous pouvez personnaliser le modèle de langue en chargeant un texte d’adaptation, à savoir un texte extrait du domaine de vocabulaire auquel vous souhaitez que le moteur s’adapte. Une fois votre modèle entraîné, les nouveaux mots qui apparaissent dans le texte d’adaptation sont reconnus, en adoptant la prononciation par défaut, et le modèle de langue apprend de nouvelles séquences probables de mots. Des modèles de langage personnalisés sont pris en charge pour l’anglais, l’espagnol, le français, l’allemand, l’italien, le chinois (simplifié), le japonais, le russe, le portugais, l’hindi et le coréen. 

Prenons un mot bien particulier tel que « Kubernetes » (dans le contexte d’Azure Kubernetes Service) comme exemple. Étant donné que Video Indexer voit ce mot pour la première fois, il le reconnaît comme « communities ». Vous devez entraîner le modèle pour le reconnaître comme « Kubernetes ». Dans d’autres cas, les mots existent, mais le modèle de langue ne les attend pas dans un contexte particulier. Par exemple, « service conteneur » n’est pas une séquence de 2 mots qu’un modèle de langue non spécialisé reconnaît comme un ensemble spécifique de mots.

Vous avez la possibilité de charger des mots sans contexte dans une liste dans un fichier texte. Cela est considéré comme une adaptation partielle. Vous pouvez aussi charger des fichiers texte de documentation ou des phrases qui se rapportent à votre contenu pour une meilleure adaptation.

Vous pouvez utiliser les API Video Indexer ou le site web pour créer et modifier des modèles de langue personnalisés, comme décrit dans les rubriques de la section [Étapes suivantes](#next-steps) de cette rubrique.

## <a name="best-practices-for-custom-language-models"></a>Bonnes pratiques pour les modèles de langue personnalisés

Comme Video Indexer apprend en se basant sur des probabilités de combinaisons de mots, pour l’optimiser :

* Donnez suffisamment d’exemples concrets de phrases telles qu’elles seraient dites.
* Placez uniquement une phrase par ligne, pas plus. Sinon, le système apprend des probabilités entre phrases.
* Vous pouvez indiquer un mot en tant que phrase pour le mettre en avant par rapport aux autres mots, mais le système apprend mieux à partir de phrases complètes.
* Lors de l’introduction de nouveaux mots ou acronymes, donnez autant d’exemples d’utilisation que possible dans une phrase complète pour fournir le plus de contexte possible au système.
* Essayez de fournir plusieurs options d’adaptation et voyez comment elles fonctionnent pour vous.
* Évitez de répéter plusieurs fois exactement la même phrase. Elle risque de fausser le reste de l’entrée.
* Évitez d’inclure des symboles peu courants (~, # @ % &), car ils seront ignorés. Les phrases dans lesquelles ils apparaissent seront également ignorées.
* Évitez les entrées trop longues, telles que des centaines de milliers de phrases, car sinon vous ne pourrez pas bénéficier pleinement de l’effet de mise en avant.

## <a name="next-steps"></a>Étapes suivantes

[Personnaliser le modèle de langue à l’aide des API](customize-language-model-with-api.md)

[Personnaliser le modèle de langue à l’aide du site web](customize-language-model-with-website.md)
