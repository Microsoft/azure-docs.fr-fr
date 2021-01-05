---
title: Principes de récupération d’urgence et de préparation sur SAP HANA sur Azure (grandes Instances) | Microsoft Docs
description: Principes de récupération d’urgence et de préparation sur SAP HANA sur Azure (grandes Instances)
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: gwallace
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7f0782738a8aa57b2bc0b87e1378972e3fa5d31c
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967854"
---
# <a name="disaster-recovery-principles"></a>Principes de la récupération d’urgence

Les grandes instances HANA offrent une fonctionnalité de récupération d’urgence entre les horodatages de grande instance HANA dans différentes régions Azure. Par exemple, si vous déployez des unités de grande instance HANA dans la région Azure USA Ouest, vous pouvez utiliser des unités de grande instance HANA dans la région USA Est comme des unités de récupération d’urgence. Comme mentionné précédemment, la récupération d’urgence n’est pas configurée automatiquement, car vous devez acheter une autre unité de grande instance HANA dans la région de la récupération d’urgence. La configuration de la récupération d’urgence fonctionne pour les configurations avec montée en puissance et avec montée en puissance parallèle. 

Dans les scénarios déployés jusqu’à présent, nos clients se servent de l’unité dans la région de la récupération d’urgence pour exécuter des systèmes de non-production qui utilisent une instance HANA installée. L’unité de grande instance HANA doit avoir la même référence SKU que celle utilisée à des fins de production. L’image suivante montre la configuration de disque entre l’unité du serveur dans la région de production Azure et la région de récupération d’urgence :

![Configuration de la récupération d’urgence du point de vue du disque](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_setup.PNG)

Comme le montre cette vue d’ensemble, vous devez commander un deuxième ensemble de volumes de disque. Les volumes de disque cibles sont de même taille que les volumes de production pour l’instance de production dans les unités de récupération d’urgence. Ces volumes de disque sont associés à l’unité du serveur de grande instance HANA sur le site de récupération d’urgence. Les volumes suivants sont répliqués de la région de production sur le site de récupération d’urgence :

- /hana/data
- /hana/logbackups 
- /hana/shared (/usr/sap compris)

Le volume /hana/log n’est pas répliqué, car le journal des transactions SAP HANA n’est pas nécessaire avec le mode de restauration depuis ces volumes qui est employé. 

La fonctionnalité de récupération d’urgence proposée est la fonctionnalité de réplication de stockage offerte par l’infrastructure des grandes instances HANA. La fonctionnalité qui est utilisée côté stockage n’est pas un flux constant de modifications répliquées de manière asynchrone à mesure que des changements sont apportés au volume de stockage. Il s’agit plutôt d’un mécanisme qui repose sur le fait que des captures instantanées de ces volumes sont régulièrement créées. La différence entre une capture instantanée déjà répliquée et une nouvelle capture instantanée non encore répliquée est ensuite transférée au site de récupération d’urgence dans les volumes de disque cibles.  Ces captures instantanées sont stockées sur les volumes et doivent, en cas de basculement dans le cadre d’une récupération d’urgence, être restaurées sur ces volumes.  

Le premier transfert de la totalité des données d’un volume doit avoir lieu avant que la quantité de données ne devienne inférieure aux données différentielles entre les captures instantanées. Ainsi, les volumes sur le site de récupération d’urgence contiennent chacun des captures instantanées de volume effectuées sur le site de production. Au final, vous pouvez utiliser ce système de récupération d’urgence pour revenir à un état antérieur et récupérer des données perdues, sans restaurer le système de production.

En cas de déploiement MCOD avec plusieurs instances SAP HANA indépendantes sur une unité de grande Instance HANA, il est probable que toutes les instances SAP HANA obtiennent un stockage répliqué du côté de la récupération d’urgence.

Lorsque vous utilisez la réplication de système HANA en tant que fonctionnalité à haute disponibilité dans votre site de production, et que vous utilisez la réplication basée sur le stockage pour le site de récupération d’urgence, les volumes des deux nœuds du site principal vers l’instance de récupération d’urgence sont répliqués. Vous devez acheter du stockage supplémentaire (de taille identique à celle du nœud principal) sur le site de récupération d’urgence pour prendre en charge la réplication à partir des deux sites principal et secondaire vers celui de la récupération d’urgence. 



>[!NOTE]
>La fonctionnalité de réplication de stockage des grandes instances HANA met en miroir et réplique les captures instantanées de stockage. Si vous n'effectuez pas de captures instantanées de stockage comme indiqué à la section Sauvegarde et restauration de cet article, aucune réplication ne peut avoir lieu sur le site de récupération d'urgence. L’exécution de captures instantanées de stockage est un prérequis pour la réplication de stockage vers le site de récupération d’urgence.



## <a name="preparation-of-the-disaster-recovery-scenario"></a>Préparation du scénario de récupération d’urgence
Dans ce scénario, vous disposez d’un système de production opérationnel sur des grandes instances HANA dans la région de production Azure. Pour les étapes qui suivent, supposons que le SID de ce système HANA est « PRD » et que vous disposez d’un système de non-production opérationnel sur des instances de grande taille HANA dans la région de récupération d’urgence Azure. Pour ce dernier, supposons que son SID est « TST ». L’image suivante montre cette configuration :

![Première étape de la configuration de la récupération d’urgence](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start1.PNG)

Si l’instance de serveur n’a pas été commandée avec l’ensemble de volume de stockage supplémentaire, l’équipe de gestion des services SAP HANA sur Azure attache l’ensemble de volume supplémentaire en tant que cible du réplica de production sur l’unité de grande instance HANA sur laquelle vous exécutez l’instance HANA TST. À cette fin, vous devez fournir le SID de votre instance HANA de production. Une fois que l’équipe de gestion des services SAP HANA sur Azure a confirmé que ces volumes ont été attachés, vous devez monter ces derniers sur l’unité de grande instance HANA.

![Étape suivante de la configuration de la récupération d’urgence](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start2.PNG)

L’étape suivante consiste à installer la deuxième instance SAP HANA sur l’unité de grande instance HANA dans la région Azure de récupération d’urgence, où vous exécutez l’instance HANA TST. La nouvelle instance SAP HANA installée doit avoir le même SID. Les utilisateurs créés doivent avoir les mêmes UID et ID de groupe que ceux de l’instance de production. Pour plus d’informations, lisez [Sauvegarde et restauration](hana-backup-restore.md). Si l’installation a réussi, vous devez :

- Exécutez l’étape 2 de la préparation de capture instantanée de stockage décrite dans [Sauvegarde et restauration](hana-backup-restore.md).
- Créez une clé publique pour l’unité de récupération d’urgence de l’unité de grande instance HANA, si ce n’est déjà fait. Consultez l’étape 3 de la préparation de capture instantanée de stockage décrite dans [Sauvegarde et restauration](hana-backup-restore.md).
- Maintenez *HANABackupCustomerDetails.txt* avec la nouvelle instance HANA et testez si la connectivité au stockage fonctionne correctement.  
- Arrêtez l’instance SAP HANA nouvellement installée sur l’unité de grande instance HANA dans la région Azure de récupération d’urgence.
- Démonter ces volumes PRD et contacter l’équipe de gestion des services SAP HANA sur Azure. Les volumes ne peuvent pas rester montés sur l’unité, car ils ne sont pas accessibles tant qu’ils fonctionnent en tant que cible de réplication de stockage.  

![Diagramme illustrant la relation de réplication entre les volumes PRD dans la région Azure de production et les volumes PRD dans la région Azure de récupération d’urgence.](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start3.PNG)

L’équipe des opérations établit la relation de réplication entre les volumes PRD dans la région Azure de production et les volumes PRD dans la région Azure de récupération d’urgence.

>[!IMPORTANT]
>Le volume /hana/log n’est pas répliqué, car il n’est pas nécessaire pour restaurer la base de données SAP HANA répliquée dans un état cohérent sur le site de récupération d’urgence.

Ensuite, définissez ou ajustez la planification des sauvegardes de capture instantanée de stockage pour atteindre vos RTO et RPO en cas d’urgence. Pour réduire l’objectif de point de récupération, définissez les intervalles de réplication suivants dans le service de grande instance HANA :
- Pour les volumes couverts par la capture instantanée combinée (type **hana**), configurez la réplication toutes les 15 minutes vers les cibles de volume de stockage équivalents sur le site de récupération d’urgence.
- Pour le volume de sauvegarde de fichier journal (type **logs** de capture instantanée), configurez la réplication toutes les 3 minutes vers les cibles de volume de stockage équivalents sur le site de récupération d’urgence.

Pour réduire l’objectif de point de récupération, effectuez le paramétrage suivant :
- Effectuez une capture instantanée de stockage de type **hana** (voir « Étape 7 : Effectuer des captures instantanées ») à intervalles de 30 minutes à 1 heure.
- Effectuez des sauvegardes de fichier journal SAP HANA toutes les 5 minutes.
- Effectuez une capture instantanée de stockage de type **logs** toutes les 5 à 15 minutes. Avec cet intervalle, vous obtenez un RPO d’environ 15 à 25 minutes.

Dans cette configuration, la séquence des sauvegardes de fichier journal, des captures instantanées de stockage, ainsi que la réplication du volume de sauvegarde de fichier journal HANA et des volumes /hana/data et /hana/shared (/usr/sap compris), pourraient ressembler aux données ci-dessous :

 ![Relation entre une capture instantanée de sauvegarde de fichier journal et un miroir de capture instantanée sur un axe temporel](./media/hana-overview-high-availability-disaster-recovery/snapmirror.PNG)

Pour améliorer le RPO dans le cadre d’une récupération d’urgence, vous pouvez copier les sauvegardes de fichier journal HANA depuis SAP HANA sur Azure (grandes instances) vers l’autre région Azure. Pour réduire encore le RPO, effectuez les étapes suivantes :

1. Sauvegardez le journal des transactions HANA aussi souvent que possible dans /hana/logbackups.
1. Utilisez rsync pour copier les sauvegardes de fichier journal sur les machines virtuelles Azure hébergées sur le partage NFS. Les machines virtuelles se trouvent dans des réseaux virtuels Azure dans la région Azure de production et dans les régions de récupération d’urgence. Vous devez connecter les deux réseaux virtuels Azure au circuit reliant les grandes instances HANA de production à Azure. Reportez-vous au graphique de la section [Considérations sur le réseau pour la récupération d’urgence avec de grandes instances HANA](#Network-considerations-for-disaster recovery-with-HANA-Large-Instances). 
1. Conservez les sauvegardes de fichier journal dans la région sur la machine virtuelle attachée au stockage NFS exporté.
1. Dans le cas d’un basculement d’urgence, complétez les sauvegardes de fichier journal disponibles sur le volume /hana/logbackups avec des sauvegardes de fichier journal plus récentes sur le partage NFS sur le site de récupération d’urgence. 
1. Lancez une sauvegarde de fichier journal pour restaurer jusqu’à la dernière sauvegarde enregistrée dans la région de la récupération d’urgence.

Lorsque les opérations de grande instance HANA confirment la relation de réplication configurée et que vous exécutez des sauvegardes de capture instantanée de stockage, la réplication des données commence.

![Étape de configuration de la récupération d’urgence avant d’établir la réplication](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start4.PNG)

Durant la réplication, les captures instantanées sur les volumes PRD dans les régions Azure de récupération d’urgence ne sont pas restaurées. Elles sont uniquement stockées. Si les volumes sont montés dans cet état, ils reflètent l’état dans lequel vous les avez démontés après l’installation de l’instance SAP HANA PRD dans l’unité du serveur de la région Azure de récupération d’urgence. Ils reflètent également les sauvegardes de stockage qui ne sont pas encore restaurées.

En cas de basculement, vous pouvez également opter pour effectuer une restauration à partir d’une capture instantanée de stockage plus ancienne au lieu de la dernière capture instantanée de stockage.

## <a name="next-steps"></a>Étapes suivantes

- Voir [Procédure de basculement en cas de récupération d’urgence](hana-failover-procedure.md).
