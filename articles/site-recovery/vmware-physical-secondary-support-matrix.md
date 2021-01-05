---
title: Prise en charge de la reprise d’activité après sinistre VMware/physique sur un site secondaire avec Azure Site Recovery
description: Résume la prise en charge de la récupération d’urgence de machines virtuelles ou de serveurs physiques VMware sur un site secondaire avec Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
services: site-recovery
ms.topic: article
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: c6f0f48df32db0beb9c0a57982d9bc87b26538d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86135149"
---
# <a name="support-matrix-for-disaster-recovery-of-vmware-vms-and-physical-servers-to-a-secondary-site"></a>Matrice de prise en charge de la récupération d’urgence des machines virtuelles VMware et serveurs physiques sur un site secondaire

Cet article résume ce qui est pris en charge lorsque vous utilisez le service [Azure Site Recovery](site-recovery-overview.md) pour la récupération d’urgence de machines virtuelles VMware ou de serveurs physiques XWindows/Linux vers un site secondaire VMware.

- Si vous souhaitez répliquer des machines virtuelles VMware ou des serveurs physiques vers Azure, passez en revue [cette matrice de prise en charge](vmware-physical-azure-support-matrix.md).
- Si vous souhaitez répliquer des machines virtuelles Hyper-V vers un site secondaire, passez en revue [cette matrice de prise en charge](hyper-v-azure-support-matrix.md).

> [!NOTE]
> La réplication de machines virtuelles locales VMware et de serveurs physiques est fournie par InMage Scout. InMage Scout est inclus dans l’abonnement au service Azure Site Recovery.

## <a name="end-of-support-announcement"></a>Annonce de fin de prise en charge
Le scénario Site Recovery de réplication entre des centres de données VMware locaux ou physiques atteint la fin de la prise en charge.

- À partir d’août 2018, le scénario ne peut pas être configuré dans le coffre Recovery Services, et le logiciel InMage Scout ne peut pas être téléchargé à partir du coffre. Les déploiements existants seront pris en charge.
- - À partir du 31 décembre 2020, le scénario ne sera plus pris en charge.
Les partenaires existants peuvent intégrer de nouveaux clients au scénario jusqu’à la fin de la prise en charge.
- En 2018 et 2019, deux mises à jour seront publiées :

    - Mise à jour 7 : résout les problèmes de configuration et de conformité réseau, et prend en charge TLS 1.2.
    - Mise à jour 8 : ajoute la prise en charge des systèmes d’exploitation Linux RHEL/CentOS 7.3/7.4/7.5 et de SUSE 12
    - Après la mise à jour 8, aucune autre mise à jour ne sera publiée. La prise en charge du correctif logiciel sera limitée pour les systèmes d’exploitation ajoutés dans la mise à jour 8 et les corrections de bogues seront basées sur le meilleur effort.

## <a name="host-servers"></a>Serveurs hôtes

**Système d’exploitation** | **Détails**
--- | ---
Serveur vCenter | vCenter 5.5, 6.0 et 6.5<br/><br/> Si vous exécutez 6.0 ou 6.5, notez que seules les fonctionnalités 5.5 sont prises en charge.


## <a name="replicated-vm-support"></a>Prise en charge de la machine virtuelle répliquée

Le tableau suivant récapitule la prise en charge du système d’exploitation pour les machines répliquées avec Site Recovery. Toute charge de travail peut être exécutée sur le système d’exploitation pris en charge.

**Système d’exploitation** | **Détails**
--- | ---
Windows Server | Windows Server 2016 64 bits, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 avec au moins SP1.
Linux | Red Hat Enterprise Linux 6.7, 6.8, 6.9, 7.1, 7.2 <br/><br/> Centos 6.5, 6.6, 6.7, 6.8, 6.9, 7.0, 7.1, 7.2 <br/><br/> Oracle Enterprise Linux 6.4, 6.5, 6.8 exécutant le noyau compatible Red Hat ou Unbreakable Enterprise Kernel Release 3 (UEK3) <br/><br/> SUSE Linux Enterprise Server 11 SP3, 11 SP4 


## <a name="linux-machine-storage"></a>Stockage de machine Linux

Seules les machines Linux avec le stockage suivant peuvent être répliquées :

- Système de fichiers (EXT3, ETX4, ReiserFS, XFS).
- Logiciel multichemin-device Mapper.
- Gestionnaire de volume (LVM2).
- Les serveurs physiques avec stockage de contrôleur HP CCISS ne sont pas pris en charge.
- Le système de fichiers ReiserFS n’est pris en charge que sur SUSE Linux Enterprise Server 11 SP3.

## <a name="network-configuration---hostguest-vm"></a>Configuration du réseau - machines virtuelles hôtes/invitées

**Configuration** | **Pris en charge**  
--- | --- 
Hôte - Association de cartes réseau | Oui 
Hôte -VLAN | Oui 
Hôte - IPv4 | Oui 
Hôte - IPv6 | Non 
Machine virtuelle invitée - Association de cartes réseau | Non
Machine virtuelle invitée - IPv4 | Oui
Machine virtuelle invitée - IPv6 | Non
Machine virtuelle invitée - Windows/Linux - Adresse IP statique | Oui
Machine virtuelle invitée - Plusieurs cartes réseau | Oui


## <a name="storage"></a>Stockage

### <a name="host-storage"></a>Stockage hôte

**Stockage (hôte)** | **Pris en charge** 
--- | --- 
NFS | Oui 
SMB 3.0 | N/A 
SAN (ISCSI) | Oui 
Chemins d’accès multiples (MPIO) | Oui 

### <a name="guest-or-physical-server-storage"></a>Stockage sur serveur physique ou invité

**Configuration** | **Pris en charge** 
--- | --- 
VMDK | Oui 
VHD/VHDX | N/A 
Machine virtuelle de 2e génération | N/A 
Disque de cluster partagé | Oui 
Disque chiffré | Non 
UEFI| Oui 
NFS | Non 
SMB 3.0 | Non 
RDM | Oui 
Disque > 1 To | Oui 
Volume avec disque à bandes > 1 To<br/><br/> LVM | Oui 
Espaces de stockage | Non 
Ajout/suppression de disque à chaud | Oui 
Exclure le disque | Oui 
Chemins d’accès multiples (MPIO) | N/A 

## <a name="vaults"></a>Coffres

**Action** | **Pris en charge** 
--- | --- 
Déplacer les coffres entre plusieurs groupes de ressources (dans ou entre les différents abonnements) | Non 
Déplacer le stockage, les réseaux, les machines virtuelles Azure entre des groupes de ressources (dans ou entre les différents abonnements) | Non 

## <a name="mobility-service-and-updates"></a>Service mobilité et mises à jour

Le Service mobilité coordonne la réplication entre les serveurs VMware locaux ou les serveurs physiques et le site secondaire. Lorsque vous configurez la réplication, assurez-vous que vous avez la version la plus récente du Service mobilité et des autres composants.

| **Mettre à jour** | **Détails** |
| --- | --- |
|Mises à jour Scout | Les mises à jour Scout sont cumulatives. <br/><br/> [En savoir plus et télécharger](vmware-physical-secondary-disaster-recovery.md#updates) les dernières mises à jour Scout |
|Mises à jour de composants | Les mises à jour Scout incluent les mises à jour pour tous les composants, y compris le serveur RX, le serveur de configuration, les serveurs maîtres et process cibles, les serveurs v-Continuum et les serveurs sources à protéger.<br/><br/> [En savoir plus](vmware-physical-secondary-disaster-recovery.md#download-and-install-component-updates).|


## <a name="next-steps"></a>Étapes suivantes

Télécharger le [guide de l’utilisateur InMage Scout](https://aka.ms/asr-scout-user-guide)

- [Répliquer des machines virtuelles Hyper-V résidant dans des clouds VMM vers un site secondaire](./hyper-v-vmm-disaster-recovery.md)
- [Répliquer des machines virtuelles VMware et des serveurs physiques vers un site secondaire](./vmware-physical-secondary-disaster-recovery.md)
