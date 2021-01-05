---
title: Prise en charge pour la migration de serveurs physiques dans Azure Migrate
description: Découvrez la prise en charge pour la migration de serveurs physiques dans Azure Migrate.
author: rahulg1190
ms.author: rahugup
ms.manager: bsiva
ms.topic: conceptual
ms.date: 06/14/2020
ms.openlocfilehash: c549cae0b7bdc5e237ec3cf704a1ead53db91449
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754026"
---
# <a name="support-matrix-for-physical-server-migration"></a>Tableau de prise en charge pour la migration de serveurs physiques

Cet article résume les paramètres et les limites de la migration de serveurs physiques avec [Azure Migrate : Migration de serveur](migrate-services-overview.md#azure-migrate-server-migration-tool). Si vous recherchez des informations sur l’évaluation de serveurs physiques VMware pour la migration vers Azure, consultez le [tableau de prise en charge de l’évaluation](migrate-support-matrix-physical.md).

## <a name="migrating-machines-as-physical"></a>Migration de machines en tant qu’ordinateurs physiques

Vous pouvez migrer des machines locales en tant que serveurs physiques à l’aide de la réplication basée sur un agent. Avec cet outil, vous pouvez effectuer la migration d’une large gamme de machines vers Azure :

- Serveurs physiques locaux.
- Machines virtuelles virtualisées par des plateformes telles que Xen ou KVM.
- Machines virtuelles Hyper-V ou machines virtuelles VMware si, pour une quelconque raison, vous ne souhaitez pas utiliser les flux [Hyper-V](tutorial-migrate-hyper-v.md) ou [VMware](server-migrate-overview.md).
- Machines virtuelles exécutées dans des clouds privés.
- Machines virtuelles exécutées dans des clouds publics tels qu’Amazon Web Services (AWS) ou Google Cloud Platform (GCP).


## <a name="migration-limitations"></a>Limitations de la migration

Vous pouvez sélectionner jusqu’à 10 machines à la fois pour la réplication. Si vous souhaitez migrer davantage de machines, répliquez-les dans des groupes de 10.


## <a name="physical-server-requirements"></a>Conditions requises des serveurs physiques

Le tableau résume la prise en charge de serveurs physiques que vous souhaitez migrer à l’aide de la migration basée sur un agent.

**Support** | **Détails**
--- | ---
**Charge de travail des machines** | Azure Migrate prend en charge la migration de toutes les charges de travail (par exemple Active Directory, SQL Server, etc.) exécutées sur une machine prise en charge.
**Systèmes d’exploitation** | Pour obtenir les informations les plus récentes, consultez [Prise en charge des systèmes d’exploitation](../site-recovery/vmware-physical-azure-support-matrix.md#replicated-machines) pour Site Recovery. Azure Migrate fournit une prise en charge identique des systèmes d’exploitation.
**Stockage invité/système de fichiers Linux** | Pour obtenir les informations les plus récentes, consultez [Prise en charge des systèmes de fichiers Linux](../site-recovery/vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage) pour Site Recovery. Azure Migrate fournit une prise en charge identique des systèmes de fichiers Linux.
**Réseau/Stockage** | Pour obtenir les informations les plus récentes, consultez les prérequis en matière de [réseau](../site-recovery/vmware-physical-azure-support-matrix.md#network) et de [stockage](../site-recovery/vmware-physical-azure-support-matrix.md#storage) pour Site Recovery. Azure Migrate fournit des exigences identiques pour le réseau/stockage.
**Conditions requises pour Azure** | Pour obtenir les informations les plus récentes, consultez les exigences d’Azure en matière de [réseau](../site-recovery/vmware-physical-azure-support-matrix.md#azure-vm-network-after-failover), de [stockage](../site-recovery/vmware-physical-azure-support-matrix.md#azure-storage) et de [calcul](../site-recovery/vmware-physical-azure-support-matrix.md#azure-compute) pour Site Recovery. Azure Migrate a des exigences identiques pour la migration de serveurs physiques.
**Service de mobilité** | L’agent du service Mobilité doit être installé sur chaque machine que vous souhaitez migrer.
**Démarrage UEFI** | Pris en charge. Les machines UEFI seront migrées vers des machines virtuelles Azure de 2e génération.  <br/><br/> Le disque du système d’exploitation doit avoir jusqu’à quatre partitions et les volumes doivent être formatés avec NTFS.
**UEFI – Démarrage sécurisé**         | Non pris en charge pour la migration.
**Disque cible** | Les machines peuvent être migrées seulement vers des disques managés (disques HDD standard, disques SSD standard, disques SSD Premium) dans Azure.
**Taille du disque** | Disque de système d’exploitation de 2 To (démarrage du BIOS) ; disque de système d’exploitation de 4 To (démarrage d’UEFI) ; 8 To pour les disques de données.
**Limites du disque** |  Jusqu’à 63 disques par machine.
**Disques/volumes chiffrés** |  Les machines avec des disques/volumes chiffrés ne sont pas prises en charge pour la migration.
**Cluster de disque partagé** | Non pris en charge.
**Disques indépendants** | Pris en charge.
**Disques directs** | Pris en charge.
**NFS** | Les volumes NFS montés en tant que volumes sur les machines ne sont pas répliqués.
**Cibles iSCSI** | Les machines avec des cibles iSCSI ne sont pas prises en charge pour la migration sans agent.
**Multipath I/O** | Non pris en charge.
**Storage vMotion** | Prise en charge
**Cartes réseau associées** | Non pris en charge.
**IPv6** | Non pris en charge.



## <a name="replication-appliance-requirements"></a>Conditions requises de l’appliance de réplication

Si vous configurez manuellement l’appliance de réplication sur un serveur physique, vérifiez qu’elle est conforme aux exigences résumées dans le tableau. Lorsque vous configurez l’appliance de réplication Azure Migrate en tant que machine virtuelle VMware à l’aide du modèle OVA fourni dans le hub Azure Migrate, l’appliance est configurée avec Windows Server 2016 et est conforme aux exigences de la prise en charge. 

- En savoir plus sur les [conditions requises de l’appliance de réplication](migrate-replication-appliance.md#appliance-requirements).
- MySQL doit être installé sur l’appliance. En savoir plus sur les [options d’installation](migrate-replication-appliance.md#mysql-installation).
- En savoir plus sur les [URL](migrate-replication-appliance.md#url-access) auxquelles l’appliance de réplication doit accéder.

## <a name="azure-vm-requirements"></a>Exigences des machines virtuelles Azure

Toutes les machines virtuelles locales répliquées sur Azure doivent respecter les exigences des machines virtuelles Azure récapitulées dans ce tableau. Quand Site Recovery vérifie les prérequis pour la réplication, la vérification échoue si certaines exigences ne sont pas satisfaites.

**Composant** | **Configuration requise** | **Détails**
--- | --- | ---
Système d’exploitation invité | Vérifie les systèmes d’exploitation pris en charge.<br/> Vous pouvez migrer les charges de travail s’exécutant sur un système d’exploitation pris en charge. | La vérification est mise en échec en cas de défaut de prise en charge.
Architecture du système d’exploitation invité | 64 bits. | La vérification est mise en échec en cas de défaut de prise en charge.
Taille du disque du système d’exploitation | Jusqu’à 2 048 Go. | La vérification est mise en échec en cas de défaut de prise en charge.
Nombre de disques du système d’exploitation | 1 | La vérification est mise en échec en cas de défaut de prise en charge.
Nombre de disques de données | 64 ou moins. | La vérification est mise en échec en cas de défaut de prise en charge.
Taille de disque de données | Jusqu’à 4 095 Go | La vérification est mise en échec en cas de défaut de prise en charge.
Adaptateurs réseau | Prise en charge de plusieurs adaptateurs réseau. |
Disque dur virtuel partagé | Non pris en charge. | La vérification est mise en échec en cas de défaut de prise en charge.
Disque FC | Non pris en charge. | La vérification est mise en échec en cas de défaut de prise en charge.
BitLocker | Non pris en charge. | Vous devez désactiver BitLocker avant d’activer la réplication pour une machine.
nom de la machine virtuelle | De 1 et 63 caractères.<br/> Uniquement des lettres, des chiffres et des traits d’union.<br/><br/> Le nom de la machine doit commencer et se terminer par une lettre ou un chiffre. |  Mettez à jour la valeur dans les propriétés de machine de Site Recovery.
Se connecter après la migration - Windows | Pour vous connecter à des machines virtuelles Azure exécutant Windows après la migration :<br/> - Avant la migration, active le protocole RDP sur la machine virtuelle locale. Vérifiez que des règles TCP et UDP sont ajoutées pour le profil **Public** et que RDP est autorisé dans **Pare-feu Windows** > **Applications autorisées** pour tous les profils.<br/> Pour l’accès VPN site à site, activez RDP et autorisez RDP dans **Pare-feu Windows** -> **Applications et fonctionnalités autorisées** pour les réseaux **Domaine et Privé**. Vérifiez aussi que la stratégie SAN du système d’exploitation est définie sur **OnlineAll**. [Plus d’informations](prepare-for-migration.md) |
Se connecter après la migration - Linux | Pour vous connecter à des machines virtuelles Azure après la migration avec SSH :<br/> Avant la migration, sur la machine locale, vérifiez que le service Secure Shell est défini sur Démarrer et que les règles de pare-feu autorisent une connexion SSH.<br/> Après le basculement, sur la machine virtuelle Azure, autorisez les connexions entrantes au port SSH pour les règles du groupe de sécurité réseau sur la machine virtuelle basculée, et pour le sous-réseau Azure auquel elle est connectée. Ajoutez aussi une adresse IP publique pour la machine virtuelle. |  


## <a name="next-steps"></a>Étapes suivantes

[Migrer](tutorial-migrate-physical-virtual-machines.md) des serveurs physiques.
