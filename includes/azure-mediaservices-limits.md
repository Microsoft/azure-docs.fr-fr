---
author: rothja
ms.service: media-services
ms.topic: include
ms.date: 09/16/2020
ms.author: jroth
ms.openlocfilehash: c315c071aeb36eea0bd1af84b1344cf2fef5f703
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91329655"
---
>[!NOTE]
>Pour les ressources qui ne sont pas corrigées, ouvrez un ticket de support pour demander une augmentation des quotas. Ne créez pas d’autres comptes Azure Media Services pour obtenir des limites supérieures.

| Ressource | Limite | 
| --- | --- | 
| Comptes Azure Media Services dans un même abonnement | 25 (fixe) |
| Unités réservées Multimédia par compte Media Services |25 (S1)<br/>10 (S2, S3)<sup>1</sup> | 
| Travaux par compte Media Services | 50 000<sup>2</sup> |
| Tâches chaînées par travail | 30 (fixe) |
| Ressources par compte Media Services | 1 000 000|
| Actifs par tâche | 50 |
| Actifs par travail | 100 |
| Localisateurs uniques associés à un actif à un moment donné | 5<sup>4</sup> |
| Canaux live par compte Media Services |5|
| Programmes dans un état Arrêté par canal |50|
| Programmes en cours d’exécution par canal |3|
| Points de terminaison de streaming arrêtés ou en cours d’exécution par compte Media Services|2|
| Unités de diffusion en continu par point de terminaison de diffusion en continu |10 |
| Comptes de stockage | 100<sup>5</sup> (fixe) |
| Stratégies | 1,000,000<sup>6</sup> |
| Taille du fichier| Dans certains scénarios, la taille maximale des fichiers pris en charge pour le traitement dans Media Services est soumise à une limite<sup>7</sup>. |

<sup>1</sup> Si vous changez le type (par exemple, de S2 à S1), les limites d’unités réservées maximales sont réinitialisées.

<sup>2</sup> Ce nombre comprend les travaux en file d’attente, terminés, actifs et annulés. Il n’inclut pas les travaux supprimés. Vous pouvez supprimer des anciens travaux à l’aide de **IJob.Delete** ou de la requête HTTP **DELETE**.

À compter du 1er avril 2017, les enregistrements de travaux de votre compte de plus de 90 jours sont supprimés automatiquement, ainsi que leurs enregistrements de tâches associés. La suppression automatique se produit même si le nombre total d’enregistrements est inférieur au quota maximal. Pour archiver les informations des travaux/tâches, utilisez le code décrit dans [Gérer les ressources avec le SDK Media Services .NET](../articles/media-services/previous/media-services-dotnet-manage-entities.md).

<sup>3</sup> Lors d’une requête visant à lister les entités de travail, un maximum de 1 000 travaux sont retournés par requête. Pour suivre l’ensemble des travaux soumis, vous pouvez utiliser les requêtes top/skip comme décrit dans [Options de requête du système OData](/previous-versions/dynamicscrm-2015/developers-guide/gg309461(v=crm.7)).

<sup>4</sup> Les localisateurs ne sont pas conçus pour gérer le contrôle d’accès par utilisateur. Pour accorder différents droits d’accès aux utilisateurs, utilisez les solutions de gestion des droits numériques (DRM). Pour plus d’informations, consultez [Protéger votre contenu avec Azure Media Services](../articles/media-services/previous/media-services-content-protection-overview.md).

<sup>5</sup> Les comptes de stockage doivent provenir du même abonnement Azure.

<sup>6</sup> Un nombre limite de 1 000 000 a été défini pour les différentes stratégies Media Services. C’est notamment le cas pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy. 

>[!NOTE]
> Appliquez le même ID de stratégie si vous utilisez toujours le même nombre de jours et les mêmes autorisations d’accès. Pour obtenir plus d’informations ainsi qu’un exemple, consultez [Gérer les ressources avec le SDK Media Services .NET](../articles/media-services/previous/media-services-dotnet-manage-entities.md#limit-access-policies).

<sup>7</sup> La taille maximale prise en charge pour un objet blob est actuellement de 5 To dans le stockage Blob Azure. Des limites supplémentaires sont applicables dans Media Services en fonction des tailles de machine virtuelle utilisées par le service. La limite de taille s’applique aux fichiers que vous chargez et également aux fichiers qui sont générés suite au traitement (encodage ou analyse) par Media Services. Si votre fichier source est supérieur à 260 Go, votre travail échouera probablement. 

Le tableau suivant indique les limites pour les unités réservées Multimédia S1, S2 et S3. Si votre fichier source dépasse la limite définie dans le tableau, votre travail d’encodage échoue. Si vous encodez des sources de résolution 4K de longue durée, vous devez obligatoirement utiliser des unités réservées Multimédia S3 afin d’obtenir les performances nécessaires. Si vous avez un contenu 4K d’une taille supérieure à la limite de 260 Go des unités réservées Multimédia S3, ouvrez un ticket de support.

|Types d’unités réservées Multimédia    |Taille maximale en entrée (Go)|
|---|---|
|S1 |    26|
|S2    | 60|
|S3    |260|
