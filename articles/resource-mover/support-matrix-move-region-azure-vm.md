---
title: Matrice de prise en charge pour le déplacement des machines virtuelles Azure vers une autre région à l'aide d'Azure Resource Mover
description: Passez en revue la matrice de prise en charge pour le déplacement des machines virtuelles Azure entre différentes régions à l'aide d'Azure Resource Mover.
author: rayne-wiselman
manager: evansma
ms.service: resource-move
ms.topic: how-to
ms.date: 10/11/2020
ms.author: raynew
ms.openlocfilehash: 4da707ab698599c8ea5dd8e1ea8647f543eb2a68
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95524247"
---
# <a name="support-for-moving-azure-vms-between-azure-regions"></a>Matrice de prise en charge pour le déplacement des machines virtuelles Azure entre différentes régions Azure

Cet article résume la prise en charge et les conditions préalables au déplacement des machines virtuelles et des ressources réseau associées entre différentes régions Azure à l'aide de Resource Mover.

> [!IMPORTANT]
> Azure Resource Mover est actuellement en préversion.


## <a name="windows-vm-support"></a>Prise en charge des machines virtuelles Windows

Resource Mover prend en charge les machines virtuelles Azure exécutant les systèmes d'exploitation Windows suivants.

**Système d’exploitation** | **Détails**
--- | ---
Windows Server 2019 | Pris en charge pour Server Core, Server avec Expérience utilisateur.
Windows Server 2016  | Prise en charge de Server Core, Server avec Expérience utilisateur.
Windows Server 2012 R2 | Pris en charge.
Windows Server 2012 | Pris en charge.
Windows Server 2008 R2 avec SP1/SP2 | Pris en charge.<br/><br/> Pour les machines exécutant Windows Server 2008 R2 avec SP1/SP2, vous devez installer une [mise à jour de la pile de maintenance (SSU)](https://support.microsoft.com/help/4490628) et une [mise à jour SHA-2](https://support.microsoft.com/help/4474419) Windows.  SHA-1 n’est pas pris en charge à partir de septembre 2019, et si la signature de code SHA-2 n’est pas activée, l’extension de l’agent ne sera pas installée/mise à niveau comme prévu. En savoir plus sur la [mise à niveau et la configuration requise pour SHA-2](https://aka.ms/SHA-2KB).
Windows 10 (x64) | Pris en charge.
Windows 8.1 (x64) | Pris en charge.
Windows 8 (x64) | Pris en charge.
Windows 7 (x64) avec SP1 et versions ultérieures | Installez une [mise à jour de la pile de maintenance (SSU)](https://support.microsoft.com/help/4490628) et une [mise à jour SHA-2](https://support.microsoft.com/help/4474419) Windows sur les machines exécutant Windows 7 avec SP1.  SHA-1 n'est plus pris en charge depuis septembre 2019. Si la signature de code SHA-2 n'est pas activée, l'étape de préparation échouera. En savoir plus sur la [mise à niveau et la configuration requise pour SHA-2](https://aka.ms/SHA-2KB).


## <a name="linux-vm-support"></a>Prise en charge des machines virtuelles Linux

Resource Move prend en charge les machines virtuelles Azure exécutant les systèmes d'exploitation Linux suivants.

**Système d’exploitation** | **Détails**
--- | ---
Red Hat Enterprise Linux | 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6,[7.7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [8.0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8.1
CentOS | 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 8.0, 8.1
Serveur LTS Ubuntu 14.04 | [Versions du noyau prises en charge](#supported-ubuntu-kernel-versions)
Serveur LTS Ubuntu 16.04 | [Version du noyau prise en charge](#supported-ubuntu-kernel-versions)<br/><br/> Sur les serveurs Ubuntu utilisant l'authentification et la connexion basées sur un mot de passe, ainsi que le package cloud-init pour configurer des machines virtuelles cloud, la connexion basée sur un mot de passe peut être désactivée lors du basculement (en fonction de la configuration de cloud-init). La connexion basée sur un mot de passe peut être réactivée sur la machine virtuelle en réinitialisant le mot de passe dans le menu Support > Résolution des problèmes > Paramètres (de la machine virtuelle basculée sur le portail Azure).
Serveur Ubuntu 18.04 LTS | [Version du noyau prise en charge](#supported-ubuntu-kernel-versions).
Debian 7 | [Versions du noyau prises en charge](#supported-debian-kernel-versions).
Debian 8 | [Versions du noyau prises en charge](#supported-debian-kernel-versions).
SUSE Linux Enterprise Server 12 | SP1, SP2, SP3, SP4. [Versions du noyau prises en charge](#supported-suse-linux-enterprise-server-12-kernel-versions)
SUSE Linux Enterprise Server 15 | 15 et 15 SP1. [(Versions du noyau prises en charge)](#supported-suse-linux-enterprise-server-15-kernel-versions)
SUSE Linux Enterprise Server 11 | SP3
SUSE Linux Enterprise Server 11 | SP4
Oracle Linux | 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery) <br/><br/> Exécutant le noyau compatible Red Hat ou le noyau Unbreakable Enterprise Kernel Release 3, 4 et 5 (UEK3, UEK4, UEK5)


### <a name="supported-ubuntu-kernel-versions"></a>Versions du noyau Ubuntu prises en charge

**Version release** | **Version du noyau** 
--- | --- 
14.04 LTS |  3.13.0-24-generic à 3.13.0-170-generic,<br/>3.16.0-25-generic à 3.16.0-77-generic,<br/>3.19.0-18-generic à 3.19.0-80-generic,<br/>4.2.0-18-generic à 4.2.0-42-generic,<br/>4.4.0-21-generic à 4.4.0-148-generic,<br/>4.15.0-1023-azure à 4.15.0-1045-azure 
LTS 16.04 |  4.4.0-21-generic à 4.4.0-171-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-74-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1066-azure
18.04 LTS | 4.15.0-20-generic à 4.15.0-74-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-37-generic </br> 5.3.0-19-generic à 5.3.0-24-generic </br> 4.15.0-1009-azure à 4.15.0-1037-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1028-azure </br> 5.3.0-1007-azure à 5.3.0-1009-azure


### <a name="supported-debian-kernel-versions"></a>Versions du noyau Debian prises en charge 

**Version release** |  **Version du noyau** 
--- |  --- 
Debian 7 |  3.2.0-4-amd64 à 3.2.0-6-amd64, 3.16.0-0.bpo.4-amd64 
Debian 8 |  3.16.0-4-amd64 à 3.16.0-10-amd64, 4.9.0-0.bpo.4-amd64 à 4.9.0-0.bpo.11-amd64 
Debian 8 |  3.16.0-4-amd64 à 3.16.0-10-amd64, 4.9.0-0.bpo.4-amd64 à 4.9.0-0.bpo.9-amd64

### <a name="supported-suse-linux-enterprise-server-12-kernel-versions"></a>Versions du noyau SUSE Linux Enterprise Server 12 prises en charge 

**Version release** | **Version du noyau** 
--- |  --- 
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3,SP4) |  Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.34-azure  


### <a name="supported-suse-linux-enterprise-server-15-kernel-versions"></a>Versions du noyau SUSE Linux Enterprise Server 15 prises en charge

**Version release** | **Version du noyau** |
--- |  --- |
SUSE Linux Enterprise Server 15 et 15 SP1 |  Tous les noyaux de stock SUSE 15 et 15 SP1 sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-8.22-azure |

## <a name="supported-linux-file-systemguest-storage"></a>Stockage invité/système de fichiers Linux pris en charge

* Systèmes de fichiers : ext3, ext4, XFS, BTRFS
* Gestionnaire de volume : LVM2
* Logiciel multichemin : Mappeur d’appareil


## <a name="supported-vm-compute-settings"></a>Paramètres de capacité de calcul de la machine virtuelle pris en charge

**Paramètre** | **Support** | **Détails**
--- | --- | ---
Taille | N'importe quelle taille de machine virtuelle Azure avec au moins deux cœurs d'UC et 1 Go de RAM | Consultez [Tailles de machine virtuelle Azure](../virtual-machines/sizes-general.md).
Groupes à haute disponibilité | Prise en charge | Pris en charge.
Zones de disponibilité | Prise en charge | Prises en charge, en fonction de la prise en charge de la région cible.
Images de la galerie Azure (publiées par Microsoft) | Prise en charge | Prises en charge si la machine virtuelle s’exécute sur un système d’exploitation pris en charge.
Images de la galerie Azure (publiées par un tiers)  | Prise en charge | Prises en charge si la machine virtuelle s’exécute sur un système d’exploitation pris en charge.
Images personnalisées (publiées par un tiers)| Prise en charge | Prises en charge si la machine virtuelle s’exécute sur un système d’exploitation pris en charge.
Machines virtuelles utilisant Site Recovery | Non pris en charge | Déplacez des ressources d'une région à l'autre pour les machines virtuelles, à l'aide de Site Recovery sur le back-end. Si vous utilisez déjà Site Recovery, désactivez la réplication, puis lancez le processus de préparation.
Stratégies Azure RBAC | Non pris en charge | Les stratégies de contrôle d’accès en fonction du rôle Azure (Azure RBAC) associées aux machines virtuelles ne sont pas copiées sur la machine virtuelle de la région cible.
Extensions | Non pris en charge | Les extensions ne sont pas copiées sur la machine virtuelle de la région cible. Installez-les manuellement une fois le déplacement terminé.


## <a name="supported-vm-storage-settings"></a>Paramètres de stockage des machines virtuelles pris en charge

Ce tableau récapitule la prise en charge du disque du système d’exploitation, du disque de données et du disque temporaire de la machine virtuelle Azure. Il est important d’observer les limites des disques de machine virtuelle et les cibles des machines virtuelles [Linux](../virtual-machines/linux/disk-scalability-targets.md) et [Windows](../virtual-machines/windows/disk-scalability-targets.md) pour éviter tout problème de performances.

> [!NOTE]
> La taille de la machine virtuelle cible doit être supérieure ou égale à celle de la machine virtuelle source. Les paramètres utilisés pour la validation sont les suivants : Nombre de disques de données, Nombre de cartes réseau, Processeurs disponibles, Mémoire en Go. Si ce n'est pas le cas, une erreur est émise.


**Composant** | **Support** | **Détails**
--- | --- | ---
Taille maximale du disque du système d’exploitation | 2048 GB | [En savoir plus](../virtual-machines/managed-disks-overview.md) sur les disques de machines virtuelles.
Disque temporaire | Non pris en charge | Le disque temporaire est toujours exclu du processus de préparation.<br/><br/> Ne conservez pas de données persistantes sur le disque temporaire. [Plus d’informations](../virtual-machines/managed-disks-overview.md#temporary-disk)
Taille maximale du disque de données | 8192 Go pour les disques managés
Taille minimale du disque de données |  2 Go pour les disques managés |
Nombre maximal de disques de données | Jusqu’à 64, en adéquation avec la prise en charge pour une taille spécifique de machine virtuelle Azure | [En savoir plus](../virtual-machines/sizes.md) sur les tailles de machines virtuelles.
Taux de modification du disque de données | 10 Mbits/s maximum par disque pour le stockage Premium. 2 Mbits/s maximum par disque pour le stockage Standard. | Si le taux moyen de modification des données du disque est en permanence supérieur à la valeur maximale, la préparation ne pourra pas suivre.<br/><br/>  En revanche, si la valeur maximale est dépassée de manière sporadique, la préparation suivra, mais les points de récupération risquent d'être légèrement différés.
Disque de données (compte de stockage Standard) | Non pris en charge. | Remplacez le type de stockage par Disque managé, puis essayez de déplacer la machine virtuelle.
Disque de données (compte de stockage Premium) | Non pris en charge | Remplacez le type de stockage par Disque managé, puis essayez de déplacer la machine virtuelle.
Disque managé (Standard) | Prise en charge  |
Disque managé (Premium) | Prise en charge |
SSD Standard | Prise en charge |
Génération 2 (démarrage UEFI) | Prise en charge
Compte de stockage des diagnostics de démarrage | Non pris en charge | Réactivez-le après avoir déplacé la machine virtuelle vers la région cible.

### <a name="limits-and-data-change-rates"></a>Limites et taux de changement des données

Le tableau suivant récapitule les limites qui ont été établies sur la base de nos tests. Celles-ci ne couvrent pas toutes les combinaisons possibles d'E/S d'application. Les résultats réels varient en fonction de votre combinaison d'E/S d'application. Deux limites sont à prendre en compte, le taux d'activité des données par disque et le taux d'activité des données par machine virtuelle.

**Cible de stockage** | **E/S moyennes de disque source** |**Activité des données moyenne de disque source** | **Total de l’activité des données de disque source par jour**
---|---|---|---
Stockage Standard | 8 Ko    | 2 Mo/s | 168 Go par disque
Disque Premium P10 ou P15 | 8 Ko    | 2 Mo/s | 168 Go par disque
Disque Premium P10 ou P15 | 16 Ko | 4 Mo/s |    336 Go par disque
Disque Premium P10 ou P15 | 32 Ko ou plus | 8 Mo/s | 672 Go par disque
Disque Premium P20 ou P30 ou P40 ou P50 | 8 Ko    | 5 Mo/s | 421 Go par disque
Disque Premium P20 ou P30 ou P40 ou P50 | 16 Ko ou plus |20 Mo/s | 1 684 Go par disque

## <a name="supported-vm-networking-settings"></a>Paramètres réseau des machines virtuelles pris en charge

**Paramètre** | **Support** | **Détails**
--- | --- | ---
Carte d’interface réseau | Prise en charge | Spécifiez une ressource existante dans la région cible, ou créez une nouvelle ressource pendant le processus de préparation. 
Équilibreur de charge interne | Prise en charge | Spécifiez une ressource existante dans la région cible, ou créez une nouvelle ressource pendant le processus de préparation.  
Équilibreur de charge public | Non prise en charge pour le moment | Spécifiez une ressource existante dans la région cible, ou créez une nouvelle ressource pendant le processus de préparation.  
Adresse IP publique | Prise en charge | Spécifiez une ressource existante dans la région cible, ou créez une nouvelle ressource pendant le processus de préparation.<br/><br/> L’adresse IP publique est spécifique à une région et n’est pas conservée dans la région cible après le déplacement. Gardez cela à l’esprit lorsque vous modifiez les paramètres de mise en réseau (y compris les règles d’équilibrage de charge) dans l’emplacement cible.
Groupe de sécurité réseau | Prise en charge | Spécifiez une ressource existante dans la région cible, ou créez une nouvelle ressource pendant le processus de préparation.  
Adresse IP (statique) réservée | Prise en charge | Vous ne pouvez actuellement pas configurer ce paramètre. La valeur par défaut est la valeur de la source. <br/><br/> Si la carte réseau de la machine virtuelle source possède une adresse IP statique et que la même adresse IP est disponible pour le sous-réseau cible, celle-ci est attribuée à la machine virtuelle cible.<br/><br/> Si la même adresse IP n'est pas disponible pour le sous-réseau cible, le déplacement initial de la machine virtuelle échouera.
Adresse IP dynamique | Prise en charge | Vous ne pouvez actuellement pas configurer ce paramètre. La valeur par défaut est la valeur de la source.<br/><br/> Si la carte réseau de la source possède une adresse IP dynamique, la carte réseau de la machine virtuelle cible est également dynamique par défaut.
Configurations IP | Prise en charge | Vous ne pouvez actuellement pas configurer ce paramètre. La valeur par défaut est la valeur de la source.

## <a name="outbound-access-requirements"></a>Conditions requises pour l'accès sortant

Les machines virtuelles Azure que vous souhaitez déplacer ont besoin d'un accès sortant.


### <a name="url-access"></a>accès URL

 Si vous utilisez un proxy de pare-feu basé sur des URL pour contrôler la connectivité sortante, autorisez l’accès à ces URL :

**Nom** | **Cloud public Azure** | **Détails** 
--- | --- | --- 
Stockage | `*.blob.core.windows.net`  | Permet d’écrire les données dans le compte de stockage de cache dans la région source à partir de la machine virtuelle. 
Azure Active Directory | `login.microsoftonline.com`  | Fournit l’autorisation et l’authentification aux URL du service Site Recovery. 
Réplication | `*.hypervrecoverymanager.windowsazure.com` | Permet à la machine virtuelle de communiquer avec le service Site Recovery. 
Service Bus | `*.servicebus.windows.net` | Permet à la machine virtuelle d’écrire des données de surveillance et de diagnostic Site Recovery. 

## <a name="nsg-rules"></a>règles de groupe de sécurité réseau
Si vous utilisez des règles de groupe de sécurité réseau (NSG) pour contrôler la connexion sortante, créez ces règles d'[étiquette de service](../virtual-network/service-tags-overview.md). Chaque règle doit autoriser l'accès sortant sur HTTPS (443).
- Créez une règle d'étiquette de stockage pour la région source.
- Créez une règle d'étiquette *AzureSiteRecovery* pour autoriser l'accès au service Site Recovery dans n'importe quelle région. Cette étiquette présente des dépendances avec ces autres étiquettes ; vous devez donc créer des règles correspondantes pour :
    - *AzureActiveDirectory*
    - **EventHub*
    - *AzureKeyVault*
    - *GuestAndHybridManagement*
- Nous vous recommandons de tester les règles dans un environnement hors production. [Consultez quelques exemples](../site-recovery/azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags). 


## <a name="next-steps"></a>Étapes suivantes

Essayez de [déplacer une machine virtuelle Azure](tutorial-move-region-virtual-machines.md) vers une autre région à l'aide de Resource Mover.