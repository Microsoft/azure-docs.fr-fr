---
title: Quotas et limites dans Azure Media Services
description: Cette rubrique décrit les quotas et les limites de Microsoft Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 10/23/2020
ms.author: inhenkel
ms.openlocfilehash: d6ca7a444f2a3d4babe220548edb10bd37784be7
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92678114"
---
<!-- If you update limits in this topic, make sure to also update https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#media-services-limits -->
# <a name="azure-media-services-quotas-and-limits"></a>Quotas et limites d'Azure Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cet article répertorie quelques-unes des limites les plus courantes de Microsoft Azure Media Services, parfois appelées des quotas.

> [!NOTE]
> Pour les ressources qui ne sont pas corrigées, ouvrez un ticket de support pour demander une augmentation des quotas. Ne créez pas d’autres comptes Azure Media Services pour obtenir des limites supérieures.

## <a name="account-limits"></a>Limites relatives aux comptes

| Ressource | Limite par défaut |
| --- | --- |
| [Comptes Media Services](media-services-account-concept.md) dans le cadre d'un même abonnement | 100 (fixe) |

## <a name="asset-limits"></a>Limites relatives aux ressources

| Ressource | Limite par défaut |
| --- | --- |
| [Ressources](assets-concept.md) par compte Media Services | 1 000 000|

## <a name="storage-limits"></a>Limites de stockage

| Ressource | Limite par défaut | 
| --- | --- | 
| Taille du fichier| Dans certains scénarios, la taille maximale des fichiers pris en charge pour le traitement dans Media Services est soumise à une limite. <sup>(1)</sup> |
| [Comptes de stockage](storage-account-concept.md) | 100<sup>(2)</sup> (fixe) |

<sup>1</sup> La taille maximale prise en charge pour un objet blob est actuellement de 5 To dans Stockage Blob Azure. Des limites supplémentaires sont applicables dans Media Services en fonction des tailles de machine virtuelle utilisées par le service. La limite de taille s’applique aux fichiers que vous chargez et également aux fichiers qui sont générés suite au traitement (encodage ou analyse) par Media Services. Si votre fichier source est supérieur à 260 Go, votre travail échouera probablement. 

Le tableau suivant indique les limites pour les unités réservées Multimédia S1, S2 et S3. Si votre fichier source dépasse la limite définie dans le tableau, votre travail d’encodage échoue. Si vous encodez des sources de résolution 4K de longue durée, vous devez obligatoirement utiliser des unités réservées Multimédia S3 afin d’obtenir les performances nécessaires. Si vous avez un contenu 4K d’une taille supérieure à la limite de 260 Go des unités réservées Multimédia S3, ouvrez un ticket de support.

|Types d’unités réservées Multimédia|Taille maximale en entrée (Go)|
|---|---|
|S1 |    26|
|S2    | 60|
|S3    |260|

<sup>2</sup> Les comptes de stockage doivent provenir du même abonnement Azure.

## <a name="jobs-encoding--analyzing-limits"></a>Limites relatives aux travaux (encodage et analyse)

| Ressource | Limite par défaut | 
| --- | --- | 
| [Travaux](transforms-jobs-concept.md) par compte Media Services | 500 000 <sup>(3)</sup> (fixe)|
| Entrées de travaux - par travail | 50 (fixe)|
| Sorties de travaux - par travail | 20 (fixe) |
| [Transformations](transforms-jobs-concept.md) par compte Media Services | 100 (fixe)|
| Sorties de transformation dans une transformation | 20 (fixe) |
| Fichiers par entrée de travail|10 (fixe)|

<sup>3</sup> Ce nombre comprend les travaux en file d’attente, terminés, actifs et annulés. Il n’inclut pas les travaux supprimés. 

Les enregistrements de travaux de votre compte qui ont plus de 90 jours sont automatiquement supprimés, même si le nombre total d’enregistrements est inférieur au quota maximal. 

## <a name="live-streaming-limits"></a>Limites relatives au streaming en direct

| Ressource | Limite par défaut | 
| --- | --- | 
| [Événements en direct](live-events-outputs-concept.md) <sup>(4)</sup> par compte Media Services |5|
| Sorties en direct par événement en direct |3 <sup>(5)</sup> |
| Durée maximale de la sortie en direct | [Taille de la fenêtre DVR](live-event-cloud-dvr.md) |

<sup>4</sup> Pour plus d'informations sur les limites relatives aux événements en direct, consultez [Comparaison et limites des types d'événements en direct](live-event-types-comparison.md).

<sup>5</sup> Les sorties en direct démarrent dès leur création et s'arrêtent lors de leur suppression.

## <a name="packaging--delivery-limits"></a>Limites relatives à l’empaquetage et à la remise

| Ressource | Limite par défaut |
| --- | --- |
| [Points de terminaison de streaming](streaming-endpoint-concept.md) (arrêtés ou en cours d'exécution) par compte Media Services | 2 |
| Unités de streaming Premium | 10 |
| [Filtres de manifeste dynamique](filters-dynamic-manifest-overview.md)|100|
| [Stratégies de diffusion en continu](streaming-policy-concept.md) | 100 <sup>(6)</sup> |
| [Localisateurs de streaming](streaming-locators-concept.md) uniques associés à une ressource à un moment donné | 100<sup>(7)</sup> (fixe) |

<sup>6</sup> Quand vous utilisez une [stratégie de streaming](/rest/api/media/streamingpolicies) personnalisée, vous devez concevoir un ensemble limité de ces stratégies pour votre compte Media Services, et les réutiliser pour vos éléments StreamingLocators chaque fois que les mêmes protocoles et options de chiffrement sont nécessaires. Vous ne devez pas créer une stratégie de diffusion en continu pour chaque localisateur de diffusion en continu.

<sup>7</sup> Les localisateurs de streaming ne sont pas conçus pour gérer le contrôle d’accès par utilisateur. Pour accorder différents droits d’accès aux utilisateurs, utilisez les solutions de gestion des droits numériques (DRM).

## <a name="protection-limits"></a>Limites relatives à la protection

| Ressource | Limite par défaut |
| --- | --- |
| Options par [stratégie de clé de contenu](content-key-policy-concept.md) |30 |
| Licences par mois pour chacun des types de DRM sur le service de remise de clé Media Services par compte|1 000 000|

## <a name="support-ticket"></a>Ticket de support

Pour les ressources qui ne sont pas fixes, vous pouvez demander d’augmenter les quotas en ouvrant un [ticket de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Incluez des informations détaillées dans la requête sur les modifications souhaitées pour les quotas, les scénarios de cas d’usage et les régions requises. <br/>Ne créez **pas** d’autres comptes Azure Media Services pour obtenir des limites supérieures.

## <a name="next-steps"></a>Étapes suivantes

[Vue d'ensemble](media-services-overview.md)
