---
title: Migrer vers des partages de fichiers Azure
description: Découvrez les migrations vers les partages de fichiers Azure et trouvez votre guide de migration.
author: fauhse
ms.service: storage
ms.topic: conceptual
ms.date: 3/18/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 995ae176a8eec58f8dc9522e6fac6fd78170014d
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94628915"
---
# <a name="migrate-to-azure-file-shares"></a>Migrer vers des partages de fichiers Azure

Cet article aborde les aspects de base de la migration vers les partages de fichiers Azure.

Cet article contient des notions de base de migration et un tableau des guides de migration. Ces guides vous aident à déplacer vos fichiers vers des partages de fichiers Azure. Les guides sont organisés en fonction de l’emplacement de vos données et du modèle de déploiement (cloud uniquement ou hybride) vers lequel vous vous déplacez.

## <a name="migration-basics"></a>Notions de base de la migration

Azure dispose de plusieurs types de stockage cloud disponibles. Un aspect fondamental de la migration de fichiers vers Azure consiste à déterminer l’option de stockage Azure adaptée à vos données.

Les [partages de fichiers Azure](storage-files-introduction.md) sont bien adaptés aux données de fichiers à usage général. Ces données incluent tout ce que vous utilisez avec un partage SMB ou NFS local. Avec [Azure File Sync](storage-sync-files-planning.md), vous pouvez mettre en cache le contenu de plusieurs partages de fichiers Azure sur des serveurs exécutant Windows en local.

Pour une application qui s’exécute actuellement sur un serveur local, le stockage de fichiers dans un partage de fichiers Azure peut être un bon choix. Vous pouvez déplacer l’application vers Azure et utiliser des partages de fichiers Azure comme stockage partagé. Vous pouvez également envisager d’utiliser des [disques Azure](../../virtual-machines/managed-disks-overview.md) dans ce scénario.

Certaines applications cloud ne dépendent pas de SMB, de l’accès local aux données de l’ordinateur ou de l’accès partagé. Pour ces applications, le stockage d’objets tels que les [objets blob Azure](../blobs/storage-blobs-overview.md) est souvent le meilleur choix.

La clé de toute migration consiste à capturer toutes les correspondances de fichiers applicables lors du déplacement de vos fichiers de leur emplacement de stockage actuel vers Azure. La fidélité de l’option de stockage Azure et les exigences de votre scénario vous aideront à choisir le stockage Azure approprié. Traditionnellement, les données de fichiers à usage général dépendent des métadonnées de fichier. Ce n’est pas forcément le cas des données d’application.

Voici les deux composants de base d’un fichier :

- **Flux de données** : le flux de données enregistre le contenu du fichier.
- **Métadonnées du fichier** : Les métadonnées de fichier comportent les sous-composants suivants :
   * Attributs de fichier tels que En lecture seule
   * Autorisations de fichier, appelées *autorisations NTFS* ou *listes de contrôle d’accès de fichiers et de dossiers*
   * Horodatages, en particulier, les horodatages de création et de dernière modification
   * Autre flux de données, qui est un espace de stockage de grandes quantités de propriétés non standard

La fidélité des fichiers dans une migration peut être définie comme la capacité à :

- Stocker toutes les informations de fichier applicables sur la source.
- Transférer des fichiers avec l'outil de migration.
- Stocker des fichiers dans le stockage cible de la migration.

Pour garantir une migration sans accroc, identifiez [le meilleur outil de copie selon vos besoins](#migration-toolbox) et associez une cible de stockage à votre source.

En tenant compte des informations précédentes, vous pouvez voir que le stockage cible pour les fichiers à usage général dans Azure est [partages de fichiers Azure](storage-files-introduction.md).

Contrairement au stockage d’objets dans les objets blob Azure, un partage de fichiers Azure peut stocker en mode natif des métadonnées de fichiers. Les partages de fichiers Azure préservent également la hiérarchie des fichiers et des dossiers, les attributs et les autorisations. Les autorisations NTFS peuvent être stockées sur des fichiers et des dossiers, car ils sont locaux.

Un utilisateur d’Active Directory, qui est son contrôleur de domaine local, peut accéder en mode natif à un partage de fichiers Azure. C’est également le cas pour un utilisateur de Microsoft Azure Active Directory Domain Services (Azure AD DS). Chacun utilise son identité actuelle pour y accéder en fonction des autorisations du partage et des listes de contrôle d’accès des fichiers et des dossiers. Ce comportement est similaire à celui d’un utilisateur qui se connecte à un partage de fichiers local.

L’autre flux de données est l’aspect principal de la fidélité des fichiers qui ne peut actuellement pas être stocké sur un fichier dans un partage de fichiers Azure. Il est conservé localement quand Azure File Sync est utilisé.

En savoir plus sur l’[authentification Azure AD](storage-files-identity-auth-active-directory-enable.md) et l’[authentification Azure AD DS](storage-files-identity-auth-active-directory-domain-service-enable.md) pour les partages de fichiers Azure.

## <a name="migration-guides"></a>Guides de migration

Le tableau suivant répertorie les guides de migration détaillés.

Comment utiliser la table :

1. Recherchez la ligne du système source sur lequel vos fichiers se trouvent actuellement.

1. Choisissez l’une de ces cibles :

   - Un déploiement hybride utilisant Azure File Sync pour mettre en cache le contenu des partages de fichiers Azure locaux
   - Partages de fichiers Azure dans le cloud

   Sélectionnez la colonne cible qui correspond à votre choix.

1. À l’intersection de la source et de la cible, la cellule répertorie les scénarios de migration disponibles. Sélectionnez-en un pour accéder directement au guide de migration détaillé.

Les scénarios sans lien n’ont pas encore de guide de migration. Consultez régulièrement cette table pour découvrir les mises à jour. Les nouveaux guides seront publiés quand ils seront disponibles.

| Source | Cible : </br>Déploiement hybride | Cible : </br>Déploiement cloud uniquement |
|:---|:--|:--|
| | Combinaison d’outils :| Combinaison d’outils : |
| Windows Server 2012 R2 et versions ultérieures | <ul><li>[Azure File Sync](storage-sync-files-deployment-guide.md)</li><li>[Azure File Sync et Azure Data Box](storage-sync-offline-data-transfer.md)</li><li>[Azure File Sync et les fichiers pré-amorcés dans le cloud](storage-sync-offline-data-transfer.md#azure-file-sync-and-pre-seeded-files-in-the-cloud)</li><li>Azure File Sync et service de migration de stockage</li></ul> | <ul><li>Azure File Sync</li><li>Azure File Sync et Data Box</li><li>Azure File Sync et service de migration de stockage</li><li>Robocopy</li></ul> |
| Windows Server 2012 et versions antérieures | <ul><li>Azure File Sync et Data Box</li><li>Azure File Sync et service de migration de stockage</li></ul> | <ul><li>Azure File Sync et service de migration de stockage</li><li>Robocopy</li></ul> |
| NAS (Network-attached storage) | <ul><li>[Azure File Sync et RoboCopy](storage-files-migration-nas-hybrid.md)</li></ul> | <ul><li>Robocopy</li></ul> |
| Linux ou Samba | <ul><li>[Azure File Sync et RoboCopy](storage-files-migration-linux-hybrid.md)</li></ul> | <ul><li>Robocopy</li></ul> |
| Microsoft Azure StorSimple Cloud Appliance 8100 ou StorSimple Cloud Appliance 8600 | <ul><li>[Azure File Sync et StorSimple Cloud Appliance 8020](storage-files-migration-storsimple-8000.md)</li></ul> | |
| StorSimple Cloud Appliance 1200 | <ul><li>[Azure File Sync](storage-files-migration-storsimple-1200.md)</li></ul> | |
| | | |

## <a name="migration-toolbox"></a>Boîte à outils de migration

### <a name="file-copy-tools"></a>Outils de copie de fichiers

Il existe plusieurs outils de copie de fichiers disponibles auprès de Microsoft et d’autres. Pour sélectionner le bon outil pour votre scénario de migration, vous devez tenir compte de ces questions fondamentales :

* L’outil prend-il en charge l’emplacement source et l’emplacement cible pour votre copie de fichier ?

* L’outil prend-il en charge le chemin de votre réseau et/ou les protocoles disponibles (par exemple REST, SMB ou NFS) entre les emplacements de stockage source et cible ?

* L’outil préserve-t-il la fidélité de fichier nécessaire prise en charge par vos emplacements sources/cibles ?

    Dans certains cas, votre stockage cible ne prend pas en charge le même niveau de fidélité que votre source. Si le stockage cible est suffisant pour répondre à vos besoins, l’outil doit correspondre uniquement aux fonctionnalités de fidélité de fichier de la cible.

* L’outil dispose-t-il de fonctionnalités permettant de l’adapter à votre stratégie de migration ?

    Par exemple, déterminez si l’outil vous permet de réduire vos temps d’arrêt.
    
    Quand un outil prend en charge une option pour mettre en miroir une source sur une cible, vous pouvez souvent l’exécuter plusieurs fois sur la même source et la même cible, tandis que la source reste accessible.

    La première fois que vous exécutez l’outil, il copie le bloc de données. Cette exécution initiale peut prendre un certain temps. Vous trouvez souvent que la mise hors connexion de la source de données pour vos processus métier dure trop longtemps.

    En mettant en miroir une source sur une cible (comme avec **robocopy /MIR**), vous pouvez réexécuter l’outil sur la même source et la même cible. L’exécution est beaucoup plus rapide, car seules les modifications de la source après l’exécution précédente doivent être transportées. Réexécuter un outil de copie de cette façon peut réduire considérablement les temps d’arrêt.

Le tableau suivant classe les outils Microsoft et leurs aptitudes actuelles pour les partages de fichiers Azure :

| Recommandé | Outil | Prise en charge des partages de fichiers Azure | Préservation de la fidélité des fichiers |
| :-: | :-- | :---- | :---- |
|![Oui, recommandé](media/storage-files-migration-overview/circle-green-checkmark.png)| Robocopy | Pris en charge. Les partages de fichiers Azure peuvent être montés comme lecteurs réseau. | Fidélité totale.* |
|![Oui, recommandé](media/storage-files-migration-overview/circle-green-checkmark.png)| Azure File Sync | Intégré en natif dans les partages de fichiers Azure. | Fidélité totale.* |
|![Oui, recommandé](media/storage-files-migration-overview/circle-green-checkmark.png)| Service de migration de stockage | Pris en charge de façon indirecte. Les partages de fichiers Azure peuvent être montés comme lecteurs réseau sur des serveurs cibles SMS. | Fidélité totale.* |
|![Oui, recommandé](media/storage-files-migration-overview/circle-green-checkmark.png)| AzCopy, version 10.4 ou ultérieure| Pris en charge. | Fidélité totale.* |
|![Oui, recommandé](media/storage-files-migration-overview/circle-green-checkmark.png)| Data Box | Pris en charge. | Le service Data Box prend désormais entièrement en charge les métadonnées. [Le service Data Box peut être utilisé en association avec Azure File Sync](storage-sync-offline-data-transfer.md). |
|![Pas vraiment recommandé](media/storage-files-migration-overview/triangle-yellow-exclamation.png)| Explorateur Stockage Azure, version 1.14 | Pris en charge. | Ne copie pas les ACL. Prend en charge les timestamps.  |
|![Non recommandé](media/storage-files-migration-overview/circle-red-x.png)| Azure Data Factory | Pris en charge. | Ne copie pas les métadonnées. |
|||||

*\* Fidélité optimale : respecte ou dépasse les fonctionnalités du partage de fichiers Azure.*

### <a name="migration-helper-tools"></a>Outils d’assistance à la migration

Cette section décrit les outils qui vous aident à planifier et à exécuter des migrations.

#### <a name="robocopy-from-microsoft-corporation"></a>RoboCopy, de Microsoft Corporation

RoboCopy est l’un des outils les mieux applicables aux migrations de fichiers. Il est intégré à Windows. La [documentation RoboCopy](/windows-server/administration/windows-commands/robocopy) principale est une ressource utile pour les nombreuses options de cet outil.

#### <a name="treesize-from-jam-software-gmbh"></a>TreeSize, de JAM Software GmbH

Azure File Sync est principalement mis à l’échelle avec le nombre d’éléments (fichiers et dossiers) et non avec la quantité totale de stockage. L’outil TreeSize vous permet de déterminer le nombre d’éléments sur vos volumes Windows Server.

Vous pouvez utiliser l’outil pour créer une perspective avant un [déploiement Azure File Sync](storage-sync-files-deployment-guide.md). Vous pouvez également l’utiliser lorsque la hiérarchisation dans le cloud est engagée après le déploiement. Dans ce scénario, vous voyez le nombre d’éléments et les répertoires qui utilisent le plus le cache de votre serveur.

La version testée de l’outil est la version 4.4.1. Elle est compatible avec les fichiers hiérarchisés dans le cloud. L’outil ne provoque pas le rappel des fichiers hiérarchisés pendant son fonctionnement normal.

## <a name="next-steps"></a>Étapes suivantes

1. Créez un plan pour le déploiement des partages de fichiers Azure (cloud uniquement ou hybrides) souhaités.
1. Passez en revue la liste des guides de migration disponibles pour trouver le guide détaillé qui correspond à votre source et votre déploiement de partages de fichiers Azure.

Voici des informations supplémentaires sur les technologies Azure Files mentionnées dans cet article :

* [Présentation des partages de fichiers Azure](storage-files-introduction.md)
* [Planification d’un déploiement de synchronisation de fichiers Azure](storage-sync-files-planning.md)
* [Azure File Sync : hiérarchisation cloud](storage-sync-cloud-tiering.md)