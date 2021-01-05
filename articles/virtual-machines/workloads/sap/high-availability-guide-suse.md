---
title: Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur SLES | Microsoft Docs
description: Guide de haute disponibilité pour SAP NetWeaver sur SUSE Linux Enterprise Server pour les applications SAP
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
ms.date: 10/22/2020
ms.author: radeltch
ms.openlocfilehash: ac70833d10c7cbcf5c4dc0fe709e2a6a58a30f26
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484207"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-suse-linux-enterprise-server-for-sap-applications"></a>Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

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

Cet article décrit comment déployer et configurer les machines virtuelles, installer l’infrastructure de cluster et installer un système SAP NetWeaver 7.50 à haute disponibilité.
Dans les exemples de configuration, commandes d’installation, et ainsi de suite, nous utilisons le numéro d’instance ASCS 00, le numéro d’instance ERS 02 et l’ID de système SAP NW1. Les noms des ressources (telles que les machines virtuelles et les réseaux virtuels) figurant dans l’exemple supposent que vous ayez utilisé le [modèle convergé][template-converged] avec l’ID de système SAP NW1 pour créer les ressources.

Commencez par lire les notes et publications SAP suivantes

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

## <a name="overview"></a>Vue d’ensemble

Pour obtenir une haute disponibilité, SAP NetWeaver nécessite un serveur NFS. Le serveur NFS est configuré dans un cluster distinct et peut être utilisé par plusieurs systèmes SAP.

![Vue d’ensemble de la haute disponibilité SAP NetWeaver](./media/high-availability-guide-suse/ha-suse.png)

Le serveur NFS, SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS et la base de données SAP HANA utilisent un nom d’hôte virtuel et des adresses IP virtuelles. Sur Azure, un équilibreur de charge est nécessaire pour utiliser une adresse IP virtuelle. Nous vous recommandons d’utiliser [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). La liste suivante illustre la configuration de l’équilibreur de charge des instances (A)SCS et ERS.

### <a name="ascs"></a>(A)SCS

* Configuration du frontend
  * Adresse IP : 10.0.0.7
* Port de la sonde
  * Port 620<strong>&lt;nr&gt;</strong>
* Règles d’équilibrage de charge
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
  * Adresse IP : 10.0.0.8
* Port de la sonde
  * Port 621<strong>&lt;nr&gt;</strong>
* Règles d’équilibrage de charge
  * Si vous utilisez Standard Load Balancer, sélectionnez **Ports haute disponibilité**
  * Si vous utilisez Basic Load Balancer, créez des règles d’équilibrage de charge pour les ports suivants
    * TCP 32<strong>&lt;nr&gt;</strong>
    * TCP 33<strong>&lt;nr&gt;</strong>
    * TCP 5<strong>&lt;nr&gt;</strong>13
    * TCP 5<strong>&lt;nr&gt;</strong>14
    * TCP 5<strong>&lt;nr&gt;</strong>16

* Configuration du backend
  * Connecté aux interfaces réseau principales de toutes les machines virtuelles qui doivent faire partie du cluster (A)SCS/ERS


## <a name="setting-up-a-highly-available-nfs-server"></a>Configuration d’un serveur NFS à haute disponibilité

SAP NetWeaver nécessite un stockage partagé pour le répertoire de transport et de profil. Pour savoir comment configurer un serveur NFS pour SAP NetWeaver, consultez [Haute disponibilité pour NFS sur les machines virtuelles Azure sur SUSE Linux Enterprise Server][nfs-ha].

## <a name="setting-up-ascs"></a>Configuration de (A)SCS

Vous pouvez utiliser un modèle Azure de GitHub pour déployer l’ensemble des ressources Azure, notamment les machines virtuelles, les groupes à haute disponibilité et l’équilibreur de charge ou vous pouvez déployer les ressources manuellement.

### <a name="deploy-linux-via-azure-template"></a>Déployer Linux via le modèle Azure

La Place de marché Azure contient une image de SUSE Linux Enterprise Server for SAP Applications 12 que vous pouvez utiliser pour déployer de nouvelles machines virtuelles. L’image Place de marché contient l’agent de ressource pour SAP NetWeaver.

Vous pouvez utiliser l’un des modèles de démarrage rapide disponibles sur GitHub pour déployer toutes les ressources nécessaires. Le modèle déploie les machines virtuelles, l’équilibrage de charge, le groupe à haute disponibilité, etc. Suivez ces étapes pour déployer le modèle :

1. Ouvrez le [modèle ASCS/SCS Multi SID][template-multisid-xscs] ou le [modèle convergé][template-converged] sur le portail Azure. 
   Le modèle ASCS/SCS crée uniquement les règles d’équilibrage de charge pour les instances de SAP NetWeaver ASCS/SCS et ERS (Linux uniquement), tandis que le modèle convergé crée également les règles d’équilibrage de charge pour une base de données (par exemple Microsoft SQL Server ou SAP HANA). Si vous prévoyez d’installer un système SAP NetWeaver et que vous souhaitez également installer la base de données sur les mêmes machines, utilisez le [modèle convergé][template-converged].
1. Entrez les paramètres suivants
   1. Préfixe de ressource (modèle ASCS/SCS Multi SID uniquement)  
      Entrez le préfixe à utiliser. Cette valeur sera utilisée comme préfixe pour les ressources déployées.
   3. ID du système SAP (modèle convergé uniquement)  
      Entrez l’ID du système SAP que vous souhaitez installer. Cet ID est utilisé comme préfixe pour les ressources déployées.
   4. Type de pile  
      Sélectionnez le type de pile de SAP NetWeaver
   5. Type de système d’exploitation  
      Sélectionnez l’une des distributions Linux. Dans cet exemple, sélectionnez SLES 12 BYOS
   6. Type de base de données  
      Sélectionnez HANA
   7. Taille du système SAP.  
      Nombre de SAP fournis par le nouveau système. Si vous ne savez pas combien de SAP sont exigés par le système, demandez à votre partenaire technologique SAP ou un intégrateur système
   8. Disponibilité du système  
      Sélectionnez la haute disponibilité (HA).
   9. Nom d’utilisateur et mot de passe d’administrateur  
      Un utilisateur pouvant être utilisé pour ouvrir une session sur la machine est créé.
   10. ID de sous-réseau  
   Si vous voulez déployer la machine virtuelle dans un réseau virtuel existant où vous avez défini un sous-réseau auquel la machine virtuelle doit être attribuée, nommez l’ID de ce sous-réseau spécifique. L’ID se présente généralement comme suit : /subscriptions/ **&lt;ID_abonnement&gt;** /resourceGroups/ **&lt;nom_groupe_ressources&gt;** /providers/Microsoft.Network/virtualNetworks/ **&lt;nom_réseau_virtuel&gt;** /subnets/ **&lt;nom_sous_réseau&gt;**

### <a name="deploy-linux-manually-via-azure-portal"></a>Déployer manuellement Linux via le portail Azure

Vous devez tout d’abord créer les machines virtuelles pour ce cluster NFS. Par la suite, vous créez un équilibreur de charge et utilisez les machines virtuelles dans le pool back-end.

1. Création d’un groupe de ressources
1. Création d'un réseau virtuel
1. Créer un groupe à haute disponibilité  
   Définir un domaine de mise à jour maximal
1. Créer la machine virtuelle 1  
   Utiliser au minimum SLES4SAP 12 SP1 (dans cet exemple, nous utiliserons l’image SLES4SAP 12 SP1 https://portal.azure.com/#create/SUSE.SUSELinuxEnterpriseServerforSAPApplications12SP1PremiumImage-ARM )  
   SLES For SAP Applications 12 SP1 est utilisé  
   Sélectionner le groupe à haute disponibilité créé précédemment  
1. Créer la machine virtuelle 2  
   Utiliser au minimum SLES4SAP 12 SP1 (dans cet exemple, nous utiliserons l’image SLES4SAP 12 SP1 https://portal.azure.com/#create/SUSE.SUSELinuxEnterpriseServerforSAPApplications12SP1PremiumImage-ARM )  
   SLES For SAP Applications 12 SP1 est utilisé  
   Sélectionner le groupe à haute disponibilité créé précédemment  
1. Ajouter au moins un disque de données sur les deux machines virtuelles  
   Les disques de données sont utilisés pour le répertoire /usr/sap/`<SAPSID`>
1. Créer un équilibreur de charge (interne, standard) :  
   1. Créer les adresses IP de serveurs frontaux
      1. Adresse IP 10.0.0.7 pour l’instance ASCS
         1. Ouvrir l’équilibrage de charge, sélectionner le pool d’adresses IP frontal et cliquer sur Ajouter
         1. Entrer le nom du nouveau pool d’adresses IP frontal (par exemple **nw1-ascs-frontend**)
         1. Définir l’affectation sur Statique et entrer l’adresse IP (par exemple **10.0.0.7**)
         1. Cliquez sur OK
      1. Adresse IP 10.0.0.8 pour les instances ASCS ERS
         * Répéter les étapes ci-dessus pour créer une adresse IP pour l’instance ERS (par exemple **10.0.0.8** et **nw1-aers-backend**)
   1. Créer le pool principal
      1. Ouvrir l’équilibrage de charge, sélectionner les pools principaux et cliquer sur Ajouter
      1. Entrer le nom du nouveau pool principal (par exemple **nw1-backend**)
      1. Cliquer sur Ajouter une machine virtuelle
      1. Sélectionnez Machine virtuelle.
      1. Sélectionnez les machines virtuelles du cluster (A)SCS et leurs adresses IP.
      1. Cliquez sur Ajouter.
   1. Créer les sondes d’intégrité
      1. Port 620 **00** pour l’instance ASCS
         1. Ouvrir l’équilibrage de charge, sélectionner les sondes d’intégrité et cliquer sur Ajouter
         1. Entrer le nom de la nouvelle sonde d’intégrité (par exemple **nw1-ascs-hp**)
         1. Sélectionner le protocole TCP et le port 620 **00**, et conserver un intervalle de 5 et un seuil de défaillance sur le plan de l’intégrité de 2
         1. Cliquez sur OK
      1. Port 621 **02** pour les instances ASCS ERS
         * Répéter les étapes ci-dessus pour créer une sonde d’intégrité pour l’instance ERS (par exemple **62102** et **nw1-aers-hp**)
   1. Règles d’équilibrage de charge
      1. Règles d’équilibreur de charge pour ASCS
         1. Ouvrir l’équilibreur de charge, sélectionner les règles d’équilibrage de charge et cliquer sur Ajouter
         1. Entrer le nom de la nouvelle règle d’équilibrage de charge (par exemple **nw1-lb-ascs**)
         1. Sélectionnez l’adresse IP du front-end, le pool de back-ends et la sonde d’intégrité créée précédemment (par exemple, **nw1-ascs-frontend**, **nw1-backend** et **nw1-ascs-hp**)
         1. Sélectionnez **Ports haute disponibilité**
         1. Augmenter le délai d’inactivité à 30 minutes
         1. **Veiller à activer IP flottante**
         1. Cliquez sur OK
         * Répétez les étapes ci-dessus pour créer des règles d’équilibrage de charge pour ERS (par exemple **nw1-lb-ers**)
1. Sinon, si votre scénario requiert l’équilibrage de charge de base (interne), procédez comme suit :  
   1. Créer les adresses IP de serveurs frontaux
      1. Adresse IP 10.0.0.7 pour l’instance ASCS
         1. Ouvrir l’équilibrage de charge, sélectionner le pool d’adresses IP frontal et cliquer sur Ajouter
         1. Entrer le nom du nouveau pool d’adresses IP frontal (par exemple **nw1-ascs-frontend**)
         1. Définir l’affectation sur Statique et entrer l’adresse IP (par exemple **10.0.0.7**)
         1. Cliquez sur OK
      1. Adresse IP 10.0.0.8 pour les instances ASCS ERS
         * Répétez les étapes ci-dessus pour créer une adresse IP pour l’instance ERS (par exemple, **10.0.0.8** et **nw1-aers-frontend**)
   1. Créer le pool principal
      1. Ouvrir l’équilibrage de charge, sélectionner les pools principaux et cliquer sur Ajouter
      1. Entrer le nom du nouveau pool principal (par exemple **nw1-backend**)
      1. Cliquer sur Ajouter une machine virtuelle
      1. Sélectionner le groupe à haute disponibilité créé précédemment
      1. Sélectionner les machines virtuelles du cluster (A)SCS
      1. Cliquez sur OK
   1. Créer les sondes d’intégrité
      1. Port 620 **00** pour l’instance ASCS
         1. Ouvrir l’équilibrage de charge, sélectionner les sondes d’intégrité et cliquer sur Ajouter
         1. Entrer le nom de la nouvelle sonde d’intégrité (par exemple **nw1-ascs-hp**)
         1. Sélectionner le protocole TCP et le port 620 **00**, et conserver un intervalle de 5 et un seuil de défaillance sur le plan de l’intégrité de 2
         1. Cliquez sur OK
      1. Port 621 **02** pour les instances ASCS ERS
         * Répéter les étapes ci-dessus pour créer une sonde d’intégrité pour l’instance ERS (par exemple **62102** et **nw1-aers-hp**)
   1. Règles d’équilibrage de charge
      1. TCP 32 **00** pour l’instance ASCS
         1. Ouvrir l’équilibreur de charge, sélectionner les règles d’équilibrage de charge et cliquer sur Ajouter
         1. Entrer le nom de la nouvelle règle d’équilibrage de charge (par exemple **nw1-lb-3200**)
         1. Sélectionner l’adresse IP du serveur frontal, le pool principal et la sonde d’intégrité créés précédemment (par exemple **nw1-ascs-frontend**)
         1. Conserver le protocole **TCP** et indiquer le port **3200**
         1. Augmenter le délai d’inactivité à 30 minutes
         1. **Veiller à activer IP flottante**
         1. Cliquez sur OK
      1. Ports supplémentaires pour l’instance ASCS
         * Répéter les étapes ci-dessus pour les ports 36 **00**, 39 **00**, 81 **00**, 5 **00** 13, 5 **00** 14, 5 **00** 16 et TCP pour l’instance ASCS
      1. Ports supplémentaires pour les instances ASCS ERS
         * Répéter les étapes ci-dessus pour les ports 33 **02**, 5 **02** 13, 5 **02** 14, 5 **02** 16 et TCP pour les instances ASCS ERS

> [!IMPORTANT]
> Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d’informations, consultez [Limitations d’équilibreur de charge Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte d’interface réseau.  

> [!Note]
> Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité Internet sortante, sauf si une configuration supplémentaire est effectuée pour autoriser le routage vers des points de terminaison publics. Pour savoir plus en détails comment bénéficier d’une connectivité sortante, voir [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

> [!IMPORTANT]
> N’activez pas les timestamps TCP sur des machines virtuelles Azure placées derrière Azure Load Balancer. L’activation des timestamps TCP entraîne l’échec des sondes d’intégrité. Définissez le paramètre **net.ipv4.tcp_timestamps** sur **0**. Pour plus d’informations, consultez [Load Balancer health probes](../../../load-balancer/load-balancer-custom-probe-overview.md) (Sondes d’intégrité Load Balancer).

### <a name="create-pacemaker-cluster"></a>Créer le cluster Pacemaker

Suivez les étapes décrites à la page [Configuration de Pacemaker sur SUSE Linux Enterprise Server dans Azure](high-availability-guide-suse-pacemaker.md) pour créer un cluster Pacemaker de base pour ce serveur (A)SCS.

### <a name="installation"></a>Installation

Les éléments suivants sont précédés de **[A]** (applicable à tous les nœuds), de **[1]** (applicable uniquement au nœud 1) ou de **[2]** (applicable uniquement au nœud 2).

1. **[A]** Installer le connecteur SUSE

   <pre><code>sudo zypper install sap-suse-cluster-connector
   </code></pre>

   > [!NOTE]
   > Le problème connu lié à l’utilisation d’un tiret dans les noms d’hôtes est résolu avec la version **3.1.1** du package **sap-suse-cluster-connector**. Veillez à utiliser au moins la version 3.1.1 du package sap-suse-cluster-connector si vous utilisez des nœuds de cluster avec un tiret dans le nom d’hôte. Dans le cas contraire, le cluster ne fonctionnera pas. 

   Vérifiez que vous avez installé la nouvelle version du connecteur de cluster SUSE SAP. L’ancienne version s’appelle sap_suse_cluster_connector et la nouvelle **sap-suse-cluster-connector**.

   ```
   sudo zypper info sap-suse-cluster-connector
   
   Information for package sap-suse-cluster-connector:
   ---------------------------------------------------
   Repository     : SLE-12-SP3-SAP-Updates
   Name           : sap-suse-cluster-connector
   <b>Version        : 3.0.0-2.2</b>
   Arch           : noarch
   Vendor         : SUSE LLC <https://www.suse.com/>
   Support Level  : Level 3
   Installed Size : 41.6 KiB
   <b>Installed      : Yes</b>
   Status         : up-to-date
   Source package : sap-suse-cluster-connector-3.0.0-2.2.src
   Summary        : SUSE High Availability Setup for SAP Products
   ```

1. **[A]** Mettre à jour les agents de ressources SAP  
   
   Un correctif pour les agents de ressources est nécessaire pour utiliser la nouvelle configuration décrite dans cet article. Vous pouvez vérifier si le correctif est déjà installé avec la commande suivante.

   <pre><code>sudo grep 'parameter name="IS_ERS"' /usr/lib/ocf/resource.d/heartbeat/SAPInstance
   </code></pre>

   La sortie doit ressembler à ce qui suit :

   <pre><code>&lt;parameter name="IS_ERS" unique="0" required="0"&gt;
   </code></pre>

   Si la commande grep ne trouve pas le paramètre IS_ERS, vous devez installer le correctif logiciel répertorié dans la [page de téléchargement SUSE](https://download.suse.com/patch/finder/#bu=suse&familyId=&productId=&dateRange=&startDate=&endDate=&priority=&architecture=&keywords=resource-agents).

   <pre><code># example for patch for SLES 12 SP1
   sudo zypper in -t patch SUSE-SLE-HA-12-SP1-2017-885=1
   # example for patch for SLES 12 SP2
   sudo zypper in -t patch SUSE-SLE-HA-12-SP2-2017-886=1
   </code></pre>

1. **[A]** Configurer la résolution de nom d’hôte

   Vous pouvez utiliser un serveur DNS ou modifier le fichier /etc/hosts sur tous les nœuds. Cet exemple montre comment utiliser le fichier /etc/hosts.
   Remplacez l’adresse IP et le nom d’hôte dans les commandes suivantes

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Insérez les lignes suivantes dans le fichier /etc/hosts. Modifiez l’adresse IP et le nom d’hôte en fonction de votre environnement   

   <pre><code># IP address of the load balancer frontend configuration for NFS
   <b>10.0.0.4 nw1-nfs</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS
   <b>10.0.0.7 nw1-ascs</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS ERS
   <b>10.0.0.8 nw1-aers</b>
   # IP address of the load balancer frontend configuration for database
   <b>10.0.0.13 nw1-db</b>
   </code></pre>

## <a name="prepare-for-sap-netweaver-installation"></a>Préparer l’installation de SAP NetWeaver

1. **[A]** Créer les répertoires partagés

   <pre><code>sudo mkdir -p /sapmnt/<b>NW1</b>
   sudo mkdir -p /usr/sap/trans
   sudo mkdir -p /usr/sap/<b>NW1</b>/SYS
   sudo mkdir -p /usr/sap/<b>NW1</b>/ASCS<b>00</b>
   sudo mkdir -p /usr/sap/<b>NW1</b>/ERS<b>02</b>
   
   sudo chattr +i /sapmnt/<b>NW1</b>
   sudo chattr +i /usr/sap/trans
   sudo chattr +i /usr/sap/<b>NW1</b>/SYS
   sudo chattr +i /usr/sap/<b>NW1</b>/ASCS<b>00</b>
   sudo chattr +i /usr/sap/<b>NW1</b>/ERS<b>02</b>
   </code></pre>

1. **[A]** Configurer autofs

   <pre><code>sudo vi /etc/auto.master
   
   # Add the following line to the file, save and exit
   +auto.master
   /- /etc/auto.direct
   </code></pre>

   Créer un fichier avec

   <pre><code>sudo vi /etc/auto.direct
   
   # Add the following lines to the file, save and exit
   /sapmnt/<b>NW1</b> -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/sapmntsid
   /usr/sap/trans -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/trans
   /usr/sap/<b>NW1</b>/SYS -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/sidsys
   </code></pre>

   Redémarrer autofs pour monter les nouveaux partages

   <pre><code>sudo systemctl enable autofs
   sudo service autofs restart
   </code></pre>

1. **[A]** Configurer le fichier SWAP

   <pre><code>sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=<b>y</b>
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=<b>2000</b>
   </code></pre>

   Redémarrer l’Agent pour activer la modification

   <pre><code>sudo service waagent restart
   </code></pre>


### <a name="installing-sap-netweaver-ascsers"></a>Installation de SAP NetWeaver ASC/ERS

1. **[1]** Créer une ressource IP virtuelle et la sonde d’intégrité pour l’instance ASCS

   > [!IMPORTANT]
   > Des tests récents ont révélé des cas où netcat cessait de répondre aux demandes en raison du backlog et de sa capacité à ne gérer qu’une seule connexion. La ressource netcat cesse d’écouter les demandes d’Azure Load Balancer et l’adresse IP flottante devient indisponible.  
   > Pour les clusters Pacemaker existants, nous vous recommandons de remplacer netcat par socat. Actuellement, nous vous recommandons d'utiliser l'agent de ressources azure-lb, qui fait partie du package resource-agents, avec la configuration requise suivante pour la version du package :
   > - Pour SLES 12 SP4/SP5, la version minimum est resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Pour SLES 15/15 SP1, la version minimum est resource-agents-4.3.0184.6ee15eb2-4.13.1.  
   >
   > Notez que la modification nécessitera un bref temps d’arrêt.  
   > Pour les clusters Pacemaker existants, si la configuration a déjà été modifiée pour utiliser socat comme décrit à la page [Azure Load-Balancer Detection Hardening](https://www.suse.com/support/kb/doc/?id=7024128), il n’est pas nécessaire de passer immédiatement à l’agent de ressources azure-lb.

   <pre><code>sudo crm node standby <b>nw1-cl-1</b>
   
   sudo crm configure primitive fs_<b>NW1</b>_ASCS Filesystem device='<b>nw1-nfs</b>:/<b>NW1</b>/ASCS' directory='/usr/sap/<b>NW1</b>/ASCS<b>00</b>' fstype='nfs4' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   sudo crm configure primitive vip_<b>NW1</b>_ASCS IPaddr2 \
     params ip=<b>10.0.0.7</b> cidr_netmask=<b>24</b> \
     op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW1</b>_ASCS azure-lb port=620<b>00</b>
   
   sudo crm configure group g-<b>NW1</b>_ASCS fs_<b>NW1</b>_ASCS nc_<b>NW1</b>_ASCS vip_<b>NW1</b>_ASCS \
      meta resource-stickiness=3000
   </code></pre>

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.

   <pre><code>sudo crm_mon -r
   
   # Node nw1-cl-1: standby
   # <b>Online: [ nw1-cl-0 ]</b>
   # 
   # Full list of resources:
   # 
   # stonith-sbd     (stonith:external/sbd): <b>Started nw1-cl-0</b>
   #  Resource Group: g-NW1_ASCS
   #      fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-0</b>
   #      nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-0</b>
   #      vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-0</b>
   </code></pre>

1. **[1]** Installer SAP NetWeaver ASCS  

   Installez SAP NetWeaver ASCS comme racine sur le premier nœud à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration frontale de l’équilibreur de charge pour l’instance ASCS, par exemple <b>nw1-ascs</b> et <b>10.0.0.7</b>, et du numéro d’instance utilisé pour la sonde de l’équilibreur de charge, par exemple <b>00</b>.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

   Si aucun sous-dossier n’est créé dans /usr/sap/**NW1**/ASCS **00** lors de l’installation, essayez de définir le propriétaire et le groupe du dossier ASCS **00**, puis réessayez.

   <pre><code>chown nw1adm /usr/sap/<b>NW1</b>/ASCS<b>00</b>
   chgrp sapsys /usr/sap/<b>NW1</b>/ASCS<b>00</b>
   </code></pre>

1. **[1]** Créer une ressource IP virtuelle et la sonde d’intégrité pour l’instance ERS

   <pre><code>sudo crm node online <b>nw1-cl-1</b>
   sudo crm node standby <b>nw1-cl-0</b>
   
   sudo crm configure primitive fs_<b>NW1</b>_ERS Filesystem device='<b>nw1-nfs</b>:/<b>NW1</b>/ASCSERS' directory='/usr/sap/<b>NW1</b>/ERS<b>02</b>' fstype='nfs4' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   sudo crm configure primitive vip_<b>NW1</b>_ERS IPaddr2 \
     params ip=<b>10.0.0.8</b> cidr_netmask=<b>24</b> \
     op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW1</b>_ERS azure-lb port=621<b>02</b>
   
   sudo crm configure group g-<b>NW1</b>_ERS fs_<b>NW1</b>_ERS nc_<b>NW1</b>_ERS vip_<b>NW1</b>_ERS
   </code></pre>

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.

   <pre><code>sudo crm_mon -r
   
   # Node <b>nw1-cl-0: standby</b>
   # <b>Online: [ nw1-cl-1 ]</b>
   # 
   # Full list of resources:
   #
   # stonith-sbd     (stonith:external/sbd): <b>Started nw1-cl-1</b>
   #  Resource Group: g-NW1_ASCS
   #      fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-1</b>
   #      nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-1</b>
   #      vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-1</b>
   #  Resource Group: g-NW1_ERS
   #      fs_NW1_ERS (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-1</b>
   #      nc_NW1_ERS (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-1</b>
   #      vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-1</b>
   </code></pre>

1. **[2]** Installer SAP NetWeaver ERS

   Installez SAP NetWeaver ERS comme racine sur le deuxième nœud à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration frontale de l’équilibreur de charge pour l’instance ERS, par exemple <b>nw1-aers</b> et <b>10.0.0.8</b>, et du numéro d’instance utilisé pour la sonde de l’équilibreur de charge, par exemple <b>02</b>.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

   > [!NOTE]
   > Utilisez SWPM SP 20 PL 05 ou ultérieur. Les versions antérieures ne définissent pas les autorisations correctement et l’installation échouera.

   Si aucun sous-dossier n’est créé dans /usr/sap/**NW1**/ERS **02** lors de l’installation, essayez de définir le propriétaire et le groupe du dossier ERS **02**, puis réessayez.

   <pre><code>chown nw1adm /usr/sap/<b>NW1</b>/ERS<b>02</b>
   chgrp sapsys /usr/sap/<b>NW1</b>/ERS<b>02</b>
   </code></pre>


1. **[1]** Adapter les profils d’instance ASCS/SCS et ERS
 
   * Profil ASCS/SCS

   <pre><code>sudo vi /sapmnt/<b>NW1</b>/profile/<b>NW1</b>_<b>ASCS00</b>_<b>nw1-ascs</b>
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # Add the keep alive parameter, if using ENSA1
   enque/encni/set_so_keepalive = true
   </code></pre>

   Pour ENSA1 et ENSA2, assurez-vous que les `keepalive`Paramètres de système d’exploitation sont définis comme décrit dans la note SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).    

   * Profil ERS

   <pre><code>sudo vi /sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b>
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # remove Autostart from ERS profile
   # Autostart = 1
   </code></pre>

1. **[A]** Configurer Keep Alive

   La communication entre le serveur d’applications SAP NetWeaver et l’ASCS/SCS est routée par l’intermédiaire d’un équilibreur de charge logiciel. L’équilibreur de charge déconnecte les connexions inactives après un délai configurable. Pour éviter ce problème, vous devez définir un paramètre dans le profil SAP NetWeaver ASCS/SCS, si vous utilisez ENSA1, et modifier les paramètres `keepalive` du système Linux sur tous les serveurs SAP pour ENSA1/ENSA2. Pour plus d’informations, consultez la [Note SAP 1410736][1410736].

   <pre><code># Change the Linux system configuration
   sudo sysctl net.ipv4.tcp_keepalive_time=300
   </code></pre>

1. **[A]** Configurer les utilisateurs SAP après l’installation

   <pre><code># Add sidadm to the haclient group
   sudo usermod -aG haclient <b>nw1</b>adm
   </code></pre>

1. **[1]** Ajouter les services ASCS et ERS SAP au fichier sapservice

   Ajoutez l’entrée de service ASCS au deuxième nœud et copiez l’entrée de service ERS dans le premier nœud.

   <pre><code>cat /usr/sap/sapservices | grep ASCS<b>00</b> | sudo ssh <b>nw1-cl-1</b> "cat >>/usr/sap/sapservices"
   sudo ssh <b>nw1-cl-1</b> "cat /usr/sap/sapservices" | grep ERS<b>02</b> | sudo tee -a /usr/sap/sapservices
   </code></pre>

1. **[1]** Créer les ressources de cluster SAP

Si vous utilisez l’architecture de serveur de file d’attente 1 (ENSA1), définissez les ressources comme suit :

   <pre><code>sudo crm configure property maintenance-mode="true"
   
   sudo crm configure primitive rsc_sap_<b>NW1</b>_ASCS<b>00</b> SAPInstance \
    operations \$id=rsc_sap_<b>NW1</b>_ASCS<b>00</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>NW1</b>_ASCS<b>00</b>_<b>nw1-ascs</b> START_PROFILE="/sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ASCS<b>00</b>_<b>nw1-ascs</b>" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
   
   sudo crm configure primitive rsc_sap_<b>NW1</b>_ERS<b>02</b> SAPInstance \
    operations \$id=rsc_sap_<b>NW1</b>_ERS<b>02</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b> START_PROFILE="/sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b>" AUTOMATIC_RECOVER=false IS_ERS=true \
    meta priority=1000
   
   sudo crm configure modgroup g-<b>NW1</b>_ASCS add rsc_sap_<b>NW1</b>_ASCS<b>00</b>
   sudo crm configure modgroup g-<b>NW1</b>_ERS add rsc_sap_<b>NW1</b>_ERS<b>02</b>
   
   sudo crm configure colocation col_sap_<b>NW1</b>_no_both -5000: g-<b>NW1</b>_ERS g-<b>NW1</b>_ASCS
   sudo crm configure location loc_sap_<b>NW1</b>_failover_to_ers rsc_sap_<b>NW1</b>_ASCS<b>00</b> rule 2000: runs_ers_<b>NW1</b> eq 1
   sudo crm configure order ord_sap_<b>NW1</b>_first_start_ascs Optional: rsc_sap_<b>NW1</b>_ASCS<b>00</b>:start rsc_sap_<b>NW1</b>_ERS<b>02</b>:stop symmetrical=false
   
   sudo crm node online <b>nw1-cl-0</b>
   sudo crm configure property maintenance-mode="false"
   </code></pre>

  SAP a introduit la prise en charge du serveur de file d’attente 2, y compris la réplication, avec SAP NW 7.52. À partir de la plateforme ABAP 1809, le serveur de file d’attente 2 est installé par défaut. Consultez la note SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) pour plus d’informations sur la prise en charge du serveur de file d’attente 2.
Si vous utilisez l’architecture de serveur de file d’attente 2 ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), définissez les ressources comme suit :

<pre><code>sudo crm configure property maintenance-mode="true"
   
   sudo crm configure primitive rsc_sap_<b>NW1</b>_ASCS<b>00</b> SAPInstance \
    operations \$id=rsc_sap_<b>NW1</b>_ASCS<b>00</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>NW1</b>_ASCS<b>00</b>_<b>nw1-ascs</b> START_PROFILE="/sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ASCS<b>00</b>_<b>nw1-ascs</b>" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000
   
   sudo crm configure primitive rsc_sap_<b>NW1</b>_ERS<b>02</b> SAPInstance \
    operations \$id=rsc_sap_<b>NW1</b>_ERS<b>02</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b> START_PROFILE="/sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b>" AUTOMATIC_RECOVER=false IS_ERS=true 
   
   sudo crm configure modgroup g-<b>NW1</b>_ASCS add rsc_sap_<b>NW1</b>_ASCS<b>00</b>
   sudo crm configure modgroup g-<b>NW1</b>_ERS add rsc_sap_<b>NW1</b>_ERS<b>02</b>
   
   sudo crm configure colocation col_sap_<b>NW1</b>_no_both -5000: g-<b>NW1</b>_ERS g-<b>NW1</b>_ASCS
   sudo crm configure order ord_sap_<b>NW1</b>_first_start_ascs Optional: rsc_sap_<b>NW1</b>_ASCS<b>00</b>:start rsc_sap_<b>NW1</b>_ERS<b>02</b>:stop symmetrical=false
   
   sudo crm node online <b>nw1-cl-0</b>
   sudo crm configure property maintenance-mode="false"
   </code></pre>

  Si vous effectuez une mise à niveau à partir d’une version antérieure et que vous passez au serveur de file d’attente 2, consultez la note SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.


   <pre><code>sudo crm_mon -r
   
   # Online: <b>[ nw1-cl-0 nw1-cl-1 ]</b>
   #
   # Full list of resources:
   #
   # stonith-sbd     (stonith:external/sbd): <b>Started nw1-cl-1</b>
   #  Resource Group: g-NW1_ASCS
   #      fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-1</b>
   #      nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-1</b>
   #      vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-1</b>
   #      rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   <b>Started nw1-cl-1</b>
   #  Resource Group: g-NW1_ERS
   #      fs_NW1_ERS (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-0</b>
   #      nc_NW1_ERS (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-0</b>
   #      vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-0</b>
   #      rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   <b>Started nw1-cl-0</b>
   </code></pre>

## <a name="sap-netweaver-application-server-preparation"></a><a name="2d6008b0-685d-426c-b59e-6cd281fd45d7"></a>Préparation du serveur d’applications SAP NetWeaver

Certaines bases de données exigent que l’installation de l’instance de base de données soit exécutée sur un serveur d’applications. Préparez les machines virtuelles de serveur d’applications pour pouvoir les utiliser dans ce cas de figure.

Les étapes ci-dessous partent du principe que vous installez le serveur d’applications sur un serveur différent des serveurs ASCS/SCS et HANA. Dans le cas contraire, certaines des étapes ci-dessous (par exemple la configuration de la résolution de nom d’hôte) ne sont pas nécessaires.

1. Configurer le système d’exploitation

   Réduisez la taille du cache d’intégrité. Pour plus d’informations, consultez [Faibles performances en écriture sur les serveurs SLES 11/12 avec une grande quantité de RAM](https://www.suse.com/support/kb/doc/?id=7010287).

   <pre><code>sudo vi /etc/sysctl.conf

   # Change/set the following settings
   vm.dirty_bytes = 629145600
   vm.dirty_background_bytes = 314572800
   </code></pre>

1. Configurer la résolution de nom d’hôte

   Vous pouvez utiliser un serveur DNS ou modifier le fichier /etc/hosts sur tous les nœuds. Cet exemple montre comment utiliser le fichier /etc/hosts.
   Remplacez l’adresse IP et le nom d’hôte dans les commandes suivantes

   ```bash
   sudo vi /etc/hosts
   ```

   Insérez les lignes suivantes dans le fichier /etc/hosts. Modifiez l’adresse IP et le nom d’hôte en fonction de votre environnement

   <pre><code># IP address of the load balancer frontend configuration for NFS
   <b>10.0.0.4 nw1-nfs</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS/SCS
   <b>10.0.0.7 nw1-ascs</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ERS
   <b>10.0.0.8 nw1-aers</b>
   # IP address of the load balancer frontend configuration for database
   <b>10.0.0.13 nw1-db</b>
   # IP address of all application servers
   <b>10.0.0.20 nw1-di-0</b>
   <b>10.0.0.21 nw1-di-1</b>
   </code></pre>

1. Créer le répertoire sapmnt

   <pre><code>sudo mkdir -p /sapmnt/<b>NW1</b>
   sudo mkdir -p /usr/sap/trans

   sudo chattr +i /sapmnt/<b>NW1</b>
   sudo chattr +i /usr/sap/trans
   </code></pre>

1. Configurer autofs

   <pre><code>sudo vi /etc/auto.master
   
   # Add the following line to the file, save and exit
   +auto.master
   /- /etc/auto.direct
   </code></pre>

   Créer un fichier avec

   <pre><code>sudo vi /etc/auto.direct
   
   # Add the following lines to the file, save and exit
   /sapmnt/<b>NW1</b> -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/sapmntsid
   /usr/sap/trans -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/trans
   </code></pre>

   Redémarrer autofs pour monter les nouveaux partages

   <pre><code>sudo systemctl enable autofs
   sudo service autofs restart
   </code></pre>

1. Configurer le fichier SWAP

   <pre><code>sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=<b>y</b>
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=<b>2000</b>
   </code></pre>

   Redémarrer l’Agent pour activer la modification

   <pre><code>sudo service waagent restart
   </code></pre>

## <a name="install-database"></a>Installer la base de données

Dans cet exemple, SAP NetWeaver est installé sur SAP HANA. Vous pouvez utiliser n’importe quelle base de données prise en charge pour cette installation. Pour plus d’informations sur l’installation de SAP HANA dans Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha]. Pour obtenir la liste des bases de données prises en charge, consultez la [note SAP 1928533][1928533].

1. Exécuter l’installation de l’instance de base de données SAP

   Installez l’instance de base de données SAP NetWeaver en tant que racine à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration frontend d’équilibreur de charge pour la base de données, par exemple <b>nw1-db</b> et <b>10.0.0.13</b>.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

## <a name="sap-netweaver-application-server-installation"></a>Installation de serveur d’applications SAP NetWeaver

Suivez ces étapes pour installer un serveur d’applications SAP.

1. Préparer le serveur d’applications

   Pour préparer le serveur d’applications, suivez la procédure décrite dans la section [Préparation du serveur d’applications SAP NetWeaver](high-availability-guide-suse.md#2d6008b0-685d-426c-b59e-6cd281fd45d7) ci-dessus.

1. Installer le serveur d’applications SAP NetWeaver

   Installer un serveur d’applications SAP NetWeaver principal ou supplémentaire.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

1. Mettre à jour la banque d’informations sécurisée SAP HANA

   Mettez à jour la banque d’informations sécurisée SAP HANA pour qu’elle pointe vers le nom virtuel de la configuration de la réplication système SAP HANA.

   Exécutez la commande suivante pour répertorier les entrées
   <pre><code>hdbuserstore List
   </code></pre>

   La sortie doit répertorier toutes les entrées et ressembler à ce qui suit
   <pre><code>DATA FILE       : /home/nw1adm/.hdb/nw1-di-0/SSFS_HDB.DAT
   KEY FILE        : /home/nw1adm/.hdb/nw1-di-0/SSFS_HDB.KEY
   
   KEY DEFAULT
     ENV : 10.0.0.14:<b>30313</b>
     USER: <b>SAPABAP1</b>
     DATABASE: <b>HN1</b>
   </code></pre>

   La sortie indique que l’adresse IP de l’entrée par défaut pointe vers la machine virtuelle et non vers l’adresse IP de l’équilibreur de charge. Cette entrée doit être modifiée pour pointer vers le nom d’hôte virtuel de l’équilibreur de charge. Assurez-vous d’utiliser le même port (**30313** dans la sortie ci-dessus) et le même nom de base de données (**HN1** dans la sortie ci-dessus) !

   <pre><code>su - <b>nw1</b>adm
   hdbuserstore SET DEFAULT <b>nw1-db:30313@HN1</b> <b>SAPABAP1</b> <b>&lt;password of ABAP schema&gt;</b>
   </code></pre>

## <a name="test-the-cluster-setup"></a>Tester la configuration du cluster

Les tests suivants sont une copie des cas de test dans les guides des meilleures pratiques de SUSE. Ils sont copiés pour des raisons pratiques. Aussi, lisez toujours les guides des meilleures pratiques et effectuez tous les tests supplémentaires qui pourraient y être ajoutés.

1. Testez HAGetFailoverConfig, HACheckConfig et HACheckFailoverConfig

   Exécutez les commandes suivantes en tant que \<sapsid>adm sur le nœud où l’instance ASCS est actuellement en cours d’exécution. Si les commandes échouent avec ÉCHEC : Mémoire insuffisante apparait, cela peut être dû à des tirets dans votre nom d’hôte. Ceci est un problème connu et il sera corrigé par SUSE dans le package sap-suse-cluster-connector.

   <pre><code>nw1-cl-0:nw1adm 54> sapcontrol -nr <b>00</b> -function HAGetFailoverConfig
   
   # 15.08.2018 13:50:36
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: Toolchain Module
   # HASAPInterfaceVersion: Toolchain Module (sap_suse_cluster_connector 3.0.1)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode:
   # HANodes: nw1-cl-0, nw1-cl-1
   
   nw1-cl-0:nw1adm 55> sapcontrol -nr 00 -function HACheckConfig
   
   # 15.08.2018 14:00:04
   # HACheckConfig
   # OK
   # state, category, description, comment
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP instance configuration, 2 ABAP instances detected
   # SUCCESS, SAP CONFIGURATION, Redundant Java instance configuration, 0 Java instances detected
   # SUCCESS, SAP CONFIGURATION, Enqueue separation, All Enqueue server separated from application server
   # SUCCESS, SAP CONFIGURATION, MessageServer separation, All MessageServer separated from application server
   # SUCCESS, SAP CONFIGURATION, ABAP instances on multiple hosts, ABAP instances on multiple hosts detected
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP SPOOL service configuration, 2 ABAP instances with SPOOL service detected
   # SUCCESS, SAP STATE, Redundant ABAP SPOOL service state, 2 ABAP instances with active SPOOL service detected
   # SUCCESS, SAP STATE, ABAP instances with ABAP SPOOL service on multiple hosts, ABAP instances with active ABAP SPOOL service on multiple hosts detected
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP BATCH service configuration, 2 ABAP instances with BATCH service detected
   # SUCCESS, SAP STATE, Redundant ABAP BATCH service state, 2 ABAP instances with active BATCH service detected
   # SUCCESS, SAP STATE, ABAP instances with ABAP BATCH service on multiple hosts, ABAP instances with active ABAP BATCH service on multiple hosts detected
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP DIALOG service configuration, 2 ABAP instances with DIALOG service detected
   # SUCCESS, SAP STATE, Redundant ABAP DIALOG service state, 2 ABAP instances with active DIALOG service detected
   # SUCCESS, SAP STATE, ABAP instances with ABAP DIALOG service on multiple hosts, ABAP instances with active ABAP DIALOG service on multiple hosts detected
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP UPDATE service configuration, 2 ABAP instances with UPDATE service detected
   # SUCCESS, SAP STATE, Redundant ABAP UPDATE service state, 2 ABAP instances with active UPDATE service detected
   # SUCCESS, SAP STATE, ABAP instances with ABAP UPDATE service on multiple hosts, ABAP instances with active ABAP UPDATE service on multiple hosts detected
   # SUCCESS, SAP STATE, SCS instance running, SCS instance status ok
   # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version (nw1-ascs_NW1_00), SAPInstance includes is-ers patch
   # SUCCESS, SAP CONFIGURATION, Enqueue replication (nw1-ascs_NW1_00), Enqueue replication enabled
   # SUCCESS, SAP STATE, Enqueue replication state (nw1-ascs_NW1_00), Enqueue replication active
   
   nw1-cl-0:nw1adm 56> sapcontrol -nr 00 -function HACheckFailoverConfig
   
   # 15.08.2018 14:04:08
   # HACheckFailoverConfig
   # OK
   # state, category, description, comment
   # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch
   </code></pre>

1. Migrer manuellement l’instance ASCS

   État des ressources avant le début du test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

   Exécutez les commandes suivantes en tant que racine pour migrer l’instance ASCS.

   <pre><code>nw1-cl-0:~ # crm resource migrate rsc_sap_NW1_ASCS00 force
   # INFO: Move constraint created for rsc_sap_NW1_ASCS00
   
   nw1-cl-0:~ # crm resource unmigrate rsc_sap_NW1_ASCS00
   # INFO: Removed migration constraints for rsc_sap_NW1_ASCS00
   
   # Remove failed actions for the ERS that occurred as part of the migration
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   État des ressources après le test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Tester HAFailoverToNode

   État des ressources avant le début du test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Exécutez les commandes suivantes en tant que \<sapsid>adm pour migrer l’instance ASCS.

   <pre><code>nw1-cl-0:nw1adm 55> sapcontrol -nr 00 -host nw1-ascs -user nw1adm &lt;password&gt; -function HAFailoverToNode ""
   
   # run as root
   # Remove failed actions for the ERS that occurred as part of the migration
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   # Remove migration constraints
   nw1-cl-0:~ # crm resource clear rsc_sap_NW1_ASCS00
   #INFO: Removed migration constraints for rsc_sap_NW1_ASCS00
   </code></pre>

   État des ressources après le test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

1. Simuler l’incident de nœud

   État des ressources avant le début du test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

   Exécutez la commande suivante en tant que racine sur le nœud où l’instance ASCS est en cours d’exécution

   <pre><code>nw1-cl-0:~ # echo b > /proc/sysrq-trigger
   </code></pre>

   Si vous utilisez SBD, Pacemaker ne doit pas démarrer automatiquement sur le nœud tué. L’état après le redémarrage du nœud devrait ressembler à ce qui suit.

   <pre><code>Online: [ nw1-cl-1 ]
   OFFLINE: [ nw1-cl-0 ]
   
   Full list of resources:
   
   stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   
   Failed Actions:
   * rsc_sap_NW1_ERS02_monitor_11000 on nw1-cl-1 'not running' (7): call=219, status=complete, exitreason='none',
       last-rc-change='Wed Aug 15 14:38:38 2018', queued=0ms, exec=0ms
   </code></pre>

   Exécutez les commandes suivantes pour démarrer Pacemaker sur le nœud tué, nettoyer les messages SBD et nettoyer les ressources ayant échoué.

   <pre><code># run as root
   # list the SBD device(s)
   nw1-cl-0:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   nw1-cl-0:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message nw1-cl-0 clear
   
   nw1-cl-0:~ # systemctl start pacemaker
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ASCS00
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   État des ressources après le test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Tester le redémarrage manuel de l’instance ASCS

   État des ressources avant le début du test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Créer un verrou d’empilement, par exemple, en modifiant un utilisateur dans la transaction su01. Exécutez les commandes suivantes en tant que \<sapsid>adm sur le nœud où l’instance ASCS est en cours d’exécution. Les commandes arrêteront l’instance ASCS et la redémarreront. Si vous utilisez l’architecture de serveur de file d’attente 1, le verrou d’empilement est censé être perdu dans ce test. Si vous utilisez l’architecture de serveur de file d’attente 2, la mise en file d’attente est conservée. 

   <pre><code>nw1-cl-1:nw1adm 54> sapcontrol -nr 00 -function StopWait 600 2
   </code></pre>

   L’instance ASCS doit maintenant être désactivée dans Pacemaker

   <pre><code>rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Stopped (disabled)
   </code></pre>

   Redémarrez l’instance ASCS sur le même nœud.

   <pre><code>nw1-cl-1:nw1adm 54> sapcontrol -nr 00 -function StartWait 600 2
   </code></pre>

   Le verrou d’empilement de la transaction su01 ne sera pas perdu et le serveur principal devrait être réinitialisé. État des ressources après le test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Tuer le processus de serveur de message

   État des ressources avant le début du test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Exécutez les commandes suivantes en tant que racine pour identifier le processus du serveur de message et le tuer.

   <pre><code>nw1-cl-1:~ # pgrep ms.sapNW1 | xargs kill -9
   </code></pre>

   Si vous ne tuez le serveur de messages qu’une seule fois, il sera redémarré par sapstart. Si vous le tuez suffisamment souvent, Pacemaker finira par déplacer l’instance ASCS vers l’autre nœud. Exécutez les commandes suivantes en tant que racine pour nettoyer l’état de la ressource de l’instance ASCS et ERS après le test.

   <pre><code>nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ASCS00
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   État des ressources après le test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

1. Tuer le processus de serveur d’empilement

   État des ressources avant le début du test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

   Exécutez la commande suivante en tant que racine sur le nœud où l’instance ASCS est en cours d’exécution pour tuer le serveur d’empilement.

   <pre><code>nw1-cl-0:~ # pgrep en.sapNW1 | xargs kill -9
   </code></pre>

   L’instance ASCS devrait immédiatement basculer vers l’autre nœud. L’instance ERS devrait également basculer après le démarrage de l’instance ASCS. Exécutez les commandes suivantes en tant que racine pour nettoyer l’état de la ressource de l’instance ASCS et ERS après le test.

   <pre><code>nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ASCS00
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   État des ressources après le test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Tuer le processus de serveur de réplication de l’empilement

   État des ressources avant le début du test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Exécutez la commande suivante en tant que racine sur le nœud où l’instance ERS est en cours d’exécution pour tuer processus de serveur de réplication de l’empilement.

   <pre><code>nw1-cl-0:~ # pgrep er.sapNW1 | xargs kill -9
   </code></pre>

   Si vous n’exécutez la commande qu’une seule fois, sapstart redémarrera le processus. Si vous l’exécutez suffisamment souvent, sapstart ne redémarrera pas le processus et la ressource se trouvera dans un état arrêté. Exécutez les commandes suivantes en tant que racine pour nettoyer l’état de la ressource de l’instance ERS après le test.

   <pre><code>nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   État des ressources après le test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Tuer le processus sapstartsrv d’empilement

   État des ressources avant le début du test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Exécutez la commande suivante en tant que racine sur le nœud où l’instance ASCS est en cours d’exécution.

   <pre><code>nw1-cl-1:~ # pgrep -fl ASCS00.*sapstartsrv
   # 59545 sapstartsrv
   
   nw1-cl-1:~ # kill -9 59545
   </code></pre>

   Le processus sapstartsrv doit toujours être redémarré par l’agent de ressource Pacemaker. État des ressources après le test :

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

## <a name="next-steps"></a>Étapes suivantes

* [Guide de haute disponibilité pour SAP NW sur les machines virtuelles Azure sur SLES pour les applications SAP multi-SID](./high-availability-guide-suse-multi-sid.md)
* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur des machines virtuelles Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha]