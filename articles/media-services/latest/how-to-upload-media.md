---
title: Charger des éléments multimédia
titleSuffix: Azure Media Services
description: Découvrez comment charger des médias pour le streaming ou l’encodage.
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
ms.openlocfilehash: 3040369e655ab91f56f271313dc4d2613f02be06
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92015844"
---
# <a name="upload-media-for-streaming-or-encoding"></a>Charger des médias pour le streaming ou l’encodage

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Dans Media Services, vous chargez vos fichiers numériques (médias) dans le conteneur d’objets blob associé à une ressource. L’entité [Asset](/rest/api/media/operations/asset) peut contenir des fichiers vidéo et audio, des images, des collections de miniatures, des pistes textuelles et des légendes (et les métadonnées concernant ces fichiers). Une fois les fichiers chargés dans le conteneur de la ressource, votre contenu est stocké en toute sécurité dans le cloud et peut faire l'objet d'un traitement et d'une diffusion en continu.

Avant de commencer, vous devez cependant recueillir ou réfléchir à quelques valeurs.

1. Le chemin local du fichier à charger
1. L’ID de la ressource (conteneur)
1. Le nom que vous souhaitez attribuer au fichier chargé, sans oublier l’extension
1. Le nom du compte de stockage que vous utilisez
1. La clé d'accès de votre compte de stockage

## <a name="portal"></a>[Portail](#tab/portal/)

[!INCLUDE [Upload files with the portal](./includes/task-upload-file-to-asset-portal.md)]

## <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/cli/)

[!INCLUDE [Upload files with the portal](./includes/task-upload-file-to-asset-cli.md)]

## <a name="rest"></a>[REST](#tab/rest/)

Une fois que vous avez [créé une ressource à l’aide de Postman ou d’une autre méthode REST et obtenu l’URL SAS de la ressource](how-to-create-asset.md?tabs=rest), utilisez les API Stockage Azure ou des kits SDK (par exemple, l’[API REST Stockage](../../storage/common/storage-rest-api-auth.md) ou le [SDK .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md).

---
<!-- add these to the tabs when available -->
Pour les autres méthodes, consultez la [documentation Stockage Azure](../../storage/blobs/index.yml) en vue d’utiliser des objets blob avec [.NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md), [Java](../../storage/blobs/storage-quickstart-blobs-java.md), [Python](../../storage/blobs/storage-quickstart-blobs-python.md) et [JavaScript (Node.js)](../../storage/blobs/storage-quickstart-blobs-nodejs.md).

## <a name="next-steps"></a>Étapes suivantes

> [Présentation de Media Services](media-services-overview.md)