---
title: Composants hérités Azure Media Services | Microsoft Docs
description: Cette rubrique traite des composants hérités Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/21/2020
ms.author: inhenkel
ms.openlocfilehash: d85cbeb99264b5b730fe585fd39f658e6448467f
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515761"
---
# <a name="azure-media-services-legacy-components"></a>Composants hérités Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Au fil du temps, nous améliorons les composants de Media Service et nous mettons hors service les composants hérités. Cet article vous aide à migrer votre application d’un composant hérité vers un composant actuel.
 
## <a name="retirement-plans-of-legacy-components-and-migration-guidance"></a>Plans de mise hors service de composants hérités et conseils de migration

Les processeurs multimédias *Windows Azure Media Encoder* (WAME) et *Azure Media Encoder* (AME) sont déconseillés.

* [Migrer de Windows Azure Media Encoder vers Media Encoder Standard](migrate-windows-azure-media-encoder.md)
* [Migrer d'Azure Media Encoder vers Media Encoder Standard](migrate-azure-media-encoder.md)

Les processeurs multimédias Media Analytics suivants sont déconseillés ou seront bientôt obsolètes :

  
 
| **Nom du processeur multimédia** | **Date de mise hors service** | **Remarques supplémentaires** |
| --- | --- | ---|
| Azure Media Indexer 2 | 1er janvier 2020 | Ce processeur multimédia sera remplacé par le [mode de base Media Services V3 AudioAnalyzerPreset](../latest/analyzing-video-audio-files-concept.md). Pour plus d’informations, consultez [Effectuer une migration depuis Azure Media Indexer 2 vers Azure Media Services Video Indexer](migrate-indexer-v1-v2.md). |
| Azure Media Indexer | 1er mars 2023 | Ce processeur multimédia sera remplacé par le [mode de base Media Services V3 AudioAnalyzerPreset](../latest/analyzing-video-audio-files-concept.md). Pour plus d’informations, consultez [Effectuer une migration depuis Azure Media Indexer 2 vers Azure Media Services Video Indexer](migrate-indexer-v1-v2.md). |
| Détection de mouvement | 1er juin 2020|Aucun plan de remplacement pour l’instant. |
| Création de résumés de vidéos |1er juin 2020|Aucun plan de remplacement pour l’instant.|
| Reconnaissance optique de caractères | 1er juin 2020 |Ce processeur multimédia a été remplacé par [Azure Media Services Video Indexer](../video-indexer/index.yml). Envisagez également d’utiliser l'[API Azure Media Services V3](../latest/analyzing-video-audio-files-concept.md). <br/>Voir [Comparer les préréglages d’Azure Media Services v3 et de Video Indexer](../video-indexer/compare-video-indexer-with-media-services-presets.md). |
| Face Detector | 1er juin 2020 | Ce processeur multimédia a été remplacé par [Azure Media Services Video Indexer](../video-indexer/index.yml). Envisagez également d’utiliser l'[API Azure Media Services V3](../latest/analyzing-video-audio-files-concept.md). <br/>Voir [Comparer les préréglages d’Azure Media Services v3 et de Video Indexer](../video-indexer/compare-video-indexer-with-media-services-presets.md). |
| Content Moderator | 1er juin 2020 |Ce processeur multimédia a été remplacé par [Azure Media Services Video Indexer](../video-indexer/index.yml). Envisagez également d’utiliser l'[API Azure Media Services V3](../latest/analyzing-video-audio-files-concept.md). <br/>Voir [Comparer les préréglages d’Azure Media Services v3 et de Video Indexer](../video-indexer/compare-video-indexer-with-media-services-presets.md). |

## <a name="next-steps"></a>Étapes suivantes

[Conseils de migration pour le passage de Media Services v2 à Media Services v3](../latest/migrate-from-v2-to-v3.md)
