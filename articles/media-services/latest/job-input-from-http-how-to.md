---
title: Créer une entrée de travail Azure Media Services à partir d’une URL HTTPS | Microsoft Docs
description: Cette rubrique explique comment créer une entrée de travail Azure Media Services à partir d’une URL HTTPS.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: e440e4393065d2bc1cad1a96b4f8c975624faa11
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89295510"
---
# <a name="create-a-job-input-from-an-https-url"></a>Créer une entrée de travail à partir d’une URL HTTPS

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Dans Media Services v3, lorsque vous soumettez des travaux pour traiter vos vidéos, vous devez indiquer à Media Services où trouver la vidéo d’entrée. Une des options consiste à spécifier une URL HTTPS comme entrée de travail (comme illustré dans cet exemple). Notez qu’AMS v3 ne prend pas en charge l’encodage de transfert mémorisé en bloc sur les URL HTTPS. Pour obtenir un exemple complet, consultez cet [exemple GitHub](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs).

> [!TIP]
> Avant de commencer le développement, consultez [Développement avec les API Media Services v3](media-services-apis-overview.md) (informations sur l’accès aux API, les conventions d’affectation de noms, etc.)

## <a name="net-sample"></a>Exemple de code .NET

Le code suivant montre comment créer un travail avec une entrée d’URL HTTP(s).

[!code-csharp[Main](../../../media-services-v3-dotnet-quickstarts/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs#SubmitJob)]

## <a name="job-error-codes"></a>Codes d’erreur des tâches

Consultez [Codes d’erreur](/rest/api/media/jobs/get#joberrorcode).

## <a name="next-steps"></a>Étapes suivantes

[Créer une entrée de travail à partir d’un fichier local](job-input-from-local-file-how-to.md).
