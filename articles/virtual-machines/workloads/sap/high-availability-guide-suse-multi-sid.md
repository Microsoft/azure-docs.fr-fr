---
title: Guide de haute disponibilité multi-SID des machines virtuelles Azure SAP sur SLES | Microsoft Docs
description: Guide de haute disponibilité multi-SID pour SAP NetWeaver sur SUSE Linux Enterprise Server pour les applications SAP
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/16/2020
ms.author: radeltch
ms.openlocfilehash: 2ce57911434aa0fdf1a5e624090633e75d98a5ad
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484241"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-suse-linux-enterprise-server-for-sap-applications-multi-sid-guide"></a>Guide de haute disponibilité multi-SID pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md

Cet article explique comment déployer plusieurs systèmes hautement disponible SAP NetWeaver ou S4HANA (c’est-à-dire, plusieurs SID) dans un cluster à deux nœuds sur des machines virtuelles Azure avec SUSE Linux Enterprise Server pour les applications SAP.  

Dans les exemples de configuration, commandes d’installation, etc., trois systèmes SAP NetWeaver 7.50 sont déployés dans un même cluster à deux nœuds à haute disponibilité. Les SID des systèmes SAP sont les suivants :
* **NW1** : Numéro d’instance ASCS **00** et nom d’hôte virtuel **msnw1ascs** ; numéro d’instance ERS **02** et nom d’hôte virtuel **msnw1ers**.  
* **NW2** : Numéro d’instance ASCS **10** et nom d’hôte virtuel **msnw2ascs** ; numéro d’instance ERS **12** et nom d’hôte virtuel **msnw2ers**.  
* **NW3** : Numéro d’instance ASCS **20** et nom d’hôte virtuel **msnw3ascs** ; numéro d’instance ERS **22** et nom d’hôte virtuel **msnw3ers**.  

Cet article ne couvre pas la couche de base de données et le déploiement des partages NFS SAP. Dans les exemples de cet article, nous utilisons les noms virtuels nw2-nfs pour les partages NW2 NFS et nw3-nfs pour les partages NW3 NFS, en partant du principe que le cluster NFS a été déployé.  

Avant de commencer, reportez-vous aux notes et documents SAP suivants :

* Note SAP [1928533][1928533], qui contient :
  * une liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP
  * des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure
  * les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données
  * la version du noyau SAP requise pour Windows et Linux sur Microsoft Azure

* La note SAP [2015553][2015553] répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure.
* La note SAP [2205917][2205917] contient des paramètres de système d’exploitation recommandés pour SUSE Linux Enterprise Server pour les applications SAP
* La note SAP [1944799][1944799] contient des instructions SAP HANA pour SUSE Linux Enterprise Server pour les applications SAP
* La note SAP [2178632][2178632] contient des informations détaillées sur toutes les métriques de surveillance rapportées pour SAP sur Azure.
* La note SAP [2191498][2191498] contient la version requise de l’agent hôte SAP pour Linux sur Azure.
* La note SAP [2243692][2243692] contient des informations sur les licences SAP sur Linux dans Azure.
* La note SAP [1984787][1984787] contient des informations sur SUSE Linux Enterprise Server 12.
* La note SAP [1999351][1999351] contient des informations de dépannage supplémentaires pour l’extension d’analyse Azure améliorée pour SAP.
* Le [WIKI de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contient toutes les notes SAP requises pour Linux.
* [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP sur Linux][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP sur Linux][dbms-guide]
* [Guides sur les meilleures pratiques pour SUSE SAP HA][suse-ha-guide] Les guides contiennent toutes les informations nécessaires pour configurer la réplication locale des systèmes Netweaver HP et SAP HANA. Utilisez ces guides comme planning de référence. Ils fournissent des informations beaucoup plus détaillées.
* [Notes de publication de SUSE High Availability Extension 12 SP3][suse-ha-12sp3-relnotes]
* [Guide du cluster multi-SID SUSE pour SLES 12 et SLES 15](https://documentation.suse.com/sbp/all/html/SBP-SAP-MULTI-SID/index.html)
* [Applications SAP NetApp su Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure]
## <a name="overview"></a>Vue d’ensemble

Les machines virtuelles participant au cluster doivent être dimensionnées de manière à exécuter toutes les ressources, en cas de basculement. Chaque SID SAP peut basculer indépendamment des autres dans le cluster à haute disponibilité multi-SID.  Si vous utilisez l'isolation SBD, les appareils SBD peuvent être partagés entre plusieurs clusters.  

Pour obtenir une haute disponibilité, SAP NetWeaver nécessite des partages NFS hautement disponibles. Dans cet exemple, nous partons du principe que les partages NFS SAP sont hébergés sur un [serveur de fichiers NFS](./high-availability-guide-suse-nfs.md) hautement disponible, susceptible d'être utilisé par plusieurs systèmes SAP. Ou que les partages sont déployés sur des [volumes NFS Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  

![Le cluster Pacemaker présente des informations détaillées sur deux clusters multi-SID, msidcl1 et msidcl2.](./media/high-availability-guide-suse/ha-suse-multi-sid.png)

> [!IMPORTANT]
> La prise en charge du clustering multi-SID de SAP ASC/ERS avec SUSE Linux comme système d’exploitation invité des machines virtuelles Azure est limitée à **cinq** SID SAP sur le même cluster. Chaque nouveau SID augmente la complexité. La combinaison de Enqueue Replication Server 1 et Enqueue Replication Server 2 SAP sur le même cluster n'est **pas prise en charge**. Le clustering multi-SID décrit l’installation de plusieurs instances de SAP ASCS/ERS avec des SID différents dans un cluster Pacemaker. Actuellement, le clustering multi-SID est uniquement pris en charge pour ASCS/ERS.  

> [!TIP]
> Le clustering multi-SID de SAP ASCS/ERS relève d'une solution particulièrement complexe. Il s'avère plus difficile à implémenter. En outre, il implique plus d'effort administratif lors de l’exécution d’activités de maintenance (application de correctifs au système d’exploitation, par exemple). Avant de procéder à l'implémentation, prenez le temps de planifier soigneusement le déploiement et tous les composants impliqués, tels que les machines virtuelles, montages NFS, adresses IP virtuelles, configurations d’équilibreur de charge, etc.  

Le serveur NFS, SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS et la base de données SAP HANA utilisent un nom d’hôte virtuel et des adresses IP virtuelles. Sur Azure, un équilibreur de charge est nécessaire pour utiliser une adresse IP virtuelle. Nous vous recommandons d’utiliser [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md).  

La liste suivante présente la configuration de l’équilibreur de charge (A)SCS et ERS pour cet exemple de cluster multi-SID avec trois systèmes SAP. Vous aurez besoin d’une adresse IP frontale, de sondes d’intégrité et de règles d’équilibrage de charge distinctes pour chaque instance ASCS et ERS des différents SID. Attribuez toutes les machines virtuelles qui font partie du cluster ASCS/ASCS à un pool principal.  

### <a name="ascs"></a>(A)SCS

* Configuration du frontend
  * Adresse IP pour NW1 :  10.3.1.14
  * Adresse IP pour NW2 :  10.3.1.16
  * Adresse IP pour NW3 :  10.3.1.13
* Ports d'analyse
  * Port 620 <strong>&lt;nr&gt;</strong>, par conséquent pour NW1, NW2 et NW3 ports d'analyse 620 **00**, 620 **10** et 620 **20**
* Règles d’équilibrage de charge - 
* créez-en une pour chaque instance, à savoir NW1/ASCS, NW2/ASCS et NW3/ASCS.
  * Si vous utilisez Standard Load Balancer, sélectionnez **Ports haute disponibilité**
  * Si vous utilisez Basic Load Balancer, créez des règles d’équilibrage de charge pour les ports suivants
    * TCP 32<strong>&lt;nr&gt;</strong>
    * TCP 36<strong>&lt;nr&gt;</strong>
    * TCP 39<strong>&lt;nr&gt;</strong>
    * TCP 81<strong>&lt;nr&gt;</strong>
    * TCP 5<strong>&lt;nr&gt;</strong>13
    * TCP 5<strong>&lt;nr&gt;</strong>14
    * TCP 5<strong>&lt;nr&gt;</strong>16

### <a name="ers"></a>ERS

* Configuration du frontend
  * Adresse IP pour NW1 10.3.1.15
  * Adresse IP pour NW2 10.3.1.17
  * Adresse IP pour NW3 10.3.1.19
* Port de la sonde
  * Port 621 <strong>&lt;Nr&gt;</strong>, par conséquent pour NW1, NW2 et N# ports d'analyse 621 **02**, 621 **12** et 621 **22**
* Règles d’équilibrage de charge - créez-en une pour chaque instance, à savoir NW1/ERS, NW2/ERS et NW3/ERS.
  * Si vous utilisez Standard Load Balancer, sélectionnez **Ports haute disponibilité**
  * Si vous utilisez Basic Load Balancer, créez des règles d’équilibrage de charge pour les ports suivants
    * TCP 32<strong>&lt;nr&gt;</strong>
    * TCP 33<strong>&lt;nr&gt;</strong>
    * TCP 5<strong>&lt;nr&gt;</strong>13
    * TCP 5<strong>&lt;nr&gt;</strong>14
    * TCP 5<strong>&lt;nr&gt;</strong>16

* Configuration du backend
  * Connecté aux interfaces réseau principales de toutes les machines virtuelles qui doivent faire partie du cluster (A)SCS/ERS

> [!IMPORTANT]
> Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d’informations, consultez [Limitations d’équilibreur de charge Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.  

> [!Note]
> Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité Internet sortante, sauf si une configuration supplémentaire est effectuée pour autoriser le routage vers des points de terminaison publics. Pour savoir plus en détails comment bénéficier d’une connectivité sortante, voir [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

> [!IMPORTANT]
> N’activez pas les timestamps TCP sur des machines virtuelles Azure placées derrière Azure Load Balancer. L’activation des timestamps TCP entraîne l’échec des sondes d’intégrité. Définissez le paramètre **net.ipv4.tcp_timestamps** sur **0**. Pour plus d’informations, consultez [Load Balancer health probes](../../../load-balancer/load-balancer-custom-probe-overview.md) (Sondes d’intégrité Load Balancer).

## <a name="sap-nfs-shares"></a>Partages NFS SAP

SAP NetWeaver nécessite un stockage partagé pour le répertoire de transport, de profil, etc. Pour un système SAP hautement disponible, il est important de disposer de partages NFS hautement disponibles. Vous devrez déterminer l’architecture de vos partages NFS SAP. Vous pouvez notamment créer un [cluster NFS hautement disponible sur des machines virtuelles Azure sur SUSE Linux Enterprise Server][nfs-ha], et celui-ci peut être partagé entre plusieurs systèmes SAP. 

Vous pouvez également déployer les partages sur des [volumes NFS Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  Avec Azure NetApp Files, vous bénéficiez d’une haute disponibilité intégrée pour les partages NFS SAP.

## <a name="deploy-the-first-sap-system-in-the-cluster"></a>Déployer le premier système SAP dans le cluster

Après avoir déterminé l’architecture des partages NFS SAP, déployez le premier système SAP dans le cluster, en suivant la documentation correspondante.

* Si vous utilisez un serveur NFS hautement disponible, consultez [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md).  
* Si vous utilisez des volumes NFS Azure NetApp Files, consultez [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md).

Les documents répertoriés ci-dessus vous guident tout au long des étapes de préparation des infrastructures nécessaires, de création du cluster, de préparation du système d’exploitation pour l’exécution de l’application SAP.  

> [!TIP]
> Testez systématiquement la fonctionnalité de basculement du cluster, une fois le premier système déployé, avant d’ajouter les SID SAP supplémentaires au cluster. Vous vous assurerez ainsi du bon fonctionnement de la fonctionnalité de cluster, avant d’ajouter des systèmes SAP supplémentaires au cluster.   

## <a name="deploy-additional-sap-systems-in-the-cluster"></a>Déployer des systèmes SAP supplémentaires au cluster

Dans cet exemple, nous partons du principe que le système **NW1** a déjà été déployé dans le cluster. Nous allons vous expliquer comment effectuer un déploiement dans les systèmes SAP du cluster **NW2** et **NW3**. 

Les éléments suivants sont précédés de **[A]** (applicable à tous les nœuds), de **[1]** (applicable uniquement au nœud 1) ou de **[2]** (applicable uniquement au nœud 2).

### <a name="prerequisites"></a>Prérequis 

> [!IMPORTANT]
> Avant de suivre les instructions pour déployer des systèmes SAP supplémentaires dans le cluster, suivez les instructions visant à déployer le premier système SAP dans le cluster, car certaines étapes sont uniquement requises lors du premier déploiement du système.  

Cette documentation suppose ce qui suit :
* Le cluster Pacemaker est déjà configuré et en cours d’exécution.  
* Au moins un système SAP (instance ASCS/ERS) est déjà déployé et est en cours d’exécution dans le cluster.  
* La fonctionnalité de basculement du cluster a été testée.  
* Les partages NFS de tous les systèmes SAP sont déployés.  

### <a name="prepare-for-sap-netweaver-installation"></a>Préparer l’installation de SAP NetWeaver

1. Ajoutez une configuration pour le système récemment déployé (à savoir, **NW2**, **NW3**) à l'instance Azure Load Balancer existante. Pour ce faire, consultez [Déployer Azure Load Balancer manuellement via le portail Azure](./high-availability-guide-suse-netapp-files.md#deploy-azure-load-balancer-manually-via-azure-portal). Ajustez les adresses IP, les ports de sonde d’intégrité et les règles d’équilibrage de charge pour votre configuration.  

2. **[A]** Configurez la résolution de noms pour les systèmes SAP supplémentaires. Vous pouvez utiliser un serveur DNS ou modifier `/etc/hosts` sur tous les nœuds. Cet exemple vous explique comment utiliser le fichier `/etc/hosts`.  Adaptez les adresses IP et les noms d’hôte à votre environnement. 

    ```
    sudo vi /etc/hosts
    # IP address of the load balancer frontend configuration for NW2 ASCS
    10.3.1.16 msnw2ascs
    # IP address of the load balancer frontend configuration for NW3 ASCS
    10.3.1.13 msnw3ascs
    # IP address of the load balancer frontend configuration for NW2 ERS
    10.3.1.17 msnw2ers
    # IP address of the load balancer frontend configuration for NW3 ERS
    10.3.1.19 msnw3ers
    # IP address for virtual host name for the NFS server for NW2
    10.3.1.31 nw2-nfs
    # IP address for virtual host name for the NFS server for NW3
    10.3.1.32 nw3-nfs
   ```

3. **[A]** Créez les répertoires partagés pour les systèmes SAP supplémentaires **NW2** et **NW3** que vous déployez sur le cluster. 

    ```
    sudo mkdir -p /sapmnt/NW2
    sudo mkdir -p /usr/sap/NW2/SYS
    sudo mkdir -p /usr/sap/NW2/ASCS10
    sudo mkdir -p /usr/sap/NW2/ERS12
    sudo mkdir -p /sapmnt/NW3
    sudo mkdir -p /usr/sap/NW3/SYS
    sudo mkdir -p /usr/sap/NW3/ASCS20
    sudo mkdir -p /usr/sap/NW3/ERS22

    
    sudo chattr +i /sapmnt/NW2
    sudo chattr +i /usr/sap/NW2/SYS
    sudo chattr +i /usr/sap/NW2/ASCS10
    sudo chattr +i /usr/sap/NW2/ERS12
    sudo chattr +i /sapmnt/NW3
    sudo chattr +i /usr/sap/NW3/SYS
    sudo chattr +i /usr/sap/NW3/ASCS20
    sudo chattr +i /usr/sap/NW3/ERS22
   ```

4. **[A]** Configurez `autofs` pour monter les systèmes de fichiers /sapmnt/SID et /usr/sap/SID/SYS pour les systèmes SAP supplémentaires que vous déployez sur le cluster. Dans cet exemple, **NW2** et **NW3**.  

   Mettez à jour le fichier `/etc/auto.direct` avec les systèmes de fichiers pour les systèmes SAP supplémentaires que vous déployez sur le cluster.  

   * Si vous utilisez un serveur de fichiers NFS, suivez les instructions contenues [ici](./high-availability-guide-suse.md#prepare-for-sap-netweaver-installation)
   * Si vous utilisez Azure NetApp Files, suivez les instructions contenues [ici](./high-availability-guide-suse-netapp-files.md#prepare-for-sap-netweaver-installation) 

   Vous devrez redémarrer le service `autofs` pour monter les partages récemment ajoutés.  

### <a name="install-ascs--ers"></a>Installer ASCS/ERS

1. Créez l'adresse IP virtuelle et les ressources de cluster de sonde d’intégrité pour l’instance ASCS du système SAP supplémentaire que vous déployez sur le cluster. L’exemple illustré ici correspond à **NW2** et **NW3** ASCS et utilise un serveur NFS hautement disponible.  

   > [!IMPORTANT]
   > Des tests récents ont révélé des cas où netcat cessait de répondre aux demandes en raison du backlog et de sa capacité à ne gérer qu’une seule connexion. La ressource netcat cesse d’écouter les demandes d’Azure Load Balancer et l’adresse IP flottante devient indisponible.  
   > Pour les clusters Pacemaker existants, nous vous recommandons de remplacer netcat par socat. Actuellement, nous vous recommandons d'utiliser l'agent de ressources azure-lb, qui fait partie du package resource-agents, avec la configuration requise suivante pour la version du package :
   > - Pour SLES 12 SP4/SP5, la version minimum est resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Pour SLES 15/15 SP1, la version minimum est resource-agents-4.3.0184.6ee15eb2-4.13.1.  
   >
   > Notez que la modification nécessitera un bref temps d’arrêt.  
   > Pour les clusters Pacemaker existants, si la configuration a déjà été modifiée pour utiliser socat comme décrit à la page [Azure Load-Balancer Detection Hardening](https://www.suse.com/support/kb/doc/?id=7024128), il n’est pas nécessaire de passer immédiatement à l’agent de ressources azure-lb.

    ```
      sudo crm configure primitive fs_NW2_ASCS Filesystem device='nw2-nfs:/NW2/ASCS' directory='/usr/sap/NW2/ASCS10' fstype='nfs4' \
       op start timeout=60s interval=0 \
       op stop timeout=60s interval=0 \
       op monitor interval=20s timeout=40s
   
      sudo crm configure primitive vip_NW2_ASCS IPaddr2 \
        params ip=10.3.1.16 cidr_netmask=24 \
        op monitor interval=10 timeout=20
   
      sudo crm configure primitive nc_NW2_ASCS azure-lb port=62010
   
      sudo crm configure group g-NW2_ASCS fs_NW2_ASCS nc_NW2_ASCS vip_NW2_ASCS \
         meta resource-stickiness=3000

      sudo crm configure primitive fs_NW3_ASCS Filesystem device='nw3-nfs:/NW3/ASCS' directory='/usr/sap/NW3/ASCS20' fstype='nfs4' \
        op start timeout=60s interval=0 \
        op stop timeout=60s interval=0 \
        op monitor interval=20s timeout=40s
   
      sudo crm configure primitive vip_NW3_ASCS IPaddr2 \
       params ip=10.3.1.13 cidr_netmask=24 \
       op monitor interval=10 timeout=20
   
      sudo crm configure primitive nc_NW3_ASCS azure-lb port=62020
   
      sudo crm configure group g-NW3_ASCS fs_NW3_ASCS nc_NW3_ASCS vip_NW3_ASCS \
        meta resource-stickiness=3000
    ```

   Les ressources créées peuvent être attribuées à différentes ressources du cluster. Lorsque vous les regroupez, elles migrent vers l’un des nœuds du cluster. Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.

2. **[1]** Installer SAP NetWeaver ASCS  

   Installez SAP NetWeaver ASCS en tant que racine à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration frontend d’équilibreur de charge pour l'instance ASCS. Par exemple, pour le système **NW2**, le nom d’hôte virtuel est <b>msnw2ascs</b>, <b>10.3.1.16</b> et le numéro d’instance que vous avez utilisé pour la sonde de l’équilibreur de charge, par exemple <b>10</b>. Pour le système **NW3**, le nom d’hôte virtuel est <b>msnw3ascs</b>, <b>10.3.1.13</b> et le numéro d’instance que vous avez utilisé pour la sonde de l’équilibreur de charge, par exemple <b>20</b>.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst. Vous pouvez utiliser le paramètre SAPINST_USE_HOSTNAME pour installer SAP à l’aide du nom d’hôte virtuel.  

     ```
      sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
     ```

   Si aucun sous-dossier n’est créé dans /usr/sap/**SID**/ASCS **Instance#** lors de l'installation, essayez de définir le propriétaire sur **sid** adm et le groupe sur sapsys pour **l'instance ASCS #** , puis réessayez.

3. **[1]** Créez une adresse IP virtuelle et des ressources de cluster de sonde d’intégrité pour l’instance ERS du système SAP supplémentaire que vous déployez sur le cluster. L’exemple illustré ici correspond à **NW2** et **NW3** ERS, avec serveur NFS hautement disponible. 

   ```
    sudo crm configure primitive fs_NW2_ERS Filesystem device='nw2-nfs:/NW2/ASCSERS' directory='/usr/sap/NW2/ERS12' fstype='nfs4' \
      op start timeout=60s interval=0 \
      op stop timeout=60s interval=0 \
      op monitor interval=20s timeout=40s
   
    sudo crm configure primitive vip_NW2_ERS IPaddr2 \
      params ip=10.3.1.17 cidr_netmask=24 \
      op monitor interval=10 timeout=20
   
    sudo crm configure primitive nc_NW2_ERS azure-lb port=62112
   
    sudo crm configure group g-NW2_ERS fs_NW2_ERS nc_NW2_ERS vip_NW2_ERS

    sudo crm configure primitive fs_NW3_ERS Filesystem device='nw3-nfs:/NW3/ASCSERS' directory='/usr/sap/NW3/ERS22' fstype='nfs4' \
      op start timeout=60s interval=0 \
      op stop timeout=60s interval=0 \
      op monitor interval=20s timeout=40s
   
    sudo crm configure primitive vip_NW3_ERS IPaddr2 \
      params ip=10.3.1.19 cidr_netmask=24 \
      op monitor interval=10 timeout=20
   
    sudo crm configure primitive nc_NW3_ERS azure-lb port=62122
   
    sudo crm configure group g-NW3_ERS fs_NW3_ERS nc_NW3_ERS vip_NW3_ERS
   ```

   Les ressources créées peuvent être attribuées à différents nœuds du cluster. Lorsque vous les regroupez, elles migrent vers l’un des nœuds du cluster. Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées.  

   Assurez-vous ensuite que les ressources du groupe ERS récemment créé sont en cours d’exécution sur le nœud de cluster, à l’inverse du nœud de cluster dans lequel l’instance ASCS correspondant au même système SAP a été installée.  Par exemple, si NW2 ASCS a été installé sur `slesmsscl1`, assurez-vous que le groupe NW2 ERS est en cours d'exécution sur `slesmsscl2`.  Vous pouvez migrer le groupe NW2 ERS vers `slesmsscl2` en exécutant la commande suivante : 

    ```
      crm resource migrate g-NW2_ERS slesmsscl2 force
    ```

4. **[2]** Installer SAP NetWeaver ERS

   Installez l’instance ERS SAP NetWeaver en tant que racine sur l'autre nœud, à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration frontend d’équilibreur de charge pour l'instance ERS. Par exemple, pour le système **NW2**, le nom d’hôte virtuel sera <b>msnw2ers</b>, <b>10.3.1.17</b> et le numéro d’instance que vous avez utilisé pour la sonde de l’équilibreur de charge, par exemple <b>12</b>. Pour le système **NW3**, le nom d’hôte virtuel sera <b>msnw3ers</b>, <b>10.3.1.19</b> et le numéro d’instance que vous avez utilisé pour la sonde de l’équilibreur de charge, par exemple <b>22</b>. 

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst. Vous pouvez utiliser le paramètre SAPINST_USE_HOSTNAME pour installer SAP à l’aide du nom d’hôte virtuel.  

    ```
     sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   > [!NOTE]
   > Utilisez SWPM SP 20 PL 05 ou ultérieur. Les versions antérieures ne définissent pas les autorisations correctement et l’installation échouera.

   Si aucun sous-dossier n’est créé dans /usr/sap/**NW2**/ERS **Instance#** lors de l'installation, essayez de définir le propriétaire sur **sid** adm et le groupe sur sapsys pour **l'instance ERS #** , puis réessayez.

   Si vous avez dû migrer le groupe ERS du système SAP récemment déployé vers un nœud de cluster différent, n’oubliez pas de supprimer la contrainte d’emplacement pour le groupe ERS. Vous pouvez supprimer cette contrainte en exécutant la commande suivante (l’exemple correspond aux systèmes SAP **NW2** et **NW3**).  

    ```
      crm resource unmigrate g-NW2_ERS
      crm resource unmigrate g-NW3_ERS
    ```

5. **[1]** Adaptez les profils d’instance ASCS/SCS et ERS au(x) système(s) SAP récemment installé(s). L’exemple illustré ci-dessous correspond à NW2. Vous devez adapter les profils ASCS/SCS et ERS pour toutes les instances SAP ajoutées au cluster.  
 
 * Profil ASCS/SCS

   ```
   sudo vi /sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # Add the keep alive parameter, if using ENSA1
   enque/encni/set_so_keepalive = true
   ```

   Pour ENSA1 et ENSA2, assurez-vous que les `keepalive`Paramètres de système d’exploitation sont définis comme décrit dans la note SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).  

 * Profil ERS

   ```
   sudo vi /sapmnt/NW2/profile/NW2_ERS12_msnw2ers
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # remove Autostart from ERS profile
   # Autostart = 1
   ```

6. **[A]** Configurez les utilisateurs SAP pour le système SAP récemment déployé, dans cet exemple **NW2** et **NW3**. 

   ```
   # Add sidadm to the haclient group
   sudo usermod -aG haclient nw2adm
   sudo usermod -aG haclient nw3adm
   ```

7. Ajoutez les services SAP ASCS et ERS pour le système SAP récemment installé au fichier `sapservice`. L’exemple ci-dessous correspond aux systèmes SAP **NW2** et **NW3**.  

   Ajoutez l’entrée de service ASCS au deuxième nœud et copiez l’entrée de service ERS dans le premier nœud. Exécutez les commandes pour chaque système SAP du nœud, où l’instance ASCS correspondant au système SAP a été installée.  

    ```
     # Execute the following commands on slesmsscl1,assuming the NW2 ASCS instance was installed on slesmsscl1
     cat /usr/sap/sapservices | grep ASCS10 | sudo ssh slesmsscl2 "cat >>/usr/sap/sapservices"
     sudo ssh slesmsscl2 "cat /usr/sap/sapservices" | grep ERS12 | sudo tee -a /usr/sap/sapservices
     # Execute the following commands on slesmsscl2, assuming the NW3 ASCS instance was installed on slesmsscl2
     cat /usr/sap/sapservices | grep ASCS20 | sudo ssh slesmsscl1 "cat >>/usr/sap/sapservices"
     sudo ssh slesmsscl1 "cat /usr/sap/sapservices" | grep ERS22 | sudo tee -a /usr/sap/sapservices
    ```

8. **[1]** Créez les ressources de cluster SAP pour le système SAP récemment installé. 

   Si vous utilisez l’architecture ENSA1 (serveur de file d’attente 1), définissez les ressources pour les systèmes SAP **NW2** et **NW3** comme suit :

    ```
     sudo crm configure property maintenance-mode="true"
    
     sudo crm configure primitive rsc_sap_NW2_ASCS10 SAPInstance \
      operations \$id=rsc_sap_NW2_ASCS10-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
    
     sudo crm configure primitive rsc_sap_NW2_ERS12 SAPInstance \
      operations \$id=rsc_sap_NW2_ERS12-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true \
      meta priority=1000
    
     sudo crm configure modgroup g-NW2_ASCS add rsc_sap_NW2_ASCS10
     sudo crm configure modgroup g-NW2_ERS add rsc_sap_NW2_ERS12
    
     sudo crm configure colocation col_sap_NW2_no_both -5000: g-NW2_ERS g-NW2_ASCS
     sudo crm configure location loc_sap_NW2_failover_to_ers rsc_sap_NW2_ASCS10 rule 2000: runs_ers_NW2 eq 1
     sudo crm configure order ord_sap_NW2_first_start_ascs Optional: rsc_sap_NW2_ASCS10:start rsc_sap_NW2_ERS12:stop symmetrical=false
   
     sudo crm configure primitive rsc_sap_NW3_ASCS20 SAPInstance \
      operations \$id=rsc_sap_NW3_ASCS20-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ASCS10_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
    
     sudo crm configure primitive rsc_sap_NW3_ERS22 SAPInstance \
      operations \$id=rsc_sap_NW3_ERS22-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW3_ERS22_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true \
      meta priority=1000
    
     sudo crm configure modgroup g-NW3_ASCS add rsc_sap_NW3_ASCS20
     sudo crm configure modgroup g-NW3_ERS add rsc_sap_NW3_ERS22
    
     sudo crm configure colocation col_sap_NW3_no_both -5000: g-NW3_ERS g-NW3_ASCS
     sudo crm configure location loc_sap_NW3_failover_to_ers rsc_sap_NW3_ASCS10 rule 2000: runs_ers_NW3 eq 1
     sudo crm configure order ord_sap_NW3_first_start_ascs Optional: rsc_sap_NW3_ASCS20:start rsc_sap_NW3_ERS22:stop symmetrical=false
     sudo crm configure property maintenance-mode="false"
    ```

   SAP a introduit la prise en charge du serveur de file d’attente 2, y compris la réplication, avec SAP NW 7.52. À partir de la plateforme ABAP 1809, le serveur de file d’attente 2 est installé par défaut. Consultez la note SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) pour plus d’informations sur la prise en charge du serveur de file d’attente 2.
   Si vous utilisez l’architecture ENSA2 ([serveur de file d’attente 2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), définissez les ressources pour les systèmes SAP **NW2** et **NW3** comme indiqué ci-après :

    ```
     sudo crm configure property maintenance-mode="true"
    
     sudo crm configure primitive rsc_sap_NW2_ASCS10 SAPInstance \
      operations \$id=rsc_sap_NW2_ASCS10-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 
    
     sudo crm configure primitive rsc_sap_NW2_ERS12 SAPInstance \
      operations \$id=rsc_sap_NW2_ERS12-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true 
    
     sudo crm configure modgroup g-NW2_ASCS add rsc_sap_NW2_ASCS10
     sudo crm configure modgroup g-NW2_ERS add rsc_sap_NW2_ERS12
    
     sudo crm configure colocation col_sap_NW2_no_both -5000: g-NW2_ERS g-NW2_ASCS
     sudo crm configure order ord_sap_NW2_first_start_ascs Optional: rsc_sap_NW2_ASCS10:start rsc_sap_NW2_ERS12:stop symmetrical=false
   
     sudo crm configure primitive rsc_sap_NW3_ASCS20 SAPInstance \
      operations \$id=rsc_sap_NW3_ASCS20-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ASCS10_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000
    
     sudo crm configure primitive rsc_sap_NW3_ERS22 SAPInstance \
      operations \$id=rsc_sap_NW3_ERS22-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW3_ERS22_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true
    
     sudo crm configure modgroup g-NW3_ASCS add rsc_sap_NW3_ASCS20
     sudo crm configure modgroup g-NW3_ERS add rsc_sap_NW3_ERS22
    
     sudo crm configure colocation col_sap_NW3_no_both -5000: g-NW3_ERS g-NW3_ASCS
     sudo crm configure order ord_sap_NW3_first_start_ascs Optional: rsc_sap_NW3_ASCS20:start rsc_sap_NW3_ERS22:stop symmetrical=false
     sudo crm configure property maintenance-mode="false"
    ```

   Si vous effectuez une mise à niveau à partir d’une version antérieure et que vous passez au serveur de file d’attente 2, consultez la note SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.
   L’exemple suivant illustre l’état des ressources de cluster, après ajout des systèmes SAP **NW2** et **NW3** au cluster. 

    ```
     sudo crm_mon -r
    
    # Online: [ slesmsscl1 slesmsscl2 ]
    
    #Full list of resources:
    
    #stonith-sbd     (stonith:external/sbd): Started slesmsscl1
    # Resource Group: g-NW1_ASCS
    #     fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    # Resource Group: g-NW1_ERS
    #     fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW2_ASCS
    #     fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW2_ERS
    #     fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    # Resource Group: g-NW3_ASCS
    #     fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW3_ERS
    #     fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    ```

   L’illustration suivante montre à quoi ressembleraient les ressources dans HA Web Konsole (Hawk), avec les ressources du système SAP **NW2** développées.  

   [![Vue d’ensemble de la haute disponibilité SAP NetWeaver](./media/high-availability-guide-suse/ha-suse-multi-sid-hawk.png)](./media/high-availability-guide-suse/ha-suse-multi-sid-hawk-detail.png#lightbox)

### <a name="proceed-with-the-sap-installation"></a>Poursuivre l’installation de SAP 

Finalisez votre installation SAP en procédant comme suit :

* [Préparer vos serveurs d'applications SAP NetWeaver](./high-availability-guide-suse.md#2d6008b0-685d-426c-b59e-6cd281fd45d7)
* [Installer une instance SGBD](./high-availability-guide-suse.md#install-database)
* [Installer un serveur d’applications SAP principal](./high-availability-guide-suse.md#sap-netweaver-application-server-installation)
* Installer une ou plusieurs instances d’application SAP supplémentaires

## <a name="test-the-multi-sid-cluster-setup"></a>Tester la configuration de cluster multi-SID

Les tests suivants sont un sous-ensemble des cas de test dans les guides des meilleures pratiques de SUSE. Ils sont inclus pour des raisons pratiques. Pour obtenir la liste complète des tests de cluster, reportez-vous à la documentation suivante :

* Si vous utilisez un serveur NFS hautement disponible, consultez [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md).  
* Si vous utilisez des volumes NFS Azure NetApp Files, consultez [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md).

Consultez systématiquement les guides des meilleures pratiques SUSE et effectuez tous les tests supplémentaires qui pourraient y être ajoutés.  
Les tests présentés se trouvent dans un cluster à deux nœuds, multi-SID, avec trois systèmes SAP installés.  

1. Testez HAGetFailoverConfig et HACheckFailoverConfig.

   Exécutez les commandes suivantes en tant que <sapsid>adm sur le nœud où l’instance ASCS est actuellement en cours d’exécution. Si les commandes échouent avec ÉCHEC : Mémoire insuffisante apparait, cela peut être dû à des tirets dans votre nom d’hôte. Ceci est un problème connu et il sera corrigé par SUSE dans le package sap-suse-cluster-connector.

   ```
    slesmsscl1:nw1adm 57> sapcontrol -nr 00 -function HAGetFailoverConfig

   # 10.12.2019 21:33:08
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl1
   # HANodes: slesmsscl1, slesmsscl2

    slesmsscl1:nw1adm 53> sapcontrol -nr 00 -function HACheckFailoverConfig

    # 19.12.2019 21:19:58
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch

    slesmsscl2:nw2adm 35> sapcontrol -nr 10 -function HAGetFailoverConfig

   # 10.12.2019 21:37:09
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl2
   # HANodes: slesmsscl2, slesmsscl1

    slesmsscl2:nw2adm 52> sapcontrol -nr 10 -function HACheckFailoverConfig

    # 19.12.2019 21:17:39
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch

    slesmsscl1:nw3adm 49> sapcontrol -nr 20 -function HAGetFailoverConfig

   # 10.12.2019 23:35:36
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl1
   # HANodes: slesmsscl1, slesmsscl2

    slesmsscl1:nw3adm 52> sapcontrol -nr 20 -function HACheckFailoverConfig

    # 19.12.2019 21:10:42
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch
   ```

2. Migrer manuellement l’instance ASCS L’exemple montre comment migrer l’instance ASCS pour le système SAP NW2.  
   État des ressources avant le début du test :
   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Exécutez les commandes suivantes en tant que racine pour migrer l’instance ASCS NW2.

   ```
    crm resource migrate rsc_sap_NW2_ASCS10 force
    # INFO: Move constraint created for rsc_sap_NW2_ASCS10
    
    crm resource unmigrate rsc_sap_NW2_ASCS10
   # INFO: Removed migration constraints for rsc_sap_NW2_ASCS10
   
   # Remove failed actions for the ERS that occurred as part of the migration
    crm resource cleanup rsc_sap_NW2_ERS12
   ```

   État des ressources après le test :

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

1. Testez HAFailoverToNode. Le test présenté ici montre comment migrer l’instance ASCS pour le système SAP NW2.  

   État des ressources avant le début du test :

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Exécutez les commandes suivantes en tant que **nw2** adm pour migrer l’instance ASCS NW2.

   ```
    slesmsscl2:nw2adm 53> sapcontrol -nr 10 -host msnw2ascs -user nw2adm password -function HAFailoverToNode ""
   
   # run as root
   # Remove failed actions for the ERS that occurred as part of the migration
   crm resource cleanup rsc_sap_NW2_ERS12
   # Remove migration constraints
   crm resource clear rsc_sap_NW2_ASCS10
   #INFO: Removed migration constraints for rsc_sap_NW2_ASCS10
   ```

   État des ressources après le test :

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

1. Simuler l’incident de nœud

   État des ressources avant le début du test :

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Exécutez la commande suivante en tant que racine sur le nœud où au moins une instance ASCS est en cours d’exécution. Dans cet exemple, nous avons exécuté la commande dans `slesmsscl2`, où les instances ASCS pour NW1 et NW3 sont en cours d’exécution.  

   ```
    slesmsscl2:~ # echo b > /proc/sysrq-trigger
   ```

   Si vous utilisez SBD, Pacemaker ne doit pas démarrer automatiquement sur le nœud tué. L’état après le redémarrage du nœud devrait ressembler à ce qui suit.

   ```
    Online: [ slesmsscl1 ]
    OFFLINE: [ slesmsscl2 ]
    Full list of resources:

    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    
    Failed Resource Actions:
    * rsc_sap_NW1_ERS02_monitor_11000 on slesmsscl1 'not running' (7): call=125, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
    * rsc_sap_NW2_ERS12_monitor_11000 on slesmsscl1 'not running' (7): call=126, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
    * rsc_sap_NW3_ERS22_monitor_11000 on slesmsscl1 'not running' (7): call=127, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
   ```

   Exécutez les commandes suivantes pour démarrer Pacemaker sur le nœud tué, nettoyer les messages SBD et nettoyer les ressources ayant échoué.

   ```bash
   # run as root
   # list the SBD device(s)
   cat /etc/sysconfig/sbd | grep SBD_DEVICE=

   # output is like:
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message slesmsscl2 clear
   
   systemctl start pacemaker
   crm resource cleanup rsc_sap_NW1_ERS02
   crm resource cleanup rsc_sap_NW2_ERS12
   crm resource cleanup rsc_sap_NW3_ERS22
   ```

   État des ressources après le test :

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
   ```

## <a name="next-steps"></a>Étapes suivantes

* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur des machines virtuelles Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha]