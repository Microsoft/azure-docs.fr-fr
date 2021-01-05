---
title: Migrer d'Azure Media Encoder vers Media Encoder Standard | Microsoft Docs
description: Cette rubrique explique comment migrer d'Azure Media Encoder vers le processeur multimédia Media Encoder Standard.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/21/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 32eab0f729280ad25f45853e05ea982b60a6d9f7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89259334"
---
# <a name="migrate-from-azure-media-encoder-to-media-encoder-standard"></a>Migrer d'Azure Media Encoder vers Media Encoder Standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Cet article décrit les étapes de la migration du processeur multimédia AME (Azure Media Encoder) hérité, qui est en phase de mise hors service, vers le processeur multimédia Media Encoder Standard. Pour connaître les dates de mise hors service, consultez la rubrique [Composants hérités](legacy-components.md).

Lors de l’encodage de fichiers avec AME, les clients ont généralement utilisé une chaîne prédéfinie nommée telle que `H264 Adaptive Bitrate MP4 Set 1080p`. Pour migrer, votre code doit être mis à jour afin d'utiliser le processeur multimédia **Media Encoder Standard** plutôt que AME, et une des [présélections système](media-services-mes-presets-overview.md) équivalentes comme `H264 Multiple Bitrate 1080p`. 

## <a name="migrating-to-media-encoder-standard"></a>Migration vers Media Encoder Standard

Voici un exemple de code C# type utilisant le processeur multimédia hérité. 

```csharp
// Declare a new job. 
IJob job = _context.Jobs.Create("AME Job"); 
// Get a media processor reference, and pass to it the name of the  
// processor to use for the specific task. 
IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Encoder"); 

// Create a task with the encoding details, using a string preset. 
// In this case " H264 Adaptive Bitrate MP4 Set 1080p" preset is used. 
ITask task = job.Tasks.AddNew("My encoding task", 
    processor, 
    " H264 Adaptive Bitrate MP4 Set 1080p", 
    TaskOptions.None); 
```

Voici la version mise à jour utilisant Media Encoder Standard.

```csharp
// Declare a new job. 
IJob job = _context.Jobs.Create("Media Encoder Standard Job"); 
// Get a media processor reference, and pass to it the name of the  
// processor to use for the specific task. 
IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard"); 

// Create a task with the encoding details, using a string preset. 
// In this case " H264 Multiple Bitrate 1080p" preset is used. 
ITask task = job.Tasks.AddNew("My encoding task", 
    processor, 
    "H264 Multiple Bitrate 1080p", 
    TaskOptions.None); 
```

### <a name="advanced-scenarios"></a>Scénarios avancés 

Si vous avez créé votre propre encodage prédéfini pour AME à l’aide de son schéma, il existe un [schéma équivalent pour Media Encoder Standard](media-services-mes-schema.md). Si vous avez des questions sur le mappage des anciens paramètres avec le nouvel encodeur, contactez-nous via mailto:amshelp@microsoft.com  
## <a name="known-differences"></a>Différences connues 

Plus robuste et plus fiable que l'encodeur AME hérité, Media Encoder Standard offre également de meilleures performances et génère une sortie de meilleure qualité. Informations supplémentaires : 

* Media Encoder Standard génère des fichiers de sortie basés sur une convention d'affectation de noms différente de celle de AME.
* Media Encoder Standard génère des artefacts tels que des fichiers contenant les [métadonnées des fichiers d'entrée](media-services-input-metadata-schema.md) et les [métadonnées des fichiers de sortie](media-services-output-metadata-schema.md).

## <a name="next-steps"></a>Étapes suivantes

* [Composants hérités](legacy-components.md)
* [Page des tarifs](https://azure.microsoft.com/pricing/details/media-services/#encoding)
