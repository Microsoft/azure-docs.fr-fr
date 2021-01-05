---
title: Prise en charge de la récupération d’urgence des machines virtuelles Hyper-V sur Azure avec Azure Site Recovery
description: Résume les composants pris en charge et les exigences pour la récupération d’urgence de machines virtuelles Hyper-V vers Azure avec Azure Site Recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 7/14/2020
ms.author: raynew
ms.openlocfilehash: 7466f037f0a39b67023b9ebcc27c2e19b27f42ab
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485261"
---
# <a name="support-matrix-for-disaster-recovery-of-on-premises-hyper-v-vms-to-azure"></a>Prendre en charge la matrice pour effectuer une récupération d’urgence de machines virtuelles Hyper-V locales vers Azure

Cet article résume les composants pris en charge ainsi que les paramètres concernant la récupération d’urgence de machines virtuelles Hyper-V locales vers Azure, à l’aide de [Azure Site Recovery](site-recovery-overview.md).

>[!NOTE]
> Site Recovery ne déplace pas et ne stocke pas les données client en dehors de la région cible dans laquelle la récupération d’urgence a été configurée pour les ordinateurs sources. S’ils le souhaitent, les clients peuvent sélectionner un coffre Recovery Services dans une autre région. Le coffre Recovery Services contient des métadonnées, mais pas de données client réelles.

## <a name="supported-scenarios"></a>Scénarios pris en charge

**Scénario** | **Détails**
--- | ---
Hyper-V avec Virtual Machine Manager <br> <br>| Vous pouvez effectuer la récupération d’urgence vers Azure pour les machines virtuelles s’exécutant sur des hôtes Hyper-V managés dans l’infrastructure System Center Virtual Machine Manager.<br/><br/> Vous pouvez déployer ce scénario dans le portail Azure ou à l’aide de PowerShell.<br/><br/> Lorsque les hôtes Hyper-V sont managés par Virtual Machine Manager, vous pouvez également effectuer la récupération d’urgence vers un site secondaire local. Pour plus d’informations sur ce scénario, consultez [ce didacticiel](hyper-v-vmm-disaster-recovery.md).
Hyper-V sans Virtual Machine Manager | Vous pouvez effectuer la récupération d’urgence vers Azure pour les machines virtuelles s’exécutant sur les hôtes Hyper-V qui ne sont pas gérés par Virtual Machine Manager.<br/><br/> Vous pouvez déployer ce scénario dans le portail Azure ou à l’aide de PowerShell.

## <a name="on-premises-servers"></a>Serveurs locaux

**Serveur** | **Configuration requise** | **Détails**
--- | --- | ---
Hyper-V (exécuté sans Virtual Machine Manager) |  Windows Server 2019, Windows Server 2016, Windows Server 2012 R2 avec les dernières mises à jour <br/><br/> **Remarque :** L’installation Server Core de ces systèmes d’exploitation est également prise en charge. | Si vous avez déjà configuré Windows Server 2012 R2 avec/ou SCVMM 2012 R2 avec Azure Site Recovery et que vous prévoyez de mettre à niveau le système d’exploitation, suivez les instructions de la [documentation.](upgrade-2012R2-to-2016.md)
Hyper-V (exécuté avec Virtual Machine Manager) | Virtual Machine Manager 2019, Virtual Machine Manager 2016, Virtual Machine Manager 2012 R2 <br/><br/> **Remarque :** L’installation Server Core de ces systèmes d’exploitation est également prise en charge.  | Si Virtual Machine Manager est utilisé, les hôtes Windows Server 2019 doivent être managés dans Virtual Machine Manager 2019. De même, les hôtes Windows Server 2016 doivent être gérés dans Virtual Machine Manager 2016.

> [!NOTE]
> Vérifiez que .NET Framework 4.6.2 ou ultérieur est présent sur le serveur local.

## <a name="replicated-vms"></a>Machines virtuelles répliquées


Le tableau suivant récapitule la prise en charge des machines virtuelles. Site Recovery prend en charge les charges de travail s’exécutant sur un système d’exploitation pris en charge.

 **Composant** | **Détails**
--- | ---
Configuration des machines virtuelles | Les machines virtuelles qui répliquent vers Azure doivent répondre aux [conditions requises par Azure](#azure-vm-requirements).
Système d’exploitation invité | N’importe quel système d’exploitation invité [pris en charge par Azure](../cloud-services/cloud-services-guestos-update-matrix.md#family-5-releases).<br/><br/> Windows Server 2016 Nano Server n’est pas pris en charge.


## <a name="vmdisk-management"></a>Gestion des machines virtuelles/disques

**Action** | **Détails**
--- | ---
Redimensionner le disque sur la machine virtuelle Hyper-V répliquée | Non pris en charge. Désactivez la réplication, effectuez la modification, puis réactivez la réplication pour la machine virtuelle.
Ajouter un disque sur la machine virtuelle Hyper-V répliquée | Non pris en charge. Désactivez la réplication, effectuez la modification, puis réactivez la réplication pour la machine virtuelle.

## <a name="hyper-v-network-configuration"></a>Configuration réseau Hyper-V

**Composant** | **Hyper-V avec Virtual Machine Manager** | **Hyper-V sans Virtual Machine Manager**
--- | --- | ---
Réseau hôte : Association de cartes réseau | Oui | Oui
Réseau hôte : VLAN | Oui | Oui
Réseau hôte : IPv4 | Oui | Oui
Réseau hôte : IPv6 | Non | Non
Réseau machines virtuelles invitées : Association de cartes réseau | Non | Non
Réseau machines virtuelles invitées : IPv4 | Oui | Oui
Réseau machines virtuelles invitées : IPv6 | Non | Oui
Réseau machines virtuelles invitées : Adresse IP statique (Windows) | Oui | Oui
Réseau machines virtuelles invitées : Adresse IP statique (Linux) | Non | Non
Réseau machines virtuelles invitées : Plusieurs cartes réseau | Oui | Oui
Proxy HTTPS | Non | Non
Lien privé d’accès au service Site Recovery | Oui. [Plus d’informations](hybrid-how-to-enable-replication-private-endpoints.md) | Oui. [Plus d’informations](hybrid-how-to-enable-replication-private-endpoints.md)




## <a name="azure-vm-network-configuration-after-failover"></a>Configuration de réseau des machines virtuelles Azure (après basculement)

**Composant** | **Hyper-V avec Virtual Machine Manager** | **Hyper-V sans Virtual Machine Manager**
--- | --- | ---
Azure ExpressRoute | Oui | Oui
ILB | Oui | Oui
ELB | Oui | Oui
Azure Traffic Manager | Oui | Oui
Plusieurs cartes réseau | Oui | Oui
Adresse IP réservée | Oui | Oui
IPv4 | Oui | Oui
Conserver l’adresse IP source | Oui | Oui
Points de terminaison de service de réseau virtuel Azure<br/> (sans pare-feu de stockage Azure) | Oui | Oui
Mise en réseau accélérée | Non | Non


## <a name="hyper-v-host-storage"></a>Stockage hôte Hyper-V

**Stockage** | **Hyper-V avec Virtual Machine Manager** | **Hyper-V sans Virtual Machine Manager**
--- | --- | --- 
NFS | NA | NA
SMB 3.0 | Oui | Oui
SAN (ISCSI) | Oui | Oui
Chemins d’accès multiples (MPIO). Testé avec :<br></br> Microsoft DSM, EMC PowerPath 5.7 SP4, EMC PowerPath DSM pour CLARiiON | Oui | Oui

## <a name="hyper-v-vm-guest-storage"></a>Stockage invité de machines virtuelles Hyper-V

**Stockage** | **Hyper-V avec Virtual Machine Manager** | **Hyper-V sans Virtual Machine Manager**
--- | --- | ---
VMDK | NA | NA
VHD/VHDX | Oui | Oui
Machine virtuelle de 2e génération | Oui | Oui
EFI/UEFI<br></br>La machine virtuelle migrée dans Azure est automatiquement convertie en machine virtuelle de démarrage du BIOS. La machine virtuelle doit exécuter Windows Server 2012 ou une version ultérieure uniquement. Le disque du système d’exploitation ne doit pas comporter plus de cinq partitions et la taille du disque du système d’exploitation doit être inférieure à 300 Go.| Oui | Oui
Disque de cluster partagé | Non | Non
Disque chiffré | Non | Non
NFS | NA | NA
SMB 3.0 | Non | Non
RDM | N/D | N/D
Disque > 1 To | Oui, jusqu’à 4,095 Go | Oui, jusqu’à 4,095 Go
Disque : secteur logique et physique de 4 K | Non pris en charge : Gen 1/Gen 2 | Non pris en charge : Gen 1/Gen 2
Disque : secteur logique de 4 K et physique de 512 octets | Oui |  Oui
Gestion des volumes logiques (LVM). LVM est pris en charge uniquement sur des disques de données. Azure ne fournit qu’un seul disque de système d’exploitation. | Oui | Oui
Volume avec disque à bandes > 1 To | Oui | Oui
Espaces de stockage | Non | Non
Ajout/suppression de disque à chaud | Non | Non
Exclure le disque | Oui | Oui
Chemins d’accès multiples (MPIO) | Oui | Oui

## <a name="azure-storage"></a>Stockage Azure

**Composant** | **Hyper-V avec Virtual Machine Manager** | **Hyper-V sans Virtual Machine Manager**
--- | --- | ---
Stockage localement redondant | Oui | Oui
Stockage géo-redondant | Oui | Oui
Stockage géo-redondant avec accès en lecture | Oui | Oui
Stockage froid | Non | Non
Stockage chaud| Non | Non
Objets blob de blocs | Non | Non
Chiffrement au repos (SSE)| Oui | Oui
Chiffrement au repos (CMK) <br></br> (Uniquement pour le basculement vers des disques managés)| Oui (via le module PowerShell Az 3.3.0 et versions ultérieures) | Oui (via le module PowerShell Az 3.3.0 et versions ultérieures)
Double chiffrement au repos <br></br> (Uniquement pour le basculement vers des disques managés) <br></br> En savoir plus sur les régions prises en charge pour [Windows](../virtual-machines/disk-encryption.md) et [Linux](../virtual-machines/disk-encryption.md) | Oui (via le module PowerShell Az 3.3.0 et versions ultérieures) | Oui (via le module PowerShell Az 3.3.0 et versions ultérieures)
Stockage Premium | Oui | Oui
Stockage Standard | Oui | Oui
Service d’importation/exportation | Non | Non
Comptes Stockage Azure avec un pare-feu activé | Oui. Pour le stockage et le cache cibles. | Oui. Pour le stockage et le cache cibles.
Modifier le compte de stockage | Non. Le compte Stockage Azure cible ne peut pas être modifié une fois la réplication activée. Pour le modifier, désactivez puis réactivez la récupération d’urgence. | Non
Option de transfert sécurisé | Oui


## <a name="azure-compute-features"></a>Fonctionnalités de Calcul Azure

**Fonctionnalité** | **Hyper-V avec Virtual Machine Manager** | **Hyper-V sans Virtual Machine Manager**
--- | --- | ---
Groupes à haute disponibilité | Oui | Oui
HUB | Oui | Oui  
Disques managés | Oui, pour le basculement.<br/><br/> La restauration automatique des disques managés n’est pas prise en charge. | Oui, pour le basculement.<br/><br/> La restauration automatique des disques managés n’est pas prise en charge.

## <a name="azure-vm-requirements"></a>Exigences des machines virtuelles Azure

Les machines virtuelles locales que vous répliquez vers Azure doivent respecter les exigences des machines virtuelles Azure décrites dans ce tableau.

**Composant** | **Configuration requise** | **Détails**
--- | --- | ---
Système d’exploitation invité | Site Recovery fonctionne sur tous les systèmes d’exploitation [pris en charge par Azure](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc794868(v=ws.10)).  | La vérification de la configuration requise est mise en échec en cas de défaut de prise en charge.
Architecture du système d’exploitation invité | 32 bits (Windows Server 2008)/64 bits | La vérification de la configuration requise est mise en échec en cas de défaut de prise en charge.
Taille du disque du système d’exploitation | Jusqu’à 2 048 Go pour les machines virtuelles de 1ère génération.<br/><br/> Jusqu’à 300 Go pour les machines virtuelles de 2e génération.  | La vérification de la configuration requise est mise en échec en cas de défaut de prise en charge.
Nombre de disques du système d’exploitation | 1 | La vérification de la configuration requise est mise en échec en cas de défaut de prise en charge.
Nombre de disques de données | 16 ou moins  | La vérification de la configuration requise est mise en échec en cas de défaut de prise en charge.
Taille du disque dur virtuel de données | Jusqu’à 4 095 Go | La vérification de la configuration requise est mise en échec en cas de défaut de prise en charge.
Adaptateurs réseau | Prise en charge de plusieurs adaptateurs réseau. |
Disque dur virtuel partagé | Non pris en charge | La vérification de la configuration requise est mise en échec en cas de défaut de prise en charge.
Disque FC | Non pris en charge | La vérification de la configuration requise est mise en échec en cas de défaut de prise en charge.
Format de disque dur | Disque dur virtuel (VHD) <br/><br/> VHDX | Site Recovery convertit automatiquement VHDX en VHD quand vous effectuez un basculement vers Azure. Lorsque vous procédez à une restauration automatique vers un site local, les machines virtuelles continuent d’utiliser le format VHDX.
BitLocker | Non pris en charge | BitLocker doit être désactivé avant d’activer la réplication pour une machine virtuelle.
nom de la machine virtuelle | Entre 1 et 63 caractères. Uniquement des lettres, des chiffres et des traits d’union. Le nom de la machine virtuelle doit commencer et se terminer par une lettre ou un chiffre. | Mettez à jour la valeur dans les propriétés de machine virtuelle de Site Recovery.
Type de machine virtuelle | Génération 1<br/><br/> Génération 2--Windows | Les machines virtuelles de 2e génération avec un type de disque de système d’exploitation de base, qui inclut un ou deux volumes de données au format VHDX et un espace disque inférieur à 300 Go sont prises en charge.<br></br>Les machines virtuelles Linux de 2e génération ne sont pas prises en charge. [Plus d’informations](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/)|

## <a name="recovery-services-vault-actions"></a>Actions de coffre Recovery Services

**Action** |  **Hyper-V avec VMM** | **Hyper-V sans VMM**
--- | --- | ---
Déplacer le coffre entre plusieurs groupes de ressources<br/><br/> Au sein et entre des abonnements | Non | Non
Déplacer le stockage, les réseaux, les machines virtuelles Azure entre des groupes de ressources<br/><br/> Au sein et entre des abonnements | Non | Non

> [!NOTE]
> Dans le cadre de la réplication de machines virtuelles Hyper-V locales sur Azure, vous ne pouvez répliquer que sur un seul locataire AD à partir d’un environnement spécifique (site Hyper-V ou Hyper-V avec VMM selon le cas).


## <a name="provider-and-agent"></a>Fournisseur et agent

Pour vous assurer que votre déploiement est compatible avec les paramètres de cet article, vérifiez que vous utilisez le fournisseur et les versions d’agent les plus récents.

**Nom** | **Description** | **Détails**
--- | --- | --- 
Fournisseur Azure Site Recovery | Coordonne les communications entre les serveurs locaux et Azure <br/><br/> Hyper-V avec Virtual Machine Manager : Installé sur les serveurs Virtual Machine Manager<br/><br/> Hyper-V sans Virtual Machine Manager : Installé sur les hôtes Hyper-V| Version la plus récente : 5.1.2700.1 (disponible dans le portail Azure)<br/><br/> [Fonctionnalités et correctifs récents](https://support.microsoft.com/help/4091311/update-rollup-23-for-azure-site-recovery)
Agent Microsoft Azure Recovery Services | Coordonne la réplication entre les machines virtuelles Hyper-V et Azure<br/><br/> Installé sur des serveurs Hyper-V locaux (avec ou sans serveur Virtual Machine Manager) | Dernier agent disponible sur le portail






## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [préparer Azure](tutorial-prepare-azure.md) à la récupération d’urgence de machines virtuelles Hyper-V locales.