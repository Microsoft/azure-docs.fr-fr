---
title: Encoder une transformation personnalisée avec Media Services v3 .NET - Azure | Microsoft Docs
description: Cette rubrique explique comment utiliser Azure Media Services v3 pour encoder une transformation personnalisée à l’aide de .NET.
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
ms.custom: seodec18
ms.openlocfilehash: b5bb505df3cad7856e0b08f04949c2e56ccec1ca
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89295612"
---
# <a name="how-to-encode-with-a-custom-transform---net"></a>Comment encoder avec une transformation personnalisée - .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Lors de l’encodage avec Azure Media Services, vous pouvez commencer rapidement avec l’un des préréglages intégrés recommandés et basés sur les bonnes pratiques, comme illustré dans le didacticiel [Streaming de fichiers](stream-files-tutorial-with-api.md). Vous pouvez également créer un préréglage personnalisé pour les besoins de votre scénario ou votre appareil.

## <a name="considerations"></a>Considérations

Lorsque vous créez des préréglages personnalisés, les considérations suivantes s’appliquent :

* Toutes les valeurs de hauteur et de largeur de contenu AVC doivent être un multiple de 4.
* Dans Azure Media Services v3, toutes les vitesses d’encodage sont données en bits par seconde. Cela diffère des préréglages avec nos API v2, qui utilisaient des kilobits par seconde comme unité. Par exemple, si la vitesse de transmission dans v2 était de 128 (kilobits/seconde), elle sera définie sur 12 8000 (bits/seconde) dans v3.

## <a name="prerequisites"></a>Conditions préalables requises 

[Créer un compte Media Services](./create-account-howto.md)

## <a name="download-the-sample"></a>Télécharger l’exemple

Clonez un dépôt GitHub qui contient l’exemple .NET Core complet sur votre machine avec la commande suivante :  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials.git
 ```
 
L’exemple de préréglage personnalisé se trouve dans le dossier [EncodeCustomTransform](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/EncodeCustomTransform/).

## <a name="create-a-transform-with-a-custom-preset"></a>Créer une transformation avec un préréglage personnalisé 

Quand vous créez une [transformation](/rest/api/media/transforms), vous devez spécifier ce qu’elle doit produire comme sortie. Le paramètre requis est un objet [TransformOutput](/rest/api/media/transforms/createorupdate#transformoutput), comme indiqué dans le code ci-dessous. Chaque objet **TransformOutput** contient un **préréglage**. Le **préréglage** décrit les instructions détaillées concernant les opérations de traitement vidéo et/ou audio qui doivent être utilisées pour générer l’objet **TransformOutput** souhaité. Le **TransformOutput** suivant crée des valeurs de sortie de codecs et de couche personnalisés.

Lorsque vous créez une [transformation](/rest/api/media/transforms), vous devez tout d’abord vérifier s’il en existe déjà une à l’aide de la méthode **Get**, comme indiqué dans le code qui suit. Dans Media Services v3, les méthodes **Get** appliquées sur des entités retournent **null** si l’entité n’existe pas (une vérification du nom ne respectant pas la casse).

### <a name="example"></a>Exemple

L’exemple suivant définit un ensemble de sorties à générer lorsque cette transformation est utilisée. Nous ajoutons tout d’abord une couche AacAudio pour l’encodage audio et deux couches H264Video pour l’encodage vidéo. Dans les couches vidéo, nous attribuons des étiquettes pour pouvoir les utiliser dans les noms de fichiers de sortie. Nous souhaitons ensuite que la sortie inclue également des miniatures. Dans l’exemple ci-dessous, nous spécifions des images au format PNG, générées à 50 % de la résolution de la vidéo d’entrée et à trois horodatages (25 %, 50 %, 75 %) de la longueur de la vidéo d’entrée. Pour fini, nous spécifions le format des fichiers de sortie : un pour la vidéo + audio et un autre pour les miniatures. Étant donné que nous avons plusieurs couches H264, nous devons utiliser des macros qui produisent des noms uniques par couche. Nous pouvons utiliser une macro `{Label}` ou `{Bitrate}` ; l’exemple montre la première.

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/EncodeCustomTransform/MediaV3ConsoleApp/Program.cs#EnsureTransformExists)]

## <a name="next-steps"></a>Étapes suivantes

[Streaming de fichiers](stream-files-tutorial-with-api.md) 
