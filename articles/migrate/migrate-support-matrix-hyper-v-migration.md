---
title: Prise en charge de la migration Hyper-V dans Azure Migrate
description: Découvrez la prise en charge de la migration Hyper-V avec Azure Migrate.
author: bsiva
ms.author: bsiva
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 04/15/2020
ms.openlocfilehash: 90da16789344754c02d46022160db71ee261a056
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754060"
---
# <a name="support-matrix-for-hyper-v-migration"></a>Matrice de prise en charge pour la migration Hyper-V

Cet article résume les paramètres et les limites de la migration de machines virtuelles Hyper-V avec [Azure Migrate : Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool). Si vous recherchez des informations sur l’évaluation des machines virtuelles Hyper-V pour la migration vers Azure, consultez la [matrice de prise en charge de l’évaluation](migrate-support-matrix-hyper-v.md).

## <a name="migration-limitations"></a>Limites de la migration

Vous pouvez sélectionner jusqu’à 10 machines virtuelles à la fois pour la réplication. Si vous souhaitez migrer davantage de machines, répliquez-les en groupes de 10.


## <a name="hyper-v-host-requirements"></a>Configuration requise pour l’hôte Hyper-V

| **Support**                | **Détails**               
| :-------------------       | :------------------- |
| **Déploiement**       | L'hôte Hyper-V peut être autonome ou déployé dans un cluster. <br/>Le logiciel de réplication Azure Migrate (fournisseur de réplication Hyper-V) est installé sur les hôtes Hyper-V.|
| **autorisations**           | Vous avez besoin des droits d'administrateur sur l'hôte Hyper-V. |
| **Système d'exploitation hôte** | Windows Server 2019, Windows Server 2016 ou Windows Server 2012 R2 avec les dernières mises à jour. Notez que l’installation Server Core de ces systèmes d’exploitation est également prise en charge. |
| **Autre configuration logicielle requise** | .NET Framework 4.7 ou version ultérieure |
| **Accès au port** |  Connexions sortantes sur le port HTTPS 443 pour envoyer les données de réplication des machines virtuelles.
| **Espace disque disponible (cache)** |  600 Go |
| **Espace disque disponible (disque de rétention)** |  600 Go |


## <a name="hyper-v-vms"></a>Machines virtuelles Hyper-V

| **Support**                  | **Détails**               
| :----------------------------- | :------------------- |
| **Système d’exploitation** | Tous les systèmes d'exploitation [Windows](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines) et [Linux](../virtual-machines/linux/endorsed-distros.md) pris en charge par Azure. |
**Windows Server 2003** | Pour les machines virtuelles exécutant Windows Server 2003, vous devez [installer Hyper-V Integration Services](prepare-windows-server-2003-migration.md) avant la migration. | 
**Machines virtuelles Linux dans Azure** | Certaines machines virtuelles peuvent nécessiter des modifications pour fonctionner dans Azure.<br/><br/> Azure Migrate effectue automatiquement les modifications pour les systèmes d’exploitation Linux suivants :<br/> - Red Hat Enterprise Linux 7.8, 7.7, 7.6, 7.5, 7.4, 7.0, 6.x<br/> - Cent OS 7.7, 7.6, 7.5, 7.4, 6.x</br> - SUSE Linux Enterprise Server 12 SP1+<br/> - SUSE Linux Enterprise Server 15 SP1 <br/>- Ubuntu 19.04, 19.10, 14.04LTS, 16.04LTS, 18.04LTS<br/> - Debian 7, 8 <br/> Oracle Linux 7.7, 7.7-CI<br/> Pour les autres systèmes d’exploitation, [modifications requises](prepare-for-migration.md#verify-required-changes-before-migrating) sont à faire manuellement.
| **Modifications nécessaires pour Azure** | Certaines machines virtuelles peuvent nécessiter des modifications pour fonctionner dans Azure. Effectuez les ajustements manuellement avant la migration. Les articles pertinents contiennent des instructions sur la façon de procéder. |
| **Démarrage Linux**                 | Si /boot se trouve sur une partition dédiée, il doit être le disque du système d’exploitation et ne pas être réparti sur plusieurs disques.<br/> Si /boot fait partie de la partition racine (/), la partition « / » doit se trouver sur le disque du système d’exploitation et ne pas s’étendre sur d’autres disques. |
| **Démarrage UEFI**                  | Pris en charge. Les machines virtuelles UEFI seront migrées vers des machines virtuelles Azure de 2e génération.  |
| **UEFI – Démarrage sécurisé**         | Non pris en charge pour la migration.|
| **Taille du disque**                  | 2 To pour le disque du système d’exploitation (démarrage du BIOS) ; 4 To pour le disque du système d’exploitation (démarrage d'UEFI) ; 4 To pour les disques de données.|
| **Nombre de disques** | Un maximum de 16 disques par machine virtuelle.|
| **Disques/volumes chiffrés**    | Non pris en charge pour la migration.|
| **RDM/disques directs**      | Non pris en charge pour la migration.|
| **Disque partagé** | Les machines virtuelles qui utilisent des disques partagés ne sont pas prises en charge pour la migration.|
| **NFS**                        | Les volumes NFS montés en tant que volumes sur les machines virtuelles ne sont pas répliqués.|
| **ISCSI**                      | Les machines virtuelles avec des cibles iSCSI ne sont pas prises en charge pour la migration.
| **Disque cible**                | Vous pouvez uniquement migrer des machines virtuelles Azure avec des disques managés. |
| **IPv6** | Non pris en charge.|
| **Association de cartes réseau** | Non pris en charge.|
| **Azure Site Recovery** | Vous ne pouvez pas répliquer en utilisant Azure Migrate Server Migration si la machine virtuelle est configurée pour la réplication avec Azure Site Recovery.|
| **Ports** | Connexions sortantes sur le port HTTPS 443 pour envoyer les données de réplication des machines virtuelles.|

### <a name="url-access-public-cloud"></a>Accès URL (cloud public)

Le logiciel du fournisseur de réplication sur les hôtes Hyper-V doit accéder à ces URL.

**URL** | **Détails**
--- | ---
login.microsoftonline.com | Contrôle d’accès et gestion des identités avec Active Directory.
backup.windowsazure.com | Transfert des données de réplication et coordination.
*.hypervrecoverymanager.windowsazure.com | Utilisé pour la gestion de la réplication.
*.blob.core.windows.net | Chargez des données dans des comptes de stockage. 
dc.services.visualstudio.com | Chargez les journaux d’applications utilisés pour la supervision interne.
time.windows.com | Vérifie la synchronisation horaire entre l’horloge système et l’heure globale.

### <a name="url-access-azure-government"></a>Accès URL (Azure Government)

Le logiciel du fournisseur de réplication sur les hôtes Hyper-V doit accéder à ces URL.

**URL** | **Détails**
--- | ---
login.microsoftonline.us | Contrôle d’accès et gestion des identités avec Active Directory.
backup.windowsazure.us | Transfert des données de réplication et coordination.
*.hypervrecoverymanager.windowsazure.us | Utilisé pour la gestion de la réplication.
*.blob.core.usgovcloudapi.net | Chargez des données dans des comptes de stockage.
dc.services.visualstudio.com | Chargez les journaux d’applications utilisés pour la supervision interne.
time.nist.gov | Vérifie la synchronisation horaire entre l’horloge système et l’heure globale.

## <a name="azure-vm-requirements"></a>Exigences des machines virtuelles Azure

Toutes les machines virtuelles locales répliquées sur Azure doivent respecter les exigences des machines virtuelles Azure récapitulées dans ce tableau.

**Composant** | **Configuration requise** | **Détails**
--- | --- | ---
Taille du disque du système d’exploitation | Jusqu’à 2 048 Go. | La vérification est mise en échec en cas de défaut de prise en charge.
Nombre de disques du système d’exploitation | 1 | La vérification est mise en échec en cas de défaut de prise en charge.
Nombre de disques de données | 16 ou moins. | La vérification est mise en échec en cas de défaut de prise en charge.
Taille de disque de données | Jusqu’à 4 095 Go | La vérification est mise en échec en cas de défaut de prise en charge.
Adaptateurs réseau | Prise en charge de plusieurs adaptateurs réseau. |
Disque dur virtuel partagé | Non pris en charge. | La vérification est mise en échec en cas de défaut de prise en charge.
Disque FC | Non pris en charge. | La vérification est mise en échec en cas de défaut de prise en charge.
BitLocker | Non pris en charge. | Vous devez désactiver BitLocker avant d’activer la réplication pour une machine.
nom de la machine virtuelle | De 1 et 63 caractères.<br/> Uniquement des lettres, des chiffres et des traits d’union.<br/><br/> Le nom de la machine doit commencer et se terminer par une lettre ou un chiffre. |  Mettez à jour la valeur dans les propriétés de machine de Site Recovery.
Se connecter après la migration - Windows | Pour vous connecter à des machines virtuelles Azure exécutant Windows après la migration :<br/><br/> - Avant la migration, activez le protocole RDP sur la machine virtuelle locale. Vérifiez que des règles TCP et UDP sont ajoutées pour le profil **Public** et que RDP est autorisé dans **Pare-feu Windows** > **Applications autorisées** pour tous les profils.<br/><br/> - Pour l’accès VPN site à site, activez RDP et autorisez RDP dans **Pare-feu Windows** -> **Applications et fonctionnalités autorisées** pour les réseaux **Domaine et Privé**. Vérifiez aussi que la stratégie SAN du système d’exploitation est définie sur **OnlineAll**. [Plus d’informations](prepare-for-migration.md) |
Se connecter après la migration - Linux | Pour vous connecter à des machines virtuelles Azure après la migration avec SSH :<br/><br/> - Avant la migration, sur la machine locale, vérifiez que le service Secure Shell est défini sur Démarrer et que les règles de pare-feu autorisent une connexion SSH.<br/><br/> - Après la migration, sur la machine virtuelle Azure, autorisez les connexions entrantes au port SSH pour les règles du groupe de sécurité réseau sur la machine virtuelle basculée, et pour le sous-réseau Azure auquel elle est connectée. Ajoutez aussi une adresse IP publique pour la machine virtuelle. |  

## <a name="next-steps"></a>Étapes suivantes

[Migrer des machines virtuelles Hyper-V](tutorial-migrate-hyper-v.md) pour la migration.
