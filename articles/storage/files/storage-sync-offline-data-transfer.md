---
title: Migrer des données vers Azure File Sync avec Azure Data Box
description: Migrez des données en bloc hors connexion d’une manière compatible avec Azure File Sync. Évitez les conflits de fichiers et conservez les listes de contrôle d’accès et l’horodatage des fichiers et des dossiers après avoir activé la synchronisation.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 02/12/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 94abb33d39765a19306a013576d43fb2602d1c37
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017625"
---
# <a name="migrate-bulk-data-to-azure-file-sync-with-azure-databox"></a>Migrer des données en bloc vers Azure File Sync avec Azure Data Box
Vous pouvez migrer des données en bloc vers Azure File Sync de deux manières :

* **Chargez vos fichiers à l’aide d’Azure File Sync.** Il s’agit de la méthode la plus simple. Déplacez vos fichiers localement vers un serveur Windows Server 2012 R2 ou version ultérieure, et installez l’agent Azure File Sync. Une fois la synchronisation configurée, vos fichiers sont téléchargés à partir du serveur (nos clients bénéficient actuellement d’une vitesse de chargement moyenne de 1 Tio sur tous les deux jours). Pour vous assurer que votre serveur n’utilise pas trop la bande passante de votre centre de données, vous pouvez configurer une [planification de limitation de bande passante](storage-sync-files-server-registration.md#ensuring-azure-file-sync-is-a-good-neighbor-in-your-datacenter).
* **Transférez vos fichiers hors connexion.** Si vous n’avez pas suffisamment de bande passante, vous ne pouvez peut-être pas charger des fichiers sur Azure dans un délai raisonnable. La difficulté ici réside dans la synchronisation initiale de l’ensemble des fichiers. Pour surmonter cette difficulté, utilisez des outils de migration en bloc hors connexion tels ceux de la [gamme Azure Data Box](https://azure.microsoft.com/services/storage/databox). 

Cet article explique comment migrer des fichiers hors connexion d’une manière compatible avec Azure File Sync. Suivez ces instructions pour éviter les conflits de fichiers et conserver vos listes de contrôle d’accès et horodatages des fichiers et dossiers après avoir activé la synchronisation.

## <a name="migration-tools"></a>Outils de migration
Le processus que nous décrivons dans cet article fonctionne non seulement pour les ressources Data Box, mais aussi pour d’autres outils de migration hors connexion. Il fonctionne également pour des outils tels qu’AzCopy, Robocopy, ainsi que pour divers outils et services partenaires fonctionnant sur Internet. Cependant, pour surmonter le défi du chargement initial, suivez les étapes décrites dans cet article pour utiliser ces outils d’une manière compatible avec Azure File Sync.

Dans certains cas, vous devez changer de serveur Windows Server avant d’adopter Azure File Sync. Le [Service de migration de stockage](/windows-server/storage/storage-migration-service/overview) peut être utile à cette fin. Que vous deviez migrer vers une version de système d’exploitation prise en charge par Azure File Sync (Windows Server 2012 R2 ou version ultérieure) ou simplement acheter un nouveau système pour Azure File Sync, le Service de migration de stockage offre de nombreuses fonctionnalités et avantages pour vous aider à opérer votre migration sans difficulté.

## <a name="benefits-of-using-a-tool-to-transfer-data-offline"></a>Avantages du recours à un outil pour transférer des données hors connexion
Voici les principaux avantages offerts par un outil de transfert tels qu’Azure Data Box pour une migration hors connexion :

- Vous n’êtes pas obligé de charger tous vos fichiers sur le réseau. Pour des espaces de noms de grande taille, cet outil peut vous aider à économiser un temps et une bande passante considérables.
- Lorsque vous utilisez Azure File Sync, quel que soit l’outil de transfert choisi (Azure Data Box, service Azure Import/Export ou autre), votre serveur en ligne charge uniquement les fichiers modifiés après la migration des données vers Azure.
- Azure File Sync synchronise vos listes de contrôle d’accès des fichiers et dossiers même si l’outil de migration en bloc hors connexion ne transporte pas de liste de contrôle d’accès.
- Data Box et Azure File Sync ne nécessitent aucun arrêt. Lorsque vous vous servez de Data Box pour transférer des données dans Azure, vous utilisez la bande passante réseau efficacement et préservez la fidélité du fichier. Vous tenez également votre espace de noms à jour en téléchargeant uniquement les fichiers modifiés après que vous avez déplacé les données vers Azure.

## <a name="prerequisites-for-the-offline-data-transfer"></a>Conditions préalables au transfert de données hors connexion
Vous ne devez pas activer la synchronisation sur le serveur que vous migrez avant d’avoir accompli votre transfert de données hors connexion. Autres aspects à prendre en considération avant de commencer :

- Si vous envisagez d’utiliser Data Box pour votre migration en bloc : Passez en revue les [prérequis de déploiement pour Data Box](../../databox/data-box-deploy-ordered.md#prerequisites).
- Planifiez votre topologie Azure File Sync finale : [Planifier un déploiement de la fonctionnalité Synchronisation de fichiers Azure](storage-sync-files-planning.md)
- Sélectionnez le ou les comptes de stockage Azure destinés à contenir les partages de fichiers avec lesquels vous souhaitez vous synchroniser. Veillez à effectuer votre migration en bloc dans des partages intermédiaires temporaires situés dans les mêmes comptes de stockage. La migration en bloc ne peut être activée qu’en utilisant un partage final (et une partage intermédiaire) résidant dans le même compte de stockage.
- Une migration en bloc ne peut être utilisée que si vous créez une nouvelle relation de synchronisation avec un emplacement serveur. Vous ne pouvez pas activer une migration en bloc avec une relation de synchronisation existante.


## <a name="process-for-offline-data-transfer"></a>Processus de transfert de données hors connexion
Voici comment configurer Azure File Sync d’une manière compatible avec des outils de migration en bloc tels qu’Azure Data Box :

![Diagramme illustrant la configuration d’Azure File Sync](media/storage-sync-files-offline-data-transfer/data-box-integration-1-600.png)

| Étape | Detail |
|---|---------------------------------------------------------------------------------------|
| ![Étape 1](media/storage-sync-files-offline-data-transfer/bullet_1.png) | [Commandez votre Data Box](../../databox/data-box-deploy-ordered.md). La gamme Data Box propose [plusieurs produits](https://azure.microsoft.com/services/storage/databox/data) répondant vos besoins. Lorsque vous recevez votre Data Box, suivez sa [documentation pour copier vos données](../../databox/data-box-deploy-copy-data.md#copy-data-to-data-box) vers ce chemin d’accès UNC sur la Data Box : *\\<DeviceIPAddres\>\<StorageAccountName_AzFile\>\<ShareName\>* . Ici, *ShareName* est le nom du partage intermédiaire. Renvoyez la Data Box à Azure. |
| ![Étape 2](media/storage-sync-files-offline-data-transfer/bullet_2.png) | Attendez que vos fichiers s’affichent dans les partages de fichiers Azure que vous avez choisis comme partages intermédiaires temporaires. *N’activez pas la synchronisation sur ces partages.* |
| ![Étape 3 :](media/storage-sync-files-offline-data-transfer/bullet_3.png) | <ul><li>Créez un partage vide pour chaque partage de fichiers que Data Box a créé pour vous. Ce nouveau partage doit se trouver dans le même compte de stockage que le partage Data Box. [Guide pratique pour créer un partage de fichiers Azure](storage-how-to-create-file-share.md).</li><li>[Créez un groupe de synchronisation](storage-sync-files-deployment-guide.md#create-a-sync-group-and-a-cloud-endpoint) dans un service de synchronisation de stockage. Référencez le partage vide en tant que point de terminaison cloud. Répétez cette étape pour chaque partage de fichiers Data Box. [Configurez Azure File Sync](storage-sync-files-deployment-guide.md).</li></ul> |
| ![Étape 4](media/storage-sync-files-offline-data-transfer/bullet_4.png) | [Ajoutez le répertoire de votre serveur en ligne comme point de terminaison de serveur](storage-sync-files-deployment-guide.md#create-a-server-endpoint). Au cours du processus, spécifiez que vous avez déjà déplacé les fichiers vers Azure et référencez les partages intermédiaires. Vous pouvez activer ou désactiver la hiérarchisation cloud en fonction des besoins. Lors de la création d’un point de terminaison de serveur sur votre serveur en ligne, référencez le partage intermédiaire. Dans le panneau **Ajouter un point de terminaison de serveur**, sous **Transfert de données hors connexion**, sélectionnez **Activé**, puis le partage intermédiaire qui doit se trouver dans le même compte de stockage que le point de terminaison cloud. Ici, la liste des partages disponibles est filtrée par compte de stockage et partages non encore synchronisés. La capture d’écran qui suit ce tableau montre comment référencer le partage DataBox lors de la création du point de terminaison de serveur dans le portail Azure. |
| ![Étape 5](media/storage-sync-files-offline-data-transfer/bullet_5.png) | Une fois le point de terminaison de serveur ajouté à l’étape précédente, les données commencent automatiquement à circuler depuis la source qui convient. La section [Synchronisation du partage](#syncing-the-share) explique quand les données circulent depuis le partage DataBox ou Windows Server. |
| |

![Capture d’écran de l’interface utilisateur du portail Azure, montrant comment activer le transfert de données hors connexion lors de la création d’un point de terminaison de serveur](media/storage-sync-files-offline-data-transfer/data-box-integration-2-600.png)

## <a name="syncing-the-share"></a>Synchronisation du partage
Une fois le point de terminaison de serveur créé, la synchronisation démarre. Le processus de synchronisation détermine si chaque fichier sur le serveur existe également dans le partage intermédiaire où la Data Box a déposé les fichiers. Si le fichier existe à cet emplacement, le processus de synchronisation le copie à partir du partage intermédiaire au lieu de le charger à partir du serveur. Si le fichier ne figure pas dans le partage intermédiaire ou si une version plus récente est disponible sur le serveur local, le processus de synchronisation charge le fichier à partir du serveur local.

Lors de la synchronisation du partage, la synchronisation fusionne les attributs de fichiers manquants, les autorisations ou les timestamps à partir des variantes de fichiers sur le serveur local, en les associant à leurs fichiers équivalents dans le partage DataBox. Ainsi, chaque fichier ou dossier se présente le plus fidèlement possible dans le partage de fichiers Azure.

> [!IMPORTANT]
> Vous pouvez activer le mode de migration en bloc uniquement lorsque vous créez un point de terminaison de serveur. Après avoir établi un point de terminaison de serveur, vous ne pouvez pas intégrer de données migrées en bloc à partir d’un serveur qui synchronise déjà dans l’espace de noms.

## <a name="acls-and-timestamps-on-files-and-folders"></a>Listes ACL et horodatages des fichiers et dossiers
Azure File Sync veille à ce que les listes de contrôle d’accès des fichiers et dossiers soient synchronisées à partir du serveur en ligne, même si l’outil de migration en bloc utilisé n’a pas initialement transporté ces listes. Le partage intermédiaire n’a donc pas besoin de contenir de listes de contrôle d’accès pour les fichiers et dossiers. Si vous activez la fonctionnalité de migration de données hors connexion lorsque vous créez un point de terminaison de serveur, toutes les listes de contrôle d’accès des fichiers sont synchronisées sur le serveur. Les horodatages récemment créés et modifiés sont également synchronisés.

## <a name="shape-of-the-namespace"></a>Forme de l’espace de noms
Lorsque vous activez la synchronisation, le contenu du serveur détermine la forme de l’espace de noms. Si des fichiers sont supprimés du serveur local après la capture instantanée et la migration de la Data Box, ces fichiers ne sont pas déplacés vers l’espace de noms en ligne en cours de synchronisation. Ils restent dans le partage intermédiaire mais ne sont pas copiés. Cela est nécessaire parce que la synchronisation conserve l’espace de noms conformément au serveur en ligne. La *capture instantanée* de la Data Box est simplement une base intermédiaire pour une copie de fichier efficace. Elle n’a aucune incidence sur la forme de l’espace de noms en ligne.

## <a name="cleaning-up-after-bulk-migration"></a>Nettoyage après la migration d’en bloc 
Lorsque le serveur achève sa synchronisation initiale de l’espace de noms, les fichiers migrés en bloc de la Data Box utilisent le partage de fichiers intermédiaire. Dans la section **Transfert de données hors connexion** du panneau **Propriétés du point de terminaison de serveur** sur le portail Azure, l’état passe de **En cours** à **Terminé**. 

![Capture d’écran du panneau Propriétés du point de terminaison de serveur dans lequel se trouvent les contrôles d’état et de désactivation pour le transfert de données hors connexion](media/storage-sync-files-offline-data-transfer/data-box-integration-3-444.png)

À ce stade, vous pouvez nettoyer le partage intermédiaire pour réduire les coûts :

1. Dans le panneau **Propriétés du point de terminaison de serveur**, lorsque l’état est **Terminé**, sélectionnez **Désactiver le transfert de données hors connexion**.
2. Songez à supprimer le partage intermédiaire pour réduire les coûts. Le partage intermédiaire ne contenant probablement pas de liste de contrôle d’accès de fichier et dossier, il n’est probablement pas très utile. Pour des besoins de sauvegarde ponctuelle, créez une [capture instantanée réelle du partage de fichiers Azure en cours de synchronisation](storage-snapshots-files.md). Vous pouvez [configurer Sauvegarde Azure pour prendre des captures instantanées]( ../../backup/backup-afs.md) selon une planification.

Désactivez le mode de transfert de données hors connexion uniquement lorsque l’état est **Terminé** ou lorsque vous souhaitez l’annuler en raison d’une configuration incorrecte. Si vous désactivez le mode au cours d’un déploiement, le chargement des fichiers à partir du serveur commence, même si votre partage intermédiaire est toujours disponible.

> [!IMPORTANT]
> Une fois le transfert de données hors connexion désactivé, il n’existe aucun moyen de l’activer, même si le partage intermédiaire de la migration en bloc est toujours disponible.

## <a name="azure-file-sync-and-pre-seeded-files-in-the-cloud"></a>Azure File Sync et les fichiers amorcés dans le cloud

Si vous avez amorcé des fichiers dans un partage de fichiers Azure par d’autres moyens que DataBox, par exemple via AzCopy, RoboCopy à partir d’une sauvegarde cloud ou toute autre méthode, vous devez toujours suivre le [processus de transfert de données hors connexion](#process-for-offline-data-transfer) décrit dans cet article. Il vous suffit d’ignorer DataBox comme méthode de transfert de vos fichiers vers le cloud. Toutefois, il est primordial de s’assurer que vous suivez toujours le processus d’amorçage des fichiers dans un *partage de fichiers intermédiaire* et non le partage de fichiers connecté Azure File Sync final.

> [!WARNING]
> **Suivez le processus d’amorçage des fichiers dans un partage intermédiaire et non le partage de fichiers connecté Azure File Sync final**. Si vous ne le faites pas, des conflits de fichiers peuvent survenir (les deux versions de fichiers seront stockées) et les fichiers supprimés sur le serveur actif peuvent revenir, s’ils existent encore dans votre ancien ensemble de fichiers. En outre, les modifications de dossiers sont fusionnées les unes avec les autres, ce qui rend très difficile la distinction entre les espaces de noms après une telle erreur.

## <a name="next-steps"></a>Étapes suivantes
- [Planifier un déploiement de la fonctionnalité Synchronisation de fichiers Azure](storage-sync-files-planning.md)
- [Déployer Azure File Sync](storage-sync-files-deployment-guide.md)