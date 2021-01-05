---
title: Unités réservées Multimédia - Azure | Microsoft Docs
description: Les unités réservées Multimédia vous permettent de mettre à l'échelle le traitement multimédia et de déterminer la vitesse de vos tâches de traitement multimédia.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/30/2020
ms.author: inhenkel
ms.openlocfilehash: df691f5b9e74432f683e52fbb896dd4d44efe2aa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91622014"
---
# <a name="media-reserved-units"></a>Unités réservées Multimédia

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Media Services vous permet de mettre à l'échelle le traitement multimédia en gérant les unités réservées Multimédia (MRU). Une MRU fournit la capacité de calcul supplémentaire requise pour l'encodage multimédia. Le nombre de MRU détermine la vitesse à laquelle vos tâches multimédias sont traitées et le nombre de tâches multimédias qui peuvent être traitées simultanément sur un compte. Par exemple, si votre compte comporte cinq MRU et que des tâches doivent être traitées, cinq tâches multimédias peuvent être exécutées simultanément. Toutes les tâches restantes sont mises en file d'attente et peuvent être sélectionnées pour être traitées dans l'ordre lorsque l'exécution d'une tâche se termine. Chaque MRU que vous approvisionnez entraîne une réservation de capacité mais ne vous fournit pas de ressource dédiée. En période de très forte demande, il se peut que vos MRU ne soient pas toutes traitées immédiatement.

Une tâche est une opération individuelle de travail sur une ressource, par l'exemple l'encodage de la diffusion en continu adaptative. Lorsque vous soumettez un travail, Media Services se charge de le répartir en opérations individuelles (c'est-à-dire en tâches) qui sont ensuite associées à des MRU distinctes.

## <a name="choosing-between-different-reserved-unit-types"></a>Choix entre les différents types d’unités réservées

Le tableau suivant vous aide à choisir entre les différentes vitesses d’encodage.  Il indique la durée d'encodage d'une vidéo 1080p de 7 minutes selon la MRU utilisée.

|Type de RU|Scénario|Exemple de résultats pour la vidéo 1080p de 7 min |
|---|---|---|
| **S1**|Encodage à débit binaire unique. <br/>Fichiers avec une résolution SD ou inférieure, insensibles à l’heure, à moindre coût.|Avec l'option « H264 – Vitesse de transmission unique – 16 x 9 SD », l'encodage à débit unique sous la forme d'un fichier MP4 de résolution SD prend environ 7 minutes.|
| **S2**|Encodage à débit binaire unique et à débit binaire multiple.<br/>Utilisation normale de l’encodage SD et HD.|L'encodage avec la présélection « H264 – Vitesse de transmission unique – 720 pixels » prend environ 6 minutes.<br/><br/>L'encodage avec la présélection « H264 – Vitesse de transmission multiple – 720 pixels » prend environ 12 minutes.|
| **S3**|Encodage à débit binaire unique et à débit binaire multiple.<br/>Vidéos avec une résolution HD complète et 4K. Encodage sensible à l’heure, plus rapide.|L’encodage avec la présélection « H264 à débit binaire simple 1080p » prend environ 3 minutes.<br/><br/>L’encodage avec la présélection « H264 à débit binaire multiple 1080p » prend environ 8 minutes.|

> [!NOTE]
> Si vous n'approvisionnez pas de MRU pour votre compte, vos tâches multimédias seront traitées avec les performances d'une MRU S1 et les tâches seront récupérées dans l'ordre. Aucune capacité de traitement n'est réservée, de sorte que le temps d'attente entre l'achèvement d'une tâche et le démarrage de la suivante dépend de la disponibilité des ressources dans le système.

## <a name="considerations"></a>Considérations

* Pour les travaux d'analyse audio et d'analyse vidéo déclenchés par Media Services v3 ou Video Indexer, il est fortement recommandé d'approvisionner le compte avec dix unités S3. Si vous avez besoin de plus de 10 MRU S3, ouvrez un ticket de support à l’aide du [Portail Azure](https://portal.azure.com/).
* Pour les tâches d'encodage qui n'ont pas de MRU, il n'existe aucune limite supérieure à la durée que vos tâches peuvent passer en file d'attente, et au maximum une seule tâche sera exécutée à la fois.

## <a name="billing"></a>Facturation

Vous êtes facturé en fonction du nombre de minutes pendant lesquelles les unités réservées Multimédia sont approvisionnées sur votre compte, qu'il y ait ou non des travaux en cours d'exécution. Pour plus d’informations, consultez la section FAQ de la page [Tarification Media Services](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="next-step"></a>Étape suivante
[Mettre à l'échelle les unités réservées Multimédia avec l'interface CLI](media-reserved-units-cli-how-to.md)
[Analyser des vidéos](analyze-videos-tutorial-with-api.md)

## <a name="see-also"></a>Voir aussi

* [Quotas et limites](limits-quotas-constraints.md)
