---
title: Restauration dans le temps pour les objets blob de blocs
titleSuffix: Azure Storage
description: La restauration dans le temps pour les objets blob en blocs offre une protection contre la suppression ou les altérations accidentelles en vous permettant de restaurer un compte de stockage à son état précédent à un moment donné.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ca09e41e6d5b83f14d2dfee4107135585b7e945a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95908793"
---
# <a name="point-in-time-restore-for-block-blobs"></a>Restauration dans le temps pour les objets blob de blocs

La limite de restauration dans le temps offre une protection contre les suppressions ou altérations accidentelles en vous permettant de restaurer des données d’objet blob de blocs à un état antérieur. La limite de restauration dans le temps est utile dans les scénarios où un utilisateur ou une application supprime accidentellement des données ou lorsqu’une erreur d’application endommage les données. La restauration dans le temps permet également de tester des scénarios qui nécessitent le rétablissement d’un jeu de données à un état connu avant d’exécuter d’autres tests.

La limite de restauration dans le temps est prise en charge pour les comptes de stockage v2 à usage général uniquement. Seules les données des niveaux d’accès chaud et froid peuvent être restaurées dans le cadre d’une limite de restauration dans le temps.

Pour savoir comment activer la restauration jusqu’à une date et heure pour un compte de stockage, consultez [Effectuer une restauration jusqu’à une date et heure sur les données d’objet blob de blocs](point-in-time-restore-manage.md).

## <a name="how-point-in-time-restore-works"></a>Fonctionnement de la restauration dans le temps

Pour activer la restauration dans le temps, vous créez une stratégie de gestion pour le compte de stockage et spécifiez une période de rétention. Pendant la période de rétention, vous pouvez restaurer les objets blob de blocs de l’état actuel à un état passé.

Pour lancer une restauration dans le temps, appelez l'opération [Restaurer les plages d’objets blob](/rest/api/storagerp/storageaccounts/restoreblobranges) et spécifiez un point de restauration en heure UTC. Vous pouvez spécifier des plages lexicographiques de noms de conteneurs et d’objets blob à restaurer ou omettre la plage pour restaurer tous les conteneurs dans le compte de stockage. Dix plages lexicographiques sont prises en charge par opération de restauration.

Stockage Azure analyse toutes les modifications apportées aux objets blob spécifiés entre le point de restauration demandé, spécifié en heure UTC, et le moment présent. L’opération de restauration est atomique, ce qui signifie qu’elle réussit entièrement à restaurer toutes les modifications ou qu’elle échoue. S’il existe des objets blob qui ne peuvent pas être restaurés, l’opération échoue, et les opérations de lecture et d’écriture sur les conteneurs concernés reprennent.

Une seule opération de restauration à la fois peut être exécutée sur un compte de stockage. Une opération de restauration ne peut pas être annulée une fois qu’elle est en cours, mais une deuxième opération de restauration peut être effectuée pour annuler la première opération.

L’opération **Restaurer les plages d'objets blob** retourne un ID de restauration qui identifie de façon unique l’opération. Pour vérifier l’état d’une restauration dans le temps, appelez l’opération **Obtenir l’état de la restauration** avec l’ID de restauration renvoyé par l’opération **Restaurer les plages d'objets blob**.

> [!IMPORTANT]
> Lorsque vous effectuez une opération de restauration, Stockage Azure bloque les opérations de données sur les objets blob de la plage en cours de restauration pendant toute la durée de l’opération. Les opérations de lecture, d’écriture et de suppression sont bloquées dans l’emplacement principal. C’est la raison pour laquelle les opérations telles que l’énumération des conteneurs sur le portail Azure peuvent ne pas se dérouler comme prévu pendant que l’opération de restauration est en cours.
>
> Les opérations de lecture à partir de l’emplacement secondaire peuvent se poursuivre pendant l’opération de restauration si le compte de stockage est géorépliqué.

> [!CAUTION]
> La limite de restauration dans le temps prend en charge la restauration des opérations sur les objets blob de blocs uniquement. Les opérations sur les conteneurs ne peuvent pas être restaurées. Si vous supprimez un conteneur du compte de stockage en appelant l’opération [Supprimer le conteneur](/rest/api/storageservices/delete-container), ce conteneur ne peut pas être restauré à l’aide d’une opération de restauration. Au lieu de supprimer un conteneur, supprimez chacun des objets blob si vous souhaitez les restaurer.

### <a name="prerequisites-for-point-in-time-restore"></a>Conditions préalables pour une restauration dans le temps

La limite de restauration dans le temps implique que les fonctionnalités Stockage Azure suivantes soient activées avant la restauration :

- [Suppression réversible](./soft-delete-blob-overview.md)
- [Flux de modification](storage-blob-change-feed.md)
- [Contrôle de version des blobs](versioning-overview.md)

### <a name="retention-period-for-point-in-time-restore"></a>Période de rétention pour une restauration dans le temps

Lorsque vous activez la restauration dans le temps pour un compte de stockage, vous spécifiez une période de rétention. Les objets blob de blocs de votre compte de stockage peuvent être restaurés au cours de la période de rétention.

La période de rétention commence lorsque vous activez la restauration dans le temps. N’oubliez pas que vous ne pouvez pas restaurer des objets blob dans un état antérieur au début de la période de rétention. Par exemple, si vous avez activé la restauration dans le temps le 1er mai avec une rétention de 30 jours, alors le 15 mai, vous pouvez effectuer une restauration pour maximum de 15 jours. Le 1er juin, vous pouvez restaurer les données entre 1 et 30 jours.

La période de rétention pour la restauration dans le temps doit être inférieure ou égale à la période de rétention spécifiée pour la suppression réversible. Par exemple, si la période de rétention de la suppression réversible est définie sur 7 jours, la période de rétention de la restauration dans le temps peut être comprise entre 1 et 6 jours.

### <a name="permissions-for-point-in-time-restore"></a>Autorisations pour une restauration dans le temps

Pour lancer une opération de restauration, un client doit disposer d’autorisations en écriture sur tous les conteneurs du compte de stockage. Pour autoriser une opération de restauration avec Azure Active Directory (Azure AD), attribuez le rôle **Contributeur de comptes de stockage** au principal de sécurité au niveau du compte de stockage, du groupe de ressources ou de l’abonnement.

## <a name="limitations-and-known-issues"></a>Limitations et problèmes connus

La restauration jusqu’à une date et heure pour les objets blob de blocs présente les limitations et les problèmes connus suivants :

- Seuls les objets blob de blocs d’un compte de stockage v2 standard à usage général peuvent être restaurés dans le cadre d’une opération de restauration jusqu’à une date et heure. Les objets blob d’ajout, les objets blob de pages et les objets blob de blocs Premium ne sont pas restaurés. 
- Si vous avez supprimé un conteneur au cours de la période de rétention, ce conteneur ne sera pas restauré lors de l’opération de restauration jusqu’à une date et heure. Si vous tentez de restaurer une plage d’objets blob incluant des objets blob dont le conteneur a été supprimé, l’opération de récupération jusqu’à une date et heure échouera. Pour en savoir plus sur la protection des conteneurs contre la suppression, consultez [Suppression réversible pour les conteneurs (préversion)](soft-delete-container-overview.md).
- Si un objet blob a été déplacé entre les niveaux chaud et froid pendant la période comprise entre le moment présent et le point de restauration, l’objet blob est restauré à son niveau précédent. La restauration d’objets blob de blocs du niveau archive n’est pas prise en charge. Par exemple, si un objet blob a été déplacé du niveau d’accès chaud au niveau de stockage archive il y a deux jours et qu’une opération de restauration restaure un point d’il y a trois jours, l’objet blob n’est pas restauré vers le niveau d’accès chaud. Pour restaurer un objet blob archivé, commencez par le déplacer en dehors du niveau archive. Pour plus d’informations, consultez [Réalimenter les données d’objets blob à partir du niveau Archive](storage-blob-rehydration.md).
- Un bloc qui a été chargé via [Put Block](/rest/api/storageservices/put-block) ou [Put Block à partir d’une URL](/rest/api/storageservices/put-block-from-url), mais n’est pas validé via [Put Block List](/rest/api/storageservices/put-block-list), ne fait pas partie d’un objet blob et n’est donc pas restauré dans le cadre d’une opération de restauration.
- Un objet blob avec un bail actif ne peut pas être restauré. Si un objet blob avec un bail actif est inclus dans la plage d’objets blob à restaurer, l’opération de restauration échoue de façon atomique. Arrêtez tout bail actif avant de lancer l’opération de restauration.
- Les instantanés ne sont pas créés ou supprimés dans le cadre d’une opération de restauration. Seul l’objet blob de base est restauré à son état précédent.
- La restauration d’espaces de noms plats et hiérarchiques Azure Data Lake Storage Gen2 n’est pas prise en charge.

> [!IMPORTANT]
> Si vous restaurez des objets blob de blocs à un point antérieur au 22 septembre 2020, les limitations de la restauration jusqu’à une date et heure seront appliquées. Microsoft vous recommande de choisir un point de restauration égal ou postérieur au 22 septembre 2020 pour tirer parti de la fonctionnalité de restauration jusqu’à une date et heure généralement disponible.

## <a name="pricing-and-billing"></a>Tarification et facturation

La facturation de la restauration dans le temps dépend de la quantité de données traitées pour effectuer l’opération de restauration. La quantité de données traitées est basée sur le nombre de modifications qui se sont produites entre le point de restauration et le moment présent. Par exemple, en supposant un taux de changement relativement constant pour bloquer les données de blob dans un compte de stockage, une opération de restauration qui remonte à 1 jour dans le passé coûterait 1/10ème d’une restauration qui remonte à 10 jours dans le passé.

Pour estimer le coût d’une opération de restauration, consultez le journal des flux de modification afin d’estimer la quantité de données qui a été modifiée au cours de la période de restauration. Par exemple, si la période de rétention du flux de modification est de 30 jours et que la taille du flux de modification est de 10 Mo, la restauration jusqu’à 10 jours dans le temps coûterait approximativement un tiers du prix indiqué pour un compte LRS dans cette région. La restauration jusqu’à un point situé 27 jours dans le passé coûterait environ neuf dixièmes du prix indiqué.

Pour plus d’informations sur la tarification pour la restauration dans le passé, consultez [Tarification des objets blob de blocs](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="next-steps"></a>Étapes suivantes

- [Effectuer une restauration jusqu’à une date et heure sur les données d’objet blob de blocs](point-in-time-restore-manage.md)
- [Prise en charge du flux de modification dans Stockage Blob Azure](storage-blob-change-feed.md)
- [Activer la suppression réversible pour les objets blob](./soft-delete-blob-enable.md)
- [Activer et gérer le contrôle de version des objets blob](versioning-enable.md)