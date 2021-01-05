---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 10/26/2020
ms.author: inhenkel
ms.openlocfilehash: 59ff0ba854fa609e6d29f3473f662a89ab5f3dbc
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95556905"
---
> [!NOTE]
> Pour les ressources qui ne sont pas corrigées, ouvrez un ticket de support pour demander une augmentation des quotas. Ne créez pas d’autres comptes Azure Media Services pour obtenir des limites supérieures.

### <a name="account-limits"></a>Limites relatives aux comptes

| Ressource | Limite par défaut |
| --- | --- |
| Comptes Media Services dans un même abonnement | 100 (fixe) |

### <a name="asset-limits"></a>Limites relatives aux ressources

| Ressource | Limite par défaut |
| --- | --- |
| Ressources par compte Media Services | 1 000 000|

### <a name="storage-media-limits"></a>Limites relatives au stockage (support)

| Ressource | Limite par défaut |
| --- | --- |
| Taille du fichier| Dans certains scénarios, la taille maximale des fichiers pris en charge pour le traitement dans Media Services est soumise à une limite. <sup>(1)</sup> |
| Comptes de stockage | 100<sup>(2)</sup> (fixe) |

<sup>1</sup> La taille maximale prise en charge pour un objet blob est actuellement de 5 To dans Stockage Blob Azure. Des limites supplémentaires sont applicables dans Media Services en fonction des tailles de machine virtuelle utilisées par le service. La limite de taille s’applique aux fichiers que vous chargez et également aux fichiers qui sont générés suite au traitement (encodage ou analyse) par Media Services. Si votre fichier source est supérieur à 260 Go, votre travail échouera probablement.

Le tableau suivant indique les limites pour les unités réservées Multimédia S1, S2 et S3. Si votre fichier source dépasse la limite définie dans le tableau, votre travail d’encodage échoue. Si vous encodez des sources de résolution 4K de longue durée, vous devez obligatoirement utiliser des unités réservées Multimédia S3 afin d’obtenir les performances nécessaires. Si vous avez un contenu 4K d’une taille supérieure à la limite de 260 Go des unités réservées Multimédia S3, ouvrez un ticket de support.

|Types d’unités réservées Multimédia|Taille maximale en entrée (Go)|
|---|---|
|S1 |    26|
|S2    | 60|
|S3    |260|

<sup>2</sup> Les comptes de stockage doivent provenir du même abonnement Azure.

### <a name="jobs-encoding--analyzing-limits"></a>Limites relatives aux travaux (encodage et analyse)

| Ressource | Limite par défaut |
| --- | --- |
| Travaux par compte Media Services | 500 000 <sup>(3)</sup> (fixe)|
| Entrées de travaux - par travail | 50 (fixe)|
| Sorties de travaux - par travail | 20 (fixe) |
| Transformations par compte Media Services | 100 (fixe)|
| Sorties de transformation dans une transformation | 20 (fixe) |
| Fichiers par entrée de travail|10 (fixe)|

<sup>3</sup> Ce nombre comprend les travaux en file d’attente, terminés, actifs et annulés. Il n’inclut pas les travaux supprimés. 

Les enregistrements de travaux de votre compte qui ont plus de 90 jours sont automatiquement supprimés, même si le nombre total d’enregistrements est inférieur au quota maximal. 

### <a name="live-streaming-limits"></a>Limites relatives au streaming en direct

| Ressource | Limite par défaut |
| --- | --- |
| Événements en direct <sup>(4)</sup> par compte Media Services |5|
| Sorties en direct par événement en direct |3 <sup>(5)</sup> |
| Durée maximale de la sortie en direct | [Taille de la fenêtre DVR](../articles/media-services/latest/live-event-cloud-dvr.md) |

<sup>4</sup> Pour plus d’informations sur les limites relatives aux événements en direct, consultez [Comparaison et limites des types d’événements en direct](../articles/media-services/latest/live-event-types-comparison.md).

<sup>5</sup> Les sorties en direct démarrent dès leur création et s’arrêtent lors de leur suppression.

### <a name="packaging--delivery-limits"></a>Limites relatives à l’empaquetage et à la remise

| Ressource | Limite par défaut |
| --- | --- |
| Points de terminaison de streaming (arrêtés ou en cours d’exécution) par compte Media Services| 2 |
| Filtres de manifeste dynamique|100|
| Stratégies de diffusion en continu | 100 <sup>(6)</sup> |
| Localisateurs de streaming uniques associés à une ressource à un moment donné | 100<sup>(7)</sup> (fixe) |

<sup>6</sup> Quand vous utilisez une [stratégie de streaming](/rest/api/media/streamingpolicies) personnalisée, vous devez concevoir un ensemble limité de ces stratégies pour votre compte Media Services, et les réutiliser pour vos éléments StreamingLocators chaque fois que les mêmes protocoles et options de chiffrement sont nécessaires. Vous ne devez pas créer une stratégie de diffusion en continu pour chaque localisateur de diffusion en continu.

<sup>7</sup> Les localisateurs de streaming ne sont pas conçus pour gérer le contrôle d’accès par utilisateur. Pour accorder différents droits d’accès aux utilisateurs, utilisez les solutions de gestion des droits numériques (DRM).

### <a name="protection-limits"></a>Limites relatives à la protection

| Ressource | Limite par défaut |
| --- | --- |
| Options de stratégie de clé de contenu | 30 |
| Licences par mois pour chacun des types de DRM sur le service de remise de clé Media Services par compte|1 000 000|

### <a name="support-ticket"></a>Ticket de support

Pour les ressources qui ne sont pas fixes, vous pouvez demander d’augmenter les quotas en ouvrant un [ticket de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Incluez des informations détaillées dans la requête sur les modifications souhaitées pour les quotas, les scénarios de cas d’usage et les régions requises. <br/>Ne créez **pas** d’autres comptes Azure Media Services pour obtenir des limites supérieures.