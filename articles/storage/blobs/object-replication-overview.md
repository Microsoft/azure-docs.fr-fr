---
title: Vue d'ensemble de la réplication d'objets
titleSuffix: Azure Storage
description: La réplication d'objets copie de manière asynchrone des objets blob de blocs entre un compte de stockage source et un compte de destination. Utilisez la réplication d’objets pour réduire la latence sur les demandes de lecture, pour renforcer l’efficacité des charges de travail de calcul, pour optimiser la distribution des données et pour réduire les coûts.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 11/13/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 47a2aae39be93361e1e0e581efb56cc678b444cd
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549087"
---
# <a name="object-replication-for-block-blobs"></a>Réplication d'objets blob de blocs

La réplication d'objets copie de manière asynchrone des objets blob de blocs entre un compte de stockage source et un compte de destination. Voici quelques scénarios pris en charge par la réplication d’objets :

- **Minimisation de la latence.** La réplication d’objets peut réduire la latence des demandes de lecture en permettant aux clients d’utiliser des données provenant d’une région plus proche en termes de proximité physique.
- **Augmentation de l’efficacité des charges de travail de calcul.** Avec la réplication d’objets, les charges de travail de calcul peuvent traiter les mêmes ensembles d’objets blob de blocs dans différentes régions.
- **Optimisation de la distribution des données.** Vous pouvez traiter ou analyser des données incluses dans un emplacement unique, puis répliquer uniquement les résultats dans d’autres régions.
- **Optimisation des coûts.** Une fois vos données répliquées, vous pouvez réduire les coûts en les déplaçant vers le niveau archive à l’aide des stratégies de gestion du cycle de vie.

Le diagramme suivant montre comment la réplication d’objets réplique les objets blob de blocs d’un compte de stockage source dans une région vers des comptes de destination dans deux régions différentes.

:::image type="content" source="media/object-replication-overview/object-replication-diagram.svg" alt-text="Diagramme montrant le fonctionnement de la réplication d’objets":::

Pour plus d'informations sur la configuration de la réplication d'objets, consultez [Configurer la réplication d'objets](object-replication-configure.md).

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="prerequisites-for-object-replication"></a>Prérequis pour la réplication d’objets

La réplication d'objets exige que les fonctionnalités de stockage Azure suivantes soient également activées :

- [Flux de modification](storage-blob-change-feed.md) : doit être activé sur le compte source. Pour plus d'informations sur l'activation du flux de modifications, consultez [Activer et désactiver le flux de modifications](storage-blob-change-feed.md#enable-and-disable-the-change-feed).
- [Contrôle de version des objets blob](versioning-overview.md) : doit être activé sur les comptes source et de destination. Pour plus d'informations sur l'activation du contrôle de version des objets blob, consultez [Activer et gérer le contrôle de version des objets blob](versioning-enable.md).

L’activation de ces fonctionnalités peut entraîner des coûts supplémentaires. Pour plus d’informations, consultez la [page des tarifs du stockage Azure](https://azure.microsoft.com/pricing/details/storage/).

## <a name="how-object-replication-works"></a>Fonctionnement de la réplication d’objets

La réplication d’objets copie de façon asynchrone les objets blob de blocs dans un conteneur selon les règles que vous configurez. Le contenu de l’objet blob, toutes les versions associées à l’objet blob, ainsi que les métadonnées et propriétés de l’objet blob sont copiés du conteneur source vers le conteneur de destination.

> [!IMPORTANT]
> Étant donné que les données d’objets blob de blocs sont répliquées de façon asynchrone, le compte source et le compte de destination ne sont pas immédiatement synchronisés. Il n’existe actuellement aucun contrat de niveau de service (SLA) sur le temps nécessaire à la réplication des données vers le compte de destination. Vous pouvez vérifier l’état de réplication sur l’objet blob source afin de déterminer si la réplication est terminée. Pour plus d’informations, consultez [Vérifier l’état de réplication d’un objet blob](object-replication-configure.md#check-the-replication-status-of-a-blob).

### <a name="blob-versioning"></a>Contrôle de version des objets blob

La réplication d’objets implique que le contrôle de version des objets blob soit activé sur les comptes source et de destination. Lorsqu’un objet blob répliqué dans le compte source est modifié, une nouvelle version de l’objet blob est créée dans le compte source et reflète l’état précédent de l’objet blob, avant modification. La version actuelle (ou objet blob de base) du compte source reflète les mises à jour les plus récentes. La version actuelle mise à jour et la nouvelle version précédente sont répliquées vers le compte de destination. Pour plus d’informations sur la façon dont les opérations d’écriture affectent les versions d’objets blob, consultez [Contrôle de version sur les opérations d’écriture](versioning-overview.md#versioning-on-write-operations).

Lorsqu’un objet blob du compte source est supprimé, la version actuelle de l’objet blob est capturée dans une version antérieure, puis supprimée. Toutes les versions antérieures de l’objet blob sont conservées même après la suppression de la version actuelle. Cet état est répliqué dans le compte de destination. Pour plus d’informations sur la façon dont les opérations de suppression affectent les versions d’objets blob, consultez [Contrôle de version sur les opérations de suppression](versioning-overview.md#versioning-on-delete-operations).

### <a name="snapshots"></a>Instantanés

La réplication d’objets ne prend pas en charge les instantanés d’objet blob. Les instantanés d’un objet blob du compte source ne sont pas répliqués vers le compte de destination.

### <a name="blob-tiering"></a>Hiérarchisation des objets blob

La réplication d’objets est prise en charge lorsque les comptes source et de destination se trouvent au niveau chaud ou froid. Les comptes source et de destination peuvent se trouver dans des niveaux différents. Toutefois, la réplication d’objets échoue si un objet blob du compte source ou de destination a été déplacé vers le niveau archive. Pour plus d’informations sur les niveaux des objets blob, consultez [Niveaux d’accès pour Stockage Blob Azure : chaud, froid et archive](storage-blob-storage-tiers.md).

### <a name="immutable-blobs"></a>Objets blob immuables

La réplication d’objets ne prend pas en charge les objets blob immuables. Si un conteneur source ou de destination dispose d’une stratégie de rétention limitée dans le temps ou d’une conservation légale, la réplication d’objets échoue. Pour plus d’informations sur les objets blob immuables, consultez [Stocker des données blob critiques pour l’entreprise avec un stockage immuable](storage-blob-immutable-storage.md).

## <a name="object-replication-policies-and-rules"></a>Stratégies et règles de réplication d’objets

Lorsque vous configurez la réplication d’objets, vous créez une stratégie de réplication qui spécifie le compte de stockage source et le compte de destination. Une stratégie de réplication comprend une ou plusieurs règles qui spécifient un conteneur source et un conteneur de destination et indiquent quels objets blob de blocs du conteneur source seront répliqués.

Une fois que vous avez configuré la réplication d’objets, Stockage Azure vérifie régulièrement le flux de modification du compte source et réplique de manière asynchrone toutes les opérations d’écriture ou de suppression dans le compte de destination. La latence de la réplication dépend de la taille de l’objet blob de blocs en cours de réplication.

### <a name="replication-policies"></a>Stratégies de réplication

Lorsque vous configurez la réplication d’objets, une stratégie de réplication est créée à la fois sur le compte source et le compte de destination par le biais du fournisseur de ressources de Stockage Azure. La stratégie de réplication est identifiée par un ID de stratégie. La stratégie sur les comptes source et de destination doit avoir le même ID de stratégie pour que la réplication ait lieu.

Un compte de stockage peut servir de compte source pour un maximum de deux comptes de destination. Les comptes source et de destination peuvent se trouver dans la même région ou dans des régions différentes. Ils peuvent également être associés à des abonnements et à des locataires Azure Active Directory (Azure AD) différents. Une seule stratégie de réplication peut être créée pour chaque paire compte source/compte de destination.

### <a name="replication-rules"></a>Règles de réplication

Les règles de réplication spécifient comment Stockage Azure va répliquer les objets blob d’un conteneur source vers un conteneur de destination. Vous pouvez spécifier jusqu’à 10 règles de réplication pour chaque stratégie de réplication. Chaque règle de réplication définit un seul conteneur source et de destination, et chaque conteneur ne peut être utilisé que dans une seule règle.

Quand vous créez une règle de réplication, par défaut, seuls les nouveaux objets blob de blocs ajoutés par la suite au conteneur source sont copiés. Vous pouvez spécifier que les objets blob de blocs, qu'ils soient nouveaux et existants, soient copiés, ou bien vous pouvez définir une étendue de copie personnalisée qui copie les objets blob de blocs créés à partir d'un moment donné.

Vous pouvez aussi spécifier un ou plusieurs filtres dans le cadre d’une règle de réplication pour filtrer les objets blob de blocs par préfixe. Lorsque vous spécifiez un préfixe, seuls les objets blob correspondant à ce préfixe dans le conteneur source sont copiés dans le conteneur de destination.

Les conteneurs source et de destination doivent tous les deux exister pour que vous puissiez les spécifier dans une règle. Une fois que vous avez créé la stratégie de réplication, le conteneur de destination bascule en lecture seule. Toute tentative d’écriture dans le conteneur de destination échoue avec le code d’erreur 409 (Conflit). En revanche, vous pouvez appeler l’opération [Définir le niveau du blob](/rest/api/storageservices/set-blob-tier) sur un objet blob dans le conteneur de destination pour le déplacer vers le niveau d’archive. Pour plus d’informations sur le niveau d’archive, consultez [Stockage Blob Azure : niveaux d’accès chaud, froid et archive](storage-blob-storage-tiers.md#archive-access-tier).

## <a name="billing"></a>Facturation

La réplication d'objets entraîne des coûts supplémentaires pour les transactions de lecture et d'écriture sur les comptes source et de destination, ainsi que des frais de sortie pour la réplication des données du compte source vers le compte de destination, et des frais de lecture pour le traitement des flux de modification.

## <a name="next-steps"></a>Étapes suivantes

- [Configurer la réplication d’objets](object-replication-configure.md)
- [Prise en charge du flux de modification dans Stockage Blob Azure](storage-blob-change-feed.md)
- [Activer et gérer le contrôle de version des objets blob](versioning-enable.md)
