---
title: Montée en charge SAP HANA avec HSR et Pacemaker sur RHEL | Microsoft Docs
description: Montée en charge SAP HANA avec HSR et Pacemaker sur RHEL
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
ms.openlocfilehash: 23a5ea2d3ffc1511bea66bb8bc3c4282b6d16cc2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489120"
---
# <a name="high-availability-of-sap-hana-scale-out-system-on-red-hat-enterprise-linux"></a>Haute disponibilité du système de montée en charge SAP HANA sur Red Hat Enterprise Linux 

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all 
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
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
[1900823]:https://launchpad.support.sap.com/#/notes/1900823
[2292690]:https://launchpad.support.sap.com/#/notes/2292690
[2455582]:https://launchpad.support.sap.com/#/notes/2455582
[2593824]:https://launchpad.support.sap.com/#/notes/2593824
[2009879]:https://launchpad.support.sap.com/#/notes/2009879

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md


Cet article explique comment déployer un système SAP HANA à haut niveau de disponibilité dans une configuration de montée en charge avec réplication de système HANA (HSR) et Pacemaker sur les machines virtuelles Red Hat Enterprise Linux Azure. Les systèmes de fichiers partagés dans l’architecture présentée sont fournis par [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) et sont montés sur NFS.  

Dans les exemples de configuration, de commandes d’installation, etc., l’instance HANA est **03** et l’ID de système HANA est **HN1**. Les exemples sont basés sur HANA 2.0 SP4 et Red Hat Enterprise Linux pour SAP 7.6. 

Avant de commencer, reportez-vous aux notes et documents SAP suivants :

* La note SAP [1928533] comprend :  
  * une liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP
  * des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure
  * les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données
  * la version du noyau SAP requise pour Windows et Linux sur Microsoft Azure
* Note SAP [2015553] : répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure
* La note SAP [2002167] recommande des paramètres de système d’exploitation pour Red Hat Enterprise Linux
* La note SAP [2009879] conseille sur SAP HANA pour Red Hat Enterprise Linux
* Note SAP [2178632] : contient des informations détaillées sur toutes les métriques de supervision rapportées pour SAP dans Azure
* Note SAP [2191498] : contient la version requise de l’agent hôte SAP pour Linux dans Azure
* Note SAP [2243692] : contient des informations sur les licences SAP sur Linux dans Azure
* Note SAP [1999351] : contient des informations supplémentaires relatives à la résolution des problèmes pour l’extension d’analyse Azure améliorée pour SAP
* Note SAP [1900823] : contient des informations sur les exigences de stockage SAP HANA
* [Wiki de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) : contient toutes les notes SAP requises pour Linux
* [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP sur Linux][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP sur Linux][dbms-guide]
* [Configuration requise pour le réseau SAP HANA](https://www.sap.com/documents/2016/08/1cd2c2fb-807c-0010-82c7-eda71af511fa.html)
* Documentation RHEL générale
  * [Vue d’ensemble des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administration des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Référence des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Guide de mise en réseau Red Hat Enterprise Linux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide)
  * [Comment configurer la réplication de système de montée en charge SAP HANA dans un cluster Pacemaker avec les systèmes de fichiers HANA sur des partages NFS](https://access.redhat.com/solutions/5423971)
* Documentation RHEL spécifique à Azure :
  * [Installation de SAP HANA sur Red Hat Enterprise Linux pour une utilisation dans Microsoft Azure](https://access.redhat.com/public-cloud/microsoft-azure)
  * [Solution Red Hat Enterprise Linux pour la montée en charge SAP HANA et la réplication de système](https://access.redhat.com/solutions/4386601)
* [Applications SAP NetApp su Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure]
* [Documentation Azure NetApp Files][anf-azure-doc] 


## <a name="overview"></a>Vue d’ensemble

Une méthode pour obtenir une haute disponibilité HANA pour les installations de montée en charge HANA consiste à configurer la réplication de système HANA et à protéger la solution avec le cluster Pacemaker afin d’autoriser le basculement automatique. En cas de défaillance d’un nœud actif, le cluster bascule les ressources HANA vers l’autre site.  
La configuration présentée montre trois nœuds HANA sur chaque site, ainsi qu’un nœud de créateur de majorité pour empêcher le scénario de split-brain. Les instructions peuvent être adaptées pour inclure plus de machines virtuelles en tant que nœuds de base de données HANA.  

Le système de fichiers partagé HANA `/hana/shared` dans l’architecture présentée est fourni par [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md). Il est monté via NFSv4.1 sur chaque nœud HANA du même site de réplication de système HANA. Les systèmes de fichiers `/hana/data` et `/hana/log` sont des systèmes de fichiers locaux et ne sont pas partagés entre les nœuds de base de données HANA. SAP HANA sera installé en mode non partagé. 

> [!TIP]
> Pour des configurations de stockage recommandées SAP HANA, consultez [Configurations de stockage SAP HANA des machines virtuelles Azure](./hana-vm-operations-storage.md).   

[![Scale-out SAP HANA avec HSR et cluster Pacemaker](./media/sap-hana-high-availability-rhel/sap-hana-high-availability-scale-out-hsr-rhel.png)](./media/sap-hana-high-availability-rhel/sap-hana-high-availability-scale-out-hsr-rhel-detail.png#lightbox)

Dans le diagramme ci-dessus, trois sous-réseaux sont représentés au sein d’un réseau virtuel Azure, qui suit les recommandations de réseau SAP HANA : 
* pour la communication client - `client` 10.23.0.0/24  
* pour la communication interne HANA entre les nœuds - `inter` 10.23.1.128/26  
* pour la réplication de système HANA - `hsr` 10.23.1.192/26  

Comme `/hana/data` et `/hana/log` sont déployés sur des disques locaux, il n’est pas nécessaire de déployer un sous-réseau distinct et des cartes réseau virtuelles distinctes pour la communication au stockage.  

Les volumes Azure NetApp sont déployés dans un sous-réseau distinct, [délégué à Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-delegate-subnet: `anf` 10.23.1.0/26.   

## <a name="set-up-the-infrastructure"></a>Configurer l’infrastructure

Les instructions suivantes supposent que vous avez déjà créé le groupe de ressources, le réseau virtuel Azure avec les trois sous-réseaux de réseau Azure : `client`, `inter` et `hsr`.

### <a name="deploy-linux-virtual-machines-via-the-azure-portal"></a>Déployer des machines virtuelles Linux via le Portail Azure
1. Déployer les machines virtuelles Azure.  
Pour la configuration présentée dans ce document, déployez sept machines virtuelles : 
   - trois machines virtuelles à servir de nœuds de base de données HANA pour le site de réplication HANA 1 : **hana-s1-db1**, **hana-s1-db2** et **hana-s1-db3**  
   - trois machines virtuelles à servir de nœuds de base de données HANA pour le site de réplication HANA 2 : **hana-s2-db1**, **hana-s2-db2** et **hana-s2-db3**  
   - une petite machine virtuelle à servir de *créateur de majorité* : **hana-s-mm**

   Les machines virtuelles, déployées en tant que nœuds HANA de base de données SAP doivent être certifiées par SAP pour HANA, telles qu’elles sont publiées dans le [Répertoire matériel SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Lors du déploiement des nœuds de base de données HANA, assurez-vous que [Accelerated Network](../../../virtual-network/create-vm-accelerated-networking-cli.md) (réseau accélérée) est sélectionné.  
  
   Pour le nœud de créateur de majorité, vous pouvez déployer une petite machine virtuelle, car cette machine virtuelle n’exécute aucune des ressources SAP HANA. La machine virtuelle de créateur de majorité est utilisée dans la configuration du cluster pour obtenir un nombre impair de nœuds de cluster dans un scénario de split-brain. La machine virtuelle de créateur de majorité n’a besoin que d’une seule interface réseau virtuelle dans le sous-réseau `client` dans cet exemple.        

   Déployer des disques managés locaux pour `/hana/data` et `/hana/log`. La configuration de stockage minimale recommandée pour `/hana/data` et `/hana/log` est décrite dans [Configurations de stockage SAP HANA des machines virtuelles Azure](./hana-vm-operations-storage.md).

   Déployez l’interface réseau principale pour chaque machine virtuelle dans le sous-réseau du réseau virtuel `client`.  
   Lorsque la machine virtuelle est déployée via le Portail Azure, le nom de l’interface réseau est généré automatiquement. Dans ces instructions, par soucis de simplicité, nous allons faire référence aux interfaces réseau principales générées automatiquement, qui sont attachées au sous-réseau `client` du réseau virtuel Azure, en tant que **hana-s1-db1-client**, **hana-s1-db2-client**, **hana-s1-db3-client**, etc.  


   > [!IMPORTANT]
   > Vérifiez que le système d’exploitation que vous sélectionnez est certifié SAP pour SAP HANA sur les types de machine virtuelle spécifiques que vous utilisez. Pour obtenir la liste des types de machines virtuelles certifiés SAP HANA et des versions de système d’exploitation correspondants, accédez au site [SAP HANA Certified IaaS platforms](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Cliquez sur les détails du type de machine virtuelle répertorié pour obtenir la liste complète des versions de système d’exploitation prises en charge par SAP HANA pour ce type.  
  

2. Créez six interfaces réseau, une pour chaque machine virtuelle de base de données HANA, dans le sous-réseau `inter` du réseau virtuel (dans cet exemple, **hana-s1-db1-inter**, **hana-s1-db2-inter**, **hana-s1-db3-inter**, **hana-s2-db1-inter**, **hana-s2-db2-inter**, et **hana-s2-db3-inter**).  

3. Créez six interfaces réseau, une pour chaque machine virtuelle de base de données HANA, dans le sous-réseau `hsr` du réseau virtuel (dans cet exemple, **hana-s1-db1-hsr**, **hana-s1-db2-hsr**, **hana-s1-db3-hsr**, **hana-s2-db1-hsr**, **hana-s2-db2-hsr**, et **hana-s2-db3-hsr**).  

4. Attachez les interfaces réseau virtuelles nouvellement créées aux machines virtuelles correspondantes :  

    a. Accédez à la machine virtuelle sur le [Portail Azure](https://portal.azure.com/#home).  

    b. Dans le volet gauche, sélectionnez **Machines virtuelles**. Filtrez sur le nom de la machine virtuelle (par exemple, **hana-s1-db1**), puis sélectionnez la machine virtuelle.  

    c. Dans le volet **Vue d’ensemble**, sélectionnez **Arrêter** pour libérer la machine virtuelle.  

    d. Sélectionnez **Mise en réseau**, puis attachez l’interface réseau. Dans la liste déroulante sous **Attacher l’interface réseau**, sélectionnez les interfaces réseau déjà créées pour les sous-réseaux `inter` et `hsr`.  
    
    e. Sélectionnez **Enregistrer**. 
 
    f. Répétez les étapes b à e pour les machines virtuelles restantes (dans notre exemple,  **hana-s1-db2**, **hana-s1-db3**, **hana-s2-db1**, **hana-s2-db2** et **hana-s2-db3**).
 
    g. Laissez les machines virtuelles dans l’état arrêté pour l’instant. Ensuite, nous allons activer [Mise en réseau accélérée](../../../virtual-network/create-vm-accelerated-networking-cli.md) pour toutes les interfaces réseau qui viennent d’être attachées.  

5. Activez la mise en réseau accélérée pour les interfaces réseau supplémentaires pour les sous-réseaux `inter` et `hsr` en procédant comme suit :  

    a. Ouvrez [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) dans le [Portail Azure](https://portal.azure.com/#home).  

    b. Exécutez les commandes suivantes pour activer la mise en réseau accélérée pour les interfaces réseau supplémentaires, qui sont attachées aux sous-réseaux `inter` et `hsr`.  

    ```
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db1-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db2-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db3-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db1-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db2-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db3-inter --accelerated-networking true
    
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db1-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db2-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db3-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db1-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db2-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db3-hsr --accelerated-networking true
    ```

7. Démarrer les machines virtuelles de base de données HANA

### <a name="deploy-azure-load-balancer"></a>Déployer Azure Load Balancer

1. Nous vous recommandons d’utiliser l’équilibreur de charge standard. Suivez ces étapes de configuration pour déployer l’équilibreur de charge standard :
   1. Commencez par créer un pool d’adresses IP frontales :

      1. Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
      1. Entrez le nom du nouveau pool d’adresses IP frontales (par exemple **hana-frontend**).
      1. Définissez l’**Affectation** sur **Statique** et entrez l’adresse IP (par exemple, **10.23.0.18**).
      1. Sélectionnez **OK**.
      1. Une fois le pool d’adresses IP frontal créé, notez son adresse IP.

   1. Ensuite, créez un pool de back-ends et ajoutez toutes les machines virtuelles du cluster au pool de back-ends :

      1. Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
      1. Entrer le nom du nouveau pool principal (par exemple **hana-backend**).
      1. Cliquez sur **Ajouter une machine virtuelle**.
      1. Sélectionnez **Machine virtuelle**.
      1. Sélectionnez les machines virtuelles du cluster SAP HANA et leurs adresses IP pour le sous-réseau `client`.
      1. Sélectionnez **Ajouter**.

   1. Créez ensuite une sonde d’intégrité :

      1. Ouvrez l’équilibrage de charge, sélectionnez les **sondes d’intégrité**, puis cliquez sur **Ajouter**.
      1. Entrez le nom de la nouvelle sonde d’intégrité (par exemple **hana-hp**).
      1. Sélectionnez **TCP** pour le protocole et le port 625 **03**. Consersez la valeur **Intervalle** à 5, et la valeur **Seuil de défaillance** à 2.
      1. Sélectionnez **OK**.

   1. Ensuite, créez les règles d’équilibrage de charge :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter**.
      1. Entrez le nom de la nouvelle règle d’équilibrage de charge (par exemple, **hana-lb**).
      1. Sélectionnez l’adresse IP frontale, le pool principal et la sonde d’intégrité que vous avez créés (par exemple,**hana-frontend**, **hana-backend** et **hana-hp**).
      1. Sélectionnez **Ports HA**.
      1. Augmentez le **délai d’inactivité** à 30 minutes.
      1. Veillez à **activer l’IP flottante** .
      1. Sélectionnez **OK**.

   > [!IMPORTANT]
   > Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d’informations, consultez [Limitations d’équilibreur de charge Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.    
   
   > [!Note]
   > Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité Internet sortante, sauf si une configuration supplémentaire est effectuée pour autoriser le routage vers des points de terminaison publics. Pour savoir plus en détails comment bénéficier d’une connectivité sortante, voir [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  


   > [!IMPORTANT]
   > N’activez pas les timestamps TCP sur des machines virtuelles Azure placées derrière Azure Load Balancer. L’activation des timestamps TCP entraîne l’échec des sondes d’intégrité. Définissez le paramètre **net.ipv4.tcp_timestamps** sur **0**. Pour plus d’informations, consultez [Load Balancer health probes](../../../load-balancer/load-balancer-custom-probe-overview.md) (Sondes d’intégrité Load Balancer).
   > Voir aussi la note SAP [2382421](https://launchpad.support.sap.com/#/notes/2382421).  

### <a name="deploy-the-azure-netapp-files-infrastructure"></a>Déployer l’infrastructure Azure NetApp Files 

Déployez les volumes ANF pour le système de fichiers `/hana/shared`. Vous aurez besoin d’un volume `/hana/shared` distinct pour chaque site de réplication de système HANA. Pour plus d’informations, consultez [Configurer l’infrastructure Azure NetApp Files](./sap-hana-scale-out-standby-netapp-files-rhel.md#set-up-the-azure-netapp-files-infrastructure).

Dans cet exemple, les volumes Azure NetApp Files suivants ont été utilisés : 

* volume **HN1**-shared-s1 (nfs://10.23.1.7/**HN1**-shared-s1)
* volume **HN1**-shared-s2 (nfs://10.23.1.7/**HN1**-shared-s2)

## <a name="operating-system-configuration-and-preparation"></a>Configuration et préparation du système d’exploitation

Les instructions des sections suivantes sont précédées de l’une des abréviations suivantes :
* **[A]**  :      applicable à tous les nœuds
* **[AH]**  :     applicable à tous les nœuds de base de données HANA
* **[M]**  :      applicable au nœud de créateur de majorité
* **[AH1]**  :    applicable à tous les nœuds de base de données HANA sur le SITE 1
* **[AH2]**  :    applicable à tous les nœuds de base de données HANA sur le SITE 2
* **[1]**  :      applicable uniquement au nœud de base de données HANA 1, le SITE 1
* **[2]**  :      applicable uniquement au nœud de base de données HANA 1, le SITE 2


Configurez et préparez votre système d’exploitation en procédant comme suit :

1. **[A]** Conservez les fichiers hôtes sur les machines virtuelles. Incluez des entrées pour tous les sous-réseaux. Les entrées suivantes ont été ajoutées à `/etc/hosts` pour cet exemple.  

    ```
     # Client subnet
     10.23.0.11 hana-s1-db1
     10.23.0.12 hana-s1-db1
     10.23.0.13 hana-s1-db2
     10.23.0.14 hana-s2-db1
     10.23.0.15 hana-s2-db2
     10.23.0.16 hana-s2-db3
     10.23.0.17 hana-s-mm
     # Internode subnet
     10.23.1.138 hana-s1-db1-inter
     10.23.1.139 hana-s1-db2-inter
     10.23.1.140 hana-s1-db3-inter
     10.23.1.141 hana-s2-db1-inter
     10.23.1.142 hana-s2-db2-inter
     10.23.1.143 hana-s2-db3-inter
     # HSR subnet
     10.23.1.202 hana-s1-db1-hsr
     10.23.1.203 hana-s1-db2-hsr
     10.23.1.204 hana-s1-db3-hsr
     10.23.1.205 hana-s2-db1-hsr
     10.23.1.206 hana-s2-db2-hsr
     10.23.1.207 hana-s2-db3-hsr
    ```


2. **[A]** Installer le package client NFS.  

    ```yum install nfs-utils ```


3. **[AH]** Red Hat pour la configuration HANA.  

    Configurez RHEL tel que décrit dans <https://access.redhat.com/solutions/2447641> et dans les notes SAP suivantes :  
   - [2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/2292690) (SAP HANA DB : Paramètres de système d’exploitation recommandés pour RHEL 7)
   - [2777782 – Base de données SAP HANA : paramètres de système d’exploitation recommandés pour RHEL 8](https://launchpad.support.sap.com/#/notes/2777782)
   - [2455582 – Linux : Exécution d’applications SAP compilées avec GCC 6.x](https://launchpad.support.sap.com/#/notes/2455582)
   - [2593824 – Linux : Exécution d’applications SAP compilées avec GCC 7.x](https://launchpad.support.sap.com/#/notes/2593824) 
   - [2886607 – Linux : Exécution d’applications SAP compilées avec GCC 9.x](https://launchpad.support.sap.com/#/notes/2886607)

## <a name="prepare-the-file-systems"></a>Préparer les systèmes de fichiers
### <a name="mount-the-shared-file-systems"></a>Monter les systèmes de fichiers partagés

Dans cet exemple, les systèmes de fichiers HANA partagés sont déployés sur Azure NetApp Files et montés sur NFSv4.  

1. **[AH]** Créez des points de montage pour les volumes de base de données HANA.  

    ```
    mkdir -p /hana/shared
    ```

2. **[AH]** Vérifiez le paramètre de domaine NFS. Assurez-vous que le domaine est configuré en tant que domaine par défaut Azure NetApp Files, autrement dit, **`defaultv4iddomain.com`** et que le mappage est défini sur **nobody**.  
   Cette étape n’est nécessaire que si vous utilisez Azure NetAppFiles NFSv4.1.  

    > [!IMPORTANT]
    > Veillez à définir le domaine NFS dans `/etc/idmapd.conf` sur la machine virtuelle pour qu’elle corresponde à la configuration de domaine par défaut sur Azure NetApp Files : **`defaultv4iddomain.com`** . En cas d’incompatibilité entre la configuration de domaine sur le client NFS (c’est-à-dire la machine virtuelle) et le serveur NFS, par exemple la configuration Azure NetApp, les autorisations pour les fichiers sur les volumes Azure NetApp montés sur les machines virtuelles s’affichent en tant que `nobody`.  

    ```
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = defaultv4iddomain.com
    [Mapping]
    Nobody-User = nobody
    Nobody-Group = nobody
    ```

3. **[AH]** Vérifiez `nfs4_disable_idmapping`. Cette option doit avoir la valeur **Y**. Pour créer la structure de répertoire où se trouve `nfs4_disable_idmapping`, exécutez la commande de montage. Vous ne serez pas en mesure de créer manuellement le répertoire sous /sys/modules, car l’accès est réservé pour le noyau/les pilotes.  
   Cette étape n’est nécessaire que si vous utilisez Azure NetAppFiles NFSv4.1.  

    ```
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    # If you need to set nfs4_disable_idmapping to Y
    mkdir /mnt/tmp
    mount 10.9.0.4:/HN1-shared /mnt/tmp
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    ```

   Pour plus d’informations sur la modification du paramètre `nfs4_disable_idmapping`, consultez https://access.redhat.com/solutions/1749883.

4. **[AH1]** Montez les volumes Azure NetApp Files partagés sur les machines virtuelles du SITE 1 de base de données HANA.  

    ```
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.23.1.7:/HN1-shared-s1 /hana/shared
    ```

5. **[AH2]** Montez les volumes Azure NetApp Files partagés sur les machines virtuelles du SITE 2 de base de données HANA.  

    ```
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.23.1.7:/HN1-shared-s2 /hana/shared
    ```


10. **[AH]** Vérifiez que les systèmes de fichiers `/hana/shared/` correspondants sont montés sur toutes les machines virtuelles de base de données HANA avec la version de protocole NFS **NFSv4**.  

    ```
    sudo nfsstat -m
    # Verify that flag vers is set to 4.1 
    # Example from SITE 1, hana-s1-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s1
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.11,local_lock=none,addr=10.23.1.7
    # Example from SITE 2, hana-s2-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s2
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.14,local_lock=none,addr=10.23.1.7
    ```

### <a name="prepare-the-data-and-log-local-file-systems"></a>Préparer les données et enregistrer les systèmes de fichiers locaux
Dans la configuration présentée, les systèmes de fichiers `/hana/data` et `/hana/log` sont déployés sur un disque managé et sont attachés localement à chaque machine virtuelle de base de données HANA. Vous devez exécuter les étapes pour créer les volumes de données et de fichier journal locaux sur chaque machine virtuelle de base de données HANA. 

Configurez la disposition du disque avec **Logical Volume Manager (LVM)** . L’exemple suivant part du principe que chaque machine virtuelle HANA dispose de trois disques de données joints qui sont utilisés pour utilisés deux volumes.

1. **[AH]** Répertoriez tous les disques disponibles :
    ```
    ls /dev/disk/azure/scsi1/lun*
    ```

   Exemple de sortie :

    ```
    /dev/disk/azure/scsi1/lun0  /dev/disk/azure/scsi1/lun1  /dev/disk/azure/scsi1/lun2 
    ```

2. **[AH]** Créez des volumes physiques pour tous les disques que vous souhaitez utiliser :
    ```
    sudo pvcreate /dev/disk/azure/scsi1/lun0
    sudo pvcreate /dev/disk/azure/scsi1/lun1
    sudo pvcreate /dev/disk/azure/scsi1/lun2
    ```

3. **[AH]** Créez un groupe de volume pour les fichiers de données. Utilisez un groupe de volume pour les fichiers journaux et un autre pour le répertoire partagé de SAP HANA :
    ```
    sudo vgcreate vg_hana_data_HN1 /dev/disk/azure/scsi1/lun0 /dev/disk/azure/scsi1/lun1
    sudo vgcreate vg_hana_log_HN1 /dev/disk/azure/scsi1/lun2
    ```

4. **[AH]** Créez les volumes logiques. 
   Un volume linéaire est créé lorsque vous utilisez `lvcreate` sans le commutateur `-i`. Nous vous suggérons de créer un volume agrégé par bandes pour obtenir de meilleures performances d’E/S, et d’aligner les tailles des bandes sur les valeurs décrites dans [Configurations de stockage de machines virtuelles SAP HANA](./hana-vm-operations-storage.md). L’argument `-i` doit indiquer le nombre de volumes physiques sous-jacents et l’argument `-I` la taille de bande. Dans ce document, deux volumes physiques sont utilisés pour le volume de données. Par conséquent, l’argument de commutateur `-i` est défini sur **2**. La taille de bande pour le volume de données est de **256 Kio**. Un volume physique étant utilisé pour le volume du fichier journal, aucun commutateur `-i` ou `-I` n’est utilisé explicitement pour les commandes de volume du fichier journal.  

   > [!IMPORTANT]
   > Utilisez le commutateur `-i` et définissez sa valeur sur le nombre de volumes physiques sous-jacents lorsque vous utilisez plusieurs volumes physiques pour chaque volume de données ou de journal. Utilisez le commutateur `-I` pour spécifier la taille de bande lors de la création d’un volume agrégé par bandes.  
   > Pour connaître les configurations de stockage recommandées, notamment les tailles de bande et le nombre de disques, consultez [Configurations de stockage de machines virtuelles SAP HANA](./hana-vm-operations-storage.md).  

    ```
    sudo lvcreate -i 2 -I 256 -l 100%FREE -n hana_data vg_hana_data_HN1
    sudo lvcreate -l 100%FREE -n hana_log vg_hana_log_HN1
    sudo mkfs.xfs /dev/vg_hana_data_HN1/hana_data
    sudo mkfs.xfs /dev/vg_hana_log_HN1/hana_log
    ```

5. **[AH]** Créez les répertoires de montage et copiez l’UUID de tous les volumes logiques :
    ```
    sudo mkdir -p /hana/data/HN1
    sudo mkdir -p /hana/log/HN1
    # Write down the ID of /dev/vg_hana_data_HN1/hana_data and /dev/vg_hana_log_HN1/hana_log
    sudo blkid
    ```

6. **[AH]** Créez des entrées `fstab` pour les volumes logiques et le montage :
    ```
    sudo vi /etc/fstab
    ```

   Insérez la ligne suivante dans le fichier `/etc/fstab` :

    ```
    /dev/disk/by-uuid/UUID of /dev/mapper/vg_hana_data_HN1-hana_data /hana/data/HN1 xfs  defaults,nofail  0  2
    /dev/disk/by-uuid/UUID of /dev/mapper/vg_hana_log_HN1-hana_log /hana/log/HN1 xfs  defaults,nofail  0  2
    ```

   Montez les nouveaux volumes :

    ```
    sudo mount -a
    ```

## <a name="installation"></a>Installation  

Dans cet exemple, pour le déploiement de SAP HANA dans une configuration de montée en charge avec HSR sur des machines virtuelles Azure, nous avons utilisé HANA 2.0 SP4.  

### <a name="prepare-for-hana-installation"></a>Préparer l’installation de HANA

1. **[AH]** Avant l’installation de HANA, définissez le mot de passe racine. Vous pouvez désactiver le mot de passe racine une fois l’installation terminée. Exécutez en tant que commande `root``passwd`.  

2. **[1,2]** Changez les autorisations sur `/hana/shared` 
    ```
    chmod 775 /hana/shared
    ```

3. **[1]** Vérifiez que vous pouvez vous connecter via SSH sur les machines virtuelles de base de données HANA dans ce site **hana-s1-db2** et **hana-s1-db3** sans être invité à entrer un mot de passe.  
   Si ce n’est pas le cas, échangez les clés SSH, comme indiqué dans [Utiliser l’authentification basée sur les clés](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s2-ssh-configuration-keypairs).  
    ```
    ssh root@hana-s1-db2
    ssh root@hana-s1-db3
    ```

4. **[2]** Vérifiez que vous pouvez vous connecter via SSH sur les machines virtuelles de base de données HANA dans ce site **hana-s2-db2** et **hana-s2-db3** sans être invité à entrer un mot de passe.  
   Si ce n’est pas le cas, échangez les clés SSH, comme indiqué dans [Utiliser l’authentification basée sur les clés](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s2-ssh-configuration-keypairs).  
    ```
    ssh root@hana-s2-db2
    ssh root@hana-s2-db3
    ```

5. **[AH]** Installez des packages supplémentaires, qui sont nécessaires pour HANA 2.0 SP4. Pour plus d’informations, consultez la note SAP [2593824](https://launchpad.support.sap.com/#/notes/2593824) pour RHEL 7. 

    ```
    # If using RHEL 7
    yum install libgcc_s1 libstdc++6 compat-sap-c++-7 libatomic1
    # If using RHEL 8
    yum install libatomic libtool-ltdl.x86_64
    ```


6. **[A]** Désactivez temporairement le pare-feu, afin qu’il n’interfère pas avec l’installation de HANA. Vous pouvez le réactiver une fois l’installation terminée. 
    ```
    # Execute as root
    systemctl stop firewalld
    systemctl disable firewalld
    ```

### <a name="hana-installation-on-the-first-node-on-each-site"></a>Installation de HANA sur le premier nœud de chaque site

1. **[1]** Installez SAP HANA en suivant les instructions fournies dans le [Guide d’installation et de mise à jour de SAP HANA 2.0](https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.04/en-US/7eb0167eb35e4e2885415205b8383584.html). Dans les instructions qui suivent, nous présentons l’installation de SAP HANA sur le premier nœud sur le SITE 1.   

   a. Démarrez le programme **hdblcm** en tant que `root` à partir du répertoire du logiciel d’installation HANA. Utilisez le paramètre `internal_network` et transmettez l’espace d’adressage pour le sous-réseau, qui est utilisé pour la communication interne HANA entre les nœuds.  

    ```
    ./hdblcm --internal_network=10.23.1.128/26
    ```

   b. À l’invite, entrez les valeurs suivantes :

     * Pour **Choisir une action** : entrer **1** (pour l’installation)
     * Pour **Composants supplémentaires pour l’installation** : entrer **2, 3**
     * Pour le chemin d’accès de l’installation : appuyer sur Entrée (utilise par défaut /hana/shared)
     * Pour **Nom d’hôte local** : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Voulez-vous ajouter des hôtes au système ?**  : entrer **n**
     * Pour **ID système SAP HANA** : entrer **HN1**
     * Pour **Numéro d’instance** [00] : entrer **03**
     * Pour **Groupe Worker hôte local** [par défaut] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Sélectionner Utilisation du système/Entrer l’index [4]**  : entrer **4** (pour personnalisé)
     * Pour **Emplacement des volumes de données** [/hana/data/HN1] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Emplacement des volumes de journaux** [/hana/log/HN1] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Restreindre l’allocation de mémoire maximale ?** [n] : entrer **n**
     * Pour **Nom d’hôte du certificat pour l’hôte hana-s1-db1** [hana-s1-db1] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Mot de passe (sapadm) utilisateur de l’agent hôte SAP** : entrer le mot de passe
     * Pour **Confirmer le mot de passe (sapadm) utilisateur de l’agent hôte SAP** : entrer le mot de passe
     * Pour **Mot de passe de l’administrateur système (hn1adm)**  : entrer le mot de passe
     * Pour **Répertoire de base de l’administrateur système**[/usr/sap/HN1/home] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Environnement de connexion de l’administrateur système** [/bin/sh] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **ID utilisateur de l’administrateur système** [1001] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Entrer l’ID du groupe d’utilisateurs (sapsys)** [79] : appuyrz sur Entrée pour accepter la valeur par défaut
     * Pour **Mot de passe utilisateur de la base de données système (système)**  : entrer le mot de passe du système
     * Pour **Confirmer le mot de passe utilisateur de la base de données système (système)**  : entrer le mot de passe du système
     * Pour **Redémarrer le système après le redémarrage de la machine ?** [n] : entrer **n** 
     * Pour **Voulez-vous continuer (y/n)**  : valider le résumé et si tout semble correct, entrer **y**

2. **[2]** Répétez l’étape précédente pour installer SAP HANA sur le premier nœud sur le SITE 2.   

3. **[1,2]** Vérifiez global. ini  

   Affichez global.ini et assurez-vous que la configuration de la communication interne SAP HANA entre les nœuds est en place. Vérifiez la section **communication**. Elle doit comporter un espace d’adressage pour le sous-réseau `inter` et `listeninterface` doit être défini sur `.internal`. Vérifiez la section **internal_hostname_resolution**. Elle doit comporter les adresses IP des machines virtuelles HANA qui appartiennent au sous-réseau `inter`.  

   ```
     sudo cat /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
     # Example from SITE1 
     [communication]
     internal_network = 10.23.1.128/26
     listeninterface = .internal
     [internal_hostname_resolution]
     10.23.1.138 = hana-s1-db1
     10.23.1.139 = hana-s1-db2
     10.23.1.140 = hana-s1-db3
   ```

4. **[1, 2]** Préparez `global.ini` pour l’installation dans un environnement non partagé, comme décrit dans la note SAP [2080991](https://launchpad.support.sap.com/#/notes/0002080991).  

   ```
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    [persistence]
    basepath_shared = no
   ```

4. **[1,2]** Redémarrez SAP HANA pour activer les modifications.  

   ```
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StopSystem
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StartSystem
   ```

6. **[1,2]** Vérifiez que l’interface client utilise les adresses IP du sous-réseau `client` pour les communications.  

    ```
    # Execute as hn1adm
    /usr/sap/HN1/HDB03/exe/hdbsql -u SYSTEM -p "password" -i 03 -d SYSTEMDB 'select * from SYS.M_HOST_INFORMATION'|grep net_publicname
    # Expected result - example from SITE 2
    "hana-s2-db1","net_publicname","10.23.0.14"
   ```

   Pour plus d’informations sur la vérification de la configuration, consultez la note SAP [2183363 – Configuration du réseau interne SAP HANA](https://launchpad.support.sap.com/#/notes/2183363).  

7. **[AH]** Modifiez les autorisations sur les répertoires de données et de journaux pour éviter une erreur d’installation de HANA.  

   ```
    sudo chmod o+w -R /hana/data /hana/log
   ```

8. **[1]** Installez les nœuds HANA secondaires. Les exemples d’instructions de cette étape sont pour le SITE 1.  
   a. Démarrez le programme **hdblcm** résident comme `root`.    
    ```
     cd /hana/shared/HN1/hdblcm
     ./hdblcm 
    ```

   b. À l’invite, entrez les valeurs suivantes :

     * Pour **Choisir une action** : entrer **2** (pour ajouter des hôtes)
     * Pour **Entrer les noms d’hôte séparés par une virgule à ajouter** : hana-s1-db2, hana-s1-db3
     * Pour **Composants supplémentaires pour l’installation** : entrer **2, 3**
     * Pour **Entrer le nom d’utilisateur racine [root]**  : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Sélectionner des rôles pour l’hôte 'hana-s1-db2' [1]**  : 1 (pour le Worker)
     * Pour **Entrer le groupe de basculement hôte pour l’hôte 'hana-s1-db2' [par défaut]**  : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Entrer le numéro de partition de stockage pour l’hôte 'hana-s1-db2' [<<assign automatically>>]**  : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Entrer le groupe de basculement hôte pour l’hôte 'hana-s1-db2' [par défaut]**  : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Sélectionner des rôles pour l’hôte 'hana-s1-db3' [1]**  : 1 (pour le Worker)
     * Pour **Entrer le groupe de basculement hôte pour l’hôte 'hana-s1-db3' [par défaut]**  : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Entrer le numéro de partition de stockage pour l’hôte 'hana-s1-db3' [<<assign automatically>>]**  : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Entrer le groupe de Worker hôte pour l’hôte 'hana-s1-db3' [par défaut]**  : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Mot de passe de l’administrateur système (hn1adm)**  : entrer le mot de passe
     * Pour **Entrer le mot de passe (sapadm) utilisateur de l’agent hôte SAP** : entrer le mot de passe
     * Pour **Confirmer le mot de passe (sapadm) utilisateur de l’agent hôte SAP** : entrer le mot de passe
     * Pour **Nom d’hôte du certificat pour l’hôte hana-s1-db2** [hana-s1-db2] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Nom d’hôte du certificat pour l’hôte hana-s1-db3** [hana-s1-db3] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Voulez-vous continuer (y/n)**  : valider le résumé et si tout semble correct, entrer **y**

9. **[2]** Répétez l’étape précédente pour installer les nœuds SAP HANA secondaires sur le SITE 2.   

## <a name="configure-sap-hana-20-system-replication"></a>Configurer la réplication de système SAP HANA 2.0

1. **[1]** Configurez la réplication de système sur le SITE 1 :

   Sauvegardez les bases de données en tant que **hn1** adm :

    ```
    hdbsql -d SYSTEMDB -u SYSTEM -p "passwd" -i 03 "BACKUP DATA USING FILE ('initialbackupSYS')"
    hdbsql -d HN1 -u SYSTEM -p "passwd" -i 03 "BACKUP DATA USING FILE ('initialbackupHN1')"
    ```

   Copiez les fichiers d’infrastructure à clé publique du système sur le site secondaire :

    ```
    scp /usr/sap/HN1/SYS/global/security/rsecssfs/data/SSFS_HN1.DAT hana-s2-db1:/usr/sap/HN1/SYS/global/security/rsecssfs/data/
    scp /usr/sap/HN1/SYS/global/security/rsecssfs/key/SSFS_HN1.KEY  hana-s2-db1:/usr/sap/HN1/SYS/global/security/rsecssfs/key/
    ```

   Créez le site principal :

    ```
    hdbnsutil -sr_enable --name=HANA_S1
    ```

2. **[2]** Configurez la réplication de système sur le SITE 2 :
    
   Inscrivez le second site pour démarrer la réplication de système. Exécutez la commande suivante en tant que <hanasi\>adm :

    ```
    sapcontrol -nr 03 -function StopWait 600 10
    hdbnsutil -sr_register --remoteHost=hana-s1-db1 --remoteInstance=03 --replicationMode=sync --name=HANA_S2
    sapcontrol -nr 03 -function StartSystem
    ```

3. **[1]** Vérifier l’état de la réplication

   Vérifiez l’état de la réplication et attendez que toutes les bases de données soient synchronisées.

    ```
    sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
    # | Database | Host          | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary     | Secondary | Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
    # |          |               |       |              |           |         |           | Host          | Port      | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
    # | -------- | ------------- | ----- | ------------ | --------- | ------- | --------- | ------------- | --------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
    # | HN1      | hana-s1-db3   | 30303 | indexserver  |         5 |       1 | HANA_S1   | hana-s2-db3   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | SYSTEMDB | hana-s1-db1   | 30301 | nameserver   |         1 |       1 | HANA_S1   | hana-s2-db1   |     30301 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db1   | 30307 | xsengine     |         2 |       1 | HANA_S1   | hana-s2-db1   |     30307 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db1   | 30303 | indexserver  |         3 |       1 | HANA_S1   | hana-s2-db1   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db2   | 30303 | indexserver  |         4 |       1 | HANA_S1   | hana-s2-db2   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    #
    # status system replication site "2": ACTIVE
    # overall system replication status: ACTIVE
    #
    # Local System Replication State
    #   
    # mode: PRIMARY
    # site id: 1
    # site name: HANA_S1
    ```

4. **[1,2]** Modifiez la configuration HANA afin que la communication pour la réplication de système HANA si elle est dirigée par le biais des interface réseau virtuelles de réplication de système HANA.   
   - Arrêter HANA sur les deux sites
    ```
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StopSystem HDB
    ```

   - Modifiez global. ini pour ajouter le mappage d’hôte pour la réplication de système HANA : utilisez les adresses IP du sous-réseau `hsr`.  
    ```
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    #Add the section
    [system_replication_hostname_resolution]
    10.23.1.202 = hana-s1-db1
    10.23.1.203 = hana-s1-db2
    10.23.1.204 = hana-s1-db3
    10.23.1.205 = hana-s2-db1
    10.23.1.206 = hana-s2-db2
    10.23.1.207 = hana-s2-db3
    ```

   - Démarrer HANA sur les deux sites
   ```
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StartSystem HDB
   ```

   Pour plus d’informations, consultez [Résolution de noms d’hôtes pour la réplication de système](https://help.sap.com/viewer/eb3777d5495d46c5b2fa773206bbfb46/1.0.12/en-US/c0cba1cb2ba34ec89f45b48b2157ec7b.html).  

5. **[AH]** Réactivez le pare-feu.  
   - Réactivez le pare-feu.
       ```
       # Execute as root
       systemctl start firewalld
       systemctl enable firewalld
       ```

   - Ouvrez les ports de pare-feu nécessaires. Vous devrez ajuster les ports pour votre numéro d’instance HANA.  

       > [!IMPORTANT]
       > Créez des règles de pare-feu pour autoriser le trafic client et de communication entre nœuds HANA. Les ports indispensables sont répertoriés ici : [Ports TCP/IP de tous les produits SAP](https://help.sap.com/viewer/ports). Les commandes suivantes sont simplement des exemples. Dans ce scénario avec le numéro de système 03 utilisé.

       ```
        # Execute as root
        sudo firewall-cmd --zone=public --add-port=30301/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30301/tcp
        sudo firewall-cmd --zone=public --add-port=30303/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30303/tcp
        sudo firewall-cmd --zone=public --add-port=30306/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30306/tcp
        sudo firewall-cmd --zone=public --add-port=30307/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30307/tcp
        sudo firewall-cmd --zone=public --add-port=30313/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30313/tcp
        sudo firewall-cmd --zone=public --add-port=30315/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30315/tcp
        sudo firewall-cmd --zone=public --add-port=30317/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30317/tcp
        sudo firewall-cmd --zone=public --add-port=30340/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30340/tcp
        sudo firewall-cmd --zone=public --add-port=30341/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30341/tcp
        sudo firewall-cmd --zone=public --add-port=30342/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30342/tcp
        sudo firewall-cmd --zone=public --add-port=1128/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=1128/tcp
        sudo firewall-cmd --zone=public --add-port=1129/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=1129/tcp
        sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40302/tcp
        sudo firewall-cmd --zone=public --add-port=40301/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40301/tcp
        sudo firewall-cmd --zone=public --add-port=40307/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40307/tcp
        sudo firewall-cmd --zone=public --add-port=40303/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40303/tcp
        sudo firewall-cmd --zone=public --add-port=40340/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40340/tcp
        sudo firewall-cmd --zone=public --add-port=50313/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=50313/tcp
        sudo firewall-cmd --zone=public --add-port=50314/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=50314/tcp
        sudo firewall-cmd --zone=public --add-port=30310/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30310/tcp
        sudo firewall-cmd --zone=public --add-port=30302/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30302/tcp
       ```

## <a name="create-a-pacemaker-cluster"></a>Créez un cluster Pacemaker

Suivez les étapes décrites sur la page [Configurer Pacemaker sur Red Hat Enterprise Linux dans Azure](high-availability-guide-rhel-pacemaker.md) pour créer un cluster Pacemaker de base pour ce serveur HANA.
Incluez toutes les machines virtuelles, y compris le créateur de majorité dans le cluster.  

> [!IMPORTANT]
> Ne définissez pas `quorum expected-votes` à 2, car il ne s’agit pas d’un cluster à deux nœuds.  
> Assurez-vous que la propriété de cluster `concurrent-fencing` est activée, afin que le délimiteur de nœud soit désérialisé.   

## <a name="create-file-system-resources"></a>Créer des ressources de système de fichiers

1. **[1,2]** Arrêtez SAP HANA sur les deux sites de réplication. Exécutez en tant que <sid\>adm.  

    ```
    sapcontrol -nr 03 -function StopSystem
    ```

2. **[AH]** Annulez le montage du système de fichiers `/hana/shared`, qui a été temporairement monté pour l’installation sur toutes les machines virtuelles de base de données HANA. Vous devez arrêter les processus et les sessions qui utilisent le système de fichiers avant de pouvoir annuler son montage. 
 
    ```
    umount /hana/shared 
    ```

3. **[1]** Créez les ressources de cluster du système de fichiers pour `/hana/shared` dans un état désactivé. Les ressources sont créées avec l’option `--disabled`, car vous devez définir les contraintes d’emplacement avant que les montages ne soient activés.  

    ```
    # /hana/shared file system for site 1
    pcs resource create fs_hana_shared_s1 --disabled ocf:heartbeat:Filesystem device=10.23.1.7:/HN1-shared-s1  directory=/hana/shared \
    fstype=nfs options='defaults,rw,hard,timeo=600,rsize=262144,wsize=262144,proto=tcp,intr,noatime,sec=sys,vers=4.1,lock,_netdev' op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 \
    op start interval=0 timeout=120 op stop interval=0 timeout=120

    # /hana/shared file system for site 2   
    pcs resource create fs_hana_shared_s2 --disabled ocf:heartbeat:Filesystem device=10.23.1.7:/HN1-shared-s1 directory=/hana/shared \
    fstype=nfs options='defaults,rw,hard,timeo=600,rsize=262144,wsize=262144,proto=tcp,intr,noatime,sec=sys,vers=4.1,lock,_netdev' op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 \
    op start interval=0 timeout=120 op stop interval=0 timeout=120

    # clone the /hana/shared file system resources for both site1 and site2
    pcs resource clone fs_hana_shared_s1 meta clone-node-max=1 interleave=true
    pcs resource clone fs_hana_shared_s2 meta clone-node-max=1 interleave=true
    ```
 
   L’attribut `OCF_CHECK_LEVEL=20` est ajouté à l’opération d’analyse afin que les opérations d’analyse effectuent un test en lecture/écriture sur le système de fichiers. Sans cet attribut, l’opération d’analyse vérifie uniquement que le système de fichiers est monté. Cela peut être un problème, car, en cas de perte de connectivité, le système de fichiers peut rester monté bien qu’il soit inaccessible.  

   L’attribut `on-fail=fence` est également ajouté à l’opération d’analyse. Avec cette option, si l’opération d’analyse échoue sur un nœud, ce dernier est immédiatement isolé. Sans cette option, le comportement par défaut consiste à arrêter toutes les ressources qui dépendent de la ressource défaillante, à redémarrer cette dernière, puis à démarrer toutes les ressources en question. Non seulement ce comportement peut prendre beaucoup de temps lorsqu’une ressource SAP HANA dépend de la ressource qui a échoué, mais il peut aussi échouer complètement. La ressource SAPHana ne peut pas s’arrêter correctement si le partage NFS contenant les binaires HANA est inaccessible.  

4. **[1]** Configurez et vérifiez les attributs du nœud. Tous les nœuds de base de données SAP HANA sur le site de réplication 1 reçoivent l’attribut `S1` et tous les nœuds de base de données SAP HANA sur le site de réplication 2 se voient attribuer l’attribut `S2`.  

    ```
    # HANA replication site 1
    pcs node attribute hana-s1-db1 NFS_SID_SITE=S1
    pcs node attribute hana-s1-db2 NFS_SID_SITE=S1
    pcs node attribute hana-s1-db3 NFS_SID_SITE=S1
    # HANA replication site 2
    pcs node attribute hana-s2-db1 NFS_SID_SITE=S2
    pcs node attribute hana-s2-db2 NFS_SID_SITE=S2
    pcs node attribute hana-s2-db3 NFS_SID_SITE=S2
    #To verify the attribute assignment to nodes execute
    pcs node attribute
    ```

5. **[1]** Configurez les contraintes, qui déterminent où les systèmes de fichiers NFS seront montés et activent les ressources du système de fichiers.  
    ```
    # Configure the constraints
    pcs constraint location fs_hana_shared_s1-clone rule resource-discovery=never score=-INFINITY NFS_SID_SITE ne S1
    pcs constraint location fs_hana_shared_s2-clone rule resource-discovery=never score=-INFINITY NFS_SID_SITE ne S2
    # Enable the file system resources
    pcs resource enable fs_hana_shared_s1
    pcs resource enable fs_hana_shared_s2
    ```

   Lorsque vous activez les ressources du système de fichiers, le cluster monte les systèmes de fichiers `/hana/shared`.
 
6. **[AH]** Vérifiez que les volumes ANF sont montés sous `/hana/shared` sur toutes les machines virtuelles de base de données HANA sur les deux sites.

    ```
    sudo nfsstat -m
    # Verify that flag vers is set to 4.1 
    # Example from SITE 1, hana-s1-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s1
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.11,local_lock=none,addr=10.23.1.7
    # Example from SITE 2, hana-s2-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s2
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.14,local_lock=none,addr=10.23.1.7
    ```

7. **[1]** Configurez les ressources d’attribut. Configurez les contraintes, qui définiront les attributs sur `true`, si les montages NFS pour `hana/shared` sont montés.  

    ```
    # Configure the attribure resources
    pcs resource create hana_nfs_s1_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs_s1_active
    pcs resource create hana_nfs_s2_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs_s2_active
    # Clone the attribure resources
    pcs resource clone hana_nfs_s1_active meta clone-node-max=1 interleave=true
    pcs resource clone hana_nfs_s2_active meta clone-node-max=1 interleave=true
    # Configure the constraints, which will set the attribute values
    pcs constraint order fs_hana_shared_s1-clone then hana_nfs_s1_active-clone
    pcs constraint order fs_hana_shared_s2-clone then hana_nfs_s2_active-clone
    ```

   > [!TIP]
   > Si votre configuration inclut d’autres systèmes de fichiers, en dehors de /`hana/shared`, qui sont montés NFS, incluez l’option `sequential=false` afin qu’il n’y ait aucune dépendance de classement entre les systèmes de fichiers. Tous les systèmes de fichiers montés NFS doivent démarrer, avant la ressource d’attribut correspondante, mais ils n’ont pas besoin de démarrer dans un ordre particulier les uns par rapport aux autres. Pour plus d’informations, consultez [Comment configurer HSR de montée en charge SAP HANA dans un cluster Pacemaker lorsque les systèmes de fichiers HANA sont des partages NFS](https://access.redhat.com/solutions/5423971).  

8. **[1]** Placez Pacemaker en mode maintenance, en vue de la création des ressources de cluster HANA.  
    ```
    pcs property set maintenance-mode=true
    ```

## <a name="create-sap-hana-cluster-resources"></a>Créer les ressources de cluster SAP HANA

1. **[A]** Installez l’agent de ressource de montée en charge HANA sur tous les nœuds de cluster, y compris le créateur de majorité.    

    ```
    yum install -y resource-agents-sap-hana-scaleout 
    ```


   > [!NOTE]
   > Consultez les [Stratégies de support pour les clusters à haute disponibilité RHEL - Gestion de SAP HANA dans un cluster](https://access.redhat.com/articles/3397471) pour la version minimale prise en charge de `resource-agents-sap-hana-scaleout` de package pour la version de votre système d’exploitation.  

2. **[1,2]** Installez le « hook de réplication de système » HANA. Le hook doit être installé sur un nœud de base de données HANA sur chaque site de réplication de système. SAP HANA doit être toujours en cours d’arrêt.        

   1. Préparer le hook en tant que `root` 
    ```
     mkdir -p /hana/shared/myHooks
     cp /usr/share/SAPHanaSR-ScaleOut/SAPHanaSR.py /hana/shared/myHooks
     chown -R hn1adm:sapsys /hana/shared/myHooks
    ```

   2. Ajuster `global.ini`
    ```
    # add to global.ini
    [ha_dr_provider_SAPHanaSR]
    provider = SAPHanaSR
    path = /hana/shared/myHooks
    execution_order = 1
    
    [trace]
    ha_dr_saphanasr = info
    ```

3. **[AH]** Le cluster nécessite une configuration de sudoers sur le nœud de cluster pour <sid\>adm. Dans cet exemple, il est possible de créer un nouveau fichier. Exécutez les commandes en tant que `root`.    
    ``` 
    cat << EOF > /etc/sudoers.d/20-saphana
    # SAPHanaSR-ScaleOut needs for srHook
     Cmnd_Alias SOK = /usr/sbin/crm_attribute -n hana_hn1_glob_srHook -v SOK -t crm_config -s SAPHanaSR
     Cmnd_Alias SFAIL = /usr/sbin/crm_attribute -n hana_hn1_glob_srHook -v SFAIL -t crm_config -s SAPHanaSR
     hn1adm ALL=(ALL) NOPASSWD: SOK, SFAIL
     EOF
    ```

4. **[1,2]** Démarrez SAP HANA sur les deux sites de réplication. Exécutez en tant que <sid\>adm.  

    ```
    sapcontrol -nr 03 -function StartSystem 
    ```

5. **[1]** Vérifiez l’installation de hook. Exécutez en tant que <sid\>adm sur le site de réplication de système HANA actif.   

    ```
    cdtrace
     awk '/ha_dr_SAPHanaSR.*crm_attribute/ \
     { printf "%s %s %s %s\n",$2,$3,$5,$16 }' nameserver_*

     # Example entries
     # 2020-07-21 22:04:32.364379 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:04:46.905661 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:04:52.092016 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:04:52.782774 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:04:53.117492 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:06:35.599324 ha_dr_SAPHanaSR SOK
    ```

6. **[1]** Créez les ressources de cluster HANA. Exécutez les commandes suivantes en tant que `root`.    
   1. Assurez-vous que le cluster est déjà en mode maintenance.  
    
   2. Ensuite, créez la ressource de topologie HANA.  
      Si vous créez un cluster RHEL **7.x**, utilisez les commandes suivantes :  
      ```
      pcs resource create SAPHanaTopology_HN1_HDB03 SAPHanaTopologyScaleOut \
       SID=HN1 InstanceNumber=03 \
       op start timeout=600 op stop timeout=300 op monitor interval=10 timeout=600

      pcs resource clone SAPHanaTopology_HN1_HDB03 meta clone-node-max=1 interleave=true
      ```

      Si vous créez un cluster RHEL **8.x**, utilisez les commandes suivantes :  
      ```
      pcs resource create SAPHanaTopology_HN1_HDB03 SAPHanaTopology \
       SID=HN1 InstanceNumber=03 meta clone-node-max=1 interleave=true \
       op methods interval=0s timeout=5 \
       op start timeout=600 op stop timeout=300 op monitor interval=10 timeout=600

      pcs resource clone SAPHanaTopology_HN1_HDB03 meta clone-node-max=1 interleave=true
      ```

   3. Ensuite, créez la ressource d’instance HANA.  
      > [!NOTE]
      > Cet article contient des références au terme *esclave*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.  
 
      Si vous créez un cluster RHEL **7.x**, utilisez les commandes suivantes :    
      ```
      pcs resource create SAPHana_HN1_HDB03 SAPHanaController \
       SID=HN1 InstanceNumber=03 PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
       op start interval=0 timeout=3600 op stop interval=0 timeout=3600 op promote interval=0 timeout=3600 \
       op monitor interval=60 role="Master" timeout=700 op monitor interval=61 role="Slave" timeout=700
     
      pcs resource master msl_SAPHana_HN1_HDB03 SAPHana_HN1_HDB03 \
       meta master-max="1" clone-node-max=1 interleave=true
      ```

      Si vous créez un cluster RHEL **8.x**, utilisez les commandes suivantes :  
      ```
      pcs resource create SAPHana_HN1_HDB03 SAPHanaController \
       SID=HN1 InstanceNumber=03 PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
       op demote interval=0s timeout=320 op methods interval=0s timeout=5 \
       op start interval=0 timeout=3600 op stop interval=0 timeout=3600 op promote interval=0 timeout=3600 \
       op monitor interval=60 role="Master" timeout=700 op monitor interval=61 role="Slave" timeout=700
     
      pcs resource promotable SAPHana_HN1_HDB03 \
       meta master-max="1" clone-node-max=1 interleave=true
      ```
      > [!IMPORTANT]
      > Nous vous recommandons, comme meilleure pratique, de définir AUTOMATED_REGISTER uniquement sur **non**, tout en effectuant des tests de basculement détaillés, afin d’éviter l’échec de l’inscription automatique de l’instance principale comme secondaire. Une fois les tests de basculement terminés, définissez AUTOMATED_REGISTER sur **oui**, afin qu’après la réplication du système de prise en charge, la réplication puisse reprendre automatiquement. 

   4. Créez une adresse IP virtuelle et les ressources associées.  
      ```
      pcs resource create vip_HN1_03 ocf:heartbeat:IPaddr2 ip=10.23.0.18 op monitor interval="10s" timeout="20s"
      sudo pcs resource create nc_HN1_03 azure-lb port=62503
      sudo pcs resource group add g_ip_HN1_03 nc_HN1_03 vip_HN1_03
      ```

   5. Créer les contraintes de cluster  
      Si vous créez un cluster RHEL **7.x**, utilisez les commandes suivantes :  
      ```
      #Start HANA topology, before the HANA instance
      pcs constraint order SAPHanaTopology_HN1_HDB03-clone then msl_SAPHana_HN1_HDB03

      pcs constraint colocation add g_ip_HN1_03 with master msl_SAPHana_HN1_HDB03 4000
      #HANA resources are only allowed to run on a node, if the node's NFS file systems are mounted. The constraint also avoids the majority maker node
      pcs constraint location SAPHanaTopology_HN1_HDB03-clone rule resource-discovery=never score=-INFINITY hana_nfs_s1_active ne true and hana_nfs_s2_active ne true
      ```
 
      Si vous créez un cluster RHEL **8.x**, utilisez les commandes suivantes :  
      ```
      #Start HANA topology, before the HANA instance
      pcs constraint order SAPHanaTopology_HN1_HDB03-clone then SAPHana_HN1_HDB03-clone

      pcs constraint colocation add g_ip_HN1_03 with master SAPHana_HN1_HDB03-clone 4000
      #HANA resources are only allowed to run on a node, if the node's NFS file systems are mounted. The constraint also avoids the majority maker node
      pcs constraint location SAPHanaTopology_HN1_HDB03-clone rule resource-discovery=never score=-INFINITY hana_nfs_s1_active ne true and hana_nfs_s2_active ne true
      ```

7. **[1]** Placez le cluster en mode maintenance. Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées.  
    ```
    sudo pcs property set maintenance-mode=false
    #If there are failed cluster resources, you may need to run the next command
    pcs resource cleanup
    ```
  
   > [!NOTE]
   > Les délais d’expiration de la configuration ci-dessus sont simplement des exemples et doivent être adaptés à la configuration HANA spécifique. Par exemple, vous devrez peut-être augmenter le délai de démarrage, si le démarrage de la base de données SAP HANA prend plus de temps.
  
## <a name="test-sap-hana-failover"></a>Test de basculement SAP HANA 

1. Avant de commencer un test, vérifiez l’état du cluster et de la réplication de système SAP HANA.  

   a. Vérifier qu’il n’y a pas d’actions de cluster ayant échoué  
     ```
     #Verify that there are no failed cluster actions
     pcs status
     # Example
     #Stack: corosync
     #Current DC: hana-s-mm (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
     #Last updated: Thu Sep 24 06:00:20 2020
     #Last change: Thu Sep 24 05:59:17 2020 by root via crm_attribute on hana-s1-db1
     #
     #7 nodes configured
     #45 resources configured
     #
     #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #
     #Active resources:
     #
     #rsc_st_azure    (stonith:fence_azure_arm):      Started hana-s-mm
     #Clone Set: fs_hana_shared_s1-clone [fs_hana_shared_s1]
     #    Started: [ hana--s1-db1 hana-s1-db2 hana-s1-db3 ]
     #Clone Set: fs_hana_shared_s2-clone [fs_hana_shared_s2]
     #    Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #Clone Set: hana_nfs_s1_active-clone [hana_nfs_s1_active]
     #    Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
     #Clone Set: hana_nfs_s2_active-clone [hana_nfs_s2_active]
     #    Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #Clone Set: SAPHanaTopology_HN1_HDB03-clone [SAPHanaTopology_HN1_HDB03]
     #    Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #Master/Slave Set: msl_SAPHana_HN1_HDB03 [SAPHana_HN1_HDB03]
     #    Masters: [ hana-s1-db1 ]
     #    Slaves: [ hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #Resource Group: g_ip_HN1_03
     #    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hana-s1-db1
     #    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hana-s1-db1
     ```

   b. Vérifier que la réplication de système SAP HANA est synchronisée

      ```
      # Verify HANA HSR is in sync
      sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
      #| Database | Host        | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary     | Secondary| Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
      #|          |             |       |              |           |         |           | Host          | Port     | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
      #| -------- | ----------- | ----- | ------------ | --------- | ------- | --------- | ------------- | -------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
      #| HN1      | hana-s1-db3 | 30303 | indexserver  |         5 |       2 | HANA_S1   | hana-s2-db3 |     30303  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
      #| HN1      | hana-s1-db2 | 30303 | indexserver  |         4 |       2 | HANA_S1   | hana-s2-db2 |     30303  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |  
      #| SYSTEMDB | hana-s1-db1 | 30301 | nameserver   |         1 |       2 | HANA_S1   | hana-s2-db1 |     30301  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
      #| HN1      | hana-s1-db1 | 30307 | xsengine     |         2 |       2 | HANA_S1   | hana-s2-db1 |     30307  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
      #| HN1      | hana-s1-db1 | 30303 | indexserver  |         3 |       2 | HANA_S1   | hana-s2-db1 |     30303  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |

      #status system replication site "1": ACTIVE
      #overall system replication status: ACTIVE

      #Local System Replication State
      #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

      #mode: PRIMARY
      #site id: 1
      #site name: HANA_S1
      ```

2. Vérifiez la configuration du cluster pour un scénario d’échec lorsqu’un nœud perd l’accès au partage NFS (`/hana/shared`).  

   Les agents de ressource SAP HANA dépendent de binaires, qui sont stockés sur `/hana/shared` pour effectuer des opérations pendant le basculement. Le système de fichiers `/hana/shared` est monté sur NFS dans la configuration présentée. Un test possible consiste à remonter le système de fichiers `/hana/shared` en *Lecture seule*. Cette approche valide le basculement du cluster en cas de perte de l’accès à `/hana/shared` sur le site de réplication de système actif.  

   **Résultat attendu** : Lorsque vous remontez `/hana/shared` en *Lecture seule*, l’opération d’analyse qui effectue une opération de lecture/écriture sur le système de fichiers échoue, car il n’est pas en mesure d’écrire dans le système de fichiers et déclenche un basculement de ressource HANA. Le même résultat est attendu lorsque votre nœud HANA perd l’accès au partage NFS.  
     
   Vous pouvez vérifier l’état des ressources de cluster en exécutant `crm_mon` ou `pcs status`. État des ressources avant le début du test :
      ```
      # Output of crm_mon
      #7 nodes configured
      #45 resources configured

      #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      #
      #Active resources:

      #rsc_st_azure    (stonith:fence_azure_arm):      Started hana-s-mm
      # Clone Set: fs_hana_shared_s1-clone [fs_hana_shared_s1]
      #    Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
      # Clone Set: fs_hana_shared_s2-clone [fs_hana_shared_s2]
      #     Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Clone Set: hana_nfs_s1_active-clone [hana_nfs_s1_active]
      #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
      # Clone Set: hana_nfs_s2_active-clone [hana_nfs_s2_active]
      #     Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Clone Set: SAPHanaTopology_HN1_HDB03-clone [SAPHanaTopology_HN1_HDB03]
      #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Master/Slave Set: msl_SAPHana_HN1_HDB03 [SAPHana_HN1_HDB03]
      #     Masters: [ hana-s1-db1 ]
      #     Slaves: [ hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Resource Group: g_ip_HN1_03
      #     nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hana-s1-db1
      #     vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hana-s1-db1
      ```

   Pour simuler l’échec de `/hana/shared` sur l’une des machines virtuelles du site de réplication principal, exécutez la commande suivante :
      ```
      # Execute as root 
      mount -o ro /hana/shared
      # Or if the above command returns an error
      sudo mount -o ro 10.23.1.7/HN1-shared-s1 /hana/shared
      ```
  
   La machine virtuelle HANA, qui a perdu l’accès à `/hana/shared` doit redémarrer ou s’arrêter en fonction de la configuration du cluster. Les ressources de cluster sont migrées vers l’autre site de réplication de système HANA.  
         
   Si le cluster n’a pas démarré sur la machine virtuelle, qui a été redémarrée, démarrez le cluster en exécutant : 

      ```
      # Start the cluster 
      pcs cluster start
      ```
   
   Au démarrage du cluster, le système de fichiers `/hana/shared` est automatiquement monté.     
   Si vous définissez AUTOMATED_REGISTER="false", vous devrez configurer la réplication de système SAP HANA sur le site secondaire. Dans ce cas, vous pouvez exécuter ces commandes pour reconfigurer SAP HANA comme secondaire.   

      ```
      # Execute on the secondary 
      su - hn1adm
      # Make sure HANA is not running on the secondary site. If it is started, stop HANA
      sapcontrol -nr 03 -function StopWait 600 10
      # Register the HANA secondary site
      hdbnsutil -sr_register --name=HANA_S1 --remoteHost=hana-s2-db1 --remoteInstance=03 --replicationMode=sync
      # Switch back to root and cleanup failed resources
      pcs resource cleanup SAPHana_HN1_HDB03
      ```

   État des ressources, après le test : 

      ```
      # Output of crm_mon
      #7 nodes configured
      #45 resources configured
    
      #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      
      #Active resources:
    
      #rsc_st_azure    (stonith:fence_azure_arm):      Started hana-s-mm
      # Clone Set: fs_hana_shared_s1-clone [fs_hana_shared_s1]
      #    Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
      # Clone Set: fs_hana_shared_s2-clone [fs_hana_shared_s2]
      #     Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Clone Set: hana_nfs_s1_active-clone [hana_nfs_s1_active]
      #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
      # Clone Set: hana_nfs_s2_active-clone [hana_nfs_s2_active]
      #     Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Clone Set: SAPHanaTopology_HN1_HDB03-clone [SAPHanaTopology_HN1_HDB03]
      #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Master/Slave Set: msl_SAPHana_HN1_HDB03 [SAPHana_HN1_HDB03]
      #     Masters: [ hana-s2-db1 ]
      #     Slaves: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db2 hana-s2-db3 ]
      # Resource Group: g_ip_HN1_03
      #     nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hana-s2-db1
      #     vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hana-s2-db1
      ```


Nous vous recommandons de tester minutieusement la configuration des clusters SAP HANA, en effectuant également les tests décrits dans [HA pour SAP HANA sur des machines virtuelles Azure sur RHEL](./sap-hana-high-availability-rhel.md#test-the-cluster-setup).


## <a name="next-steps"></a>Étapes suivantes

* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur des machines virtuelles Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha].