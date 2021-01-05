---
title: Guide multi-SID de haute disponibilité des machines virtuelles pour SAP NW sur RHEL | Microsoft Docs
description: Créez une haute disponibilité pour SAP NW sur les machines virtuelles Azure RHEL multi-SID.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/16/2020
ms.author: radeltch
ms.openlocfilehash: b944ed37fe8df5fd4964342d8c0f52a040612ee4
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486400"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-red-hat-enterprise-linux-for-sap-applications-multi-sid-guide"></a>Guide multi-SID de haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur Red Hat Enterprise Linux for SAP Applications

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability-rhel.md
[glusterfs-ha]:high-availability-guide-rhel-glusterfs.md

Cet article explique comment déployer plusieurs systèmes hautement disponibles SAP NetWeaver (soit plusieurs SID) dans un cluster à deux nœuds sur des machines virtuelles Azure avec Red Hat Enterprise Linux for SAP Applications.  

Dans les exemples de configuration, commandes d’installation, etc., trois systèmes SAP NetWeaver 7.50 sont déployés dans un même cluster à deux nœuds à haute disponibilité. Les SID des systèmes SAP sont les suivants :
* **NW1** : Numéro d’instance ASCS **00** et nom d’hôte virtuel **msnw1ascs** ; numéro d’instance ERS **02** et nom d’hôte virtuel **msnw1ers**.  
* **NW2** : Numéro d’instance ASCS **10** et nom d’hôte virtuel **msnw2ascs** ; numéro d’instance ERS **12** et nom d’hôte virtuel **msnw2ers**.  
* **NW3** : Numéro d’instance ASCS **20** et nom d’hôte virtuel **msnw3ascs** ; numéro d’instance ERS **22** et nom d’hôte virtuel **msnw3ers**.  

Cet article ne couvre pas la couche de base de données et le déploiement des partages NFS SAP. Dans les exemples de cet article, nous utilisons le volume [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md) **sapMSID** pour les partages NFS, en supposant que le volume est déjà déployé avec le protocole NFSv3 et que les chemins des fichiers suivants existent pour les ressources de cluster des instances ASCS et ERS des systèmes SAP NW1, NW2 et NW3 :  

* volume sapMSID (nfs://10.42.0.4/sapmnt<b>NW1</b>)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW1</b>ascs)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW1</b>sys)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW1</b>ers)
* volume sapMSID (nfs://10.42.0.4/sapmnt<b>NW2</b>)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW2</b>ascs)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW2</b>sys)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW2</b>ers)
* volume sapMSID (nfs://10.42.0.4/sapmnt<b>NW3</b>)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW3</b>ascs)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW3</b>sys)
* volume sapMSID (nfs://10.42.0.4/usrsap<b>NW3</b>ers)

Avant de commencer, reportez-vous aux notes et documents SAP suivants :

* Note SAP [1928533], qui contient :
  * une liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP
  * des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure
  * les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données
  * la version du noyau SAP requise pour Windows et Linux sur Microsoft Azure
* [Documentation Azure NetApp Files][anf-azure-doc]
* La note SAP [2015553] répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure.
* La note SAP [2002167] recommande des paramètres de système d’exploitation pour Red Hat Enterprise Linux
* La note SAP [2009879] conseille sur SAP HANA pour Red Hat Enterprise Linux
* La note SAP [2178632] contient des informations détaillées sur toutes les métriques de surveillance rapportées pour SAP sur Azure.
* La note SAP [2191498] contient la version requise de l’agent hôte SAP pour Linux sur Azure.
* La note SAP [2243692] contient des informations sur les licences SAP sur Linux dans Azure.
* La note SAP [1999351] contient des informations de dépannage supplémentaires pour l’extension d’analyse Azure améliorée pour SAP.
* Le [WIKI de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contient toutes les notes SAP requises pour Linux.
* [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP sur Linux][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP sur Linux][dbms-guide]
* [SAP Netweaver dans le cluster pacemaker](https://access.redhat.com/articles/3150081)
* Documentation RHEL générale
  * [Vue d’ensemble des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administration des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Référence des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Configuration d’ASC/ERS pour SAP Netweaver avec des ressources autonomes dans RHEL 7.5](https://access.redhat.com/articles/3569681)
  * [Configuration de SAP S/4HANA ASCS/ERS avec Standalone Enqueue Server 2 (ENSA2) dans Pacemaker sur RHEL](https://access.redhat.com/articles/3974941)
* Documentation RHEL spécifique à Azure :
  * [Stratégies de prise en charge des clusters à haute disponibilité RHEL - Machines virtuelles Microsoft Azure en tant que membres du cluster](https://access.redhat.com/articles/3131341)
  * [Installation et configuration d’un cluster à haute disponibilité Red Hat Enterprise Linux 7.4 (et versions ultérieures) sur Microsoft Azure](https://access.redhat.com/articles/3252491)
* [Applications SAP NetApp su Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Vue d’ensemble

Les machines virtuelles participant au cluster doivent être dimensionnées de manière à exécuter toutes les ressources, en cas de basculement. Chaque SID SAP peut basculer indépendamment des autres dans le cluster à haute disponibilité multi-SID.  

Des partages hautement disponibles sont requis par SAP NetWeaver pour la haute disponibilité. Dans cette documentation, nous présentons des exemples pour lesquels les partages SAP sont déployés sur des [volumes NFS Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md). Il est également possible d’héberger les partages sur un cluster [GlusterFS](./high-availability-guide-rhel-glusterfs.md) hautement disponible, qui peut être utilisé par plusieurs systèmes SAP.  

![Vue d’ensemble de la haute disponibilité SAP NetWeaver](./media/high-availability-guide-rhel/ha-rhel-multi-sid.png)

> [!IMPORTANT]
> La prise en charge du clustering multi-SID de SAP ASCS/ERS avec Red Hat Linux comme système d’exploitation invité sur les machines virtuelles Azure est limitée à **cinq** SID SAP sur le même cluster. Chaque nouveau SID augmente la complexité. La combinaison de Enqueue Replication Server 1 et Enqueue Replication Server 2 SAP sur le même cluster n'est **pas prise en charge**. Le clustering multi-SID décrit l’installation de plusieurs instances de SAP ASCS/ERS avec des SID différents dans un cluster Pacemaker. Actuellement, le clustering multi-SID est uniquement pris en charge pour ASCS/ERS.  

> [!TIP]
> Le clustering multi-SID de SAP ASCS/ERS relève d'une solution particulièrement complexe. Il s'avère plus difficile à implémenter. En outre, il implique plus d'effort administratif lors de l’exécution d’activités de maintenance (application de correctifs au système d’exploitation, par exemple). Avant de procéder à l'implémentation, prenez le temps de planifier soigneusement le déploiement et tous les composants impliqués, tels que les machines virtuelles, montages NFS, adresses IP virtuelles, configurations d’équilibreur de charge, etc.  

SAP NetWeaver ASCS, SAP NetWeaver SCS et SAP NetWeaver ERS utilisent un nom d’hôte virtuel et des adresses IP virtuelles. Sur Azure, un équilibreur de charge est nécessaire pour utiliser une adresse IP virtuelle. Nous vous recommandons d’utiliser [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md).  

La liste suivante présente la configuration de l’équilibreur de charge (A)SCS et ERS pour cet exemple de cluster multi-SID avec trois systèmes SAP. Vous aurez besoin d’une adresse IP frontale, de sondes d’intégrité et de règles d’équilibrage de charge distinctes pour chaque instance ASCS et ERS des différents SID. Attribuez toutes les machines virtuelles qui font partie du cluster ASCS/ASCS à un pool back-end d’un seul équilibrage de charge interne.  

### <a name="ascs"></a>(A)SCS

* Configuration du frontend
  * Adresse IP pour NW1 :  10.3.1.50
  * Adresse IP pour NW2 :  10.3.1.52
  * Adresse IP pour NW3 :  10.3.1.54

* Ports d'analyse
  * Port 620 <strong>&lt;nr&gt;</strong>, par conséquent pour NW1, NW2 et NW3 ports d'analyse 620 **00**, 620 **10** et 620 **20**
* Règles d’équilibrage de charge : créez-en une par instance, à savoir NW1/ASCS, NW2/ASCS et NW3/ASCS.
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
  * Adresse IP pour NW1 : 10.3.1.51
  * Adresse IP pour NW2 : 10.3.1.53
  * Adresse IP pour NW3 : 10.3.1.55

* Port de la sonde
  * Port 621 <strong>&lt;nr&gt;</strong>, soit 621 **02**, 621 **12** et 621 **22** pour NW1, NW2 et N3
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

## <a name="sap-shares"></a>Partages SAP

SAP NetWeaver nécessite un stockage partagé pour le répertoire de transport, de profil, etc. Pour un système SAP hautement disponible, il est important de disposer de partages hautement disponibles. Vous devrez déterminer l’architecture de vos partages SAP. Il est possible de les déployer sur des [volumes NFS Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  Avec Azure NetApp Files, vous bénéficiez d’une haute disponibilité intégrée pour les partages NFS SAP.

Vous pouvez également créer un [cluster GlusterFS sur des machines virtuelles Azure sur Red Hat Enterprise Linux for SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md), partageable entre plusieurs systèmes SAP. 

## <a name="deploy-the-first-sap-system-in-the-cluster"></a>Déployer le premier système SAP dans le cluster

Après avoir déterminé l’architecture des partages SAP, déployez le premier système SAP dans le cluster, en suivant la documentation correspondante.

* Si vous utilisez des volumes NFS Azure NetApp Files, consultez [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux avec Azure NetApp Files for SAP Applications](./high-availability-guide-rhel-netapp-files.md).  
* Si vous utilisez un cluster GlusterFS, consultez [GlusterFS sur les machines virtuelles Azure sur Red Hat Enterprise Linux for SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md).  

Les documents ci-dessus vous guident tout au long des étapes de préparation de l’infrastructure nécessaire, de création du cluster et de préparation du système d’exploitation à l’exécution de l’application SAP.  

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

1. Ajoutez une configuration pour le système récemment déployé (à savoir, **NW2**, **NW3**) à l'instance Azure Load Balancer existante. Pour ce faire, consultez [Déployer Azure Load Balancer manuellement via le portail Azure](./high-availability-guide-rhel-netapp-files.md#deploy-linux-manually-via-azure-portal). Ajustez les adresses IP, les ports de sonde d’intégrité et les règles d’équilibrage de charge pour votre configuration.  

2. **[A]** Configurez la résolution de noms pour les systèmes SAP supplémentaires. Vous pouvez utiliser un serveur DNS ou modifier `/etc/hosts` sur tous les nœuds. Cet exemple vous explique comment utiliser le fichier `/etc/hosts`.  Adaptez les adresses IP et les noms d’hôte à votre environnement. 

    ```
    sudo vi /etc/hosts
    # IP address of the load balancer frontend configuration for NW2 ASCS
    10.3.1.52 msnw2ascs
    # IP address of the load balancer frontend configuration for NW3 ASCS
    10.3.1.54 msnw3ascs
    # IP address of the load balancer frontend configuration for NW2 ERS
    10.3.1.53 msnw2ers
    # IP address of the load balancer frontend configuration for NW3 ERS
    10.3.1.55 msnw3ers
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

4. **[A]** Ajoutez les entrées de montage pour les systèmes de fichiers /sapmnt/SID et /usr/sap/SID/SYS des systèmes SAP supplémentaires que vous déployez sur le cluster. Dans cet exemple, **NW2** et **NW3**.  

   Mettez à jour le fichier `/etc/fstab` avec les systèmes de fichiers pour les systèmes SAP supplémentaires que vous déployez sur le cluster.  

   * Si vous utilisez Azure NetApp Files, suivez les instructions contenues [ici](./high-availability-guide-rhel-netapp-files.md#prepare-for-sap-netweaver-installation)  
   * Si vous utilisez un cluster GlusterFS, suivez les instructions contenues [ici](./high-availability-guide-rhel.md#prepare-for-sap-netweaver-installation)  

### <a name="install-ascs--ers"></a>Installer ASCS/ERS

1. Créez l’adresse IP virtuelle et les ressources de cluster de sonde d’intégrité des instances ASCS des systèmes SAP supplémentaires que vous déployez sur le cluster. L’exemple indiqué ici concerne **NW2** et **NW3** ASCS et utilise NFS sur des volumes Azure NetApp Files avec le protocole NFSv3.  

    ```
    sudo pcs resource create fs_NW2_ASCS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW2ascs' \
    directory='/usr/sap/NW2/ASCS10' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW2_ASCS

    sudo pcs resource create vip_NW2_ASCS IPaddr2 \
    ip=10.3.1.52 cidr_netmask=24 \
     --group g-NW2_ASCS
    
    sudo pcs resource create nc_NW2_ASCS azure-lb port=62010 \
     --group g-NW2_ASCS

    sudo pcs resource create fs_NW3_ASCS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW3ascs' \
    directory='/usr/sap/NW3/ASCS20' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-NW3_ASCS

    sudo pcs resource create vip_NW3_ASCS IPaddr2 \
    ip=10.3.1.54 cidr_netmask=24 \
    --group g-NW3_ASCS

    sudo pcs resource create nc_NW3_ASCS azure-lb port=62020 \
    --group g-NW3_ASCS
    ```

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.  

2. **[1]** Installer SAP NetWeaver ASCS  

   Installez SAP NetWeaver ASCS en tant que racine à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration frontend d’équilibreur de charge pour l'instance ASCS. Par exemple, pour le système **NW2**, le nom d’hôte virtuel est <b>msnw2ascs</b>, <b>10.3.1.52</b> et le numéro d’instance que vous avez utilisé pour la sonde de l’équilibreur de charge, par exemple <b>10</b>. Pour le système **NW3**, le nom d’hôte virtuel est <b>msnw3ascs</b>, <b>10.3.1.54</b> et le numéro d’instance que vous avez utilisé pour la sonde de l’équilibreur de charge, par exemple <b>20</b>. Notez le nœud de cluster sur lequel vous avez installé ASCS pour chaque SID SAP.  

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst. Vous pouvez utiliser le paramètre SAPINST_USE_HOSTNAME pour installer SAP à l’aide du nom d’hôte virtuel.  

    ```
    # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again
    sudo firewall-cmd --zone=public --add-port=4237/tcp
    sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   Si aucun sous-dossier n’est créé dans /usr/sap/**SID**/ASCS **Instance#** lors de l'installation, essayez de définir le propriétaire sur **sid** adm et le groupe sur sapsys pour **l'instance ASCS #** , puis réessayez.

3. **[1]** Créez une adresse IP virtuelle et des ressources de cluster de sonde d’intégrité pour l’instance ERS du système SAP supplémentaire que vous déployez sur le cluster. L’exemple indiqué ici concerne **NW2** et **NW3** ERS et utilise NFS sur des volumes Azure NetApp Files avec le protocole NFSv3.  

    ```
    sudo pcs resource create fs_NW2_AERS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW2ers' \
    directory='/usr/sap/NW2/ERS12' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW2_AERS

    sudo pcs resource create vip_NW2_AERS IPaddr2 \
    ip=10.3.1.53 cidr_netmask=24 \
     --group g-NW2_AERS

    sudo pcs resource create nc_NW2_AERS azure-lb port=62112 \
     --group g-NW2_AERS

    sudo pcs resource create fs_NW3_AERS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW3ers' \
    directory='/usr/sap/NW3/ERS22' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW3_AERS

    sudo pcs resource create vip_NW3_AERS IPaddr2 \
    ip=10.3.1.55 cidr_netmask=24 \
     --group g-NW3_AERS

    sudo pcs resource create nc_NW3_AERS azure-lb port=62122 \
     --group g-NW3_AERS
   ```

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées.  

   Assurez-vous ensuite que les ressources du groupe ERS récemment créé sont en cours d’exécution sur le nœud de cluster, à l’inverse du nœud de cluster dans lequel l’instance ASCS correspondant au même système SAP a été installée.  Par exemple, si NW2 ASCS a été installé sur `rhelmsscl1`, assurez-vous que le groupe NW2 ERS est en cours d'exécution sur `rhelmsscl2`.  Pour migrer le groupe NW2 ERS vers `rhelmsscl2`, exécutez la commande suivante pour l’une des ressources de cluster du groupe : 

    ```
      pcs resource move fs_NW2_AERS rhelmsscl2
    ```

4. **[2]** Installer SAP NetWeaver ERS

   Installez l’instance ERS SAP NetWeaver en tant que racine sur l'autre nœud, à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration frontend d’équilibreur de charge pour l'instance ERS. Par exemple, pour le système **NW2**, le nom d’hôte virtuel est <b>msnw2ers</b>, <b>10.3.1.53</b> et le numéro d’instance que vous avez utilisé pour la sonde de l’équilibreur de charge, par exemple <b>12</b>. Pour le système **NW3**, le nom d’hôte virtuel est <b>msnw3ers</b>, <b>10.3.1.55</b> et le numéro d’instance que vous avez utilisé pour la sonde de l’équilibreur de charge, par exemple <b>22</b>. 

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst. Vous pouvez utiliser le paramètre SAPINST_USE_HOSTNAME pour installer SAP à l’aide du nom d’hôte virtuel.  

    ```
    # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again
    sudo firewall-cmd --zone=public --add-port=4237/tcp
    sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   > [!NOTE]
   > Utilisez SWPM SP 20 PL 05 ou ultérieur. Les versions antérieures ne définissent pas les autorisations correctement et l’installation échouera.

   Si aucun sous-dossier n’est créé dans /usr/sap/**NW2**/ERS **Instance#** lors de l'installation, essayez de définir le propriétaire sur **sid** adm et le groupe sur sapsys pour **l'instance ERS #** , puis réessayez.

   Si vous avez dû migrer le groupe ERS du système SAP récemment déployé vers un nœud de cluster différent, n’oubliez pas de supprimer la contrainte d’emplacement pour le groupe ERS. Vous pouvez supprimer cette contrainte en exécutant la commande suivante (l’exemple correspond aux systèmes SAP **NW2** et **NW3**). Veillez à supprimer les contraintes temporaires de la ressource que vous avez utilisée dans la commande pour déplacer le groupe de clusters ERS.

    ```
      pcs resource clear fs_NW2_AERS
      pcs resource clear fs_NW3_AERS
    ```

5. **[1]** Adaptez les profils d’instance ASCS/SCS et ERS au(x) système(s) SAP récemment installé(s). L’exemple illustré ci-dessous correspond à NW2. Vous devez adapter les profils ASCS/SCS et ERS pour toutes les instances SAP ajoutées au cluster.  
 
   * Profil ASCS/SCS

      ```
      sudo vi /sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs
   
      # Change the restart command to a start command
      #Restart_Program_01 = local $(_EN) pf=$(_PF)
      Start_Program_01 = local $(_EN) pf=$(_PF)

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
   
      # remove Autostart from ERS profile
      # Autostart = 1
      ```

6. **[A]** Mettre à jour le fichier /usr/sap/sapservices

   Pour empêcher le démarrage des instances par le script de démarrage sapinit, commentez toutes les instances gérées par Pacemaker dans le fichier `/usr/sap/sapservices`.  L’exemple ci-dessous correspond aux systèmes SAP **NW2** et **NW3**.  

   ```
    # On the node where ASCS was installed, comment out the line for the ASCS instacnes
    #LD_LIBRARY_PATH=/usr/sap/NW2/ASCS10/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW2/ASCS10/exe/sapstartsrv pf=/usr/sap/NW2/SYS/profile/NW2_ASCS10_msnw2ascs -D -u nw2adm
    #LD_LIBRARY_PATH=/usr/sap/NW3/ASCS20/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW3/ASCS20/exe/sapstartsrv pf=/usr/sap/NW3/SYS/profile/NW3_ASCS20_msnw3ascs -D -u nw3adm

    # On the node where ERS was installed, comment out the line for the ERS instacnes
    #LD_LIBRARY_PATH=/usr/sap/NW2/ERS12/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW2/ERS12/exe/sapstartsrv pf=/usr/sap/NW2/ERS12/profile/NW2_ERS12_msnw2ers -D -u nw2adm
    #LD_LIBRARY_PATH=/usr/sap/NW3/ERS22/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW3/ERS22/exe/sapstartsrv pf=/usr/sap/NW3/ERS22/profile/NW3_ERS22_msnw3ers -D -u nw3adm
   ```

7. **[1]** Créez les ressources de cluster SAP pour le système SAP récemment installé.  

   Si vous utilisez l’architecture ENSA1 (serveur de file d’attente 1), définissez les ressources pour les systèmes SAP **NW2** et **NW3** comme suit :

    ```
     sudo pcs property set maintenance-mode=true

    sudo pcs resource create rsc_sap_NW2_ASCS10 SAPInstance \
    InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_ASCS

    sudo pcs resource create rsc_sap_NW2_ERS12 SAPInstance \
    InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_AERS

    sudo pcs constraint colocation add g-NW2_AERS with g-NW2_ASCS -5000
    sudo pcs constraint location rsc_sap_NW2_ASCS10 rule score=2000 runs_ers_NW2 eq 1
    sudo pcs constraint order g-NW2_ASCS then g-NW2_AERS kind=Optional symmetrical=false

    sudo pcs resource create rsc_sap_NW3_ASCS20 SAPInstance \
    InstanceName=NW3_ASCS20_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_ASCS

    sudo pcs resource create rsc_sap_NW3_ERS22 SAPInstance \
    InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW2_ERS22_msnw3ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_AERS

    sudo pcs constraint colocation add g-NW3_AERS with g-NW3_ASCS -5000
    sudo pcs constraint location rsc_sap_NW3_ASCS20 rule score=2000 runs_ers_NW3 eq 1
    sudo pcs constraint order g-NW3_ASCS then g-NW3_AERS kind=Optional symmetrical=false

    sudo pcs property set maintenance-mode=false
    ```

   SAP a introduit la prise en charge du serveur de file d’attente 2, y compris la réplication, avec SAP NW 7.52. À partir de la plateforme ABAP 1809, le serveur de file d’attente 2 est installé par défaut. Consultez la note SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) pour plus d’informations sur la prise en charge du serveur de file d’attente 2.
   Si vous utilisez l’architecture ENSA2 ([serveur de file d’attente 2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), définissez les ressources pour les systèmes SAP **NW2** et **NW3** comme indiqué ci-après :

    ```
     sudo pcs property set maintenance-mode=true

    sudo pcs resource create rsc_sap_NW2_ASCS10 SAPInstance \
    InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_ASCS

    sudo pcs resource create rsc_sap_NW2_ERS12 SAPInstance \
    InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_AERS

    sudo pcs constraint colocation add g-NW2_AERS with g-NW2_ASCS -5000
    sudo pcs constraint order g-NW2_ASCS then g-NW2_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-NW2_ASCS then stop g-NW2_AERS symmetrical=false

    sudo pcs resource create rsc_sap_NW3_ASCS20 SAPInstance \
    InstanceName=NW3_ASCS20_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_ASCS

    sudo pcs resource create rsc_sap_NW3_ERS22 SAPInstance \
    InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW2_ERS22_msnw3ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_AERS

    sudo pcs constraint colocation add g-NW3_AERS with g-NW3_ASCS -5000
    sudo pcs constraint order g-NW3_ASCS then g-NW3_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-NW3_ASCS then stop g-NW3_AERS symmetrical=false

    sudo pcs property set maintenance-mode=false
    ```

   Si vous effectuez une mise à niveau à partir d’une version antérieure et que vous passez au serveur de file d’attente 2, consultez la note SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   > [!NOTE]
   > Les délais d’expiration de la configuration ci-dessus sont simplement des exemples et doivent être adaptés à la configuration SAP spécifique. 

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.
   L’exemple suivant illustre l’état des ressources de cluster, après ajout des systèmes SAP **NW2** et **NW3** au cluster. 

    ```
     sudo pcs status

    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    ```

8. **[A]** Ajoutez des règles de pare-feu pour ASCS et ERS sur les deux nœuds.  L’exemple ci-dessous montre celles des systèmes SAP **NW2** et **NW3**.  

   ```
    # NW2 - ASCS
    sudo firewall-cmd --zone=public --add-port=62010/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62010/tcp
    sudo firewall-cmd --zone=public --add-port=3210/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3210/tcp
    sudo firewall-cmd --zone=public --add-port=3610/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3610/tcp
    sudo firewall-cmd --zone=public --add-port=3910/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3910/tcp
    sudo firewall-cmd --zone=public --add-port=8110/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=8110/tcp
    sudo firewall-cmd --zone=public --add-port=51013/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51013/tcp
    sudo firewall-cmd --zone=public --add-port=51014/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51014/tcp
    sudo firewall-cmd --zone=public --add-port=51016/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51016/tcp
    # NW2 - ERS
    sudo firewall-cmd --zone=public --add-port=62112/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62112/tcp
    sudo firewall-cmd --zone=public --add-port=3312/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3312/tcp
    sudo firewall-cmd --zone=public --add-port=51213/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51213/tcp
    sudo firewall-cmd --zone=public --add-port=51214/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51214/tcp
    sudo firewall-cmd --zone=public --add-port=51216/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51216/tcp
    # NW3 - ASCS
    sudo firewall-cmd --zone=public --add-port=62020/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62020/tcp
    sudo firewall-cmd --zone=public --add-port=3220/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3220/tcp
    sudo firewall-cmd --zone=public --add-port=3620/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3620/tcp
    sudo firewall-cmd --zone=public --add-port=3920/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3920/tcp
    sudo firewall-cmd --zone=public --add-port=8120/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=8120/tcp
    sudo firewall-cmd --zone=public --add-port=52013/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52013/tcp
    sudo firewall-cmd --zone=public --add-port=52014/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52014/tcp
    sudo firewall-cmd --zone=public --add-port=52016/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52016/tcp
    # NW3 - ERS
    sudo firewall-cmd --zone=public --add-port=62122/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62122/tcp
    sudo firewall-cmd --zone=public --add-port=3322/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3322/tcp
    sudo firewall-cmd --zone=public --add-port=52213/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52213/tcp
    sudo firewall-cmd --zone=public --add-port=52214/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52214/tcp
    sudo firewall-cmd --zone=public --add-port=52216/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52216/tcp
   ```

### <a name="proceed-with-the-sap-installation"></a>Poursuivre l’installation de SAP 

Finalisez votre installation SAP en procédant comme suit :

* [Préparer vos serveurs d'applications SAP NetWeaver](./high-availability-guide-rhel-netapp-files.md#2d6008b0-685d-426c-b59e-6cd281fd45d7)
* [Installer une instance SGBD](./high-availability-guide-rhel-netapp-files.md#install-database)
* [Installer un serveur d’applications SAP principal](./high-availability-guide-rhel-netapp-files.md#sap-netweaver-application-server-installation)
* Installer une ou plusieurs instances d’application SAP supplémentaires

## <a name="test-the-multi-sid-cluster-setup"></a>Tester la configuration de cluster multi-SID

Les tests suivants représentent une partie des cas de test des guides des meilleures pratiques de Red Hat. Ils sont inclus pour des raisons pratiques. Pour obtenir la liste complète des tests de cluster, reportez-vous à la documentation suivante :

* Si vous utilisez des volumes NFS Azure NetApp Files, consultez [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur RHEL avec Azure NetApp Files for SAP Applications](./high-availability-guide-rhel-netapp-files.md).
* Si vous utilisez un cluster `GlusterFS` hautement disponible, consultez [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur RHEL for SAP Applications](./high-availability-guide-rhel.md).  

Consultez systématiquement les guides des meilleures pratiques Red Hat et effectuez tous les tests qui y sont ajoutés.  
Les tests présentés se trouvent dans un cluster à deux nœuds, multi-SID, avec trois systèmes SAP installés.  

1. Migrer manuellement l’instance ASCS L’exemple montre comment migrer l’instance ASCS pour le système SAP NW3.

   État des ressources avant le début du test :

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
   ```

   Exécutez les commandes suivantes en tant que racine pour migrer l’instance NW3 ASCS.

   ```
    pcs resource move rsc_sap_NW3_ASCS200
    # Clear temporary migration constraints
    pcs resource clear rsc_sap_NW3_ASCS20

    # Remove failed actions for the ERS that occurred as part of the migration
    pcs resource cleanup rsc_sap_NW3_ERS22
   ```

   État des ressources après le test :

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
   ```

1. Simuler l’incident de nœud

   État des ressources avant le début du test :

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
   ```

   Exécutez la commande suivante en tant que racine sur un nœud où au moins une instance ASCS est en cours d’exécution. Dans cet exemple, nous avons exécuté la commande sur `rhelmsscl1`, où les instances ASCS pour NW1, NW2 et NW3 sont en cours d’exécution.  

   ```
   echo c > /proc/sysrq-trigger
   ```

   L’état après le test, et après le nœud, qui a rencontré un incident, se présente ainsi.

   ```
    Full list of resources:

    rsc_st_azure    (stonith:fence_azure_arm):      Started rhelmsscl2
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
   ```

   S’il y a des messages d’échec pour certaines ressources, nettoyez l’état de ces ressources. Par exemple :

   ```
   pcs resource cleanup rsc_sap_NW1_ERS02
   ```

## <a name="next-steps"></a>Étapes suivantes

* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur des machines virtuelles Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha]