---
title: Éléments multimédias
titleSuffix: Azure Media Services
description: Découvrez-en plus sur les éléments multimédias et leur utilisation par Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 99101f31ec955c822c784180dd8b04bfb8e5f73c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89291600"
---
# <a name="assets-in-azure-media-services-v3"></a>Actifs multimédias dans Azure Media Service v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Dans Azure Media Services, l’[actif multimédia](/rest/api/media/assets) est un concept clé. C’est là que vous entrez du contenu multimédia (par exemple par le biais d’un chargement ou d’une ingestion en direct), où vous générez du contenu multimédia (à partir d’une sortie de travail) et à partir duquel vous publiez du contenu multimédia (pour le streaming). 

Un actif multimédia est mappé à un conteneur d’objets blob dans le [compte Stockage Azure](storage-account-concept.md) et les fichiers contenus dans l’actif multimédia sont stockés sous forme d’objets blob de blocs dans ce conteneur. Les actifs multimédias contiennent des informations sur les fichiers numériques stockés dans Stockage Azure (notamment des données vidéo, des données audio, des images, des collections de miniatures, des pistes de texte et des fichiers de sous-titres).

Media Services prend en charge les niveaux d’objets blob quand le compte utilise le stockage v2 universel (GPv2). Avec GPv2, vous pouvez déplacer les fichiers vers un [stockage Froid ou Archive](../../storage/blobs/storage-blob-storage-tiers.md). Le stockage **Archive** est approprié pour archiver les fichiers sources quand ils ne sont plus nécessaires (par exemple une fois qu’ils ont été encodés).

Le niveau de stockage **Archive** est recommandé uniquement pour les fichiers sources très volumineux qui ont déjà été encodés et dont la sortie de travail d’encodage a été placée dans un conteneur d’objets blob de sortie. Les objets blob présents dans le conteneur de sortie que vous voulez associer à un élément multimédia et utiliser pour diffuser en continu ou analyser votre contenu doivent exister dans un niveau de stockage **Chaud** ou **Froid**.

## <a name="naming"></a>Dénomination 

### <a name="assets"></a>Éléments multimédias

Les noms des éléments multimédias doivent être uniques. Les noms de ressources Media Services v3 (par exemple Assets, Jobs, Transforms) sont sujets à des restrictions d'affectation de noms d'Azure Resource Manager. Pour plus d'informations, consultez [Conventions d'affectation de noms](media-services-apis-overview.md#naming-conventions).

### <a name="blobs"></a>Objets blob

Les noms des fichiers/objets blob au sein d’une ressource doivent respecter les [exigences en matière de nom d’objet blob](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) et de [nom NTFS](/windows/win32/fileio/naming-a-file). Ces exigences se justifient par le fait que les fichiers peuvent être copiés du stockage d’objets blob vers un disque NTFS local à des fins de traitement.

## <a name="next-steps"></a>Étapes suivantes

[Présentation de Media Services](media-services-overview.md)

## <a name="see-also"></a>Voir aussi

[Différences entre Media Services v2 et v3](migrate-from-v2-to-v3.md)
