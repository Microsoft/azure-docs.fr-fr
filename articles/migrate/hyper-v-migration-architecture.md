---
title: Comment fonctionne la migration Hyper-V dans Azure Migrate ?
description: En savoir plus sur la migration d’Hyper-V avec Azure Migrate
author: bsiva
ms.author: bsiva
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 11/19/2019
ms.openlocfilehash: 239918cc19eefbef9e3c3f12d5ddd3bb5434b490
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751017"
---
# <a name="how-does-hyper-v-replication-work"></a>Fonctionnement de la réplication Hyper-V

Cet article fournit une vue d’ensemble de l’architecture et des processus utilisés lorsque vous migrez des machines virtuelles Hyper-V à l’aide de l’outil de migration de serveur Azure Migrate.

[Azure Migrate](migrate-services-overview.md) offre un hub central pour suivre la découverte, l’évaluation et la migration vers Azure de vos applications et charges de travail locales, ainsi que de machines virtuelles cloud privées/publiques. Le hub fournit des outils Azure Migrate pour l’évaluation et la migration ainsi que des offres de fournisseurs de logiciels indépendants (ISV) tiers.

## <a name="agentless-migration"></a>Migration sans agent

L’outil de migration de serveur Azure Migrate fournit une réplication sans agent pour les machines virtuelles Hyper-V locales, à l’aide d’un flux de travail de migration optimisé pour Hyper-V. Vous installez un agent logiciel uniquement sur des ordinateurs hôtes Hyper-V ou des nœuds de cluster. Rien ne doit être installé sur les machines virtuelles Hyper-V.

## <a name="server-migration-and-azure-site-recovery"></a>Migration de serveur et Azure Site Recovery

La migration de serveur Azure Migrate est un outil qui permet de migrer des charges de travail locales et des machines virtuelles basées sur le cloud vers Azure. Site Recovery est un outil de récupération d’urgence. Les outils partagent certains composants technologiques courants utilisés pour la réplication des données, mais servent des objectifs différents. 


## <a name="architectural-components"></a>Composants architecturaux

![Diagramme montrant un réseau Hyper-V source avec un canal de données HTTPS vers Microsoft Azure, et des détails expliqués dans un tableau.](./media/hyper-v-replication-architecture/architecture.png)



**Composant** | **Déploiement** | 
--- | --- 
**Fournisseur de réplication** | Le fournisseur Microsoft Azure Site Recovery est installé sur les hôtes Hyper-V et inscrit auprès de la migration du serveur Azure Migration.<br/> Le fournisseur orchestre la réplication pour les machines virtuelles Hyper-V.
**Agent Recovery Services** | L’agent Microsoft Azure Recovery Service gère la réplication des données. Il collabore avec le fournisseur pour répliquer des données à partir de machines virtuelles Hyper-V vers Azure.<br/> Les données répliquées sont chargées vers un compte de stockage dans votre abonnement Azure. L’outil de migration de serveur traite les données répliquées et les applique aux disques de réplication de l’abonnement. Les disques de réplica sont utilisés pour créer les machines virtuelles Azure lors de la migration.

- Les composants sont installés par un fichier d’installation unique, téléchargé à partir de la migration de serveur Azure Migrate dans le portail.
- Le fournisseur et l’appliance utilisent des connexions sortantes du port HTTPS 443 pour communiquer avec la migration de serveur Azure Migrate.
- Les communications en provenance du fournisseur et de l’agent sont sécurisées et chiffrées.


## <a name="replication-process"></a>Processus de réplication

1. Lorsque vous activez la réplication pour une machine virtuelle Hyper-V, la réplication initiale commence.
2. Un instantané de machine virtuelle Hyper-V est pris.
3. Les disques durs virtuels sur la machine virtuelle sont répliqués un par un, jusqu’à ce qu’ils soient tous copiés sur Azure. Le temps de réplication initiale dépend de la taille de la machine virtuelle et de la bande passante réseau.
4. Les modifications de disque qui se produisent pendant la réplication initiale sont suivies à l’aide de la réplication Hyper-V et stockées dans des fichiers journaux (fichiers HRL).
    - Ces fichiers journaux se trouvent dans le même dossier que les disques.
    - À chaque disque correspond un fichier .HRL, qui est envoyé à la mémoire auxiliaire.
    - L’instantané et les fichiers journaux consomment des ressources disque pendant la réplication initiale.
4. Après que la réplication initiale s’achève, l’instantané de machine virtuelle est supprimé et la réplication delta commence.
5. Les modifications incrémentielles du disque sont suivies dans les fichiers HRL. Les journaux de réplication sont régulièrement chargés dans un compte de stockage Azure par l’agent de Recovery Services.


## <a name="performance-and-scaling"></a>Performances et mise à l’échelle

Les performances de réplication pour Hyper-V sont influencées par des facteurs qui incluent la taille de machine virtuelle, le taux de modification des données (évolution) des machines virtuelles, l’espace disponible sur l’hôte Hyper-V pour le stockage des fichiers journaux, la bande passante de chargement pour les données de réplication et le stockage cible dans Azure.

- Si vous répliquez plusieurs machines en même temps, utilisez le [Planificateur de déploiement Azure Site Recovery](../site-recovery/hyper-v-deployment-planner-overview.md) pour Hyper-V, afin d’optimiser la réplication.
- Planifiez la réplication Hyper-V et distribuez-la sur les comptes de stockage Azure, en fonction de la capacité.

### <a name="control-upload-throughput"></a>Contrôle du débit de chargement

Vous pouvez limiter la quantité de bande passante utilisée pour télécharger des données vers Azure sur chaque hôte Hyper-V. Soyez prudent ! Si vous définissez une valeur trop faible, elle risque d’avoir un impact négatif sur la réplication et de retarder la migration.


1. Connectez-vous à l’hôte Hyper-V ou au nœud de cluster.
2. Exécutez **C:\Program Files\Microsoft Azure Recovery Services Agent\bin\wabadmin.msc** pour ouvrir le composant logiciel enfichable MMC de Windows Sauvegarde Azure.
3. Dans le composant logiciel enfichable, sélectionnez **Modifier les propriétés**.
4. Dans **Limitation**, sélectionnez la case **Activer la limitation de la bande passante sur Internet pour les opérations de sauvegarde**. Définissez les limites pour les heures ouvrées et non ouvrées. Les plages valides vont de 512 Kbits/s à 1 023 Mbits/s.
I

### <a name="influence-upload-efficiency"></a>Améliorer l’efficacité du chargement

Si vous disposez d’une bande passante de rechange pour la réplication et que vous souhaitez augmenter les chargements, vous pouvez augmenter le nombre de threads alloués pour la tâche de chargement, comme suit :

1. Ouvrez le registre avec Regedit.
2. Accédez à la clé HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication\UploadThreadsPerVM
3. Augmentez la valeur du nombre de threads utilisés pour le téléchargement de données pour chaque machine virtuelle de réplication. La valeur par défaut est 4 et la valeur maximale est 32. 




## <a name="next-steps"></a>Étapes suivantes

Essayez la [migration Hyper-V](tutorial-migrate-hyper-v.md) à l’aide de la migration de serveur Azure Migrate.
