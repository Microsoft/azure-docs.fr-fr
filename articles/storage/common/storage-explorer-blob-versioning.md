---
title: Guide du contrôle de version des blobs d’Explorateur Stockage Azure | Microsoft Docs
description: Guide de contrôle de version des blobs pour Explorateur Stockage Azure
services: storage
author: chuye
ms.service: storage
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: chuye
ms.openlocfilehash: e20733cb4b93fcfac7606895746645727f12d6c8
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92783825"
---
# <a name="azure-storage-explorer-blob-versioning-guide"></a>Guide du contrôle de version des blobs d’Explorateur Stockage Azure

Explorateur Stockage Microsoft Azure permet d’accéder aux versions des blobs et de les gérer facilement. Ce guide vous aidera à comprendre le fonctionnement du contrôle de version des blobs dans Explorateur Stockage. Avant de continuer, il est recommandé d’en savoir plus sur [le contrôle de version des blobs](../blobs/versioning-overview.md).

## <a name="terminology"></a>Terminologie

Cette section fournit quelques définitions pour aider à comprendre l’utilisation de ces termes dans cet article.

- Suppression réversible : autre fonctionnalité de protection automatique des données. Vous pouvez en savoir plus sur la suppression réversible [ici](../blobs/soft-delete-blob-overview.md).
- Blob actif : blob ou version de blob créés dans un état actif. Vous pouvez uniquement utiliser des blobs ou des versions de blob dans un état actif.
- Blob supprimé de manière réversible : blob ou version de blob marqués comme supprimés de manière réversible. Les blobs supprimés de manière réversible sont conservés uniquement pendant la période de rétention.
- Version de blob : blob créé avec le contrôle de version des blobs activé. Chaque version de blob est associée à un ID de version.
- Version actuelle : version de blob marquée comme étant la version actuelle.
- Version précédente : version de blob qui n’est pas la version actuelle.
- Blob sans version : blob créé avec le contrôle de version des blobs désactivé. Un blob sans version n’a pas d’ID de version.

## <a name="view-blob-versions"></a>Afficher les versions de blob

Explorateur Stockage prend en charge quatre vues différentes pour l’affichage des blobs.

| Affichage | Blobs sans version actifs | Blobs sans version supprimés de manière réversible | Versions de blob |
| ---- | :----------: | :-----------: | :------------------: |
| Blobs actifs | Oui | Non | Version actuelle uniquement |
| Blobs actifs et blobs supprimés de manière réversible | Oui | Oui | Version actuelle uniquement |
| Blobs actifs et blobs sans la version actuelle | Oui | Non | Version actuelle ou version active la plus récente |
| Tous les blobs et blobs sans la version actuelle | Oui | Oui | Version actuelle ou version la plus récente |

### <a name="active-blobs"></a>Blobs actifs

Dans cette vue, Explorateur Stockage affiche :

- Blobs sans version actifs
- Versions actuelles

### <a name="active-blobs-and-soft-deleted-blobs"></a>Blobs actifs et blobs supprimés de manière réversible

Dans cette vue, Explorateur Stockage affiche :

- Blobs sans version actifs
- Blobs sans version supprimés de manière réversible
- Versions actuelles.

### <a name="active-blobs-and-blobs-without-current-version"></a>Blobs actifs et blobs sans la version actuelle

Dans cette vue, Explorateur Stockage affiche :

- Blobs sans version actifs
- Versions actuelles
- Versions précédentes actives les plus récentes. 

Pour les blobs qui n’ont pas de version actuelle, mais qui ont une version précédente active, Explorateur Stockage affiche leur version précédente active la plus récente comme représentation de ce blob.

### <a name="all-blobs-and-blobs-without-current-version"></a>Tous les blobs et blobs sans la version actuelle

Dans cette vue, Explorateur Stockage affiche :

- Blobs sans version actifs
- Blobs sans version supprimés de manière réversible
- Versions actuelles
- Versions précédentes les plus récentes. 

Pour les blobs qui n’ont pas de version actuelle, Explorateur Stockage affiche leur version précédente la plus récente comme représentation de ce blob.

> [!Note]
> En raison de la limitation du service, Explorateur Stockage nécessite un traitement supplémentaire pour obtenir une vue hiérarchique de vos répertoires virtuels lors du listage des versions de blob. Il faudra plus de temps pour répertorier les blobs dans les vues suivantes :
> 
> - Blobs actifs et blobs sans la version actuelle
> - Tous les blobs et blobs sans la version actuelle

## <a name="manage-blob-versions"></a>Gérer les versions de blob

### <a name="view-versions-of-a-blob"></a>Afficher les versions d’un blob

Explorateur Stockage fournit une commande **Gérer les versions** pour afficher toutes les versions d’un blob. Pour afficher les versions d’un blob, sélectionnez le blob dont vous souhaitez consulter les versions et sélectionnez **Gérer l’historique &rarr; Gérer les versions** à partir de la barre d’outils ou du menu contextuel.

### <a name="download-blob-versions"></a>Télécharger des versions de blob

Pour télécharger une ou plusieurs versions de blob, sélectionnez les versions de blob que vous souhaitez télécharger, puis sélectionnez **Télécharger** à partir de la barre d’outils ou du menu contextuel.

Si vous téléchargez plusieurs versions d’un blob, les fichiers téléchargés comporteront leurs ID de version au début de leur nom de fichier.

### <a name="delete-blob-versions"></a>Supprimer des versions de blob

Pour supprimer une ou plusieurs versions de blob, sélectionnez les versions de blob que vous souhaitez supprimer, puis sélectionnez **Supprimer** à partir de la barre d’outils ou du menu contextuel.

Les versions de blob sont soumises à votre stratégie de suppression réversible. Si la suppression réversible est activée, les versions de blob seront supprimées de manière réversible. La suppression d’une version actuelle est un cas particulier. En supprimant une version actuelle, une version précédente active la remplace automatiquement.

### <a name="promote-blob-version"></a>Promouvoir une version de blob

Vous pouvez restaurer le contenu d’un blob en promouvant une version précédente pour qu’elle devienne la version actuelle. Sélectionnez la version du blob que vous souhaitez promouvoir, puis sélectionnez **Promouvoir la version** à partir de la barre d’outils ou du menu contextuel.

Les blobs sans version seront remplacés par la version de blob promue. Assurez-vous de ne plus avoir besoin de ces données ou sauvegardez les données vous-même avant de confirmer l’opération. Les versions actuelles deviennent automatiquement des versions précédentes, donc Explorateur Stockage ne demande pas de confirmation.

### <a name="undelete-blob-version"></a>Annuler la suppression d’une version de blob

Les suppressions de version de blob ne peuvent pas être annulées individuellement. Elles doivent toutes être annulées en même temps. Pour annuler la suppression de toutes les versions d’un blob, sélectionnez l’une des versions du blob et sélectionnez **Annuler la suppression des éléments sélectionnés** dans la barre d’outils ou dans le menu contextuel.

### <a name="change-access-tier-of-blob-versions"></a>Modifier le niveau d’accès des versions de blob

Chaque version de blob a son propre niveau d’accès. Pour modifier le niveau d’accès des versions de blob, sélectionnez les versions de blob pour lesquelles vous souhaitez modifier le niveau d’accès, puis sélectionnez **Modifier le niveau d’accès…** dans le menu contextuel.

## <a name="see-also"></a>Voir aussi

* [Contrôle de version des blobs](../blobs/versioning-overview.md)
* [Suppression réversible pour les objets blob](../blobs/soft-delete-blob-overview.md)