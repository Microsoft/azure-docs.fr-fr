---
title: Codes d’erreur d’encodage Azure Media Services | Microsoft Docs
description: Cette rubrique répertorie les codes d’erreur susceptibles d’être renvoyés à la suite d’une erreur d’exécution d’un travail d’encodage.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: ce4e939f-5aee-41f9-859d-e4429815e9f2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 6e56dbe1d1236a567ed6f59acfcca325a6c9ee7e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89269027"
---
# <a name="encoding-error-codes"></a>Codes d’erreur d’encodage

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Le tableau suivant répertorie les codes d’erreur susceptibles d’être renvoyés à la suite d’une erreur d’exécution d’une tâche de codage.  Pour obtenir les détails de l’erreur dans votre code .NET, utilisez la classe [ErrorDetails](/previous-versions/azure/jj126075(v=azure.100)) . Pour obtenir les détails de l’erreur dans votre code REST, utilisez l’API REST [ErrorDetail](/rest/api/media/operations/errordetail) .

| ErrorDetail.Code | Causes possibles de l’erreur |
| --- | --- |
| Unknown |Erreur inconnue lors de l’exécution de la tâche. |
| ErrorDownloadingInputAssetMalformedContent |Catégorie d’erreurs se produisant lors du téléchargement d’éléments multimédias d’entrée : noms de fichier incorrects, fichiers de longueur nulle, formats incorrects, etc. |
| ErrorDownloadingInputAssetServiceFailure |Catégorie d’erreurs regroupant les problèmes côté service, par exemple les erreurs réseau ou de stockage lors du téléchargement. |
| ErrorParsingConfiguration |Catégorie d’erreurs se produisant quand la tâche \<see cref="MediaTask.PrivateData"/> (configuration) n’est pas valide. Par exemple, la configuration n’est pas une valeur système prédéfinie valide ou contient du code XML non valide. |
| ErrorExecutingTaskMalformedContent |Catégorie d’erreurs se produisant pendant l’exécution de la tâche. Des problèmes dans les fichiers multimédias d’entrée provoquent une défaillance. |
| ErrorExecutingTaskUnsupportedFormat |Catégorie d’erreurs se produisant quand le processeur multimédia ne peut pas traiter les fichiers fournis. Le format de média n’est pas pris en charge ou ne correspond pas à la configuration. Cela peut arriver si vous tentez de générer une sortie audio uniquement à partir d’un élément multimédia qui contient seulement des données vidéo. |
| ErrorProcessingTask |Catégorie d’erreurs qui ne sont pas liées au contenu que le processeur multimédia rencontre pendant le traitement de la tâche. |
| ErrorUploadingOutputAsset |Catégorie d’erreurs se produisant pendant le chargement de l’élément multimédia de sortie. |
| ErrorCancelingTask |Catégorie d’erreurs se produisant à la suite d’échecs d’annulation de la tâche. |
| TransientError |Catégorie d’erreurs pour couvrir les problèmes temporaires (par ex. problèmes réseau temporaires avec Azure Storage) |

Pour obtenir de l’aide de la part de l’équipe **Media Services** , ouvrez un [ticket de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-articles"></a>Articles connexes
* [Exécution de tâches d’encodage avancées via la personnalisation des présélections Media Encoder Standard](media-services-custom-mes-presets-with-dotnet.md)
* [Quotas et limitations](media-services-quotas-and-limitations.md)

<!--Reference links in article-->
[1]: https://azure.microsoft.com/pricing/details/media-services/
