---
title: Matrice de support MABS et System Center DPM
description: Cet article résume la prise en charge de la Sauvegarde Azure quand vous utilisez un serveur de Sauvegarde Microsoft Azure (MABS) ou System Center DPM pour sauvegarder des ressources locales et celles de machines virtuelles Azure.
ms.date: 02/17/2019
ms.topic: conceptual
ms.openlocfilehash: 0180135da793aaf7869441ee290f6125ea88fc88
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92276961"
---
# <a name="support-matrix-for-backup-with-microsoft-azure-backup-server-or-system-center-dpm"></a>Tableau de prise en charge pour la sauvegarde avec un serveur de sauvegarde Microsoft Azure ou System Center DPM

Vous pouvez utiliser le [service Sauvegarde Azure](backup-overview.md) pour sauvegarder des machines et des charges de travail locales ainsi que des machines virtuelles Azure. Cet article récapitule les paramètres de prise en charge et les limitations associés à la sauvegarde de machines avec un serveur de sauvegarde Microsoft Azure (MABS) ou System Center Data Protection Manager (DPM) et Sauvegarde Azure.

## <a name="about-dpmmabs"></a>À propos de DPM/MABS

[System Center DPM](/system-center/dpm/dpm-overview) est une solution d'entreprise qui configure, facilite et gère la sauvegarde et la récupération des machines et des données d'entreprise. Cette solution fait partie de la suite de produits [System Center](https://www.microsoft.com/system-center/pricing).

MABS est un produit serveur qui permet de sauvegarder des serveurs physiques locaux, des machines virtuelles et les applications qu'elles exécutent.

MABS est basé sur System Center DPM et fournit des fonctionnalités similaires, à quelques différences près :

- Aucune licence System Center n’est nécessaire pour exécuter MABS.
- Azure fournit un stockage de sauvegarde à long terme pour MABS et DPM. DPM vous permet aussi de sauvegarder des données sur bande pour le stockage à long terme. MABS n’offre pas cette fonctionnalité.
- [Vous pouvez sauvegarder un serveur DPM principal avec un serveur DPM secondaire](/system-center/dpm/back-up-the-dpm-server). Le serveur secondaire protège la base de données du serveur principal et les réplicas de la source de données stockés sur le serveur principal. En cas d’échec du serveur principal, le serveur secondaire peut continuer à protéger les charges de travail qui sont protégées par le serveur principal, jusqu’à ce que le serveur principal soit de nouveau disponible.  MABS n’offre pas cette fonctionnalité.

Vous pouvez télécharger MABS à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=57520). Il peut être exécuté localement ou sur une machine virtuelle Azure.

DPM et MABS prennent en charge la sauvegarde d’un large éventail d’applications et de systèmes d’exploitation (serveur et clients). Ils proposent plusieurs scénarios de sauvegarde :

- Vous pouvez sauvegarder au niveau de la machine avec une sauvegarde de l’état du système ou une sauvegarde complète.
- Vous pouvez sauvegarder des volumes, des partages, des dossiers et des fichiers spécifiques.
- Vous pouvez sauvegarder des applications spécifiques à l'aide de paramètres optimisés prenant en compte les applications.

## <a name="dpmmabs-backup"></a>Sauvegarde DPM/MABS

La sauvegarde à l'aide de DPM/MABS et de Sauvegarde Azure fonctionne comme ceci :

1. L'agent de protection DPM/MABS est installé sur chaque machine à sauvegarder.
1. Les machines et applications sont sauvegardées dans le stockage local sur DPM/MABS.
1. L’agent MARS (Microsoft Azure Recovery Services) est installé sur le serveur DPM/MABS.
1. L'agent MARS sauvegarde les disques DPM/MABS dans un coffre Recovery Services de sauvegarde sur Azure à l'aide de Sauvegarde Azure.

Pour plus d'informations :

- [Découvrez-en plus](backup-architecture.md#architecture-back-up-to-dpmmabs) sur l’architecture MABS.
- [Passez en revue](backup-support-matrix-mars-agent.md) ce qui est pris en charge pour l'agent MARS.

## <a name="supported-scenarios"></a>Scénarios pris en charge

**Scénario** | **Agent** | **Lieu**
--- | --- | ---
**Sauvegarde de machines/charges de travail locales** | L'agent de protection DPM/MABS s'exécute sur les machines que vous souhaitez sauvegarder.<br/><br/> Agent MARS sur le serveur DPM/MABS.<br/> La version minimale de l’agent Microsoft Azure Recovery Services ou un agent Azure Backup, requise pour activer cette fonctionnalité est 2.0.8719.0.  | DPM/MABS doit s'exécuter localement.

## <a name="supported-deployments"></a>Déploiements pris en charge

DPM/MABS peut être déployé comme décrit dans le tableau suivant.

**Déploiement** | **Support** | **Détails**
--- | --- | ---
**Déploiement local** | Serveur physique<br/><br/>Machine virtuelle Hyper-V<br/><br/> Machine virtuelle VMware | Pour plus d’informations, reportez-vous à la [matrice de protection](backup-mabs-protection-matrix.md). 
**Déployé comme machine virtuelle Azure Stack** | MABS uniquement | Vous ne pouvez pas utiliser DPM pour sauvegarder des machines virtuelles Azure Stack.
**Déployé comme machine virtuelle Azure** | Protège les machines virtuelles Azure et les charges de travail qui s'exécutent sur celles-ci. | DPM/MABS exécuté dans Azure ne peut pas sauvegarder les machines locales.

## <a name="supported-mabs-and-dpm-operating-systems"></a>Systèmes d’exploitation pris en charge pour MABS et DPM

Sauvegarde Azure peut sauvegarder les instances de DPM/MABS qui exécutent l'un des systèmes d'exploitation suivants. Les systèmes d’exploitation doivent exécuter les derniers Service Packs et les dernières mises à jour.

**Scénario** | **DPM/MABS**
--- | ---
**MABS sur une machine virtuelle Azure** |  Windows 2016 Datacenter.<br/><br/> Windows 2019 Datacenter.<br/><br/> Nous vous recommandons de commencer avec une image de la Place de marché.<br/><br/> Standard_A4_v2 minimum avec quatre cœurs et 8 Go de RAM.
**DPM sur une machine virtuelle Azure** | System Center 2012 R2 avec Update 3 ou ultérieur.<br/><br/> Système d’exploitation Windows conforme aux [exigences de System Center](/system-center/dpm/prepare-environment-for-dpm#dpm-server).<br/><br/> Nous vous recommandons de commencer avec une image de la Place de marché.<br/><br/> Standard_A4_v2 minimum avec quatre cœurs et 8 Go de RAM.
**MABS localement** |  MABS v3 et versions ultérieures : Windows Server 2016 ou Windows Server 2019
**DPM localement** | Serveur physique/machine virtuelle Hyper-V : System Center 2012 SP1 ou ultérieur.<br/><br/> Machine virtuelle VMware : System Center 2012 R2 avec Update 5 ou ultérieur.

>[!NOTE]
>L'installation du serveur de sauvegarde Azure n'est pas prise en charge sur Windows Server Core ou Microsoft Hyper-V Server.

## <a name="management-support"></a>Prise en charge de la gestion

**Problème** | **Détails**
--- | ---
**Installation** | Installez DPM/MABS sur une machine à usage unique.<br/><br/> N’installez pas DPM/MABS sur un contrôleur de domaine, sur une machine sur laquelle est installé le rôle Serveur d’applications, sur une machine qui exécute Microsoft Exchange Server ou System Center Operations Manager, ou encore sur un nœud de cluster.<br/><br/> [Passez en revue toutes les exigences système de DPM](/system-center/dpm/prepare-environment-for-dpm#dpm-server).
**Domaine** | DPM/MABS doit être joint à un domaine. Effectuez d’abord l’installation, puis joignez DPM/MABS à un domaine. Le déplacement de DPM/MABS vers un nouveau domaine après le déploiement n’est pas pris en charge.
**Stockage** | Le stockage de sauvegarde moderne (MBS) est pris en charge pour DPM 2016/MABS v2 et versions ultérieures. Il n’est pas disponible pour MABS v1.
**Mise à niveau de MABS** | Vous pouvez installer directement MABS v3 ou effectuer la mise à niveau vers MABS v3 à partir de MABS v2. [Plus d’informations](backup-azure-microsoft-azure-backup.md#upgrade-mabs)
**Déplacement de MABS** | Il est possible de déplacer MABS sur un nouveau serveur tout en conservant le stockage si vous utilisez MBS.<br/><br/> Le serveur doit avoir le même nom que l’original. Vous ne pouvez pas changer le nom si vous souhaitez conserver le même pool de stockage et utiliser la même base de données MABS pour stocker les points de récupération de données.<br/><br/> Vous devez effectuer une sauvegarde de la base de données MABS, car vous devrez la restaurer.

## <a name="mabs-support-on-azure-stack"></a>Prise en charge de MABS sur Azure Stack

Vous pouvez déployer MABS sur une machine virtuelle Azure Stack pour gérer la sauvegarde des machines virtuelles et des charges de travail Azure Stack à partir d’un même emplacement.

**Composant** | **Détails**
--- | ---
**MABS sur une machine virtuelle Azure Stack** | Au moins la taille A2. Nous vous recommandons de commencer avec une image Windows Server 2012 R2 ou Windows Server 2016 de Place de marché Azure.<br/><br/> N'installez rien d'autre sur la machine virtuelle MABS.
**Stockage MABS** | Utilisez un compte de stockage distinct pour la machine virtuelle MABS. L'agent MARS exécuté sur MABS a besoin d'un stockage temporaire comme emplacement de cache et comme destination de la restauration des données du cloud.
**Pool de stockage MABS** | La taille du pool de stockage MABS est déterminée par le nombre et la taille des disques joints à la machine virtuelle MABS. Chaque taille de machine virtuelle Azure Stack correspond à un nombre maximal de disques. Par exemple, la taille A2 correspond à quatre disques.
**Conservation MABS** | Ne conservez pas les données sauvegardées sur des disques MABS locaux plus de cinq jours.
**Scale-up de MABS** | Pour effectuer un scale-up de votre déploiement, vous pouvez augmenter la taille de la machine virtuelle MABS. Par exemple, vous pouvez passer de la série A à la série D.<br/><br/> Vous pouvez également décharger régulièrement des données avec des sauvegardes vers Azure. Si nécessaire, vous pouvez déployer des serveurs MABS supplémentaires.
**.NET Framework sur MABS** | .NET Framework 3.3 SP1 ou ultérieur doit être installé sur la machine virtuelle MABS.
**Domaine MABS** | La machine virtuelle MABS doit être jointe à un domaine. Un utilisateur de domaine disposant de privilèges administratifs doit installer MABS sur la machine virtuelle.
**Sauvegarde des données d’une machine virtuelle Azure Stack** | Vous pouvez sauvegarder des fichiers, des dossiers et des applications.
**Sauvegarde prise en charge** | Vous pouvez sauvegarder les machines virtuelles exécutant les systèmes d'exploitation suivants :<br/><br/> Canal semi-annuel Windows Server (Datacenter, Enterprise, Standard)<br/><br/> Windows Server 2016, Windows Server 2012 R2, Windows Server 2008 R2
**Prise en charge de SQL Server pour les machines virtuelles Azure Stack** | Sauvegardez SQL Server 2016, SQL Server 2014 et SQL Server 2012 SP1.<br/><br/> Sauvegardez et récupérez une base de données.
**Prise en charge de SharePoint pour les machines virtuelles Azure Stack** | SharePoint 2016, SharePoint 2013, SharePoint 2010.<br/><br/> Sauvegardez et restaurez une batterie de serveurs, une base de données, un front-end et un serveur web.
**Configuration réseau requise pour les machines virtuelles sauvegardées** | Toutes les machines virtuelles de la charge de travail Azure Stack doivent appartenir au même réseau virtuel et au même abonnement.

## <a name="dpmmabs-networking-support"></a>Prise en charge du réseau pour DPM/MABS

### <a name="url-access"></a>accès URL

Le serveur DPM/MABS doit accéder à ces URL :

- `http://www.msftncsi.com/ncsi.txt`
- `*.Microsoft.com`
- `*.WindowsAzure.com`
- `*.microsoftonline.com`
- `*.windows.net`

### <a name="azure-expressroute-support"></a>Support Azure ExpressRoute

Vous pouvez sauvegarder vos données sur Azure ExpressRoute avec le Peering publique (disponible pour les anciens circuits) et le Peering Microsoft. La sauvegarde sur un peering privé n’est pas prise en charge.

Avec le Peering public : Garantissez l’accès aux domaines/adresses suivants :

- `http://www.msftncsi.com/ncsi.txt`
- `microsoft.com`
- `.WindowsAzure.com`
- `.microsoftonline.com`
- `.windows.net`

Avec le peering Microsoft, sélectionnez les services/régions et les valeurs de communauté pertinentes suivants :

- Azure Active Directory (12076:5060)
- Région Microsoft Azure (en fonction de l’emplacement de votre coffre Recovery Services)
- Stockage Azure (en fonction de l’emplacement de votre coffre Recovery Services)

Pour plus d’informations, consultez [Exigences du routage ExpressRoute](../expressroute/expressroute-routing.md).

>[!NOTE]
>Le peering public Azure est déconseillé pour les nouveaux circuits.

### <a name="dpmmabs-connectivity-to-azure-backup"></a>Connectivité de DPM/MABS à Sauvegarde Azure

La connectivité au service Sauvegarde Azure est obligatoire pour que les sauvegardes fonctionnent correctement, et l’abonnement Azure doit être actif. Le tableau suivant montre ce qui se passe si ces deux conditions ne sont pas remplies.

**MABS à Azure** | **Abonnement** | **Sauvegarde/restauration**
--- | --- | ---
Connecté | Actif | Sauvegarde sur disque DPM/MABS.<br/><br/> Sauvegarde sur Azure.<br/><br/> Restauration à partir d'un disque.<br/><br/> Restauration à partir d'Azure.
Connecté | Expiré/déprovisionné | Aucune sauvegarde sur disque ou Azure.<br/><br/> Si l'abonnement a expiré, vous pouvez restaurer à partir d'un disque ou d'Azure.<br/><br/> Si l'abonnement est désactivé, vous ne pouvez pas restaurer à partir d'un disque ou d'Azure. Les points de récupération Azure sont supprimés.
Aucune connectivité pendant plus de 15 jours | Actif | Aucune sauvegarde sur disque ou Azure.<br/><br/> Vous pouvez restaurer à partir d’un disque ou d’Azure.
Aucune connectivité pendant plus de 15 jours | Expiré/déprovisionné | Aucune sauvegarde sur disque ou Azure.<br/><br/> Si l'abonnement a expiré, vous pouvez restaurer à partir d'un disque ou d'Azure.<br/><br/> Si l'abonnement est désactivé, vous ne pouvez pas restaurer à partir d'un disque ou d'Azure. Les points de récupération Azure sont supprimés.

## <a name="domain-and-domain-trusts-support"></a>Prise en charge des domaines et des approbations de domaines

|Condition requise |Détails |
|---------|---------|
|Domaine    | Le serveur DPM/MABS doit se trouver dans un domaine Windows Server 2019, Windows Server 2016, Windows Server 2012 R2 ou Windows Server 2012.        |
|Approbation de domaines   |  DPM/MABS prend en charge la protection des données sur plusieurs forêts pour autant que vous établissiez une relation d'approbation bidirectionnelle au niveau de la forêt entre les forêts distinctes.   <BR><BR>   DPM/MABS peut protéger des serveurs et des stations de travail sur plusieurs domaines au sein d'une forêt ayant une relation d'approbation bidirectionnelle avec le domaine du serveur DPM/MABS. Pour protéger des ordinateurs dans des groupes de travail ou des domaines non approuvés, consultez [Sauvegarder et restaurer des charges de travail dans des groupes de travail et des domaines non approuvés.](/system-center/dpm/back-up-machines-in-workgroups-and-untrusted-domains)  |

## <a name="dpmmabs-storage-support"></a>Prise en charge du stockage pour DPM/MABS

Les données sauvegardées sur DPM/MABS sont stockées sur un disque local.

**Stockage** | **Détails**
--- | ---
**MBS** | Le stockage de sauvegarde moderne (MBS) est pris en charge pour DPM 2016/MABS v2 et versions ultérieures. Il n’est pas disponible pour MABS v1.
**Stockage MABS sur une machine virtuelle Azure** | Les données sont stockées sur des disques Azure joints à la machine virtuelle DPM/MABS et gérés dans DPM/MABS. Le nombre de disques utilisables pour le pool de stockage DPM/MABS est limité par la taille de la machine virtuelle.<br/><br/> Machine virtuelle A2 : 4 disques ; machine virtuelle A3 : 8 disques ; machine virtuelle A4 : 16 disques, avec une taille maximale de 1 To pour chaque disque. Ce paramètre détermine le pool de stockage de sauvegarde total disponible.<br/><br/> La quantité de données que vous pouvez sauvegarder varie selon le nombre et la taille des disques attachés.
**Conservation des données MABS sur une machine virtuelle Azure** | Nous vous recommandons de conserver les données une journée sur un disque Azure DPM/MABS et de sauvegarder les données DPM/MABS dans le coffre pour les conserver plus longtemps. Vous pouvez ainsi protéger une plus grande quantité de données en les déchargeant sur Sauvegarde Azure.

### <a name="modern-backup-storage-mbs"></a>Stockage de sauvegarde moderne (MBS)

À partir de la version DPM 2016/MABS v2 (exécuté sur Windows Server 2016), vous pouvez tirer parti du stockage de sauvegarde moderne (MBS).

- Les sauvegardes MBS sont stockées sur un disque ReFS (Resilient File System).
- MBS utilise le clonage de bloc ReFS pour accélérer les sauvegardes et optimiser l'utilisation de l'espace de stockage.
- Lorsque vous ajoutez des volumes au pool de stockage DPM/MABS local, vous les configurez avec des lettres de lecteur. Vous pouvez ensuite configurer le stockage de charge de travail sur des volumes différents.
- Quand vous créez des groupes de protection pour sauvegarder des données sur DPM/MABS, vous sélectionnez le lecteur que vous souhaitez utiliser. Par exemple, vous pouvez stocker des sauvegardes pour SQL ou d’autres charges de travail ayant des IOPS élevées sur un lecteur hautes performances, et stocker les charges de travail sauvegardées moins fréquemment sur un lecteur moins performant.

## <a name="supported-backups-to-mabs"></a>Sauvegardes prises en charge sur MABS

Pour plus d’informations sur les différents serveurs et charges de travail que vous pouvez protéger à l’aide du serveur de sauvegarde Azure, reportez-vous à la [matrice de protection du serveur de sauvegarde Azure](./backup-mabs-protection-matrix.md#protection-support-matrix).

## <a name="supported-backups-to-dpm"></a>Sauvegardes prises en charge sur DPM

Pour plus d’informations sur les différents serveurs et charges de travail que vous pouvez protéger avec Data Protection Manager, reportez-vous à l’article [Que peut sauvegarder DPM ?](/system-center/dpm/dpm-protection-matrix).

- Les charges de travail en cluster sauvegardées par DPM/MABS doivent se trouver dans le même domaine que DPM/MABS ou dans un domaine enfant/approuvé.
- Vous pouvez utiliser l’authentification NTLM/par certificat pour sauvegarder les données dans des groupes de travail ou des domaines non approuvés.

## <a name="next-steps"></a>Étapes suivantes

- [Découvrez-en plus](backup-architecture.md#architecture-back-up-to-dpmmabs) sur l’architecture MABS.
- [Passez en revue](backup-support-matrix-mars-agent.md) ce qui est pris en charge pour l’agent MARS.
- [Configurez](backup-azure-microsoft-azure-backup.md) un serveur MABS.
- [Configurez DPM](/system-center/dpm/install-dpm).
