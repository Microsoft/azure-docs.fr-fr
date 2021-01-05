---
title: Présélection pour l’encodage sensible au contenu – Azure Media Services
description: Cet article décrit l’encodage sensible au contenu dans Microsoft Azure Media Services v3.
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
ms.custom: devx-track-csharp
ms.openlocfilehash: 5d08e09905841f6068f2bac45680a8e5a011c158
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89297363"
---
# <a name="use-the-content-aware-encoding-preset-to-find-the-optimal-bitrate-value-for-a-given-resolution"></a>Utiliser la présélection de l’encodage sensible au contenu pour rechercher la valeur optimale de vitesse de transmission pour une résolution donnée

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Afin de préparer le contenu pour une diffusion en [streaming à débit adaptatif](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming), la vidéo doit être encodée à plusieurs débits (du plus élevé au plus faible). Cela garantit une dégradation appropriée de la qualité, la vitesse de transmission et la résolution de la vidéo sont réduites en parallèle. Un tel codage à vitesses de transmission multiples utilise ce qu’on appelle une échelle d’encodage, c’est-à-dire un tableau de résolutions et de vitesses de transmission. Consultez les [présélections d’encodage intégrées](/rest/api/media/transforms/createorupdate#encodernamedpreset) de Media Services.

Vous devez être conscient du contenu que vous traitez et personnaliser ou ajuster l’échelle d’encodage à la complexité de la vidéo. À chaque résolution, il existe un débit au-delà duquel toute augmentation de la qualité n’est pas perceptive : l’encodeur fonctionne à la valeur de débit optimale. Le niveau suivant d’optimisation consiste à sélectionner les résolutions en fonction du contenu, par exemple, une vidéo de présentation PowerPoint ne bénéficie d’aucune amélioration en passant sous 720p. Pour aller plus loin, l’encodeur peut être chargé afin d’optimiser les paramètres pour chaque scène de la vidéo. 

La présélection [Adaptive Streaming](autogen-bitrate-ladder.md) (diffusion en continu adaptative) de Microsoft résout le problème de variabilité de la qualité et de la résolution des vidéos sources. Nos clients bénéficient d’une grande variété de contenus, certains en 1080p, d’autres en 720p, et quelques-uns en SD voire à des résolutions inférieures. En outre, tout le contenu source n’affiche pas les mezzanines haute qualité proposées par les studios de cinéma ou de télévision. La présélection Adaptive Streaming résout ces problèmes en s'assurant que l'échelle de débit ne dépasse jamais la résolution ou le débit moyen de la mezzanine en entrée. Toutefois, cette présélection n’examine pas les propriétés sources autres que la résolution et la vitesse de transmission.

## <a name="the-content-aware-encoding"></a>Encodage sensible au contenu 

La présélection d’encodage sensible au contenu étend le mécanisme de diffusion en continu à vitesse de transmission adaptative, en incorporant une logique personnalisée permettant à l’encodeur de rechercher la valeur optimale de vitesse de transmission pour une résolution donnée, mais sans nécessiter une analyse informatique complète. Cette présélection produit un ensemble de fichiers MP4 alignés sur le groupe d’images. Étant donné un contenu d’entrée, le service effectue une analyse initiale légère du contenu d’entrée et utilise les résultats pour déterminer le nombre optimal de couches, le débit approprié et les paramètres de résolution pour la livraison par diffusion en continu adaptative. Cette présélection est particulièrement efficace pour les vidéos de complexité faible et moyenne, où les fichiers de sortie sont à des vitesses de transmission inférieures à la présélection Diffusion en continu adaptative, mais à une qualité qui offre toujours une bonne expérience aux viewers. La sortie contiendra des fichiers MP4 avec vidéo et audio entrelacées.

Les exemples de graphiques suivants illustrent la comparaison à l’aide de mesures de qualité comme [PSNR](https://en.wikipedia.org/wiki/Peak_signal-to-noise_ratio) et [VMAF](https://en.wikipedia.org/wiki/Video_Multimethod_Assessment_Fusion). La source avait été créée en concaténant de petits clips très complexes extraits de films et de séries TV, destinés à soumettre l’encodeur à un test de contrainte. Par définition, cette présélection produit des résultats qui varient selon le contenu : cela signifie également que pour certains contenus, la réduction du débit ou l’amélioration de la qualité reste minime.

![Courbe de distorsion du débit (RD) avec PSNR](media/content-aware-encoding/msrv1.png)

**Figure 1 : Courbe de distorsion du débit (RD) avec la métrique PSNR pour une source très complexe**

![Courbe de distorsion du débit (RD) avec VMAF](media/content-aware-encoding/msrv2.png)

**Figure 2 : Courbe de distorsion du débit (RD) avec la métrique VMAF pour une source très complexe**

Voici les résultats pour une autre catégorie de contenu source, où l’encodeur a pu déterminer que l’entrée était de mauvaise qualité (de nombreux artefacts de compression en raison d’une faible vitesse de transmission). Notez qu’avec la présélection sensible au contenu, l’encodeur a décidé de ne produire qu’une seule couche de sortie, à une vitesse de transmission suffisamment faible pour que la plupart des clients puissent lire le flux sans saccade.

![Courbe de distorsion du débit (RD) avec PSNR](media/content-aware-encoding/msrv3.png)

**Figure 3 : Courbe de distorsion du débit (RD) avec PSNR pour une entrée de faible qualité (à 1080p)**

![Courbe de distorsion du débit (RD) avec VMAF](media/content-aware-encoding/msrv4.png)

**Figure 4 : Courbe de distorsion du débit (RD) avec VMAF pour une entrée de faible qualité (à 1080p)**

## <a name="how-to-use-the-content-aware-encoding-preset"></a>Comment utiliser la présélection d’encodage sensible au contenu 

Vous pouvez créer des transformations qui utilisent cette présélection comme suit. 

Consultez la section [Étapes suivantes](#next-steps) pour accéder à des didacticiels qui utilisent la transformation des sorties. La ressource de sortie peut être fournie à partir de points de terminaison de streaming Media Services dans des protocoles tels que MPEG-DASH et HLS (comme indiqué dans les didacticiels).

> [!NOTE]
> Veillez à utiliser la présélection **ContentAwareEncoding**, et non ContentAwareEncodingExperimental.

```csharp
TransformOutput[] output = new TransformOutput[]
{
   new TransformOutput
   {
      // The preset for the Transform is set to one of Media Services built-in sample presets.
      // You can customize the encoding settings by changing this to use "StandardEncoderPreset" class.
      Preset = new BuiltInStandardEncoderPreset()
      {
         // This sample uses the new preset for content-aware encoding
         PresetName = EncoderNamedPreset.ContentAwareEncoding
      }
   }
};
```

> [!NOTE]
> Les travaux d’encodage à l’aide de la présélection `ContentAwareEncoding` sont facturés en fonction des minutes de sortie. 
  
## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Télécharger, encoder et diffuser des vidéos avec Media Services v3](stream-files-tutorial-with-api.md)
* [Tutoriel : Encoder un fichier distant basé sur une URL et diffuser la vidéo en continu - REST](stream-files-tutorial-with-rest.md)
* [Tutoriel : Encoder un fichier distant basé sur une URL et diffuser la vidéo en continu – CLI](stream-files-cli-quickstart.md)
* [Tutoriel : Encoder un fichier distant basé sur une URL et diffuser la vidéo en continu – .NET](stream-files-dotnet-quickstart.md)
* [Tutoriel : Encoder un fichier distant basé sur une URL et diffuser la vidéo en continu – Node.js](stream-files-nodejs-quickstart.md)
