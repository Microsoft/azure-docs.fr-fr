---
title: Création d’un processeur multimédia à l’aide du Kit de développement logiciel Azure Media Services pour .NET | Microsoft Docs
description: Apprenez à créer un composant processeur multimédia pour encoder, chiffrer ou déchiffrer un contenu multimédia, ou convertir son format pour Azure Media Services. Les exemples de code sont écrits en C# et utilisent le Kit de développement logiciel (SDK) Media Services pour .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: dbf9496f-c6f0-42a7-aa36-70f89dcb8ea2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/03/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 3215eef337ca1c9a84f2f4d917dbea8ea2e090f0
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93346687"
---
# <a name="how-to-get-a-media-processor-instance"></a>Procédure : Obtention d'une instance de processeur multimédia

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-get-media-processor.md)
> * [REST](media-services-rest-get-media-processor.md)

## <a name="overview"></a>Vue d’ensemble

Dans Media Services, un processeur multimédia est un composant qui gère une tâche de traitement spécifique, telle que l’encodage, la conversion de format, le chiffrement ou le déchiffrement de contenu multimédia. Le plus souvent, vous devez créer un processeur multimédia lorsque vous créez une tâche visant à encoder, à chiffrer ou à convertir le format du contenu multimédia.

## <a name="azure-media-processors"></a>Processeurs multimédias Azure

La rubrique suivante fournit une liste de processeurs multimédias :

* [Processeurs multimédias d’encodage](scenarios-and-availability.md)
* [Processeurs multimédias Analytics](scenarios-and-availability.md)

## <a name="get-media-processor"></a>Obtention d'un processeur multimédia

La méthode suivante montre comment obtenir une instance de processeur multimédia. L'exemple de code implique l'utilisation au niveau du module d'une variable nommée **_context** pour référencer le contexte du serveur, comme décrit à la section [Procédure : Se connecter à Media Services par programmation](media-services-use-aad-auth-to-access-ams-api.md).

```csharp
private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
{
    var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
    ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

    if (processor == null)
    throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

    return processor;
}
```

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment obtenir une instance de processeur multimédia, consultez la rubrique [Encodage d’un élément multimédia](media-services-dotnet-encode-with-media-encoder-standard.md) pour savoir comment utiliser Media Encoder Standard afin d’encoder un élément multimédia.