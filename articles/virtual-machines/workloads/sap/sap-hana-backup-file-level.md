---
title: Sauvegarde SAP HANA sur Azure au niveau fichier | Microsoft Docs
description: Il existe deux grandes méthodes permettant d’effectuer des sauvegardes SAP HANA sur des machines virtuelles Azure. Cet article aborde la sauvegarde SAP HANA sur Azure au niveau fichier.
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ums.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/01/2020
ms.author: juergent
ms.openlocfilehash: 70b0f8178a94735a6ef37a225044984508cc2233
ms.sourcegitcommit: 86acfdc2020e44d121d498f0b1013c4c3903d3f3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97617134"
---
# <a name="sap-hana-azure-backup-on-file-level"></a>Sauvegarde SAP HANA sur Azure au niveau fichier

## <a name="introduction"></a>Introduction

Cet article est associé au [Guide de sauvegarde pour SAP HANA sur des machines virtuelles Azure](./sap-hana-backup-guide.md), qui offre une vue d’ensemble permettant de bien démarrer avec cette sauvegarde et fournit des informations détaillées sur le service Sauvegarde Azure et les captures instantanées de stockage. 

Différents types de machines virtuelles dans Azure autorisent un nombre différent de disques durs virtuels attachés. Les détails exacts sont documentés dans [Tailles des machines virtuelles Linux dans Azure](../../sizes.md). Pour les tests indiqués dans cette documentation, nous avons utilisé une machine virtuelle Azure GS5 qui permet d’attacher 64 disques de données. Pour les grands systèmes SAP HANA, un nombre important de disques peut déjà être utilisé pour les données et les fichiers journaux, éventuellement en association avec un entrelacement logiciel pour un débit d’E/S de disque optimal. Pour plus d’informations sur les configurations de disque suggérées pour les déploiements de SAP HANA sur les machines virtuelles Azure, lisez l’article [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md). Les recommandations préconisées incluent notamment les recommandations d’espace disque pour les sauvegardes locales.

La méthode standard pour gérer la sauvegarde/restauration au niveau fichier est d’utiliser une sauvegarde basée sur les fichiers via SAP HANA Studio ou via les instructions SQL SAP HANA. Pour plus d’informations, consultez le [document de référence sur SQL SAP HANA et les vues système](https://help.sap.com/viewer/4fe29514fd584807ac9f2a04f6754767/2.0.05/en-US/3859e48180bb4cf8a207e15cf25a7e57.html).

![Cette illustration montre la boîte de dialogue de l’élément du menu de sauvegarde dans SAP HANA Studio](media/sap-hana-backup-file-level/backup-menue-dialog.png)

Cette illustration montre la boîte de dialogue de l’élément du menu de sauvegarde dans SAP HANA Studio. Lorsque vous choisissez le type &quot;fichier&quot;, vous devez indiquer le chemin d’accès de l’emplacement dans le système de fichiers où SAP HANA inscrit les fichiers de sauvegarde. La restauration fonctionne de la même façon.

Bien que ce choix semble simple et direct, certains points doivent être pris en compte. Les machines virtuelles Azure ont une limite en termes de nombre de disques de données pouvant être attachés. Les systèmes de fichiers de la machine virtuelle ne disposent peut-être pas de la capacité nécessaire pour stocker les fichiers de sauvegarde SAP HANA, en fonction de la taille de la base de données et des contraintes de débit du disque. Cela peut nécessiter que un entrelacement logiciel sur plusieurs disques de données. Plus loin dans cet article, nous aborderons différentes options permettant de déplacer ces fichiers de sauvegarde et de gérer les restrictions concernant la taille des fichiers et les performances lors du traitement de plusieurs téraoctets de données.

Une autre possibilité, qui offre davantage de liberté en ce qui concerne la capacité totale, est le stockage Blob Azure. Bien que les objets blob soient également limités à 1 To, la capacité totale d’un conteneur d’objets blob unique est actuellement de 500 To. En outre, cela offre également la possibilité aux clients de sélectionner ce que l’on appelle le stockage Blob &quot;froid&quot;, qui représente un avantage en termes de coût. Pour plus d’informations sur le stockage Blob froid, consultez [Stockage Blob Azure : niveaux d’accès chaud, froid et archive](../../../storage/blobs/storage-blob-storage-tiers.md?tabs=azure-portal).

Pour plus de sécurité, utilisez un compte de stockage géo-répliqué pour stocker les sauvegardes SAP HANA. Pour plus d’informations sur la redondance et la réplication du stockage, consultez [Redondance de Stockage Azure](../../../storage/common/storage-redundancy.md).

Vous pouvez placer des disques durs virtuels dédiés aux sauvegardes SAP HANA dans un compte de stockage de sauvegarde dédié géo-répliqué. Vous pouvez également copier les disques durs virtuels sur lesquels sont enregistrées les sauvegardes SAP HANA vers un compte de stockage géo-répliqué, ou vers un compte de stockage qui se trouve dans une autre région.

## <a name="azure-blobxfer-utility-details"></a>Détails de l’utilitaire blobxfer Azure

Pour stocker les répertoires et les fichiers sur le stockage Azure, vous pouvez utiliser l’interface de ligne de commande ou PowerShell, ou encore développer un outil à l’aide d’un des [kits de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/). Il existe également un utilitaire prêt à l’emploi, AzCopy, pour la copie de données dans Stockage Azure. (consultez l’article [Transfert de données avec l’utilitaire de ligne de commande AzCopy](../../../storage/common/storage-use-azcopy-v10.md).)

Par conséquent, blobxfer a été utilisé pour copier les fichiers de sauvegarde SAP HANA. Il s’agit d’un outil Open Source utilisé par de nombreux clients dans leur environnement de production. Il est disponible sur [GitHub](https://github.com/Azure/blobxfer). Cet outil permet de copier des données directement sur le stockage Blob Azure ou sur le système de partage de fichiers Azure. Il inclut également de nombreuses fonctionnalités utiles telles que le code de hachage md5 ou le parallélisme automatique lors de la copie d’un répertoire contenant plusieurs fichiers.

## <a name="sap-hana-backup-performance"></a>Performances de sauvegarde SAP HANA
Ce chapitre aborde les aspects liés aux performances. Il se peut que les chiffres atteints ne représentent pas l’état actuel de la technologie. En effet, le débit offert par le stockage Azure fait l’objet d’un développement constant et s’améliore régulièrement. Vous devez donc effectuer des tests individuels pour votre configuration et votre région Azure.

![Cette capture d’écran provient de la console de sauvegarde SAP HANA dans SAP HANA Studio](media/sap-hana-backup-file-level/backup-console-hana-studio.png)

Cette capture d’écran montre la console de sauvegarde SAP HANA de SAP HANA Studio. Environ 42 minutes ont été nécessaires pour effectuer une sauvegarde de 230 Go sur un seul disque de stockage HDD Standard Azure attaché à la machine virtuelle HANA avec le système de fichiers XFS.

![Cette capture d’écran provient de YaST sur la machine virtuelle de test SAP HANA](media/sap-hana-backup-file-level/one-backup-disk.png)

Cette capture d’écran provient de YaST sur la machine virtuelle de test SAP HANA. Vous pouvez voir l’unique disque de 1 To utilisé pour la sauvegarde SAP HANA. Il a fallu environ 42 minutes pour sauvegarder 230 Go. En outre, cinq disques de 200 Go ont été liés et le RAID logiciel md0 a été créé, avec un entrelacement sur ces cinq disques de données Azure.

![Répétition de la même sauvegarde sur le RAID logiciel avec entrelacement sur cinq disques de données de stockage Azure standard liés](media/sap-hana-backup-file-level/five-backup-disks.png)

La répétition de la même sauvegarde sur le RAID logiciel avec entrelacement sur cinq disques de données de stockage Azure standard liés a fait passer le temps nécessaire à la sauvegarde de 42 minutes à 10 minutes. Les disques ont été liés sans mise en cache sur la machine virtuelle. Cet exercice démontre l’importance du débit d’écriture sur le disque pour obtenir un temps de sauvegarde correct. Vous pouvez passer au stockage SSD Standard Azure ou au Stockage Premium Azure pour accélérer davantage le processus et ainsi bénéficier de performances optimales. Le stockage HDD Standard Azure n’est généralement pas recommandé et n’a été utilisé qu’à des fins de démonstration. Il est recommandé d’utiliser au minimum un stockage SSD Standard Azure ou un Stockage Premium Azure pour les systèmes de production.

## <a name="copy-sap-hana-backup-files-to-azure-blob-storage"></a>Copier les fichiers de sauvegarde SAP HANA vers le stockage Blob Azure
Il se peut que les chiffres mentionnés quant aux performances, à la durée des sauvegardes et à la durée des copies ne représentent pas l’état actuel de la technologie Azure. Microsoft améliore régulièrement le stockage Azure pour offrir un débit supérieur et des latences inférieures. Ces chiffres sont donc fournis à des fins de démonstration uniquement. Vous devez effectuer des tests correspondant à vos propres besoins dans la région Azure de votre choix pour pouvoir déterminer la méthode qui vous convient le mieux.

Une autre option pour stocker rapidement les fichiers de sauvegarde SAP HANA est le stockage Blob Azure. Un conteneur d’objets blob unique a une limite d’environ 500 To, suffisante pour les systèmes SAP HANA utilisant les types de machine virtuelle M32ts, M32ls, M64ls et GS5 d’Azure, pour conserver les sauvegardes SAP HANA suffisantes. Les clients ont le choix entre le stockage Blob &quot;chaud&quot; et &quot;froid&quot; (voir [Stockage Blob Azure : niveaux d’accès chaud, froid et archive](../../../storage/blobs/storage-blob-storage-tiers.md?tabs=azure-portal)).

Grâce à l’outil blobxfer, il est facile de copier les fichiers de sauvegarde SAP HANA directement dans le stockage Blob Azure.

![Vous pouvez voir ici les fichiers d’une sauvegarde de fichiers SAP HANA complète](media/sap-hana-backup-file-level/list-of-backups.png)

Vous pouvez voir les fichiers d’une sauvegarde de fichiers SAP HANA complète. La taille du plus gros des quatre fichiers est d’environ 230 Go.

![Il a fallu environ 3 000 secondes pour copier les 230 Go dans un conteneur d’objets blob du compte de stockage Azure standard](media/sap-hana-backup-file-level/copy-duration-blobxfer.png)

En n’utilisant pas le code de hachage md5 lors du premier test, il a fallu environ 3 000 secondes pour copier les 230 Go dans un conteneur d’objets blob du compte de stockage Azure standard.

La console de sauvegarde HANA Studio permet de limiter la taille de fichier maximale des fichiers de sauvegarde HANA. Dans l’environnement d’exemple, elle a amélioré les performances en permettant d’avoir plusieurs fichiers de sauvegarde plus petits au lieu d’un gros fichier de 230 Go.

La définition de la limite de taille de fichier de sauvegarde côté HANA n’améliore pas le temps de sauvegarde, car les fichiers sont écrits de manière séquentielle. La limite de taille de fichier a été définie sur 60 Go, donc la sauvegarde a créé quatre fichiers de données volumineux au lieu d’un seul fichier de 230 Go. L’utilisation de plusieurs fichiers de sauvegarde peut devenir une nécessité pour la sauvegarde de bases de données HANA si vos cibles de sauvegarde sont limitées en termes de taille de fichiers ou de blobs.

![Pour tester le parallélisme de l’outil blobxfer, la taille de fichier maximale pour les sauvegardes HANA a ensuite été définie sur 15 Go](media/sap-hana-backup-file-level/parallel-copy-multiple-backup-files.png)

Pour tester le parallélisme de l’outil blobxfer, la taille de fichier maximale pour les sauvegardes HANA a ensuite été définie sur 15 Go, ce qui a engendré la création de 19 fichiers de sauvegarde. Cette configuration a fait passer le temps nécessaire pour que blobxfer copie les 230 Go sur le stockage Blob Azure de 3 000 secondes à 875 secondes.

Quand vous explorez la copie de sauvegardes effectuées à partir de disques locaux vers d’autres emplacements comme le Stockage Blob Azure, gardez à l’esprit que la bande passante utilisée par un éventuel processus de copie parallèle est prise en compte dans le quota réseau ou le quota de stockage de votre type de machine virtuelle spécifique. Vous devez donc trouver un compromis entre la durée de la copie et la bande passante réseau et de stockage nécessaire à la charge de travail normale s’exécutant dans la machine virtuelle. 


## <a name="copy-sap-hana-backup-files-to-nfs-share"></a>Copier les fichiers de sauvegarde SAP HANA vers le partage NFS

Microsoft Azure offre des partages NFS avec [Azure NetApp Files](https://azure.microsoft.com/services/netapp/). Vous pouvez créer plusieurs volumes d’une capacité d’une douzaine de To pour stocker et gérer les sauvegardes. Vous pouvez également créer des instantanés de ces volumes avec la technologie de NetApp. Azure NetApp Files (ANF) est disponible selon trois niveaux de service offrant différents débits de stockage. Pour en savoir plus, consultez l’article [Niveaux de service pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Vous pouvez créer et monter un volume NFS à partir d’ANF comme décrit dans l’article [Démarrage rapide : Configurer Azure NetApp Files et créer un volume NFS](../../../azure-netapp-files/azure-netapp-files-quickstart-set-up-account-create-volumes.md?tabs=azure-portal).

Au-delà de l’utilisation de volumes NFS natifs d’Azure avec ANF, vous disposez de différentes méthodes pour créer vos propres déploiements fournissant des partages NFS sur Azure. Elles présentent toutes le même inconvénient : vous devez déployer et gérer ces solutions vous-même. Certaines de ces méthodes sont documentées dans les articles suivants :

- [Haute disponibilité pour NFS sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md)
- [GlusterFS sur les machines virtuelles Azure sur Red Hat Enterprise Linux pour SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md)

Les partages NFS créés à l’aide des méthodes décrites dans les articles ci-dessus peuvent être utilisés pour exécuter directement des sauvegardes HANA sur des disques locaux ou pour copier des sauvegardes effectuées sur des disques locaux vers ces partages NFS.

> [!NOTE]
> SAP HANA prend en charge NFS v3 et NFS v4.x. Les autres formats comme SMB avec système de fichiers CIFS ne sont pas pris en charge pour l’écriture de sauvegardes HANA. Consultez également la [note de support SAP n° 1820529](https://launchpad.support.sap.com/#/notes/1820529).

## <a name="copy-sap-hana-backup-files-to-azure-files"></a>Copier les fichiers de sauvegarde SAP HANA dans Azure Files

Il est possible de monter un partage Azure Files dans une machine virtuelle Linux Azure. L’article [Guide pratique pour utiliser le Stockage Fichier Azure avec Linux](../../../storage/files/storage-how-to-use-files-linux.md) fournit des informations détaillées sur la manière d’effectuer la configuration. Pour connaître les limitations relatives aux fichiers Premium Azure et Azure Files, lisez l’article [Objectifs de performance et d’extensibilité d’Azure Files](../../../storage/files/storage-files-scale-targets.md).

> [!NOTE]
> SAP HANA ne prend pas en charge le protocole SMB avec système de fichiers CIFS pour l’écriture de sauvegardes HANA. Consultez également la [note de support SAP n° 1820529](https://launchpad.support.sap.com/#/notes/1820529). Ainsi, vous pouvez utiliser cette solution uniquement comme destination finale d’une sauvegarde de base de données HANA effectuée directement sur des disques locaux attachés.
> 

Dans le cadre d’un test effectué avec Azure Files (et non Azure Premium Files), il a fallu environ 929 secondes pour copier 19 fichiers de sauvegarde d’un volume total de 230 Go. Avec Azure Premium Files, l’opération devrait être beaucoup plus courte. Toutefois, n’oubliez pas que vous devez trouver un compromis entre les avantages d’une copie rapide et les exigences de votre charge de travail en termes de bande passante réseau. Étant donné que chaque type de machine virtuelle Azure applique un quota de bande passante réseau, votre charge de travail et la copie des fichiers de sauvegarde doivent respecter la plage correspondante.

![Cette illustration montre qu’il a fallu environ 929 secondes pour copier 19 fichiers de sauvegarde SAP HANA](media/sap-hana-backup-file-level/parallel-copy-to-azure-files.png)


Le stockage de fichiers de sauvegarde SAP HANA sur Azure Files peut représenter une option intéressante. L’amélioration de la latence et du débit d’Azure Premium Files offrent, effectivement un précieux avantage.

## <a name="next-steps"></a>Étapes suivantes
* L’article [Guide de sauvegarde pour SAP HANA sur les machines virtuelles Azure](sap-hana-backup-guide.md) fournit une vue d’ensemble et des informations sur la mise en route.
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur Azure (grandes instances), consultez [Haute disponibilité et récupération d’urgence de SAP HANA (grandes instances) sur Azure](hana-overview-high-availability-disaster-recovery.md).
