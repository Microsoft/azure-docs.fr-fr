---
title: Comment générer des miniatures à l’aide d’Azure Media Services Encoder Standard avec .NET
description: Cette rubrique vous explique comment utiliser .NET pour coder un élément multimédia tout en générant des miniatures à l’aide de Media Encoder Standard.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 12/01/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 887ac4197321b4015eff7cc839abd8f57ae67760
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512016"
---
# <a name="how-to-generate-thumbnails-using-encoder-standard-with-net"></a>Comment générer des miniatures à l’aide de Media Encoder Standard avec .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Vous pouvez utiliser Media Encoder Standard pour générer une ou plusieurs miniatures à partir de votre vidéo d’entrée au format de fichier d’image [JPEG](https://en.wikipedia.org/wiki/JPEG) ou [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics).

## <a name="recommended-reading-and-practice"></a>Lecture et pratiques recommandées

Nous vous recommandons de vous familiariser avec les transformations personnalisées en lisant [Comment encoder avec une transformation -.NET personnalisée](customize-encoder-presets-how-to.md).

## <a name="transform-code-example"></a>Exemple de code de transformation

L’exemple de code ci-dessous crée uniquement une miniature.  Vous devez définir les paramètres suivants :

- **début** : position dans la vidéo d’entrée à partir de laquelle commencer la génération des miniatures. La valeur peut être au format ISO 8601 (par exemple, PT05S pour commencer à 5 secondes) ou un nombre de cadres (par exemple, 10 pour commencer au dixième cadre) ou une valeur relative à la durée du flux (par exemple, 10 % pour commencer à 10 % de la durée du flux). Prend également en charge une macro {Best}, qui indique à l’encodeur de sélectionner la meilleure miniature parmi les premières secondes de la vidéo pour ne produire qu’une seule miniature, quels que soient les autres paramètres pour Étape et Plage. La valeur par défaut est macro {Best}.
- **étape** : intervalles selon lesquelles les miniatures sont générées. La valeur peut être au format ISO 8601 (par exemple, PT05S pour commencer toutes les 5 secondes) ou un nombre de cadres (par exemple, 30 pour une image tous les 30 cadres) ou une valeur relative à la durée du flux (par exemple, 10 % pour une image à chaque 10 % de la durée du flux). La valeur de l’étape affecte la première miniature générée, qui peut ne pas être exactement celle qui est spécifiée lors de l’heure de début de la présélection de la transformation. Cela est dû à l’encodeur, qui tente de sélectionner la meilleure miniature entre l’heure de début et la position de l’étape à partir de l’heure de début comme première sortie. Étant donné que la valeur par défaut est 10 %, si le flux a une durée longue, la première miniature générée peut être éloignée de celle spécifiée au moment de l’heure de début. Essayez de sélectionner une valeur raisonnable pour l’étape si la première miniature est censée être proche de l’heure de début ou définissez la valeur de la plage sur 1 si une seule miniature est nécessaire au moment de l’heure de début.
- **plage** : position relative à l’heure de début de la présélection de transformation dans la vidéo d’entrée à laquelle arrêter la génération de miniatures. La valeur peut être au format ISO 8601 (par exemple, PT5M30S pour s’arrêter à 5 minutes et 30 secondes à partir de l’heure de début) ou un nombre de cadres (par exemple, 300 pour s’arrêter au niveau de la 300e trame à partir du cadre à l’heure de début. Si cette valeur est 1, cela signifie qu’il ne doit produire qu’une seule miniature à l’heure de début, ou une valeur relative à la durée du flux (par exemple, 50 % pour s’arrêter à la moitié de la durée du flux à partir de l’heure de début). La valeur par défaut est 100 %, ce qui signifie qu’elle s’arrête à la fin du flux.
- **couches** : collection de couches d’images de sortie à produire par l’encodeur.

```csharp

private static Transform EnsureTransformExists(IAzureMediaServicesClient client, string resourceGroupName, string accountName, string transformName)
{
    // Does a Transform already exist with the desired name? Assume that an existing Transform with the desired name
    // also uses the same recipe or Preset for processing content.
    Transform transform = client.Transforms.Get(resourceGroupName, accountName, transformName);

    if (transform == null)
    {
        // Create a new Transform Outputs array - this defines the set of outputs for the Transform
        TransformOutput[] outputs = new TransformOutput[]
        {
            // Create a new TransformOutput with a custom Standard Encoder Preset
            // This demonstrates how to create custom codec and layer output settings

          new TransformOutput(
                new StandardEncoderPreset(
                    codecs: new Codec[]
                    {
                        // Generate a set of PNG thumbnails
                        new PngImage(
                            start: "25%",
                            step: "25%",
                            range: "80%",
                            layers: new PngLayer[]{
                                new PngLayer(
                                    width: "50%",
                                    height: "50%"
                                )
                            }
                        )
                    },
                    // Specify the format for the output files for the thumbnails
                    formats: new Format[]
                    {
                        new PngFormat(
                            filenamePattern:"Thumbnail-{Basename}-{Index}{Extension}"
                        )
                    }
                ),
                onError: OnErrorType.StopProcessingJob,
                relativePriority: Priority.Normal
            )
        };

        string description = "A transform that includes thumbnails.";
        // Create the custom Transform with the outputs defined above
        transform = client.Transforms.CreateOrUpdate(resourceGroupName, accountName, transformName, outputs, description);
    }

    return transform;
}
```

## <a name="next-steps"></a>Étapes suivantes
[Générer des miniatures avec REST](media-services-generate-thumbnails-rest.md)
