---
title: Identifier et transcrire automatiquement du contenu multilingue avec Video Indexer
titleSuffix: Azure Media Services
description: Cette rubrique montre comment identifier et transcrire automatiquement du contenu multilingue avec Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 09/01/2019
ms.author: juliako
ms.openlocfilehash: f0dede42891069bb5d01ddc33f3797c20c5493d7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "72968736"
---
# <a name="automatically-identify-and-transcribe-multi-language-content-preview"></a>Identifier et transcrire automatiquement du contenu multilingue (préversion)

Video Indexer prend en charge l’identification automatique des langues et la transcription en contenu multilingue. Ce processus implique d’identifier automatiquement la langue parlée dans différents segments à partir d’un support audio, d’envoyer chaque segment du fichier multimédia à transcrire et de regrouper toutes les transcriptions en une seule. 

## <a name="choosing-multilingual-identification-on-indexing-with-portal"></a>Choix d’une identification multilingue pour l’indexation à l’aide du portail

Vous pouvez choisir la **détection multilingue** lors du chargement et de l’indexation de votre vidéo. Vous pouvez également choisir la **détection multilingue** lors de la réindexation de votre vidéo. Les étapes suivantes décrivent le processus de réindexation :

1. Accédez au site web [Video Indexer](https://vi.microsoft.com/) et connectez-vous.
1. Accédez à la page de la **bibliothèque** et pointez sur le nom de la vidéo que vous souhaitez réindexer. 
1. Dans le coin inférieur droit, cliquez sur le bouton **Réindexer la vidéo**. 
1. Dans la boîte de dialogue **Réindexer la vidéo**, choisissez la **détection multilingue** dans la zone de liste déroulante **Langue source de la vidéo**.

    * Quand une vidéo est indexée comme multilingue, la page des insights inclut cette option et un type d’insight supplémentaire s’affiche, ce qui permet à l’utilisateur de voir quel segment est transcrit dans quelle « langue parlée ».
    * La traduction dans toutes les langues est entièrement disponible à partir de la transcription multilingue.
    * Tous les autres insights s’affichent dans la langue principale détectée, c’est-à-dire la langue la plus présente dans le support audio.
    * Le sous-titrage sur le lecteur est également disponible en plusieurs langues.

![Utilisation du portail](./media/multi-language-identification-transcription/portal-experience.png)

## <a name="choosing-multilingual-identification-on-indexing-with-api"></a>Choix d’une identification multilingue pour l’indexation à l’aide de l’API

Lors de l’indexation ou de la [réindexation](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-Index-Video?) d’une vidéo à l’aide de l’API, choisissez l’option `multi-language detection` dans le paramètre `sourceLanguage`.

### <a name="model-output"></a>Sortie du modèle

Le modèle récupère toutes les langues détectées dans la vidéo dans une liste

```json
"sourceLanguage": null,
"sourceLanguages": [
    "es-ES",
    "en-US"
],
```

Par ailleurs, chaque instance de la section de transcription inclut la langue dans laquelle elle a été transcrite

```json
{
  "id": 136,
  "text": "I remember well when my youth Minister took me to hear Doctor King I was a teenager.",
  "confidence": 0.9343,
  "speakerId": 1,
  "language": "en-US",
  "instances": [
    {
       "adjustedStart": "0:21:10.42",
       "adjustedEnd": "0:21:17.48",
       "start": "0:21:10.42",
       "end": "0:21:17.48"
    }
  ]
},
```

## <a name="guidelines-and-limitations"></a>Instructions et limitations

* Ensemble de langues prises en charge : anglais, français, allemand, espagnol.
* Prise en charge du contenu multilingue avec jusqu’à trois langues prises en charge.
* Si l’audio contient des langues autres que la liste prise en charge ci-dessus, le résultat est incertain.
* Longueur minimale du segment à détecter pour chaque langue : 15 secondes.
* Le décalage de la détection de langue est de 3 secondes en moyenne.
* Le discours est supposé être continu. Des alternances fréquentes entre les langues peuvent affecter les performances des modèles.
* La façon de parler des orateurs non natifs peut affecter les performances du modèle (par exemple, quand ils utilisent leur langue maternelle et qu’ils passent à une autre langue).
* Le modèle est conçu pour reconnaître des conversations spontanées avec une qualité acoustique correcte (et non des commandes vocales, du chant, etc.).
* La création et la modification de projets ne sont actuellement pas disponibles pour les vidéos multilingues.
* Les modèles linguistiques personnalisés ne sont pas disponibles lors de l’utilisation de la détection multilingue.
* L’ajout de mots clés n’est pas pris en charge.
* Lorsque vous exportez des fichiers de sous-titres, la langue n’est pas indiquée.
* L’API de mise à jour de la transcription ne prend pas en charge les fichiers de plusieurs langues.

## <a name="next-steps"></a>Étapes suivantes

[Présentation de Video Indexer](video-indexer-overview.md)
