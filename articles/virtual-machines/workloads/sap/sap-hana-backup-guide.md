---
title: Guide de sauvegarde pour SAP HANA sur Machines Virtuelles Azure | Microsoft Docs
description: Le guide de sauvegarde pour SAP HANA couvre deux méthodes de sauvegarde clés pour SAP HANA sur machines virtuelles Azure
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ums.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/01/2020
ms.author: juergent
ms.openlocfilehash: 340ce84ebe662b976f32048a47fd98d793083065
ms.sourcegitcommit: 86acfdc2020e44d121d498f0b1013c4c3903d3f3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97617100"
---
# <a name="backup-guide-for-sap-hana-on-azure-virtual-machines"></a>Guide de sauvegarde pour SAP HANA sur Machines Virtuelles Azure

## <a name="getting-started"></a>Mise en route

Le guide de sauvegarde pour SAP HANA exécuté sur des machines virtuelles Azure aborde uniquement les sujets spécifiques à Azure. Pour obtenir des informations générales sur la sauvegarde SAP HANA, consultez la documentation SAP HANA. Nous attendons de vous que vous connaissiez les principales stratégies de sauvegarde des bases de données, les raisons et les motivations nécessaires pour avoir une stratégie de sauvegarde saine et valide, et que vous connaissiez les exigences de votre entreprise en ce qui concerne la procédure de sauvegarde, la durée de rétention des sauvegardes et la procédure de restauration.

La plateforme SAP HANA est officiellement prise en charge sur différents types de machine virtuelle Azure, comme les machines virtuelles Azure de la série M. Pour obtenir la liste complète des machines virtuelles Azure certifiées SAP HANA et des unités de grande Instance HANA, consultez [Find Certified IaaS Platforms](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) (Rechercher des plateformes IaaS certifiées). Microsoft Azure offre un certain nombre d’unités dans lesquelles SAP HANA s’exécute de manière non virtualisée sur des serveurs physiques. Ce service est appelé [Grandes instances HANA](hana-overview-architecture.md). Ce guide ne traite pas des processus et des outils de sauvegarde pour les grandes instances HANA. Il ne traitera que des machines virtuelles Azure. Pour plus d’informations sur les processus de sauvegarde/restauration avec de grandes instances HANA, lisez l’article [Sauvegarde et restauration des Grandes instances HANA (HLI)](./hana-backup-restore.md).

Cet article couvre trois méthodes de sauvegarde pour SAP HANA sur machines virtuelles Azure :

- Sauvegarde HANA via [le service Sauvegarde Azure](../../../backup/backup-overview.md) 
- Sauvegarde HANA sur le système de fichiers dans une machine virtuelle Azure Linux (voir [Sauvegarde SAP HANA sur Azure au niveau fichier](sap-hana-backup-file-level.md))
- Sauvegarde HANA à partir de captures instantanées de stockage en utilisant la fonctionnalité de capture instantanée du stockage blob Azure manuellement ou le service Sauvegarde Azure


SAP HANA offre une API de sauvegarde qui permet l’intégration directe d’outils de sauvegarde tiers à SAP HANA. Les produits tels que le service Sauvegarde Azure ou [CommVault](https://azure.microsoft.com/resources/protecting-sap-hana-in-azure/) utilisent cette interface propriétaire pour déclencher des sauvegardes de base de données SAP HANA ou de fichiers journaux de phase de restauration par progression. 


Pour plus d’informations sur les logiciels SAP pris en charge sur Azure, consultez l’article [Logiciels SAP dont le déploiement est pris en charge sur Azure](./sap-supported-product-on-azure.md).

## <a name="azure-backup-service"></a>Service Azure Backup

Le premier scénario présenté est un scénario dans lequel soit le service Sauvegarde Azure utilise l’interface SAP HANA `backint` pour effectuer une sauvegarde en continu à partir d’une base de données SAP HANA, soit vous utilisez une capacité plus générique du service Sauvegarde Azure pour créer une capture instantanée de disque cohérente par rapport aux applications et la transférer au service Sauvegarde Azure.

Sauvegarde Azure intègre la solution de sauvegarde pour SAP HANA à l’aide de l’interface SAP HANA propriétaire appelée [backint](https://www.sap.com/dmc/exp/2013_09_adpd/enEN/#/d/solutions?id=8f3fd455-a2d7-4086-aa28-51d8870acaa5) et est certifié comme tel. Pour plus d’informations sur la solution, ses capacités et les régions Azure où elle est disponible, consultez l’article [Matrice de prise en charge pour la sauvegarde des bases de données SAP HANA sur des machines virtuelles Azure](../../../backup/sap-hana-backup-support-matrix.md#scenario-support). Pour plus d’informations et pour connaître les principes relatifs au service Sauvegarde Azure pour HANA, consultez l’article [À propos de la sauvegarde de base de données SAP HANA dans les machines virtuelles Azure](../../../backup/sap-hana-db-about.md). 

La seconde possibilité de tirer parti du service Sauvegarde Azure consiste à créer une sauvegarde cohérente par rapport aux applications à l’aide de captures instantanées de disque de Stockage Premium Azure. D’autres stockages Azure certifiés HANA, comme le [disque Ultra Azure](../../disks-enable-ultra-ssd.md) et [Azure NetApp Files](https://azure.microsoft.com/services/netapp/), ne prennent pas en charge ce type de capture instantanée par le biais du service Sauvegarde Azure. En lisant les articles suivants :

- [Planification de votre infrastructure de sauvegarde de machines virtuelles dans Azure](../../../backup/backup-azure-vms-introduction.md)
- [Sauvegarde cohérente des applications des machines virtuelles Linux Azure](../../../backup/backup-azure-linux-app-consistent.md) 

vous verrez cette séquence d’activités émerger :

- Sauvegarde Azure doit exécuter un script de pré-instantané qui place l’application (SAP HANA dans ce cas-ci) dans un état cohérent.
- Dans la mesure où cet état cohérent est confirmé, Sauvegarde Azure exécute les captures instantanées de disque.
- Une fois les captures instantanées terminées, Sauvegarde Azure annule l’activité qu’il a effectuée dans le script de pré-instantané.
- Une fois l’exécution réussie, Sauvegarde Azure diffuse en continu les données dans le coffre de sauvegarde.

Dans le cas de SAP HANA, la plupart des clients utilisent Accélérateur d’écriture Azure pour les volumes qui contiennent le journal de phase de restauration par progression SAP HANA. Le service Sauvegarde Azure exclut automatiquement ces volumes des captures instantanées. Cette exclusion ne nuit pas à la capacité de restauration de HANA, bien qu’elle bloque la capacité de restauration avec presque tous les autres systèmes de gestion de base de données (SGBD) pris en charge par SAP.

L’inconvénient de cette possibilité est dû au fait que vous devez développer votre propre script de pré-instantané et de post-instantané. Le script de pré-instantané doit créer une capture instantanée HANA et gérer les éventuels cas d’exception, tandis que le script de post-instantané doit à nouveau supprimer la capture instantanée HANA. Pour plus d’informations sur la logique requise, commencez par la [note de support SAP no 2039883](https://launchpad.support.sap.com/#/notes/2039883). Les considérations de la section « Cohérence des données SAP HANA lors de la création de captures instantanées de stockage » de cet article s’appliquent dans leur intégralité à ce type de sauvegarde.

> [!NOTE]
> Les sauvegardes qui sont basées sur des captures instantanées de disque pour SAP HANA dans les déploiements où plusieurs conteneurs de base de données sont utilisés nécessitent HANA 2.0 SP04 en version minimale.
> 

Pour plus d’informations sur les captures instantanées de stockage, voir plus loin dans ce document.

![Ce schéma illustre deux possibilités d’enregistrement de l’état actuel de la machine virtuelle.](media/sap-hana-backup-guide/azure-backup-service-for-hana.png)

## <a name="other-hana-backup-methods"></a>Autres méthodes de sauvegarde HANA
Trois autres méthodes de sauvegarde ou chemins d’accès peuvent être envisagés :

- Sauvegarde sur un partage NFS basé sur Azure NetApp Files (ANF). ANF a à nouveau la possibilité de créer des captures instantanées des volumes sur lesquels vous stockez des sauvegardes. Étant donné le débit dont vous avez finalement besoin pour écrire les sauvegardes, cette méthode peut devenir très coûteuse. Elle reste néanmoins facile à mettre en place puisque HANA peut écrire les sauvegardes directement dans le partage NFS natif Azure
- Exécution de la sauvegarde HANA sur les disques attachés aux machines virtuelles de SSD Standard ou Stockage Premium Azure. À l’étape suivante, vous pouvez copier ces fichiers de sauvegarde sur Stockage Blob Azure. Cette stratégie peut être intéressante en matière de prix.
- Exécution de la sauvegarde HANA sur les disques attachés aux machines virtuelles de SSD Standard ou Stockage Premium Azure. L’étape suivante consiste à effectuer régulièrement des captures instantanées du disque. Après la première capture instantanée, vous pouvez utiliser des instantanés incrémentiels pour réduire les coûts.

![Ce schéma illustre les possibilités de sauvegarde de fichiers SAP HANA dans la machine virtuelle.](media/sap-hana-backup-guide/other-hana-backup-paths.png)

Ce schéma illustre les possibilités de sauvegarde de fichiers SAP HANA dans la machine virtuelle et de stockage des fichiers de sauvegarde HANA à un autre emplacement à l’aide de différents outils. Toutefois, toutes les solutions qui n’impliquent pas un service de sauvegarde tiers ou le service Sauvegarde Azure ont plusieurs contraintes en commun. Certaines d’entre elles peuvent être répertoriées, telles que l’administration de rétention, le processus de restauration automatique et la récupération automatique jusqu’à une date et heure dans le cadre du service Sauvegarde Azure ou d’autres suites et services de sauvegarde tiers spécialisés. Un grand nombre de ces services tiers peuvent être exécutés sur Azure. 


## <a name="sap-resources-for-hana-backup"></a>Ressources SAP pour la sauvegarde HANA

### <a name="sap-hana-backup-documentation"></a>Documentation sur la sauvegarde SAP HANA

- [Introduction to SAP HANA Administration](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.00/en-US) (Introduction à l’administration de HANA SAP)
- [Planning Your Backup and Recovery Strategy](https://help.sap.com/saphelp_hanaplatform/helpdata/en/ef/085cd5949c40b788bba8fd3c65743e/content.htm) (Planification de votre stratégie de sauvegarde et de récupération)
- [Schedule HANA Backup using ABAP DBACOCKPIT](https://www.hanatutorials.com/p/schedule-hana-backup-using-abap.html) (Planifier la sauvegarde HANA avec ABAP DBACOCKPIT)
- [Schedule Data Backups (SAP HANA Cockpit)](https://help.sap.com/saphelp_hanaplatform/helpdata/en/6d/385fa14ef64a6bab2c97a3d3e40292/frameset.htm) (Planifier des sauvegardes de données (cockpit SAP HANA))
- FAQ sur la sauvegarde SAP HANA dans la [note SAP 1642148](https://launchpad.support.sap.com/#/notes/1642148)
- FAQ sur les captures instantanées de base de données et de stockage SAP HANA dans la [note SAP 2039883](https://launchpad.support.sap.com/#/notes/2039883)
- Systèmes de fichiers en réseau inappropriés pour la sauvegarde et la récupération dans la [note SAP 1820529](https://launchpad.support.sap.com/#/notes/1820529)

### <a name="how-to-verify-correctness-of-sap-hana-backup"></a>Comment vérifier qu’une sauvegarde SAP HANA est correcte
Indépendamment de votre méthode de sauvegarde, l’exécution d’un test de restauration sur un autre système est une nécessité absolue. Cette approche permet de s’assurer qu’une sauvegarde est correcte et que les processus internes de sauvegarde et de restauration fonctionnent comme prévu. Bien que la restauration des sauvegardes puisse être une contrainte en local en raison de ses besoins en infrastructure, il est beaucoup plus facile procéder dans le cloud en fournissant temporairement les ressources nécessaires à cet effet. Il est correct que des outils fournis avec HANA peuvent vérifier la capacité de restauration des fichiers de sauvegarde. Toutefois, l’objectif des exercices de restauration fréquents est de tester le processus de restauration d’une base de données et de former ce processus avec le personnel d’exploitation.

Gardez à l’esprit qu’il ne suffit pas d’effectuer une simple restauration et de vérifier si HANA est opérationnel et en cours d’exécution. Vous devez exécuter une vérification de cohérence de table pour vous assurer que la base de données restaurée est correcte. SAP HANA offre plusieurs types de vérifications de cohérence. Ceux-ci sont décrits dans la [note SAP 1977584](https://launchpad.support.sap.com/#/notes/1977584).

Vous trouverez également des informations sur la vérification de cohérence de table sur le site web SAP à la page [Table and Catalog Consistency Checks](https://help.sap.com/saphelp_hanaplatform/helpdata/en/25/84ec2e324d44529edc8221956359ea/content.htm#loio9357bf52c7324bee9567dca417ad9f8b) (Vérifications de cohérence de table et de catalogue).

### <a name="pros-and-cons-of-hana-backup-versus-storage-snapshot"></a>Avantages et inconvénients de la sauvegarde HANA par rapport à la capture instantanée de stockage

SAP ne privilégie pas la sauvegarde HANA par rapport à la capture instantanée de stockage. Il répertorie leurs avantages et inconvénients, et chacun peut ainsi déterminer la méthode à utiliser en fonction de la situation et de la technologie de stockage disponible (voir [Planning Your Backup and Recovery Strategy](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.05/en-US/ef085cd5949c40b788bba8fd3c65743e.html) (Planification de votre stratégie de sauvegarde et de récupération) pour plus d’informations).

Sur Azure, sachez que la fonctionnalité de capture instantanée de blobs Azure ne garantit pas la cohérence du système de fichiers sur plusieurs disques (voir la page [Utilisation d’instantanés blob avec PowerShell](/archive/blogs/cie/using-blob-snapshots-with-powershell)). 

Il faut également comprendre les implications liées à la facturation en cas d’utilisation fréquente de captures instantanées d’objets blob, comme décrit dans cet article : [Présentation des frais liés aux instantanés](/rest/api/storageservices/understanding-how-snapshots-accrue-charges). Cet aspect n’est pas aussi évident que lors de l’utilisation de disques virtuels Azure.

### <a name="sap-hana-data-consistency-when-taking-storage-snapshots"></a>Cohérence des données SAP HANA lors de la création de captures instantanées de stockage

Comme indiqué précédemment, la description des capacités de sauvegarde de captures instantanées de Sauvegarde Azure, du système de fichiers et de la cohérence des applications est obligatoire lors de la création de captures instantanées de stockage. Pour éviter les problèmes, le plus simple consisterait à arrêter SAP HANA, voire la machine virtuelle entière. Cela n’est pas possible pour une instance de production.

> [!NOTE]
> Les sauvegardes qui sont basées sur des captures instantanées de disque pour SAP HANA dans les déploiements où plusieurs conteneurs de base de données sont utilisés nécessitent HANA 2.0 SP04 en version minimale.
> 

Le stockage Azure n’assure pas la cohérence du système de fichiers sur plusieurs disques ou volumes attachés à une machine virtuelle pendant le processus d’instantané. Cela signifie que la cohérence de l’application pendant la capture instantanée doit être fournie par l’application, SAP HANA dans cet exemple. La [note SAP 2039883](https://launchpad.support.sap.com/#/notes/2039883) fournit des informations importantes sur les sauvegardes SAP HANA effectuées par captures instantanées du stockage. Par exemple, avec le système de fichiers XFS, il est nécessaire d’exécuter **xfs\_freeze** avant de démarrer une capture instantanée de stockage pour garantir la cohérence de l’application (voir la page [xfs\_freeze(8) – Linux man page](https://linux.die.net/man/8/xfs_freeze) pour plus d’informations sur **xfs\_freeze**).

En supposant qu’un système de fichiers XFS couvre quatre disques virtuels Azure, les étapes suivantes permettent d’obtenir une capture instantanée cohérente qui représente la zone de données HANA :

1. Créer une capture instantanée des données HANA préparées
1. Geler le système de fichiers de tous les disques/volumes (par exemple, à l’aide de **xfs\_freeze**)
1. Créer toutes les captures instantanées d’objets blob nécessaires sur Azure
1. Dégeler le système de fichiers
1. Confirmer la capture instantanée des données HANA (supprimera la capture instantanée)

Lorsque vous utilisez la capacité de Sauvegarde Azure à effectuer des sauvegarde de captures instantanées cohérentes par rapport aux applications, vous devez coder/scripter l’étape no 1 pour le script de pré-instantané. Le service Sauvegarde Azure exécutera les étapes no 2 et no 3. Les étapes no 4 et no 5 doivent être fournies à nouveau par votre code dans le script de post-instantané. Si vous n’utilisez pas le service Sauvegarde Azure, vous devez également coder/scripter vous-même les étapes no 2 et no 3.
Pour plus d’informations sur la création de captures instantanées de données HANA, consultez les articles suivants :

- HANA data snapshots (Captures instantanées de données HANA, https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/en-US/ac114d4b34d542b99bc390b34f8ef375.html ).
- Pour plus d’informations sur l’étape no 1, consultez l’article [Create a Data Snapshot (Native SQL)](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/en-US/9fd1c8bb3b60455caa93b7491ae6d830.html) [Création d’une capture instantanée de données (native SQL)]. 
- Pour plus d’informations sur la confirmation/suppression des captures instantanées de données HANA, le cas échéant, de l’étape no 5, consultez l’article [Create a Data Snapshot (Native SQL)](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/en-US/9fd1c8bb3b60455caa93b7491ae6d830.html) [Création d’une capture instantanée de données (native SQL)]. 

Il est important de confirmer la capture instantanée HANA. En raison de la &quot;copie pour écriture&quot;, il se peut que SAP HANA ne nécessite pas d’espace disque supplémentaire en mode de préparation de capture instantanée. Par ailleurs, il n’est pas possible de démarrer de nouvelles sauvegardes tant que la capture instantanée SAP HANA n’a pas été confirmée.


### <a name="sap-hana-backup-scheduling-strategy"></a>Stratégie de planification de sauvegarde SAP HANA

L’article SAP HANA [Planning Your Backup and Recovery Strategy](https://help.sap.com/saphelp_hanaplatform/helpdata/en/ef/085cd5949c40b788bba8fd3c65743e/content.htm) (Planification de votre stratégie de sauvegarde et de récupération) fournit un plan de base pour effectuer les sauvegardes. Fiez-vous à la documentation SAP relative à HANA et à vos expériences avec d’autres SGBD pour définir la stratégie et le processus de sauvegarde/restauration de SAP HANA. L’ordre des différents types de sauvegardes et la durée de rétention dépendent fortement des Contrats de niveau de service que vous devez établir.


### <a name="sap-hana-backup-encryption"></a>Chiffrement de sauvegarde SAP HANA

SAP HANA assure le chiffrement des données et du journal. Si le journal et les données SAP HANA ne sont pas chiffrés, les sauvegardes ne le sont pas par défaut. Toutefois, SAP HANA propose un chiffrement de sauvegarde distinct, comme indiqué dans l’article [SAP HANA Backup Encryption](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.03/en-US/5f837a57ce5e468d9db21c8683bc84da.html) (Chiffrement de sauvegarde SAP HANA). Si vous exécutez des versions antérieures de SAP HANA, vous devrez peut-être vérifier si le chiffrement de sauvegarde faisait partie des fonctionnalités déjà fournies.  


## <a name="next-steps"></a>Étapes suivantes
* L’article [Sauvegarde SAP HANA sur Azure au niveau fichier](sap-hana-backup-file-level.md) décrit l’option de sauvegarde basée sur des fichiers.
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur Azure (grandes instances), consultez [Haute disponibilité et récupération d’urgence de SAP HANA (grandes instances) sur Azure](hana-overview-high-availability-disaster-recovery.md).
