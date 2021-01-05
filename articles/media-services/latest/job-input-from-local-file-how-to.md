---
title: Créer une entrée de travail Azure Media Services à partir d’un fichier local | Microsoft Docs
description: Cet article explique comment créer une entrée de travail Azure Media Services à partir d’un fichier local.
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
ms.openlocfilehash: 02aaae89f7544392765b46361b1617e0fd7dae8c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89296804"
---
# <a name="create-a-job-input-from-a-local-file"></a>Créer une entrée de travail à partir d’un fichier local

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Dans Media Services v3, lorsque vous soumettez des travaux pour traiter vos vidéos, vous devez indiquer à Media Services où trouver la vidéo d’entrée. La vidéo d’entrée peut être stockée sous la forme d’une ressource Media Service, dans laquelle vous créez une ressource d’entrée basée sur un fichier (stocké en local ou dans le stockage Blob Azure). Cette rubrique montre comment créer une entrée de travail à partir d’un fichier local. Pour obtenir un exemple complet, consultez cet [exemple GitHub](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs).

## <a name="prerequisites"></a>Prérequis 

* [Créer un compte Media Services](./create-account-howto.md).

## <a name="net-sample"></a>Exemple de code .NET

Le code suivant montre comment créer une ressource d’entrée et l’utiliser comme entrée pour le travail. La fonction CreateInputAsset effectue les actions suivantes :

* Elle crée la ressource
* Elle récupère une [URL SAP](../../storage/common/storage-sas-overview.md) accessible en écriture vers le [conteneur de stockage](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container) de la ressource
* Elle charge le fichier dans le conteneur de stockage à l’aide de l’URL SAP

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateInputAsset)]

L’extrait de code suivant crée un élément multimédia de sortie s’il n’existe pas déjà :

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateOutputAsset)]

L'extrait de code suivant envoie un travail d'encodage :

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#SubmitJob)]

## <a name="job-error-codes"></a>Codes d’erreur des tâches

Consultez [Codes d’erreur](/rest/api/media/jobs/get#joberrorcode).

## <a name="next-steps"></a>Étapes suivantes

[Créer une entrée de travail à partir d’une URL HTTPS](job-input-from-http-how-to.md).
