---
title: Prendre en charge la matrice de récupération d’urgence VMware/physique à l’aide d’Azure Site Recovery
description: Résume la prise en charge de la récupération d’urgence des machines virtuelles et des serveurs physiques VMware sur Azure en utilisant Azure Site Recovery.
ms.topic: conceptual
ms.date: 07/14/2020
ms.openlocfilehash: 6cc67a6af9e11e5667479eadfe277d400c2d4947
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492214"
---
# <a name="support-matrix-for-disaster-recovery--of-vmware-vms-and-physical-servers-to-azure"></a>Matrice de prise en charge de la reprise d’activité des machines virtuelles VMware et serveurs physiques sur Azure

Cet article répertorie les composants et les paramètres pris en charge pour la récupération d’urgence de machines virtuelles et de serveurs physiques VMware sur Azure en utilisant [Azure Site Recovery](site-recovery-overview.md).

- [Apprenez-en davantage](vmware-azure-architecture.md) sur l’architecture de récupération d’urgence de machine virtuelle/serveur physique VMware.
- Pour essayer la récupération d’urgence, suivez nos [didacticiels](tutorial-prepare-azure.md).

> [!NOTE]
> Site Recovery ne déplace pas et ne stocke pas les données client en dehors de la région cible dans laquelle la récupération d’urgence a été configurée pour les ordinateurs sources. S’ils le souhaitent, les clients peuvent sélectionner un coffre Recovery Services dans une autre région. Le coffre Recovery Services contient des métadonnées, mais pas de données client réelles.

## <a name="deployment-scenarios"></a>Scénarios de déploiement

**Scénario** | **Détails**
--- | ---
Récupération d’urgence des machines virtuelles VMware | Réplication de machines virtuelles VMware locales dans Azure. Vous pouvez déployer ce scénario dans le portail Azure ou à l’aide de [PowerShell](vmware-azure-disaster-recovery-powershell.md).
Récupération d’urgence des serveurs physiques | Réplication de serveurs physiques Windows/Linux locaux sur Azure. Vous pouvez déployer ce scénario dans le portail Azure.

## <a name="on-premises-virtualization-servers"></a>Serveurs de virtualisation locaux

**Serveur** | **Configuration requise** | **Détails**
--- | --- | ---
Serveur vCenter | La version 7.0 et les mises à jour ultérieures dans cette version, 6.7, 6.5, 6.0 ou 5.5 | Nous vous recommandons d’utiliser un serveur vCenter dans votre déploiement de récupération d’urgence.
Hôtes vSphere | La version 7.0 et les mises à jour ultérieures dans cette version, 6.7, 6.5, 6.0 ou 5.5 | Nous vous recommandons d’héberger les hôtes vSphere et les serveurs vCenter dans le même réseau que le serveur de traitement. Le serveur de processus s’exécute par défaut sur le serveur de configuration. [Plus d’informations](vmware-physical-azure-config-process-server-overview.md)

## <a name="site-recovery-configuration-server"></a>Serveur de configuration Site Recovery

Le serveur de configuration est une machine locale qui exécute les composants de Site Recovery, comprenant le serveur de configuration, le serveur de traitement et le serveur cible maître.

- Pour les machines virtuelles VMware, vous définissez le serveur de configuration en téléchargeant un modèle OVF pour créer une machine virtuelle VMware.
- Pour les serveurs physiques, vous définissez l’ordinateur serveur de configuration manuellement.

**Composant** | **Configuration requise**
--- |---
Cœurs d’unité centrale | 8
RAM | 16 Go
Nombre de disques | 3 disques<br/><br/> Les disques comprennent le disque du système d’exploitation, le disque de cache du serveur de traitement et le lecteur de rétention pour la restauration automatique.
Espace disque libre | 600 Go d’espace pour le cache du serveur de processus.
Espace disque libre | 600 Go d’espace pour le lecteur de rétention.
Système d’exploitation  | Windows Server 2012 R2 ou Windows Server 2016 avec l’Expérience utilisateur <br/><br> Si vous envisagez d’utiliser le serveur cible maître intégré de cet appareil pour la restauration automatique, vérifiez que la version du système d’exploitation est identique ou supérieure à celle des éléments répliqués.|
Paramètres régionaux du système d’exploitation | Anglais (en-us)
[PowerCLI](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1) | Non nécessaire pour le serveur de configuration version [9.14](https://support.microsoft.com/help/4091311/update-rollup-23-for-azure-site-recovery) ou ultérieure.
Rôles Windows Server | N’activez pas Active Directory Domain Services, Internet Information Services (IIS) ou Hyper-V
Stratégies de groupe| - Empêcher l’accès à l’invite de commandes <br/> - Empêcher l’accès aux outils de modification du Registre <br/> - Logique de confiance pour les pièces jointes <br/> - Activer l’exécution des scripts <br/> - [En savoir plus](/previous-versions/windows/it-pro/windows-7/gg176671(v=ws.10))|
IIS | Assurez-vous d’effectuer les tâches suivantes :<br/><br/> - Vérifier l’absence d’un site web par défaut préexistant <br/> - Activer [l’authentification anonyme](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731244(v=ws.10)) <br/> - Activer le paramètre [FastCGI](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753077(v=ws.10))  <br/> - Vérifier qu’aucune application/aucun site web préexistants n’écoutent le port 443<br/>
Type de carte réseau | VMXNET3 (en cas de déploiement comme machine virtuelle VMware)
Type d’adresse IP | statique
Ports | 443 utilisé pour l’orchestration du canal de contrôle<br/>9443 utilisé pour le transport de données

## <a name="replicated-machines"></a>Machines répliquées

Site Recovery assure la réplication de toutes les charges de travail exécutées sur une machine prise en charge.

**Composant** | **Détails**
--- | ---
Paramètres de la machine | Les ordinateurs qui répliquent vers Azure doivent répondre aux [conditions requises par Azure](#azure-vm-requirements).
Charge de travail de machine | Site Recovery assure la réplication de toutes les charges de travail exécutées sur une machine prise en charge. [Plus d’informations](./site-recovery-workload.md)
Nom de l'ordinateur | Vérifier que le nom d’affichage de la machine ne se trouve pas dans les [noms de ressources réservées Azure](../azure-resource-manager/templates/error-reserved-resource-name.md)<br/><br/> Les noms de volume logique ne sont pas sensibles à la casse. Vérifiez que deux volumes sur un appareil ne portent pas le même nom. Exemple : Les volumes portant les noms « voLUME1 » et « volume1 » ne peuvent pas être protégés par Azure Site Recovery.

### <a name="for-windows"></a>Pour Windows

**Système d’exploitation** | **Détails**
--- | ---
Windows Server 2019 | Pris en charge à partir du [Correctif cumulatif 34](https://support.microsoft.com/help/4490016) (version 9.22 de Mobility Service).
Windows Server 2016 64 bits | Pris en charge pour Server Core, Server avec Expérience utilisateur.
Windows Server 2012 R2/Windows Server 2012 | Pris en charge.
Windows Server 2008 R2 avec SP1 et au-delà. | Pris en charge.<br/><br/> À partir de la version [9.30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) de l’agent du service Mobility, vous devez installer les [mises à jour de la pile de maintenance (SSU)](https://support.microsoft.com/help/4490628) et [SHA-2](https://support.microsoft.com/help/4474419) sur les ordinateurs exécutant Windows 2008 R2 avec SP1 ou une version ultérieure. SHA-1 n’est pas pris en charge à partir de septembre 2019, et si la signature de code SHA-2 n’est pas activée, l’extension de l’agent ne sera pas installée/mise à niveau comme prévu. En savoir plus sur la [mise à niveau et la configuration requise pour SHA-2](https://aka.ms/SHA-2KB).
Windows Server 2008 avec SP2 ou version ultérieure (64 bits/32 bits) |  Pris en charge pour la migration uniquement. [Plus d’informations](migrate-tutorial-windows-server-2008.md)<br/><br/> À partir de la version [9.30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) de l’agent du service Mobility, vous devez installer les [mises à jour de la pile de maintenance (SSU)](https://support.microsoft.com/help/4493730) et [SHA-2](https://support.microsoft.com/help/4474419) sur les ordinateurs Windows 2008 SP2. SHA-1 n’est pas pris en charge à partir de septembre 2019, et si la signature de code SHA-2 n’est pas activée, l’extension de l’agent ne sera pas installée/mise à niveau comme prévu. En savoir plus sur la [mise à niveau et la configuration requise pour SHA-2](https://support.microsoft.com/en-us/help/4472027/2019-sha-2-code-signing-support-requirement-for-windows-and-wsus).
Windows 10, Windows 8.1, Windows 8 | Seul le système 64 bits est pris en charge. Le système 32 bits n’est pas pris en charge.
Windows 7 avec SP1 64 bits | Pris en charge à partir du [Correctif cumulatif 36](https://support.microsoft.com/help/4503156) (version 9.22 de Mobility Service). </br></br> À partir de la version [9.30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) de l’agent du service Mobility, vous devez installer les [mises à jour de la pile de maintenance (SSU)](https://support.microsoft.com/help/4490628) et [SHA-2](https://support.microsoft.com/help/4474419) sur les ordinateurs Windows 7 SP1.  SHA-1 n’est pas pris en charge à partir de septembre 2019, et si la signature de code SHA-2 n’est pas activée, l’extension de l’agent ne sera pas installée/mise à niveau comme prévu. En savoir plus sur la [mise à niveau et la configuration requise pour SHA-2](https://support.microsoft.com/en-us/help/4472027/2019-sha-2-code-signing-support-requirement-for-windows-and-wsus).

### <a name="for-linux"></a>Pour Linux

**Système d’exploitation** | **Détails**
--- | ---
Linux | Seul le système 64 bits est pris en charge. Le système 32 bits n’est pas pris en charge.<br/><br/>Les [composants Linux Integration Services (LIS)](https://www.microsoft.com/download/details.aspx?id=55106) doivent être installés sur chaque serveur Linux. Il sont nécessaires pour démarrer le serveur dans Azure après le test de basculement et le basculement. Si les [composants](https://www.microsoft.com/download/details.aspx?id=55106) LIS intégrés sont manquants, veillez à les installer avant d’activer la réplication pour les machines à démarrer dans Azure. <br/><br/> Site Recovery orchestre le basculement pour exécuter des serveurs Linux dans Azure. Toutefois, les fournisseurs Linux peuvent limiter la prise en charge aux versions de distribution qui n’ont pas atteint leur fin de vie.<br/><br/> Dans les distributions Linux, seuls les noyaux de stockage qui font partie de la version/mise à jour mineure de distribution sont pris en charge.<br/><br/> La mise à niveau des machines protégées sur des versions de distribution majeures Linux n’est pas prise en charge. Pour effectuer la mettre à niveau, désactivez la réplication, mettez à niveau le système d’exploitation, puis réactivez la réplication.<br/><br/> [Apprenez-en davantage](https://support.microsoft.com/help/2941892/support-for-linux-and-open-source-technology-in-azure) sur la prise en charge des technologies Linux et open source dans Azure.
Red Hat Enterprise Linux | 5.2 à 5.11</b><br/> 6.1 à 6.10</b> </br> 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4564347/), [version bêta 7.9](https://support.microsoft.com/help/4578241/), [7.9](https://support.microsoft.com/help/4590304/) </br> [8.0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8.1, [8.2](https://support.microsoft.com/help/4570609), [8.3](https://support.microsoft.com/help/4597409/) <br/> Les [composants Linux Integration Services (LIS)](https://www.microsoft.com/download/details.aspx?id=55106) ne sont pas préinstallés sur un petit nombre d’anciens noyaux des serveurs exécutant Red Hat Enterprise Linux 5.2 à 5.11 et 6.1 à 6.10. Si les [composants](https://www.microsoft.com/download/details.aspx?id=55106) LIS intégrés sont manquants, veillez à les installer avant d’activer la réplication pour les machines à démarrer dans Azure.
Linux : CentOS | 5.2 à 5.11</b><br/> 6.1 à 6.10</b><br/> </br> 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4564347/), [7.9](https://support.microsoft.com/help/4578241/) </br> [8.0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8.1, [8.2](https://support.microsoft.com/help/4570609), [8.3](https://support.microsoft.com/help/4597409/) <br/><br/> Les [composants Linux Integration Services (LIS)](https://www.microsoft.com/download/details.aspx?id=55106) ne sont pas préinstallés sur un petit nombre d’anciens noyaux des serveurs exécutant CentOS 5.2 à 5.11 et 6.1 à 6.10. Si les [composants](https://www.microsoft.com/download/details.aspx?id=55106) LIS intégrés sont manquants, veillez à les installer avant d’activer la réplication pour les machines à démarrer dans Azure.
Ubuntu | Serveur LTS Ubuntu 14.04* [(voir les versions du noyau prises en charge)](#ubuntu-kernel-versions)<br/>Serveur LTS Ubuntu 16.04* [(voir les versions du noyau prises en charge)](#ubuntu-kernel-versions) </br> Serveur LTS Ubuntu 18.04* [(voir les versions du noyau prises en charge)](#ubuntu-kernel-versions) </br> Serveur LTS Ubuntu 20.04* [(voir les versions du noyau prises en charge)](#ubuntu-kernel-versions) </br> (*prend en charge toutes les versions 14.04.* x *, 16.04.* x *, 18.04.* x *et 20.04.* x*)
Debian | Debian 7/Debian 8 (avec prise en charge de toutes les versions 7. *x* et 8. *x*) ; Debian 9 (prend en charge les versions 9.1 à 9.13. Debian 9.0 n’est pas pris en charge.) [(Examiner les versions du noyau prises en charge)](#debian-kernel-versions)
SUSE Linux | SUSE Linux Enterprise Server 12 SP1, SP2, SP3, SP4, [SP5](https://support.microsoft.com/help/4570609) [(voir les versions du noyau prises en charge)](#suse-linux-enterprise-server-12-supported-kernel-versions) <br/> SUSE Linux Enterprise Server 15, 15 SP1 [(voir les versions du noyau prises en charge)](#suse-linux-enterprise-server-15-supported-kernel-versions) <br/> SUSE Linux Enterprise Server 11 SP3. [Veillez à télécharger la dernière version du programme d’installation de l’agent de mobilité sur le serveur de configuration](vmware-physical-mobility-service-overview.md#download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server). </br> SUSE Linux Enterprise Server 11 SP4 </br> **Remarque** : La mise à niveau de machines répliquées de SUSE Linux Enterprise Server 11 SP3 vers SP4 n’est pas prise en charge. Pour effectuer la mise à niveau, désactivez la réplication, puis réactiver-la après la mise à niveau. <br/>|
Oracle Linux | 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4573888/), [7.9](https://support.microsoft.com/help/4597409/), [8.0](https://support.microsoft.com/help/4573888/), [8.1](https://support.microsoft.com/help/4573888/), [8.2](https://support.microsoft.com/help/4573888/), [8.3](https://support.microsoft.com/help/4597409/)  <br/> Exécutant le noyau compatible Red Hat ou le noyau Unbreakable Enterprise Kernel Release 3, 4 et 5 (UEK3, UEK4, UEK5)<br/><br/>8.1<br/>L’exécution sur tous les noyaux UEK et le noyau RedHat <= 3.10.0-1062.* est prise en charge dans la version [9.35](https://support.microsoft.com/help/4573888/). La prise en charge des autres noyaux RedHat est disponible dans la version [9.36](https://support.microsoft.com/help/4578241/)

> [!Note]
>- Pour chacune des versions de Windows, Azure Site Recovery prend uniquement en charge les builds du [canal de maintenance à long terme (LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc).  Les versions du [Canal semi-annuel](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel) ne sont actuellement pas prises en charge.
>- Vérifiez que pour les versions de Linux, Azure Site Recovery ne prend pas en charge les images de système d’exploitation personnalisées. Seuls les noyaux de stockage qui font partie de la version/mise à jour mineure de distribution sont pris en charge.

### <a name="ubuntu-kernel-versions"></a>Version du noyau Ubuntu

**Version prise en charge** | **Version du service Mobilité** | **Version du noyau** |
--- | --- | --- |
14.04 LTS | [9.35](https://support.microsoft.com/help/4573888/), [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/), [9.38](https://support.microsoft.com/help/4590304/) | 3.13.0-24-generic à 3.13.0-170-generic,<br/>3.16.0-25-generic à 3.16.0-77-generic,<br/>3.19.0-18-generic à 3.19.0-80-generic,<br/>4.2.0-18-generic à 4.2.0-42-generic,<br/>4.4.0-21-generic à 4.4.0-148-generic,<br/>4.15.0-1023-azure à 4.15.0-1045-azure |
|||
LTS 16.04 | [9.39](https://support.microsoft.com/help/4597409/) | 4.4.0-21-generic à 4.4.0-194-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-123-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1098-azure|
LTS 16.04 | [9.38](https://support.microsoft.com/help/4590304/) | 4.4.0-21-generic à 4.4.0-190-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-118-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1096-azure|
LTS 16.04 | [9.37](https://support.microsoft.com/help/4582666/) | 4.4.0-21-generic à 4.4.0-189-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-115-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1093-azure |
LTS 16.04 | [9.36](https://support.microsoft.com/help/4578241/)| 4.4.0-21-generic à 4.4.0-186-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-112-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1092-azure |
LTS 16.04 | [9.35](https://support.microsoft.com/help/4573888/) | 4.4.0-21-generic à 4.4.0-184-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-106-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1089-azure |
|||
18.04 LTS | [9.39](https://support.microsoft.com/help/4597409/) | 4.15.0-20-generic à 4.15.0-123-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-63-generic </br> 5.3.0-19-generic à 5.3.0-69-generic </br> 5.4.0-37-generic à 5.4.0-53-generic</br> 4.15.0-1009-azure à 4.15.0-1099-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1035-azure </br> 5.4.0-1020-azure à 5.4.0-1031-azure|
18.04 LTS | [9.38](https://support.microsoft.com/help/4590304/) | 4.15.0-20-generic à 4.15.0-118-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-61-generic </br> 5.3.0-19-generic à 5.3.0-67-generic </br> 5.4.0-37-generic à 5.4.0-48-generic</br> 4.15.0-1009-azure à 4.15.0-1096-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1035-azure </br> 5.4.0-1020-azure à 5.4.0-1026-azure|
18.04 LTS | [9.37](https://support.microsoft.com/help/4582666/) | 4.15.0-20-generic à 4.15.0-115-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-60-generic </br> 5.3.0-19-generic à 5.3.0-66-generic </br> 5.4.0-37-generic à 5.4.0-45-generic</br> 4.15.0-1009-azure à 4.15.0-1093-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1035-azure </br> 5.4.0-1020-azure à 5.4.0-1023-azure|
18.04 LTS | [9.36](https://support.microsoft.com/help/4578241/) | 4.15.0-20-generic à 4.15.0-112-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-58-generic </br> 5.3.0-19-generic à 5.3.0-64-generic </br> 5.4.0-37-generic à 5.4.0-42-generic</br> 4.15.0-1009-azure à 4.15.0-1092-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1032-azure </br> 5.4.0-1020-azure à 5.4.0-1022-azure|
18.04 LTS | [9.35](https://support.microsoft.com/help/4573888/) | 4.15.0-20-generic à 4.15.0-108-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-52-generic </br> 5.3.0-19-generic à 5.3.0-61-generic </br> 4.15.0-1009-azure à 4.15.0-1089-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1031-azure|
|||
20.04 LTS |[9.39](https://support.microsoft.com/help/4597409/) | 5.4.0-26-generic à 5.4.0-53 </br> -generic 5.4.0-1010-azure à 5.4.0-1031-azure
20.04 LTS |[9.38](https://support.microsoft.com/help/4590304/) | 5.4.0-26-generic à 5.4.0-48 </br> -generic 5.4.0-1010-azure à 5.4.0-1026-azure
20.04 LTS |[9.37](https://support.microsoft.com/help/4582666/) | 5.4.0-26-generic à 5.4.0-45 </br> -generic 5.4.0-1010-azure à 5.4.0-1023-azure
20.04 LTS |[9.36](https://support.microsoft.com/help/4578241/) | 5.4.0-26-generic à 5.4.0-42 </br> -generic 5.4.0-1010-azure à 5.4.0-1022-azure

### <a name="debian-kernel-versions"></a>Versions du noyau Debian


**Version prise en charge** | **Version du service Mobilité** | **Version du noyau** |
--- | --- | --- |
Debian 7 | [9.35](https://support.microsoft.com/help/4573888/), [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/), [9.38](https://support.microsoft.com/help/4590304/), [9.39](https://support.microsoft.com/help/4597409/)| 3.2.0-4-amd64 à 3.2.0-6-amd64, 3.16.0-0.bpo.4-amd64 |
|||
Debian 8 | [9.35](https://support.microsoft.com/help/4573888/), [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/), [9.38](https://support.microsoft.com/help/4590304/), [9.39](https://support.microsoft.com/help/4597409/) | 3.16.0-4-amd64 à 3.16.0-11-amd64, 4.9.0-0.bpo.4-amd64 à 4.9.0-0.bpo.11-amd64 |
|||
Debian 9.1 | [9.39](https://support.microsoft.com/help/4597409/) | 4.9.0-1-amd64 à 4.9.0-14-amd64 </br> 4.19.0-0.bpo.1-amd64 à 4.19.0-0.bpo.12-amd64 </br> 4.19.0-0.bpo.1-cloud-amd64 à 4.19.0-0.bpo.12-cloud-amd64 </br> Debian 9.1 | [9.38](https://support.microsoft.com/help/4590304/) | 4.9.0-1-amd64 à 4.9.0-13-amd64 </br> 4.19.0-0.bpo.1-amd64 à 4.19.0-0.bpo.11-amd64 </br> 4.19.0-0.bpo.1-cloud-amd64 à 4.19.0-0.bpo.11-cloud-amd64 </br> 
Debian 9.1 | [9.37](https://support.microsoft.com/help/4582666/) | 4.9.0-3-amd64 to 4.9.0-13-amd64, 4.19.0-0.bpo.6-amd64 à 4.19.0-0.bpo.10-amd64, 4.19.0-0.bpo.6-cloud-amd64 à 4.19.0-0.bpo.10-cloud-amd64

### <a name="suse-linux-enterprise-server-12-supported-kernel-versions"></a>Versions du noyau prises en charge de SUSE Linux Enterprise Server 12

**Version release** | **Version du service Mobilité** | **Version du noyau** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4, SP5) | [9.39](https://support.microsoft.com/help/4597409/) | Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.43-azure </br> 4.12.14-16.7-azure à 4.12.14-16.34-azure |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4, SP5) | [9.38](https://support.microsoft.com/help/4590304/) | Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.43-azure </br> 4.12.14-16.7-azure à 4.12.14-16.28-azure |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4, SP5) | [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/),  | Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.43-azure </br> 4.12.14-16.7-azure à 4.12.14-16.22-azure  |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4, SP5) | [9.35](https://support.microsoft.com/help/4573888/) | Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.43-azure </br> 4.12.14-16.7-azure à 4.12.14-16.19-azure  |

### <a name="suse-linux-enterprise-server-15-supported-kernel-versions"></a>Versions du noyau prises en charge de SUSE Linux Enterprise Server 15

**Version release** | **Version du service Mobilité** | **Version du noyau** |
--- | --- | --- |
SUSE Linux Enterprise Server 15, SP1, SP2 | [9.39](https://support.microsoft.com/help/4597409/)  | Par défaut, tous les [noyaux de stock SUSE 15 et 15](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure à 4.12.14-8.47-azure </br> 5.3.18-16-azure </br> 5.3.18-18.5-azure à 5.3.18-18.21-azure
SUSE Linux Enterprise Server 15, SP1, SP2 | [9.38](https://support.microsoft.com/help/4590304/)  | Par défaut, tous les [noyaux de stock SUSE 15 et 15](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure à 4.12.14-8.44-azure </br> 5.3.18-16-azure </br> 5.3.18-18.5-azure à 5.3.18-18.18-azure
SUSE Linux Enterprise Server 15 et 15 SP1 | [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/)  | Par défaut, tous les [noyaux de stock SUSE 15 et 15](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure à 4.12.14-8.38-azure
SUSE Linux Enterprise Server 15 et 15 SP1 | [9.35](https://support.microsoft.com/help/4573888/)  | Par défaut, tous les [noyaux de stock SUSE 15 et 15](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure à 4.12.14-8.33-azure 

## <a name="linux-file-systemsguest-storage"></a>Stockage invité/système de fichiers Linux

**Composant** | **Pris en charge**
--- | ---
Systèmes de fichiers | ext3, ext4, XFS, BTRFS (conditions applicables conformément à ce tableau)
Approvisionnement de la gestion des volumes logiques (LVM)| Allocation statique : oui <br></br> Allocation dynamique : non
Gestionnaire de volume | - LVM est pris en charge.<br/> - /boot sur LVM est pris en charge à partir du [Correctif cumulatif 31](https://support.microsoft.com/help/4478871/) (version 9.20 de Mobility Service). Il n’est pas pris en charge dans les versions antérieures de Mobility Service.<br/> - Les disques de système d’exploitation multiples ne sont pas pris en charge.
Dispositif de stockage paravirtualisé | Les appareils exportés par les pilotes paravirtualisés ne sont pas pris en charge.
Unités de bloc d’entrée et de sortie en file d’attente | Non pris en charge.
Serveurs physiques avec le contrôleur de stockage HP CCISS | Non pris en charge.
Convention de nommage pour les appareils/points de montage | Le nom de l’appareil ou le nom du point de montage doit être unique.<br/> Vérifiez qu’aucun appareil ou point de montage ne possède un nom sensible à la casse. Par exemple, nommer la même machine virtuelle *appareil1* et *Appareil1* n’est pas pris en charge.
Répertoires | Si vous exécutez une version de Mobility Service antérieure à la version 9.20 (publiée dans le [Correctif cumulatif 31](https://support.microsoft.com/help/4478871/)), les restrictions suivantes s’appliquent :<br/><br/> - Les répertoires suivants (s’ils sont configurés en tant que partitions/systèmes de fichiers séparés) doivent se trouver sur le même disque de système d’exploitation, sur le serveur source : /(root), /boot, /usr, /usr/local, /var, /etc.</br> - Le répertoire /boot doit se trouver sur une partition de disque et ne doit pas être un volume LVM.<br/><br/> À partir de la version 9.20, ces restrictions ne s’appliquent pas. 
Répertoire de démarrage | - Les disques de démarrage ne doivent pas être au format de partition GPT. Il s’agit d’une limitation d’architecture Azure. Les disques GPT sont pris en charge en tant que disques de données.<br/><br/> Plusieurs disques de démarrage ne sont pas pris en charge sur une machine virtuelle<br/><br/> - /boot sur un volume LVM s’étendant sur plusieurs disques n’est pas pris en charge.<br/> - Une machine sans un disque de démarrage ne peuvent pas être répliquée.
Espace libre requis| 2 Go sur la partition/root <br/><br/> 250 Mo dans le dossier d’installation
XFSv5 | Les fonctionnalités XFSv5 sur des systèmes de fichiers XFS, tels que les sommes de contrôle de métadonnées, sont prises en charge (à partir de la version 9.10 de Mobility Service).<br/> Utilisez l’utilitaire xfs_info pour vérifier le superbloc XFS pour la partition. Si `ftype` est défini sur 1, les fonctionnalités XFSv5 sont utilisées.
BTRFS | BTRFS est pris en charge à partir du [Correctif cumulatif 34](https://support.microsoft.com/help/4490016) (version 9.22 de Mobility Service). BTRFS n’est pas pris en charge si :<br/><br/> - Le sous-volume du système de fichiers BTRFS est modifié après l’activation de la protection.</br> - Le système de fichiers BTRFS s’étend sur plusieurs disques.</br> - Le système de fichiers BTRFS prend en charge RAID.

## <a name="vmdisk-management"></a>Gestion des machines virtuelles/disques

**Action** | **Détails**
--- | ---
Redimensionner le disque sur la machine virtuelle répliquée | Pris en charge sur la machine virtuelle source avant le basculement, directement dans les propriétés de la machine virtuelle. Vous n’avez pas besoin de désactiver/réactiver la réplication.<br/><br/> Si vous modifiez la machine virtuelle source après le basculement, les modifications ne sont pas capturées.<br/><br/> Si vous modifiez la taille du disque sur la machine virtuelle Azure après le basculement, lorsque vous effectuez une restauration automatique, Site Recovery crée une machine virtuelle avec les mises à jour.
Ajouter un disque à la machine virtuelle répliquée | Non pris en charge.<br/> Désactivez la réplication pour la machine virtuelle, ajoutez le disque, puis réactivez la réplication.

> [!NOTE]
> Aucune modification de l’identité du disque n’est prise en charge. Par exemple, si le partitionnement du disque a été modifié de GPT en MBR ou vice versa, cela modifie l’identité du disque. Dans ce scénario, la réplication s’interrompt et une nouvelle configuration est nécessaire. 

## <a name="network"></a>Réseau

**Composant** | **Pris en charge**
--- | ---
Association de cartes réseau de réseau hôte | Pris en charge pour les machines virtuelles VMware <br/><br/>Non pris en charge pour la réplication des machines physiques
Réseau hôte VLAN | Oui.
Réseau hôte IPv4 | Oui.
Réseau hôte IPv6 | Non.
Association de cartes de réseau invité/serveur | Non.
Réseau invité/serveur IPv4 | Oui.
Réseau invité/serveur IPv6 | Non.
Adresse IP statique du réseau invité/serveur (Windows) | Oui.
Adresse IP statique du réseau invité/serveur (Linux) | Oui. <br/><br/>Les machines virtuelles sont configurées pour utiliser le protocole DHCP lors de la restauration automatique.
Plusieurs cartes réseau invité/serveur | Oui.
Lien privé d’accès au service Site Recovery | Oui. [Plus d’informations](hybrid-how-to-enable-replication-private-endpoints.md)


## <a name="azure-vm-network-after-failover"></a>Réseau de machines virtuelles Azure (après le basculement)

**Composant** | **Pris en charge**
--- | ---
Azure ExpressRoute | Oui
ILB | Oui
ELB | Oui
Azure Traffic Manager | Oui
Plusieurs cartes réseau | Oui
Adresses IP réservées | Oui
IPv4 | Oui
Conserver l’adresse IP source | Oui
Points de terminaison de service de réseau virtuel Azure<br/> | Oui
Mise en réseau accélérée | Non

## <a name="storage"></a>Stockage
**Composant** | **Pris en charge**
--- | ---
Disque dynamique | Le disque du système d’exploitation doit être un disque de base. <br/><br/>Les disques de données peuvent être des disques dynamiques
Configuration des disques Docker | Non
Hôte NFS | Oui pour VMware<br/><br/> Non pour les serveurs physiques
Hôte SAN (iSCSI/FC) | Oui
vSAN hôte | Oui pour VMware<br/><br/> N/A pour les serveurs physiques
Multipath hôte (MPIO) | Oui, testé avec : Microsoft DSM, EMC PowerPath 5.7 SP4, EMC PowerPath DSM pour CLARiiON
Volumes virtuels hôtes (VVols) | Oui pour VMware<br/><br/> N/A pour les serveurs physiques
VMDK invité/serveur | Oui
Disque de cluster partagé invité/serveur | Non
Disque chiffré invité/serveur | Non
NFS invité/serveur | Non
iSCSI invité/serveur | Pour la migration : Oui<br/>Pour la récupération d’urgence : Non. iSCSI est automatiquement restauré en tant que disque attaché à la machine virtuelle
SMB 3.0 invité/serveur | Non
RDM invité/serveur | Oui<br/><br/> N/A pour les serveurs physiques
Disque invité/serveur > 1 To | Oui, le disque doit être d’une taille supérieure à 1024 Mo<br/><br/>Jusqu'à 8 192 Go lors de la réplication sur des disques managés (à partir de la version 9.26)<br></br> Jusqu'à 4 095 Go lors de la réplication vers des comptes de stockage
Disque invité/serveur avec une taille de secteur logique de 4 Ko et une taille de secteur physique de 4 K | Non
Disque invité/serveur avec une taille de secteur logique de 4 K et une taille de secteur physique de 512 octets | Non
Volume invité/serveur avec disque à bandes > 4 To | Oui
Gestion des volumes logiques (LVM)| Allocation statique : oui <br></br> Allocation dynamique : non
Invité/serveur - Espaces de stockage | Non
Ajout/retrait à chaud de disque d’Invité/de serveur | Non
Invité/serveur - Exclure le disque | Oui
Multipath invité/serveur (MPIO) | Non
Partitions GPT invité/serveur | Cinq partitions sont prises en charge à partir du [Correctif cumulatif 37](https://support.microsoft.com/help/4508614/) (version 9.25 de Mobility Service). Auparavant, elles étaient au nombre de quatre.
ReFS | Resilient File System est pris en charge avec le service Mobility version 9.23 ou ultérieure
Démarrage EFI/UEFI invité/serveur | - Pris en charge pour tous les [systèmes d'exploitation UEFI de la Place de marché Azure](../virtual-machines/generation-2.md#generation-2-vm-images-in-azure-marketplace) avec l'agent de mobilité Site Recovery 9.30 et versions ultérieures. <br/> - Le type de démarrage UEFI sécurisé n’est pas pris en charge. [En savoir plus.](../virtual-machines/generation-2.md#on-premises-vs-azure-generation-2-vms)

## <a name="replication-channels"></a>Canaux de réplication

|**Type de réplication**   |**Pris en charge**  |
|---------|---------|
|Transferts de données déchargées (ODX)    |       Non  |
|Amorçage hors connexion        |   Non      |
| Azure Data Box | Non

## <a name="azure-storage"></a>Stockage Azure

**Composant** | **Pris en charge**
--- | ---
Stockage localement redondant | Oui
Stockage géo-redondant | Oui
Stockage géo-redondant avec accès en lecture | Oui
Stockage froid | Non
Stockage chaud| Non
Objets blob de blocs | Non
Chiffrement au repos (SSE)| Oui
Chiffrement au repos (CMK)| Oui (via le module PowerShell Az 3.3.0 et versions ultérieures)
Double chiffrement au repos | Oui (par le biais du module PowerShell Az 3.3.0 et versions ultérieures). Découvrez-en plus sur les régions prises en charge pour [Windows](../virtual-machines/disk-encryption.md) et [Linux](../virtual-machines/disk-encryption.md).
Stockage Premium | Oui
Option de transfert sécurisé | Oui
Service Import/Export | Non
Pare-feu de Stockage Azure pour les réseaux virtuels | Oui.<br/> Configurés sur le compte de stockage de cache/stockage cible (utilisé pour stocker les données de réplication).
Comptes de stockage v2 à usage général (niveaux chaud et froid) | Oui (les coûts de transaction sont sensiblement plus élevées pour la version V2 que pour la version V1)

## <a name="azure-compute"></a>Calcul Azure

**Fonctionnalité** | **Pris en charge**
--- | ---
Groupes à haute disponibilité | Oui
Zones de disponibilité | Non
HUB | Oui
Disques managés | Oui

## <a name="azure-vm-requirements"></a>Exigences des machines virtuelles Azure

Les machines virtuelles locales répliquées sur Azure doivent respecter les exigences des machines virtuelles Azure décrites dans ce tableau. Quand Site Recovery vérifie les prérequis pour la réplication, la vérification échoue si certaines exigences ne sont pas satisfaites.

**Composant** | **Configuration requise** | **Détails**
--- | --- | ---
Système d’exploitation invité | Vérifiez les [systèmes d’exploitation pris en charge](#replicated-machines) pour les machines répliquées. | La vérification est mise en échec en cas de défaut de prise en charge.
Architecture du système d’exploitation invité | 64 bits. | La vérification est mise en échec en cas de défaut de prise en charge.
Taille du disque du système d’exploitation | Jusqu’à 2 048 Go. | La vérification est mise en échec en cas de défaut de prise en charge.
Nombre de disques du système d’exploitation | 1 </br> une partition système et de démarrage sur différents disques n’est pas prise en charge | La vérification est mise en échec en cas de défaut de prise en charge.
Nombre de disques de données | 64 ou moins. | La vérification est mise en échec en cas de défaut de prise en charge.
Taille de disque de données | Jusqu'à 8 192 Go lors de la réplication sur un disque managé (à partir de la version 9.26)<br></br>Jusqu'à 4 095 Go lors de la réplication vers un compte de stockage| La vérification est mise en échec en cas de défaut de prise en charge.
Adaptateurs réseau | Prise en charge de plusieurs adaptateurs réseau. |
Disque dur virtuel partagé | Non pris en charge. | La vérification est mise en échec en cas de défaut de prise en charge.
Disque FC | Non pris en charge. | La vérification est mise en échec en cas de défaut de prise en charge.
BitLocker | Non pris en charge. | Vous devez désactiver BitLocker avant d’activer la réplication pour une machine. |
nom de la machine virtuelle | De 1 et 63 caractères.<br/><br/> Uniquement des lettres, des chiffres et des traits d’union.<br/><br/> Le nom de la machine doit commencer et se terminer par une lettre ou un chiffre. |  Mettez à jour la valeur dans les propriétés de machine de Site Recovery.

## <a name="resource-group-limits"></a>Limites de groupe de ressources

Pour plus d’informations sur le nombre de machines virtuelles pouvant être protégées sous un groupe de ressources unique, reportez-vous à l’article sur les [limites et quotas d’abonnement](../azure-resource-manager/management/azure-subscription-service-limits.md#resource-group-limits).

## <a name="churn-limits"></a>Limites d’activité

Le tableau suivant présente les limites d’Azure Site Recovery.
- Ces limites sont basées sur nos tests, mais ne couvrent pas toutes les combinaisons d’E/S d’application possibles.
- Les résultats réels varient en fonction de la combinaison d’E/S de votre application.
- Pour des résultats optimaux, nous recommandons d’exécuter l’[outil Planificateur de déploiement](site-recovery-deployment-planner.md) et d’effectuer des tests d’application étendus, notamment des tests de basculement pour obtenir une image réelle des performances de votre application.

**Cible de réplication** | **Taille d’E/S moyenne de disque source** |**Activité des données moyenne de disque source** | **Total de l’activité des données de disque source par jour**
---|---|---|---
Stockage Standard | 8 Ko    | 2 Mo/s | 168 Go par disque
Disque Premium P10 ou P15 | 8 Ko    | 2 Mo/s | 168 Go par disque
Disque Premium P10 ou P15 | 16 Ko | 4 Mo/s |    336 Go par disque
Disque Premium P10 ou P15 | 32 Ko ou plus | 8 Mo/s | 672 Go par disque
Disque Premium P20 ou P30 ou P40 ou P50 | 8 Ko    | 5 Mo/s | 421 Go par disque
Disque Premium P20 ou P30 ou P40 ou P50 | 16 Ko ou plus |20 Mo/s | 1 684 Go par disque


**Activité de données sources** | **Limite maximale**
---|---
Activité des données de pointe sur tous les disques d’une machine virtuelle | 54 Mo/s
Activité des données maximale par jour prise en charge par un serveur de processus | 2 To

- Il s’agit de moyennes en partant sur un chevauchement d’E/S de 30 pour cent.
- Site Recovery est capable de gérer un débit plus élevé en fonction du ratio de chevauchement, de tailles d’écriture plus grandes et du comportement d’E/S des charges de travail réelles.
- Ces valeurs supposent un backlog typique d’environ cinq minutes. Autrement dit, une fois que les données sont chargées, elles sont traitées, et un point de récupération est créé dans un délai de cinq minutes.

## <a name="vault-tasks"></a>Tâches de coffre

**Action** | **Pris en charge**
--- | ---
Déplacer le coffre entre plusieurs groupes de ressources | Non
Déplacer le coffre au sein des abonnements et entre ceux-ci | Non
Déplacer le stockage, les réseaux, les machines virtuelles Azure entre des groupes de ressources | Non
Déplacer le stockage, le réseau et les machines virtuelles Azure au sein des abonnements et entre ceux-ci | Non


## <a name="obtain-latest-components"></a>Obtenir les composants les plus récents

**Nom** | **Description** | **Détails**
--- | --- | ---
Serveur de configuration | Installé en local.<br/> Coordonne les communications entre les serveurs ou machines physiques VMware locaux et Azure. | - [Apprenez-en davantage sur](vmware-physical-azure-config-process-server-overview.md) le serveur de configuration.<br/> - [Apprenez-en davantage sur](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server) la mise à niveau vers la dernière version.<br/> - [Apprenez-en davantage sur](vmware-azure-deploy-configuration-server.md) l’installation du serveur de configuration.
Serveur de traitement | Installé par défaut sur le serveur de configuration.<br/> Reçoit les données de réplication, les optimise avec une mise en cache, une compression et un chiffrement, puis les envoie à Azure.<br/> À mesure que croît votre déploiement, vous pouvez ajouter des serveurs de traitement afin de gérer de plus grands volumes de trafic de réplication. | - [Apprenez-en davantage sur](vmware-physical-azure-config-process-server-overview.md) le serveur de processus.<br/> - [Apprenez-en davantage sur](vmware-azure-manage-process-server.md#upgrade-a-process-server) la mise à niveau vers la dernière version.<br/> - [Apprenez-en davantage sur](vmware-physical-large-deployment.md#set-up-a-process-server) la configuration de serveurs de processus scale-out.
Service Mobilité | Installé sur une machine virtuelle ou des serveurs physiques VMware que vous souhaitez répliquer.<br/> Coordonne la réplication entre les serveurs VMware/serveurs physiques et Azure.| - [Apprenez-en davantage sur](vmware-physical-mobility-service-overview.md) Mobility Service.<br/> - [Apprenez-en davantage sur](vmware-physical-manage-mobility-service.md#update-mobility-service-from-azure-portal) la mise à niveau vers la dernière version.<br/>



## <a name="next-steps"></a>Étapes suivantes
[Découvrez comment](tutorial-prepare-azure.md) préparer Azure à la récupération d’urgence de machines virtuelles VMware.

[9.32 UR]: https://support.microsoft.com/en-in/help/4538187/update-rollup-44-for-azure-site-recovery
[9.31 UR]: https://support.microsoft.com/en-in/help/4537047/update-rollup-43-for-azure-site-recovery
[9.30 UR]: https://support.microsoft.com/en-in/help/4531426/update-rollup-42-for-azure-site-recovery
[9.29 UR]: https://support.microsoft.com/en-in/help/4528026/update-rollup-41-for-azure-site-recovery
[9.28 UR]: https://support.microsoft.com/en-in/help/4521530/update-rollup-40-for-azure-site-recovery
[9.27 UR]: https://support.microsoft.com/en-in/help/4517283/update-rollup-39-for-azure-site-recovery
[9.26 UR]: https://support.microsoft.com/en-in/help/4513507/update-rollup-38-for-azure-site-recovery
[9.25 UR]: https://support.microsoft.com/en-in/help/4508614/update-rollup-37-for-azure-site-recovery
[9.24 UR]: https://support.microsoft.com/en-in/help/4503156
[9.23 UR]: https://support.microsoft.com/en-in/help/4494485/update-rollup-35-for-azure-site-recovery
[9.22 UR]: https://support.microsoft.com/help/4489582/update-rollup-33-for-azure-site-recovery
[9.21 UR]: https://support.microsoft.com/help/4485985/update-rollup-32-for-azure-site-recovery
[9.20 UR]: https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery
[9.19 UR]: https://support.microsoft.com/help/4468181/azure-site-recovery-update-rollup-30
[9.18 UR]: https://support.microsoft.com/help/4466466/update-rollup-29-for-azure-site-recovery