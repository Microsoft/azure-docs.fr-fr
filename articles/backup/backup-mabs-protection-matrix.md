---
title: Matrice de protection MABS V3 UR1 (serveur de sauvegarde Azure)
description: Cet article fournit une matrice de prise en charge répertoriant toutes les charges de travail, les types de données et les installations que protège le serveur de sauvegarde Azure.
ms.date: 03/19/2020
ms.topic: conceptual
ms.openlocfilehash: d37d51ee781dfbc5bcd56fa3158a622202a979c6
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754179"
---
# <a name="mabs-azure-backup-server-v3-ur1-protection-matrix"></a>Matrice de protection MABS V3 UR1 (serveur de sauvegarde Azure)

Cet article répertorie les différents serveurs et charges de travail que vous pouvez protéger avec le serveur de sauvegarde Azure. La matrice suivante répertorie ce qui peut être protégé avec le serveur de sauvegarde Azure.

Utilisez la matrice suivante pour MABS v3 UR1 :

* Charges de travail : type de technologie pour la charge de travail.

* Version : version de MABS prise en charge pour les charges de travail.

* Installation de MABS : ordinateur/emplacement où vous souhaitez installer MABS.

* Protection et récupération : listez les informations détaillées sur les charges de travail, telles que le conteneur de stockage pris en charge ou le déploiement pris en charge.

>[!NOTE]
>La prise en charge de l’agent de protection 32 bits est déconseillée avec MABS v3 UR1. Consultez [Dépréciation de l’agent de protection 32 bits](backup-mabs-whats-new-mabs.md#32-bit-protection-agent-deprecation).

## <a name="protection-support-matrix"></a>Matrice de prise en charge de la protection

Les sections suivantes détaillent la matrice de prise en charge de protection pour MABS :

* [Sauvegarde des applications](#applications-backup)
* [Sauvegarde des machines virtuelles](#vm-backup)
* [Linux](#linux)

## <a name="applications-backup"></a>Sauvegarde des applications

| **Charge de travail**               | **Version**                                                  | **Installation du serveur de sauvegarde Azure**                       | **Serveur de sauvegarde Azure pris en charge** | **Protection et récupération**                                  |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------- | ------------------------------------------------------------ |
| Ordinateurs clients (64 bits) | Windows 10                                                  | Serveur physique  <br><br>    Machine virtuelle Hyper-V   <br><br>   Machine virtuelle VMware | V3 UR1                            | Volume, partage, dossier, fichiers, volumes dédupliqués   <br><br>   Les volumes protégés doivent être au format NTFS. Les formats FAT et FAT32 ne sont pas pris en charge.  <br><br>    Les volumes doivent être d’au moins 1 Go. Le serveur de sauvegarde Azure utilise le service VSS pour prendre un l’instantané des données, et cet instantané fonctionne seulement si le volume est d’au moins 1 Go. |
| Serveurs (64 bits)          | Windows Server 2019, 2016, 2012 R2, 2012                    | Machine virtuelle Azure (quand la charge de travail s’exécute en tant que machine virtuelle Azure)  <br><br>    Serveur physique  <br><br>    Machine virtuelle Hyper-V <br><br>     Machine virtuelle VMware  <br><br>    Azure Stack | V3 UR1                            | Volume, partage, dossier, fichier <br><br>    Volumes dédupliqués (NTFS uniquement)  <br><br>   État système et sauvegarde complète (non pris en charge quand la charge de travail s’exécute en tant que machine virtuelle Azure) |
| Serveurs (64 bits)          | Windows Server 2008 R2 SP1, Windows Server 2008 SP2 (vous devez installer [Windows Management Framework](https://www.microsoft.com/download/details.aspx?id=54616)) | Serveur physique  <br><br>    Machine virtuelle Hyper-V  <br><br>      Machine virtuelle VMware  <br><br>   Azure Stack | V3 UR1                            | Volume, partage, dossier, fichier, état système/sauvegarde complète        |
| SQL Server                | SQL Server 2019, 2017, 2016 et [Service Packs pris en charge](https://support.microsoft.com/lifecycle/search?alpha=SQL%20Server%202016), 2014 et [Service Packs](https://support.microsoft.com/lifecycle/search?alpha=SQL%20Server%202014) pris en charge | Serveur physique  <br><br>     Machine virtuelle Hyper-V   <br><br>     Machine virtuelle VMware  <br><br>   Machine virtuelle Azure (quand la charge de travail s’exécute en tant que machine virtuelle Azure)  <br><br>     Azure Stack | V3 UR1                            | Tous les scénarios de déploiement : base de données       <br><br>  MABS v3 UR1 prend en charge la sauvegarde des bases de données SQL sur les volumes ReFS                  |
| Exchange                   | Exchange 2019, 2016                                         | Serveur physique   <br><br>   Machine virtuelle Hyper-V  <br><br>      Machine virtuelle VMware  <br><br>   Azure Stack  <br><br>    Machine virtuelle Azure (quand la charge de travail s’exécute en tant que machine virtuelle Azure) | V3 UR1                            | Protéger (tous les scénarios de déploiement) : serveur Exchange autonome, base de données dans un groupe de disponibilité de base de données  <br><br>    Récupérer (tous les scénarios de déploiement) : boîte aux lettres, bases de données de boîtes aux lettres dans un groupe de disponibilité de base de données    <br><br>  La sauvegarde d’Exchange sur ReFS est prise en charge avec MABS v3 UR1 |
| SharePoint                 | SharePoint 2019, 2016 avec les derniers Service Packs                       | Serveur physique  <br><br>    Machine virtuelle Hyper-V <br><br>    Machine virtuelle VMware  <br><br>   Machine virtuelle Azure (quand la charge de travail s’exécute en tant que machine virtuelle Azure)   <br><br>   Azure Stack | V3 UR1                            | Protéger (tous les scénarios de déploiement) : batterie de serveurs, contenu de serveur web front-end  <br><br>    Récupérer (tous les scénarios de déploiement) : batterie de serveurs, base de données, application web, élément de fichier ou de liste, recherche SharePoint, serveur web front-end  <br><br>    La protection d’une batterie de serveurs SharePoint qui utilise la fonctionnalité AlwaysOn de SQL Server 2012 pour les bases de données de contenu n’est pas prise en charge. |

## <a name="vm-backup"></a>Sauvegarde des machines virtuelles

| **Charge de travail**                                                 | **Version**                                             | **Installation du serveur de sauvegarde Azure**                      | **Serveur de sauvegarde Azure pris en charge** | **Protection et récupération**                                 |
| ------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| Hôte Hyper-V : agent de protection MABS sur un serveur hôte, un cluster ou une machine virtuelle Hyper-V | Windows Server 2019, 2016, 2012 R2, 2012               | Serveur physique  <br><br>    Machine virtuelle Hyper-V <br><br>    Machine virtuelle VMware | V3 UR1                             | Protéger :  ordinateurs Hyper-V, volumes partagés de cluster  <br><br>    Récupérer : machine virtuelle, récupération au niveau élément de fichiers et de dossiers disponible uniquement pour Windows, volumes, disques durs virtuels |
| Machines virtuelles VMware                                                  | Serveur VMware 5.5, 6.0 ou 6.5, 6.7 (version sous licence) | Machine virtuelle Hyper-V  <br><br>   Machine virtuelle VMware         | V3 UR1                             | Protéger :  Machines virtuelles VMware sur volumes partagés de cluster (CSV), NFS et stockage SAN   <br><br>     Récupérer :  machine virtuelle, récupération au niveau élément de fichiers et de dossiers disponible uniquement pour Windows, volumes, disques durs virtuels <br><br>    Les applications virtuelles VMware ne sont pas prises en charge. |

## <a name="linux"></a>Linux

| **Charge de travail** | **Version**                               | **Installation du serveur de sauvegarde Azure**                      | **Serveur de sauvegarde Azure pris en charge** | **Protection et récupération**                                 |
| ------------ | ----------------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| Linux        | Linux s’exécutant en tant qu’invité [Hyper-V](back-up-hyper-v-virtual-machines-mabs.md) ou [VMware](backup-azure-backup-server-vmware.md) | Serveur physique, machine virtuelle Hyper-V locale, machine virtuelle Windows dans VMWare | V3 UR1                             | Hyper-V doit s’exécuter sur Windows Server 2012 R2, Windows Server 2016 ou Windows Server 2019. Protéger :  toute la machine virtuelle   <br><br>   Récupérer : toute la machine virtuelle   <br><br>    Seuls les instantanés cohérents au niveau des fichiers sont pris en charge.    <br><br>   Pour obtenir la liste complète des versions et distributions Linux prises en charge, consultez l’article [Linux sur les distributions approuvées par Azure](../virtual-machines/linux/endorsed-distros.md). |

## <a name="azure-expressroute-support"></a>Support Azure ExpressRoute

Vous pouvez sauvegarder vos données sur Azure ExpressRoute avec le Peering publique (disponible pour les anciens circuits) et le Peering Microsoft. La sauvegarde sur un peering privé n’est pas prise en charge.

Avec le Peering public : Garantissez l’accès aux domaines/adresses suivants :

* `http://www.msftncsi.com/ncsi.txt`
* `microsoft.com`
* `.WindowsAzure.com`
* `.microsoftonline.com`
* `.windows.net`

Avec le peering Microsoft, sélectionnez les services/régions et les valeurs de communauté pertinentes suivants :

* Azure Active Directory (12076:5060)
* Région Microsoft Azure (en fonction de l’emplacement de votre coffre Recovery Services)
* Stockage Azure (en fonction de l’emplacement de votre coffre Recovery Services)

Pour plus d’informations, consultez [Exigences du routage ExpressRoute](../expressroute/expressroute-routing.md).

>[!NOTE]
>Le peering public Azure est déconseillé pour les nouveaux circuits.

## <a name="cluster-support"></a>Prise en charge des clusters

Le serveur de sauvegarde Azure peut protéger les données dans les applications en cluster suivantes :

* Serveurs de fichiers

* SQL Server

* Hyper-V : si vous protégez un cluster Hyper-V en utilisant l’agent de protection MABS avec montée en charge, vous ne pouvez pas ajouter de protection secondaire pour les charges de travail Hyper-V protégées.

* Exchange Server : le serveur de sauvegarde Azure peut protéger des clusters de disques non partagés pour les versions d’Exchange Server prises en charge (réplication continue en cluster) et peut également protéger Exchange Server configuré pour la réplication continue locale.

* SQL Server : le serveur de sauvegarde Azure ne prend en charge la sauvegarde des bases de données SQL Server hébergées sur des volumes partagés de cluster.

Le serveur de sauvegarde Azure peut protéger les charges de travail de cluster qui se trouvent dans le même domaine que le serveur MABS, et dans un domaine enfant ou approuvé. Si vous voulez protéger des sources de données dans des domaines ou des groupes de travail non approuvés, utilisez NTLM ou l’authentification par certificat pour un serveur unique, ou l’authentification par certificat uniquement pour un cluster.
