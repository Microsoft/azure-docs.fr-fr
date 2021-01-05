---
title: Haute disponibilité des machines virtuelles Azure pour SAP NW sur RHEL avec Azure NetApp Files | Microsoft Docs
description: Créez une haute disponibilité pour SAP NW sur les machines virtuelles Azure RHEL à l’aide d’Azure NetApp Files.
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
ms.date: 10/22/2020
ms.author: radeltch
ms.openlocfilehash: 3b33351f840cb590d0adea42f9404917b2f30ca8
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484258"
---
# <a name="azure-virtual-machines-high-availability-for-sap-netweaver-on-red-hat-enterprise-linux-with-azure-netapp-files-for-sap-applications"></a>Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux avec Azure NetApp Files pour les applications SAP

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

Cet article décrit comment déployer et configurer les machines virtuelles, installer l’infrastructure de cluster et installer un système SAP NetWeaver 7.50 à haute disponibilité à l’aide d’[Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).
Dans les exemples de configuration, commandes d’installation, et ainsi de suite, L’instance ASCS correspond au numéro 00, l’instance ERS au numéro 01, l’instance d’application principale (PAS) au numéro 02 et l’instance d’application (AAS) au numéro 03. L’assurance qualité d’ID de système SAP est utilisée. 

La couche de base de données n’est pas couverte en détail dans cet article.  

Commencez par lire les notes et publications SAP suivantes :

* [Documentation Azure NetApp Files][anf-azure-doc] 
* Note SAP [1928533], qui contient :
  * une liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP
  * des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure
  * les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données
  * la version du noyau SAP requise pour Windows et Linux sur Microsoft Azure

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
  * [Configure SAP S/4HANA ASCS/ERS with Standalone Enqueue Server 2 (ENSA2) in Pacemaker on RHEL ](https://access.redhat.com/articles/3974941) (Configurer SAP S/4HANA ASC/ERS avec le serveur de file d’attente autonome 2 (ENSA2) dans Pacemaker sur RHEL)
* Documentation RHEL spécifique à Azure :
  * [Stratégies de prise en charge des clusters à haute disponibilité RHEL - Machines virtuelles Microsoft Azure en tant que membres du cluster](https://access.redhat.com/articles/3131341)
  * [Installation et configuration d’un cluster à haute disponibilité Red Hat Enterprise Linux 7.4 (et versions ultérieures) sur Microsoft Azure](https://access.redhat.com/articles/3252491)
* [Applications SAP NetApp su Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Vue d’ensemble

La haute disponibilité pour les services centraux SAP Netweaver nécessite un stockage partagé.
Pour y parvenir sur Red Hat Linux, il était jusqu'à présent nécessaire de générer un cluster GlusterFS hautement disponible distinct. 

Maintenant, il est possible d’atteindre la haute disponibilité SAP Netweaver à l’aide d’un stockage partagé, déployé sur Azure NetApp Files. L’utilisation d’Azure NetApp Files pour le stockage partagé élimine le besoin de [cluster GlusterFS](./high-availability-guide-rhel-glusterfs.md) supplémentaire. Pacemaker est toujours nécessaire pour la haute disponibilité des services (ASCS/SCS) centraux de SAP Netweaver.

![Vue d’ensemble de la haute disponibilité SAP NetWeaver](./media/high-availability-guide-rhel/high-availability-guide-rhel-anf.png)

SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS et la base de données SAP HANA utilisent un nom d’hôte virtuel et des adresses IP virtuelles. Sur Azure, un équilibreur de charge est nécessaire pour utiliser une adresse IP virtuelle. Nous vous recommandons d’utiliser [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). La liste suivante montre la configuration de l’équilibreur de charge avec des adresses IP front-end distinctes pour les instances (A)SCS et ERS.

### <a name="ascs"></a>(A)SCS

* Configuration du frontend
  * Adresse IP 192.168.14.9
* Port de la sonde
  * Port 620<strong>&lt;nr&gt;</strong>
* Règles d’équilibrage de charge
  * Si vous utilisez Standard Load Balancer, sélectionnez **Ports haute disponibilité**
  * TCP 32<strong>&lt;nr&gt;</strong>
  * TCP 36<strong>&lt;nr&gt;</strong>
  * TCP 39<strong>&lt;nr&gt;</strong>
  * TCP 81<strong>&lt;nr&gt;</strong>
  * TCP 5<strong>&lt;nr&gt;</strong>13
  * TCP 5<strong>&lt;nr&gt;</strong>14
  * TCP 5<strong>&lt;nr&gt;</strong>16

### <a name="ers"></a>ERS

* Configuration du frontend
  * Adresse IP 192.168.14.10
* Port de la sonde
  * Port 621<strong>&lt;nr&gt;</strong>
* Règles d’équilibrage de charge
  * Si vous utilisez Standard Load Balancer, sélectionnez **Ports haute disponibilité**
  * TCP 32<strong>&lt;nr&gt;</strong>
  * TCP 33<strong>&lt;nr&gt;</strong>
  * TCP 5<strong>&lt;nr&gt;</strong>13
  * TCP 5<strong>&lt;nr&gt;</strong>14
  * TCP 5<strong>&lt;nr&gt;</strong>16

* Configuration du backend
  * Connecté aux interfaces réseau principales de toutes les machines virtuelles qui doivent faire partie du cluster (A)SCS/ERS

## <a name="setting-up-the-azure-netapp-files-infrastructure"></a>Configuration de l’infrastructure Azure NetApp Files 

SAP NetWeaver nécessite un stockage partagé pour le répertoire de transport et de profil.  Avant de poursuivre la configuration de l’infrastructure Azure NetApp Files, familiarisez-vous avec la [documentation Azure NetApp Files][anf-azure-doc]. Vérifiez si la région Azure sélectionnée est compatible avec Azure NetApp Files. Le lien suivant indique la disponibilité d’Azure NetApp Files pour chaque région Azure : [Disponibilité d’Azure NetApp Files pour chaque région Azure][anf-avail-matrix].

Azure NetApp Files est disponible dans plusieurs [régions Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Avant de déployer Azure NetApp Files, demandez l’intégration à Azure NetApp Files, en suivant les [instructions d’inscription à Azure NetApp Files][anf-register]. 

### <a name="deploy-azure-netapp-files-resources"></a>Déployer des ressources Azure NetApp Files  

Les étapes supposent que vous avez déjà déployé un [réseau virtuel Azure](../../../virtual-network/virtual-networks-overview.md). Les ressources Azure NetApp Files et les machines virtuelles, où les ressources Azure NetApp Files seront montées, doivent être déployées dans le même réseau virtuel Azure ou dans des réseaux virtuels homologués Azure.  

1. Si ce n’est pas encore fait, demandez l’[intégration à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).  
2. Créez le compte de NetApp dans la région Azure sélectionnée en suivant les [instructions de création de compte NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).  
3. Configurez le pool de capacité Azure NetApp Files en suivant les [instructions de configuration du pool de capacité Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).  
L’architecture de SAP Netweaver présentée dans cet article utilise un seul pool de capacité Azure NetApp Files, référence SKU Premium. Nous vous recommandons la référence SKU Premium de Azure NetApp Files pour la charge de travail applicative de SAP Netweaver sur Azure.  
4. Déléguez un sous-réseau pour Azure NetApp Files, comme décrit dans les [instructions de délégation d’un sous-réseau pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

5. Déployez des volumes Azure NetApp Files en suivant les [instructions de création de volume pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md). Déployez les volumes dans le [sous-réseau](/rest/api/virtualnetwork/subnets) Azure NetApp Files désigné. Les adresses IP des volumes Azure NetApp sont attribuées automatiquement. Gardez à l’esprit que les ressources Azure NetApp Files et les machines virtuelles Azure doivent être dans le même réseau virtuel Azure ou dans des réseaux virtuels homologués Azure. Dans cet exemple, nous utilisons deux volumes Azure NetApp Files : sap<b>QAS</b> et transSAP. Les chemins d’accès de fichier montés sur les points de montage correspondants sont /usrsap<b>qas</b>/sapmnt<b>QAS</b>, /usrsap<b>qas</b>/usrsap<b>QAS</b>sys, etc.  

   1. volume sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/sapmnt<b>QAS</b>)
   2. volume sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>ascs)
   3. volume sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>sys)
   4. volume sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>ers)
   5. volume transSAP (nfs://192.168.24.4/transSAP)
   6. volume sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>pas)
   7. volume sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>aas)
  
Dans cet exemple, nous avons utilisé Azure NetApp Files pour tous les systèmes de fichiers SAP Netweaver afin d’illustrer comment Azure NetApp Files peut être utilisé. Les systèmes de fichiers SAP qui n’ont pas besoin d’être montés par le biais de NFS peuvent également être déployés en tant que [stockage sur disque Azure](../../disks-types.md#premium-ssd). Dans cet exemple <b>a-e</b> doit se trouver sur Azure NetApp Files et <b>f-g</b> (autrement dit, /usr/sap/<b>QAS</b>/D<b>02</b>, /usr/sap/<b>QAS</b>/D<b>03</b>) peut être déployé en tant que stockage sur disque Azure. 

### <a name="important-considerations"></a>Points importants à prendre en compte

Lorsque vous envisagez d’utiliser Azure NetApp Files pour SAP Netweaver sur une architecture à haute disponibilité SUSE, tenez compte des considérations importantes suivantes :

- La taille de pool de capacité minimale est de 4 Tio. La taille du pool de capacité peut être augmentée par incréments de 1 Tio.
- Le volume minimal est de 100 Gio.
- Azure NetApp Files et toutes les machines virtuelles, où les volumes Azure NetApp Files seront montés, doivent être déployés dans le même réseau virtuel Azure ou dans des [réseaux virtuels homologués](../../../virtual-network/virtual-network-peering-overview.md) de la même région. L’accès à Azure NetApp Files via le peering de réseaux virtuels dans la même région est maintenant pris en charge. L’accès à Azure NetApp via le peering global n’est pas encore pris en charge.
- Le réseau virtuel sélectionné doit avoir un sous-réseau délégué à Azure NetApp Files.
- Azure NetApp Files propose une [stratégie d’exportation](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md) : vous pouvez contrôler les clients autorisés, le type d’accès (lecture et écriture, lecture seule, etc.). 
- La fonctionnalité Azure NetApp Files ne tient pas encore compte des zones. Actuellement, la fonctionnalité Azure NetApp Files n’est pas déployée dans toutes les zones de disponibilité d’une région Azure. Méfiez-vous de l’impact potentiel sur les temps de latence dans certaines régions Azure. 
- Les volumes Azure NetApp Files peuvent être déployés en tant que volumes NFSv3 ou NFSv4.1. Les deux protocoles sont pris en charge pour la couche application SAP (ASCS/ERS, serveurs d’applications SAP). 

## <a name="setting-up-ascs"></a>Configuration de (A)SCS

Dans cet exemple, les ressources ont été déployées manuellement le [portail Azure](https://portal.azure.com/#home).

### <a name="deploy-linux-manually-via-azure-portal"></a>Déployer manuellement Linux via le portail Azure

Vous devez d’abord créer les volumes Azure NetApp Files. Déployez les machines virtuelles. Par la suite, vous créez un équilibreur de charge et utilisez les machines virtuelles dans le pool back-end.

1. Créer un équilibreur de charge (interne, standard) :  
   1. Créer les adresses IP de serveurs frontaux
      1. Adresse IP 192.168.14.9 pour l’instance ASCS
         1. Ouvrir l’équilibrage de charge, sélectionner le pool d’adresses IP frontal et cliquer sur Ajouter
         1. Entrer le nom du nouveau pool d’adresses IP frontal (par exemple, **frontend.QAS.ASCS**)
         1. Définir l’affectation sur Statique et entrer l’adresse IP (par exemple, **192.168.14.9**)
         1. Cliquez sur OK
      1. Adresse IP 192.168.14.10 pour les instances ASCS ERS
         * Répéter les étapes du point « a » afin de créer une adresse IP pour l’instance ERS (par exemple, **192.168.14.10** et **frontend.QAS.ERS**)
   1. Créer le pool principal
      1. Ouvrir l’équilibrage de charge, sélectionner les pools principaux et cliquer sur Ajouter
      1. Entrer le nom du nouveau pool principal (par exemple, **backend.QAS**)
      1. Cliquer sur Ajouter une machine virtuelle
      1. Sélectionnez une machine virtuelle. 
      1. Sélectionnez les machines virtuelles du cluster (A)SCS et leurs adresses IP.
      1. Cliquez sur Ajouter.
   1. Créer les sondes d’intégrité
      1. Port 620 **00** pour l’instance ASCS
         1. Ouvrir l’équilibrage de charge, sélectionner les sondes d’intégrité et cliquer sur Ajouter
         1. Entrer le nom de la nouvelle sonde d’intégrité (par exemple, **health.QAS.ASCS**).
         1. Sélectionner le protocole TCP et le port 620 **00**, et conserver un intervalle de 5 et un seuil de défaillance sur le plan de l’intégrité de 2
         1. Cliquez sur OK
      1. Port 621 **01** pour les instances ASCS ERS
            * Répéter les étapes du point « c » afin de créer une sonde d’intégrité pour l’instance ERS (par exemple, 621 **01** et **health.QAS.ERS**)
   1. Règles d’équilibrage de charge
      1. Règles d’équilibreur de charge pour ASCS
         1. Ouvrir l’équilibreur de charge, sélectionner les règles d’équilibrage de charge et cliquer sur Ajouter
         1. Entrer le nom de la nouvelle règle d’équilibreur de charge (par exemple, **lb.QAS.ASCS**)
         1. Sélectionner l’adresse IP du serveur frontal pour l’instance ASCS, le pool principal et la sonde d’intégrité créés précédemment (par exemple, **frontend.QAS.ASCS**, **backend.QAS** et **health.QAS.ASCS**)
         1. Sélectionnez **Ports haute disponibilité**
         1. Augmenter le délai d’inactivité à 30 minutes
         1. **Veiller à activer IP flottante**
         1. Cliquez sur OK
         * Répétez les étapes ci-dessus pour créer des règles d’équilibrage de charge pour ERS (par exemple **lb.QAS.ERS**)
1. Sinon, si votre scénario requiert l’équilibrage de charge de base (interne), procédez comme suit :  
   1. Créer les adresses IP de serveurs frontaux
      1. Adresse IP 192.168.14.9 pour l’instance ASCS
         1. Ouvrir l’équilibrage de charge, sélectionner le pool d’adresses IP frontal et cliquer sur Ajouter
         1. Entrer le nom du nouveau pool d’adresses IP frontal (par exemple, **frontend.QAS.ASCS**)
         1. Définir l’affectation sur Statique et entrer l’adresse IP (par exemple, **192.168.14.9**)
         1. Cliquez sur OK
      1. Adresse IP 192.168.14.10 pour les instances ASCS ERS
         * Répéter les étapes du point « a » afin de créer une adresse IP pour l’instance ERS (par exemple, **192.168.14.10** et **frontend.QAS.ERS**)
   1. Créer le pool principal
      1. Ouvrir l’équilibrage de charge, sélectionner les pools principaux et cliquer sur Ajouter
      1. Entrer le nom du nouveau pool principal (par exemple, **backend.QAS**)
      1. Cliquer sur Ajouter une machine virtuelle
      1. Sélectionner le groupe à haute disponibilité créé précédemment pour l’instance ASCS 
      1. Sélectionner les machines virtuelles du cluster (A)SCS
      1. Cliquez sur OK
   1. Créer les sondes d’intégrité
      1. Port 620 **00** pour l’instance ASCS
         1. Ouvrir l’équilibrage de charge, sélectionner les sondes d’intégrité et cliquer sur Ajouter
         1. Entrer le nom de la nouvelle sonde d’intégrité (par exemple, **health.QAS.ASCS**).
         1. Sélectionner le protocole TCP et le port 620 **00**, et conserver un intervalle de 5 et un seuil de défaillance sur le plan de l’intégrité de 2
         1. Cliquez sur OK
      1. Port 621 **01** pour les instances ASCS ERS
            * Répéter les étapes du point « c » afin de créer une sonde d’intégrité pour l’instance ERS (par exemple, 621 **01** et **health.QAS.ERS**)
   1. Règles d’équilibrage de charge
      1. TCP 32 **00** pour l’instance ASCS
         1. Ouvrir l’équilibreur de charge, sélectionner les règles d’équilibrage de charge et cliquer sur Ajouter
         1. Entrer le nom de la nouvelle règle d’équilibrage de charge (par exemple, **lb.QAS.ASCS.3200**).
         1. Sélectionner l’adresse IP du serveur frontal pour l’instance ASCS, le pool principal et la sonde d’intégrité créés précédemment (par exemple, **frontend.QAS.ASCS**)
         1. Conserver le protocole **TCP** et indiquer le port **3200**
         1. Augmenter le délai d’inactivité à 30 minutes
         1. **Veiller à activer IP flottante**
         1. Cliquez sur OK
      1. Ports supplémentaires pour l’instance ASCS
         * Répéter les étapes du point « d » pour les ports 36 **00**, 39 **00**, 81 **00**, 5 **00** 13, 5 **00** 14, 5 **00** 16 et TCP pour l’instance ASCS
      1. Ports supplémentaires pour les instances ASCS ERS
         * Répéter les étapes du point « d » pour les ports 32 **01**, 33 **01**, 5 **01** 13, 5 **01** 14, 5 **01** 16 et TCP pour les instances ASCS ERS

      > [!IMPORTANT]
      > Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d’informations, consultez [Limitations d’équilibreur de charge Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.  

      > [!Note]
      > Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité Internet sortante, sauf si une configuration supplémentaire est effectuée pour autoriser le routage vers des points de terminaison publics. Pour savoir plus en détails comment bénéficier d’une connectivité sortante, voir [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

      > [!IMPORTANT]
      > N’activez pas les timestamps TCP sur des machines virtuelles Azure placées derrière Azure Load Balancer. L’activation des timestamps TCP entraîne l’échec des sondes d’intégrité. Définissez le paramètre **net.ipv4.tcp_timestamps** sur **0**. Pour plus d’informations, consultez [Load Balancer health probes](../../../load-balancer/load-balancer-custom-probe-overview.md) (Sondes d’intégrité Load Balancer).

## <a name="disable-id-mapping-if-using-nfsv41"></a>Désactiver le mappage d’ID (en cas d’utilisation de NFSv4.1)

Les instructions de cette section s’appliquent seulement si vous utilisez des volumes Azure NetApp Files avec le protocole NFSv4.1. Effectuez la configuration sur toutes les machines virtuelles où des volumes Azure NetApp Files NFSv4.1 seront montés.  

1. Vérifiez le paramètre de domaine NFS. Assurez-vous que le domaine est configuré en tant que domaine par défaut Azure NetApp Files, par exemple **`defaultv4iddomain.com`** et que le mappage est défini sur **nobody**.  

    > [!IMPORTANT]
    > Veillez à définir le domaine NFS dans `/etc/idmapd.conf` sur la machine virtuelle pour qu’elle corresponde à la configuration de domaine par défaut sur Azure NetApp Files : **`defaultv4iddomain.com`** . En cas d’incompatibilité entre la configuration de domaine sur le client NFS (c’est-à-dire la machine virtuelle) et le serveur NFS, par exemple la configuration Azure NetApp, les autorisations pour les fichiers sur les volumes Azure NetApp montés sur les machines virtuelles s’affichent en tant que `nobody`.  

    <pre><code>
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = <b>defaultv4iddomain.com</b>
    [Mapping]
    Nobody-User = <b>nobody</b>
    Nobody-Group = <b>nobody</b>
    </code></pre>

4. **[A]** Vérifiez `nfs4_disable_idmapping`. Cette option doit avoir la valeur **Y**. Pour créer la structure de répertoire où se trouve `nfs4_disable_idmapping`, exécutez la commande de montage. Vous ne serez pas en mesure de créer manuellement le répertoire sous /sys/modules, car l’accès est réservé pour le noyau/les pilotes.  

    <pre><code>
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    # If you need to set nfs4_disable_idmapping to Y
    mkdir /mnt/tmp
    mount 192.168.24.5:/sap<b>QAS</b>
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    </code></pre>

   Pour plus d’informations sur la modification du paramètre `nfs4_disable_idmapping`, consultez https://access.redhat.com/solutions/1749883.

### <a name="create-pacemaker-cluster"></a>Créer le cluster Pacemaker

Suivez les étapes décrites sur la page [Configuration de Pacemaker sur Red Hat Enterprise Linux dans Azure](high-availability-guide-rhel-pacemaker.md) pour créer un cluster Pacemaker de base pour ce serveur (A)SCS.

### <a name="prepare-for-sap-netweaver-installation"></a>Préparer l’installation de SAP NetWeaver

Les éléments suivants sont précédés de **[A]** (applicable à tous les nœuds), de **[1]** (applicable uniquement au nœud 1) ou de **[2]** (applicable uniquement au nœud 2).

1. **[A]** Configurer la résolution de nom d’hôte

   Vous pouvez utiliser un serveur DNS ou modifier le fichier /etc/hosts sur tous les nœuds. Cet exemple montre comment utiliser le fichier /etc/hosts.
   Remplacez l’adresse IP et le nom d’hôte dans les commandes suivantes

    ```
    sudo vi /etc/hosts
    ```

   Insérez les lignes suivantes dans le fichier /etc/hosts. Modifiez l’adresse IP et le nom d’hôte en fonction de votre environnement

    ```
    # IP address of cluster node 1
    192.168.14.5    anftstsapcl1
    # IP address of cluster node 2
    192.168.14.6     anftstsapcl2
    # IP address of the load balancer frontend configuration for SAP Netweaver ASCS
    192.168.14.9    anftstsapvh
    # IP address of the load balancer frontend configuration for SAP Netweaver ERS
    192.168.14.10    anftstsapers
    ```

1. **[1]**  Créer des répertoires SAP dans le volume Azure NetApp Files.  
   Montez temporairement le volume Azure NetApp Files sur l’une des machines virtuelles et créez les répertoires SAP (chemins d’accès de fichier).  

    ```
     # mount temporarily the volume
     sudo mkdir -p /saptmp
     # If using NFSv3
     sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 192.168.24.5:/sapQAS /saptmp
     # If using NFSv4.1
     sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys,tcp 192.168.24.5:/sapQAS /saptmp
     # create the SAP directories
     sudo cd /saptmp
     sudo mkdir -p sapmntQAS
     sudo mkdir -p usrsapQASascs
     sudo mkdir -p usrsapQASers
     sudo mkdir -p usrsapQASsys
     sudo mkdir -p usrsapQASpas
     sudo mkdir -p usrsapQASaas
     # unmount the volume and delete the temporary directory
     sudo cd ..
     sudo umount /saptmp
     sudo rmdir /saptmp
    ``` 

1. **[A]** Créer les répertoires partagés

   ```
   sudo mkdir -p /sapmnt/QAS
   sudo mkdir -p /usr/sap/trans
   sudo mkdir -p /usr/sap/QAS/SYS
   sudo mkdir -p /usr/sap/QAS/ASCS00
   sudo mkdir -p /usr/sap/QAS/ERS01
   
   sudo chattr +i /sapmnt/QAS
   sudo chattr +i /usr/sap/trans
   sudo chattr +i /usr/sap/QAS/SYS
   sudo chattr +i /usr/sap/QAS/ASCS00
   sudo chattr +i /usr/sap/QAS/ERS01
   ```

1. **[A]** Installer le client NFS et autres exigences

   ```
   sudo yum -y install nfs-utils resource-agents resource-agents-sap
   ```

1. **[A]** Vérifier la version de ressource-agents-sap

   Vérifiez que la version du package de ressources-agents-sap installé est au moins 3.9.5-124.el7
   ```
   sudo yum info resource-agents-sap
   
   # Loaded plugins: langpacks, product-id, search-disabled-repos
   # Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
   # Installed Packages
   # Name        : resource-agents-sap
   # Arch        : x86_64
   # Version     : 3.9.5
   # Release     : 124.el7
   # Size        : 100 k
   # Repo        : installed
   # From repo   : rhel-sap-for-rhel-7-server-rpms
   # Summary     : SAP cluster resource agents and connector script
   # URL         : https://github.com/ClusterLabs/resource-agents
   # License     : GPLv2+
   # Description : The SAP resource agents and connector script interface with
   #          : Pacemaker to allow SAP instances to be managed in a cluster
   #          : environment.
   ```


1. **[A]** Ajouter des entrées de montage

   Si vous utilisez NFSv3 :
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
    192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=3
    192.168.24.5:/sapQAS/usrsapQASsys /usr/sap/QAS/SYS nfs rw,hard,rsize=65536,wsize=65536,vers=3
    192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=3
   ```

   Si vous utilisez NFSv4.1 :
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
    192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
    192.168.24.5:/sapQAS/usrsapQASsys /usr/sap/QAS/SYS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
    192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   ```

   > [!NOTE]
   > Veillez à ce que les volumes Azure NetApp Files correspondent à la version du protocole NFS lors du montage des volumes. Si les volumes Azure NetApp Files sont créés en tant que volumes NFSv3, utilisez la configuration NFSv3 correspondante. Si les volumes de Azure NetApp Files sont créés en tant que volumes NFSv4.1, suivez les instructions pour désactiver le mappage d’ID et veillez à utiliser la configuration NFSv4.1 correspondante. Dans cet exemple, les volumes Azure NetApp Files ont été créés en tant que volumes NFSv3.  

   Monter les nouveaux partages

   ```
   sudo mount -a  
   ```

1. **[A]** Configurer le fichier SWAP

   ```
   sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=y
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=2000
   ```

   Redémarrer l’Agent pour activer la modification

   ```
   sudo service waagent restart
   ```

1. **[A]** Configuration de RHEL

   Configurez RHEL comme décrit dans la note SAP [2002167]

### <a name="installing-sap-netweaver-ascsers"></a>Installation de SAP NetWeaver ASC/ERS

1. **[1]** Créer une ressource IP virtuelle et la sonde d’intégrité pour l’instance ASCS

   ```
   sudo pcs node standby anftstsapcl2
   # If using NFSv3
   sudo pcs resource create fs_QAS_ASCS Filesystem device='192.168.24.5:/sapQAS/usrsapQASascs' \
     directory='/usr/sap/QAS/ASCS00' fstype='nfs' force_unmount=safe \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-QAS_ASCS
   
   # If using NFSv4.1
   sudo pcs resource create fs_QAS_ASCS Filesystem device='192.168.24.5:/sapQAS/usrsapQASascs' \
     directory='/usr/sap/QAS/ASCS00' fstype='nfs' force_unmount=safe options='sec=sys,vers=4.1' \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-QAS_ASCS
   
   sudo pcs resource create vip_QAS_ASCS IPaddr2 \
     ip=192.168.14.9 cidr_netmask=24 \
     --group g-QAS_ASCS
   
   sudo pcs resource create nc_QAS_ASCS azure-lb port=62000 \
     --group g-QAS_ASCS
   ```

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.

   ```
   sudo pcs status
   
   # Node anftstsapcl2: standby
   # Online: [ anftstsapcl1 ]
   #
   # Full list of resources:
   #
   # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
   ```

1. **[1]** Installer SAP NetWeaver ASCS  

   Installez SAP NetWeaver ASCS comme racine sur le premier nœud à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration front-end de l’équilibreur de charge pour l’instance ASCS, par exemple <b>anftstsapvh</b>, <b>192.168.14.9</b>, et du numéro d’instance utilisé pour la sonde de l’équilibreur de charge, par exemple <b>00</b>.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst.

   ```
   # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again.
   sudo firewall-cmd --zone=public  --add-port=4237/tcp
   
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=<virtual_hostname>
   ```

   Si aucun sous-dossier n’est créé dans /usr/sap/**QAS**/ASCS **00** lors de l’installation, essayez de définir le propriétaire et le groupe du dossier ASCS **00**, puis réessayez.

   ```
   sudo chown qasadm /usr/sap/QAS/ASCS00
   sudo chgrp sapsys /usr/sap/QAS/ASCS00
   ```

1. **[1]** Créer une ressource IP virtuelle et la sonde d’intégrité pour l’instance ERS

   ```
   sudo pcs node unstandby anftstsapcl2
   sudo pcs node standby anftstsapcl1
   
   # If using NFSv3
   sudo pcs resource create fs_QAS_AERS Filesystem device='192.168.24.5:/sapQAS/usrsapQASers' \
     directory='/usr/sap/QAS/ERS01' fstype='nfs' force_unmount=safe \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-QAS_AERS
   
   # If using NFSv4.1
   sudo pcs resource create fs_QAS_AERS Filesystem device='192.168.24.5:/sapQAS/usrsapQASers' \
     directory='/usr/sap/QAS/ERS01' fstype='nfs' force_unmount=safe options='sec=sys,vers=4.1' \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-QAS_AERS
   
   sudo pcs resource create vip_QAS_AERS IPaddr2 \
     ip=192.168.14.10 cidr_netmask=24 \
    --group g-QAS_AERS
   
   sudo pcs resource create nc_QAS_AERS azure-lb port=62101 \
    --group g-QAS_AERS
   ```
 
   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.

   ```
   sudo pcs status
   
   # Node anftstsapcl1: standby
   # Online: [ anftstsapcl2 ]
   #
   # Full list of resources:
   #
   # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl2
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2<
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
   #  Resource Group: g-QAS_AERS
   #      fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
   #      nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
   #      vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
   ```

1. **[2]** Installer SAP NetWeaver ERS  

   Installez SAP NetWeaver ERS comme racine sur le deuxième nœud à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration front-end de l’équilibreur de charge pour l’instance ERS, par exemple <b>anftstsapers</b> et <b>192.168.14.10</b>, et du numéro d’instance utilisé pour la sonde de l’équilibreur de charge, par exemple <b>01</b>.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst.

   ```
   # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again.
   sudo firewall-cmd --zone=public  --add-port=4237/tcp

   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=<virtual_hostname>
   ```

   Si aucun sous-dossier n’est créé dans /usr/sap/**QAS**/ERS **01** lors de l’installation, essayez de définir le propriétaire et le groupe du dossier ERS **01**, puis réessayez.

   ```
   sudo chown qaadm /usr/sap/QAS/ERS01
   sudo chgrp sapsys /usr/sap/QAS/ERS01
   ```

1. **[1]** Adapter les profils d’instance ASCS/SCS et ERS

   * Profil ASCS/SCS

   ```
   sudo vi /sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the keep alive parameter, if using ENSA1
   enque/encni/set_so_keepalive = true
   ```

   Pour ENSA1 et ENSA2, assurez-vous que les `keepalive`Paramètres de système d’exploitation sont définis comme décrit dans la note SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).  

   * Profil ERS

   ```
   sudo vi /sapmnt/QAS/profile/QAS_ERS01_anftstsapers
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # remove Autostart from ERS profile
   # Autostart = 1
   ```


1. **[A]** Configurer Keep Alive

   La communication entre le serveur d’applications SAP NetWeaver et l’ASCS/SCS est routée par l’intermédiaire d’un équilibreur de charge logiciel. L’équilibreur de charge déconnecte les connexions inactives après un délai configurable. Pour éviter ce problème, vous devez définir un paramètre dans le profil SAP NetWeaver ASCS/SCS, si vous utilisez ENSA1, et modifier les paramètres `keepalive` du système Linux sur tous les serveurs SAP pour ENSA1/ENSA2. Pour plus d’informations, consultez la [Note SAP 1410736][1410736].

   ```
   # Change the Linux system configuration
   sudo sysctl net.ipv4.tcp_keepalive_time=300
   ```

1. **[A]** Mettre à jour le fichier /usr/sap/sapservices

   Pour empêcher le démarrage des instances par le script de démarrage sapinit, toutes les instances gérées par Pacemaker doivent être commentées à partir du fichier /usr/sap/sapservices. Ne commentez pas l’instance SAP HANA si elle sera utilisée avec HANA SR.

   ```
   sudo vi /usr/sap/sapservices
   
   # On the node where you installed the ASCS, comment out the following line
   # LD_LIBRARY_PATH=/usr/sap/QAS/ASCS00/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/QAS/ASCS00/exe/sapstartsrv pf=/usr/sap/QAS/SYS/profile/QAS_ASCS00_anftstsapvh -D -u qasadm
   
   # On the node where you installed the ERS, comment out the following line
   # LD_LIBRARY_PATH=/usr/sap/QAS/ERS01/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/QAS/ERS01/exe/sapstartsrv pf=/usr/sap/QAS/ERS01/profile/QAS_ERS01_anftstsapers -D -u qasadm
   ```

1. **[1]** Créer les ressources de cluster SAP

   Si vous utilisez l’architecture de serveur de file d’attente 1 (ENSA1), définissez les ressources comme suit :

   ```
   sudo pcs property set maintenance-mode=true
   
    sudo pcs resource create rsc_sap_QAS_ASCS00 SAPInstance \
    InstanceName=QAS_ASCS00_anftstsapvh START_PROFILE="/sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_ASCS
   
    sudo pcs resource create rsc_sap_QAS_ERS01 SAPInstance \
    InstanceName=QAS_ERS01_anftstsapers START_PROFILE="/sapmnt/QAS/profile/QAS_ERS01_anftstsapers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_AERS
      
    sudo pcs constraint colocation add g-QAS_AERS with g-QAS_ASCS -5000
    sudo pcs constraint location rsc_sap_QAS_ASCS00 rule score=2000 runs_ers_QAS eq 1
    sudo pcs constraint order g-QAS_ASCS then g-QAS_AERS kind=Optional symmetrical=false
    
    sudo pcs node unstandby anftstsapcl1
    sudo pcs property set maintenance-mode=false
    ```

   SAP a introduit la prise en charge du serveur de file d’attente 2, y compris la réplication, avec SAP NW 7.52. À partir de la plateforme ABAP 1809, le serveur de file d’attente 2 est installé par défaut. Consultez la note SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) pour plus d’informations sur la prise en charge du serveur de file d’attente 2.
   Si vous utilisez l’architecture de serveur de file d’attente 2 ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), installez l’agent de ressource resource-agents-sap-4.1.1-12.el7.x86_64 ou une version ultérieure et définissez les ressources comme suit :

    ```
    sudo pcs property set maintenance-mode=true
    
    sudo pcs resource create rsc_sap_QAS_ASCS00 SAPInstance \
    InstanceName=QAS_ASCS00_anftstsapvh START_PROFILE="/sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_ASCS
   
    sudo pcs resource create rsc_sap_QAS_ERS01 SAPInstance \
    InstanceName=QAS_ERS01_anftstsapers START_PROFILE="/sapmnt/QAS/profile/QAS_ERS01_anftstsapers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_AERS
      
    sudo pcs constraint colocation add g-QAS_AERS with g-QAS_ASCS -5000
    sudo pcs constraint order g-QAS_ASCS then g-QAS_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-QAS_ASCS then stop g-QAS_AERS symmetrical=false
   
    sudo pcs node unstandby anftstsapcl1
    sudo pcs property set maintenance-mode=false
    ```

   Si vous effectuez une mise à niveau à partir d’une version antérieure et que vous passez au serveur de file d’attente 2, consultez la note SAP [2641322](https://launchpad.support.sap.com/#/notes/2641322). 

   > [!NOTE]
   > Les délais d’expiration de la configuration ci-dessus sont simplement des exemples et doivent être adaptés à la configuration SAP spécifique. 

   Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.

    ```
    sudo pcs status
    
    # Online: [ anftstsapcl1 anftstsapcl2 ]
    #
    # Full list of resources:
    #
    # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl2
    #  Resource Group: g-QAS_ASCS
    #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
    #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
    #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
    #      rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    #  Resource Group: g-QAS_AERS
    #      fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
    #      nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
    #      vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
    #      rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. **[A]** Ajouter des règles de pare-feu pour ASCS et ERS sur les deux nœuds.
   ```
   # Probe Port of ASCS
   sudo firewall-cmd --zone=public --add-port=62000/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=62000/tcp
   sudo firewall-cmd --zone=public --add-port=3200/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3200/tcp
   sudo firewall-cmd --zone=public --add-port=3600/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3600/tcp
   sudo firewall-cmd --zone=public --add-port=3900/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3900/tcp
   sudo firewall-cmd --zone=public --add-port=8100/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=8100/tcp
   sudo firewall-cmd --zone=public --add-port=50013/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50013/tcp
   sudo firewall-cmd --zone=public --add-port=50014/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50014/tcp
   sudo firewall-cmd --zone=public --add-port=50016/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50016/tcp
   # Probe Port of ERS
   sudo firewall-cmd --zone=public --add-port=62101/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=62101/tcp
   sudo firewall-cmd --zone=public --add-port=3301/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3301/tcp
   sudo firewall-cmd --zone=public --add-port=50113/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50113/tcp
   sudo firewall-cmd --zone=public --add-port=50114/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50114/tcp
   sudo firewall-cmd --zone=public --add-port=50116/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50116/tcp
   ```

## <a name="sap-netweaver-application-server-preparation"></a><a name="2d6008b0-685d-426c-b59e-6cd281fd45d7"></a>Préparation du serveur d’applications SAP NetWeaver

   Certaines bases de données exigent que l’installation de l’instance de base de données soit exécutée sur un serveur d’applications. Préparez les machines virtuelles de serveur d’applications pour pouvoir les utiliser dans ce cas de figure.  

   Les étapes ci-dessous partent du principe que vous installez le serveur d’applications sur un serveur différent des serveurs ASCS/SCS et HANA. Dans le cas contraire, certaines des étapes ci-dessous (par exemple la configuration de la résolution de nom d’hôte) ne sont pas nécessaires.  

   Les éléments suivants sont précédés de [**A]** (applicable à PAS et à AAS), de **[P]** (applicable uniquement à PAS) ou de **[S]** (applicable uniquement à AAS).  

1. **[A]** Configurer la résolution de noms d’hôte Vous pouvez utiliser un serveur DNS ou modifier le fichier /etc/hosts sur tous les nœuds. Cet exemple montre comment utiliser le fichier /etc/hosts.
   Remplacez l’adresse IP et le nom d’hôte dans les commandes suivantes :  

   ```
   sudo vi /etc/hosts
   ```

   Insérez les lignes suivantes dans le fichier /etc/hosts. Changez l’adresse IP et le nom d’hôte en fonction de votre environnement.

   ```
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS
   192.168.14.9 anftstsapvh
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS ERS
   192.168.14.10 anftstsapers
   192.168.14.7 anftstsapa01
   192.168.14.8 anftstsapa02
   ```

1. **[A]** Créer le répertoire sapmnt
   ```
   sudo mkdir -p /sapmnt/QAS
   sudo mkdir -p /usr/sap/trans

   sudo chattr +i /sapmnt/QAS
   sudo chattr +i /usr/sap/trans
   ```

1. **[A]** Installer le client NFS et autres exigences  

   ```
   sudo yum -y install nfs-utils uuidd
   ```

1. **[A]** Ajouter des entrées de montage  
   Si vous utilisez NFSv3 :
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
   192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=3
   192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=3
   ```

   Si vous utilisez NFSv4.1 :
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
   192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   ```

   Monter les nouveaux partages

   ```
   sudo mount -a
   ```

1. **[P]** Créer et monter le répertoire PAS  
   Si vous utilisez NFSv3 :
   ```
   sudo mkdir -p /usr/sap/QAS/D02
   sudo chattr +i /usr/sap/QAS/D02
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASpas /usr/sap/QAS/D02 nfs rw,hard,rsize=65536,wsize=65536,vers=3
   
   # Mount
   sudo mount -a
   ```

   Si vous utilisez NFSv4.1 :
   ```
   sudo mkdir -p /usr/sap/QAS/D02
   sudo chattr +i /usr/sap/QAS/D02
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASpas /usr/sap/QAS/D02 nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   
   # Mount
   sudo mount -a
   ```

1. **[S]** Créer et monter le répertoire AAS  
   Si vous utilisez NFSv3 :
   ```
   sudo mkdir -p /usr/sap/QAS/D03
   sudo chattr +i /usr/sap/QAS/D03
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASaas /usr/sap/QAS/D03 nfs rw,hard,rsize=65536,wsize=65536,vers=3
   
   # Mount
   sudo mount -a
   ```

   Si vous utilisez NFSv4.1 :
   ```
   sudo mkdir -p /usr/sap/QAS/D03
   sudo chattr +i /usr/sap/QAS/D03
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASaas /usr/sap/QAS/D03 nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   
   # Mount
   sudo mount -a
   ```

1. **[A]** Configurer le fichier SWAP
 
   ```
   sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=y
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=2000
   ```

   Redémarrer l’Agent pour activer la modification

   ```
   sudo service waagent restart
   ```

## <a name="install-database"></a>Installer la base de données

Dans cet exemple, SAP NetWeaver est installé sur SAP HANA. Vous pouvez utiliser n’importe quelle base de données prise en charge pour cette installation. Pour plus d’informations sur l’installation de SAP HANA dans Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure sur Red Hat Enterprise Linux][sap-hana-ha]. For a list of supported databases, see [SAP Note 1928533][1928533].

1. Exécuter l’installation de l’instance de base de données SAP

   Installez l’instance de base de données SAP NetWeaver en tant que racine à l’aide d’un nom d’hôte virtuel mappé à l’adresse IP de la configuration frontend d’équilibreur de charge pour la base de données.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst.

   ```
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin
   ```

## <a name="sap-netweaver-application-server-installation"></a>Installation de serveur d’applications SAP NetWeaver

Suivez ces étapes pour installer un serveur d’applications SAP.

1. Préparer le serveur d’applications

   Pour préparer le serveur d’applications, suivez la procédure décrite dans la section [Préparation du serveur d’applications SAP NetWeaver](high-availability-guide-rhel.md#2d6008b0-685d-426c-b59e-6cd281fd45d7) ci-dessus.

1. Installer le serveur d’applications SAP NetWeaver

   Installer un serveur d’applications SAP NetWeaver principal ou supplémentaire.

   Vous pouvez utiliser le paramètre sapinst SAPINST_REMOTE_ACCESS_USER pour autoriser un utilisateur non racine à se connecter à sapinst.

   ```
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin
   ```

1. Mettre à jour la banque d’informations sécurisée SAP HANA

   Mettez à jour la banque d’informations sécurisée SAP HANA pour qu’elle pointe vers le nom virtuel de la configuration de la réplication système SAP HANA.

   Exécutez la commande suivante pour répertorier les entrées en tant qu’\<sapsid>adm

   ```
   hdbuserstore List
   ```

   La sortie doit répertorier toutes les entrées et ressembler à ce qui suit
   ```
   DATA FILE       : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.DAT
   KEY FILE        : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.KEY
   
   KEY DEFAULT
     ENV : 192.168.14.4:30313
     USER: SAPABAP1
     DATABASE: QAS
   ```

   La sortie indique que l’adresse IP de l’entrée par défaut pointe vers la machine virtuelle et non vers l’adresse IP de l’équilibreur de charge. Cette entrée doit être modifiée pour pointer vers le nom d’hôte virtuel de l’équilibreur de charge. Assurez-vous d’utiliser le même port (**30313** dans la sortie ci-dessus) et le même nom de base de données (**QAS** dans la sortie ci-dessus) !

   ```
   su - qasadm
   hdbuserstore SET DEFAULT qasdb:30313@QAS SAPABAP1 <password of ABAP schema>
   ```

## <a name="test-the-cluster-setup"></a>Tester la configuration du cluster

1. Migrer manuellement l’instance ASCS

   État des ressources avant le début du test :

   ```
    rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Exécutez les commandes suivantes en tant que racine pour migrer l’instance ASCS.

   ```
   [root@anftstsapcl1 ~]# pcs resource move rsc_sap_QAS_ASCS00
   
   [root@anftstsapcl1 ~]# pcs resource clear rsc_sap_QAS_ASCS00
   
   # Remove failed actions for the ERS that occurred as part of the migration
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   État des ressources après le test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. Simuler l’incident de nœud

   État des ressources avant le début du test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

   Exécutez la commande suivante en tant que racine sur le nœud où l’instance ASCS est en cours d’exécution

   ```
   [root@anftstsapcl2 ~]# echo b > /proc/sysrq-trigger
   ```

   L’état après le redémarrage du nœud devrait ressembler à ce qui suit.

   ```
   Online: [ anftstsapcl1 anftstsapcl2 ]
   
   Full list of resources:
   
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   
   Failed Actions:
   * rsc_sap_QAS_ERS01_monitor_11000 on anftstsapcl1 'not running' (7): call=45, status=complete, exitreason='',
   ```

   Utilisez la commande suivante pour nettoyer les ressources ayant échoué.

   ```
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   État des ressources après le test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Tuer le processus de serveur de message

   État des ressources avant le début du test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```
   
   Exécutez les commandes suivantes en tant que racine pour identifier le processus du serveur de message et le tuer.

   ```
   [root@anftstsapcl1 ~]# pgrep ms.sapQAS | xargs kill -9
   ```

   Si vous ne tuez le serveur de messages qu’une seule fois, il sera redémarré par `sapstart`. Si vous le tuez suffisamment souvent, Pacemaker finira par déplacer l’instance ASCS vers l’autre nœud. Exécutez les commandes suivantes en tant que racine pour nettoyer l’état de la ressource de l’instance ASCS et ERS après le test.

   ```
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ASCS00
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   État des ressources après le test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. Tuer le processus de serveur d’empilement

   État des ressources avant le début du test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

   Exécutez la commande suivante en tant que racine sur le nœud où l’instance ASCS est en cours d’exécution pour tuer le serveur d’empilement.

   ```
   [root@anftstsapcl2 ~]# pgrep en.sapQAS | xargs kill -9
   ```

   L’instance ASCS devrait immédiatement basculer vers l’autre nœud. L’instance ERS devrait également basculer après le démarrage de l’instance ASCS. Exécutez les commandes suivantes en tant que racine pour nettoyer l’état de la ressource de l’instance ASCS et ERS après le test.

   ```
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ASCS00
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   État des ressources après le test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Tuer le processus de serveur de réplication de l’empilement

   État des ressources avant le début du test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Exécutez la commande suivante en tant que racine sur le nœud où l’instance ERS est en cours d’exécution pour tuer processus de serveur de réplication de l’empilement.

   ```
   [root@anftstsapcl2 ~]# pgrep er.sapQAS | xargs kill -9
   ```

   Si vous n’exécutez la commande qu’une seule fois, `sapstart` redémarrera le processus. Si vous l’exécutez suffisamment souvent, `sapstart` ne redémarrera pas le processus et la ressource se trouvera dans un état arrêté. Exécutez les commandes suivantes en tant que racine pour nettoyer l’état de la ressource de l’instance ERS après le test.

   ```
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   État des ressources après le test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Tuer le processus sapstartsrv d’empilement

   État des ressources avant le début du test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Exécutez la commande suivante en tant que racine sur le nœud où l’instance ASCS est en cours d’exécution.

   ```
   [root@anftstsapcl1 ~]# pgrep -fl ASCS00.*sapstartsrv
   # 59545 sapstartsrv
   
   [root@anftstsapcl1 ~]# kill -9 59545
   ```

   Le processus sapstartsrv doit toujours être redémarré par l’agent de ressource Pacemaker dans le cadre de la supervision. État des ressources après le test :

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

## <a name="next-steps"></a>Étapes suivantes

* [Guide de haute disponibilité pour SAP NW sur les machines virtuelles Azure sur RHEL pour les applications SAP multi-SID](./high-availability-guide-rhel-multi-sid.md)
* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur Azure (grandes instances), consultez [Haute disponibilité et récupération d’urgence de SAP HANA (grandes instances) sur Azure](hana-overview-high-availability-disaster-recovery.md).
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur des machines virtuelles Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha]