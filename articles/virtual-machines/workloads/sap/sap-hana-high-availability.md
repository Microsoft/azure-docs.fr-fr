---
title: Haute disponibilité de SAP HANA sur des machines virtuelles Azure sur SLES | Microsoft Docs
description: Haute disponibilité de SAP HANA sur les machines virtuelles Azure sur SUSE Linux Enterprise Server
services: virtual-machines-linux
documentationcenter: ''
author: rdeltcheva
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/16/2020
ms.author: radeltch
ms.openlocfilehash: 5af2c40dd1efa542ac13bd4cf96ba3017810bf00
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608671"
---
# <a name="high-availability-of-sap-hana-on-azure-vms-on-suse-linux-enterprise-server"></a>Haute disponibilité de SAP HANA sur les machines virtuelles Azure sur SUSE Linux Enterprise Server

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
[2388694]:https://launchpad.support.sap.com/#/notes/2388694
[401162]:https://launchpad.support.sap.com/#/notes/401162

[hana-ha-guide-replication]:sap-hana-high-availability.md#14c19f65-b5aa-4856-9594-b81c7e4df73d
[hana-ha-guide-shared-storage]:sap-hana-high-availability.md#498de331-fa04-490b-997c-b078de457c9d
[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/

[suse-hana-ha-guide]:https://www.suse.com/docrep/documents/ir8w88iwu7/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf
[sap-swcenter]:https://launchpad.support.sap.com/#/softwarecenter
[template-multisid-db]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json

En développement local, vous pouvez utiliser soit la réplication système HANA soit un stockage partagé pour établir une haute disponibilité pour SAP HANA.
Sur les machines virtuelles Azure, la réplication système SAP HANA dans Azure est la seule fonction de haute disponibilité prise en charge actuellement. La réplication SAP HANA se compose d’un nœud principal et d’au moins un nœud secondaire. Les modifications apportées aux données sur le nœud principal sont répliquées vers les nœuds secondaires de manière synchrone ou asynchrone.

Cet article décrit comment déployer et configurer les machines virtuelles, installer l’infrastructure de cluster, et installer et configurer la réplication système SAP HANA.
Les exemples de configuration et les commandes d’installation utilisent le numéro d’instance **03** et l’ID système HANA **HN1**.

Commencez par lire les notes et publications SAP suivantes :

* Note SAP [1928533], qui contient :
  * La liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP.
  * Des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure.
  * Les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données.
  * La version du noyau SAP requise pour Windows et Linux sur Microsoft Azure.
* La note SAP [2015553] répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure.
* La note SAP [2205917] contient des paramètres de système d’exploitation recommandés pour SUSE Linux Enterprise Server pour les applications SAP.
* La note SAP [1944799] contient des instructions SAP HANA pour SUSE Linux Enterprise Server pour les applications SAP.
* La note SAP [2178632] contient des informations détaillées sur toutes les métriques de surveillance qui sont rapportées pour SAP sur Azure.
* La note SAP [2191498] contient la version requise de l’agent hôte SAP pour Linux sur Azure.
* La note SAP [2243692] contient des informations sur les licences SAP sur Linux dans Azure.
* La note SAP [1984787] contient des informations sur SUSE Linux Enterprise Server 12.
* La note SAP [1999351] contient des informations de dépannage supplémentaires pour l’extension d’analyse Azure améliorée pour SAP.
* La note SAP [401162] a des informations sur la façon d’éviter le message « adresse déjà utilisée » quand vous configurez la réplication de système HANA.
* Le [WIKI de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contient toutes les notes SAP requises pour Linux.
* [Plateformes IaaS certifiées SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure)
* Guide [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide].
* [Déploiement de machines virtuelles Azure pour SAP sur Linux][deployment-guide] (cet article).
* Guide de [déploiement SGBD de machines virtuelles Azure pour SAP sur Linux][dbms-guide].
* [Guides des meilleures pratiques de SUSE Linux Enterprise Server pour applications SAP 12 SP3][sles-for-sap-bp]
  * Configuration d’une infrastructure à performances optimisées SAP HANA SR (SLES pour les applications SAP 12 SP1). Le guide contient toutes les informations requises pour configurer la réplication système SAP HANA en développement local. Utilisez ce guide comme référence.
  * Configuration d’une infrastructure à coût optimisé SAP HANA SR (SLES pour les applications SAP 12 SP1)

## <a name="overview"></a>Vue d’ensemble

Pour atteindre la haute disponibilité, SAP HANA est installé sur deux machines virtuelles. Les données sont répliquées à l’aide de la réplication de système HANA.

![Présentation de la haute disponibilité de SAP HANA](./media/sap-hana-high-availability/ha-suse-hana.png)

La configuration de la réplication du système SAP HANA utilise un nom d’hôte virtuel dédié et des adresses IP virtuelles. Sur Azure, un équilibreur de charge est nécessaire pour utiliser une adresse IP virtuelle. La liste suivante illustre la configuration de l’équilibreur de charge :

* Configuration front-end : Adresse IP 10.0.0.13 pour hn1-db
* Configuration back-end : Connecté aux interfaces réseau principales de toutes les machines virtuelles qui doivent faire partie de la réplication de système HANA
* Port de la sonde : Port 62503
* Règles d’équilibrage de charge : 30313 TCP, 30315 TCP, 30317 TCP

## <a name="deploy-for-linux"></a>Déploiement pour Linux

L’agent de ressource pour SAP HANA est inclus dans SUSE Linux Enterprise Server for SAP Applications.
La Place de marché Azure contient une image de SUSE Linux Enterprise Server for SAP Applications 12 que vous pouvez utiliser pour déployer de nouvelles machines virtuelles.

### <a name="deploy-with-a-template"></a>Déployer les mises à niveau avec un modèle

Vous pouvez utiliser un des modèles de démarrage rapide disponibles sur GitHub pour déployer toutes les ressources nécessaires. Le modèle déploie les machines virtuelles, l’équilibrage de charge, le groupe à haute disponibilité, etc.
Suivez ces étapes pour déployer le modèle :

1. Ouvrez le [modèle de base de données][template-multisid-db] ou le [modèle convergé][template-converged] sur le portail Azure. 
    Le modèle de base de données crée les règles d’équilibrage de charge pour une base de données uniquement. Le modèle convergent crée également les règles d’équilibrage de charge pour une instance ASCS/SCS et ERS (Linux uniquement). Si vous prévoyez d’installer un système SAP NetWeaver et souhaitez installer l’instance ASC/SCS sur les mêmes machines, utilisez le [modèle convergé][template-converged].

1. Entrez les paramètres suivants :
    - **ID du système SAP** : Entrez l’ID du système SAP que vous souhaitez installer. Cet ID est utilisé comme préfixe pour les ressources déployées.
    - **Type de pile** : (Ce paramètre est applicable uniquement si vous utilisez le modèle convergé.) Sélectionnez le type de pile de SAP NetWeaver.
    - **Type de système d’exploitation** : Sélectionnez l’une des distributions Linux. Dans cet exemple, sélectionnez **SLES 12**.
    - **Type de base de données** : sélectionnez **HANA**.
    - **Taille du système SAP** : entrez le nombre de SAP que le nouveau système va fournir. Si vous ne savez pas combien de SAP sont exigés par le système, demandez à votre partenaire technologique SAP ou un intégrateur système.
    - **Disponibilité du système** : Sélectionnez la haute disponibilité **(HA)** .
    - **Nom d’utilisateur et mot de passe administrateur** : Un utilisateur pouvant être utilisé pour se connecter à la machine est créé.
    - **Sous-réseau nouveau ou existant** : Détermine s’il faut créer un réseau virtuel et un sous-réseau, ou utiliser un sous-réseau existant. Si vous disposez déjà d’un réseau virtuel connecté à votre réseau local, sélectionnez **Existant**.
    - **ID de sous-réseau** : Si vous voulez déployer la machine virtuelle dans un réseau virtuel existant où vous avez défini un sous-réseau auquel la machine virtuelle doit être attribuée, nommez l’ID de ce sous-réseau spécifique. L’identifiant se présente généralement sous la forme **/subscriptions/\<subscription ID>/resourceGroups/\<resource group name>/providers/Microsoft.Network/virtualNetworks/\<virtual network name>/subnets/\<subnet name>** .

### <a name="manual-deployment"></a>Déploiement manuel

> [!IMPORTANT]
> Vérifiez que le système d’exploitation que vous sélectionnez est certifié SAP pour SAP HANA sur les types de machine virtuelle spécifiques que vous utilisez. La liste des types de machine virtuelle certifiés SAP HANA et des systèmes d’exploitation correspondants peut être consultée dans la page [SAP HANA Certified IaaS Platforms](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Cliquez sur les détails du type de machine virtuelle listé pour obtenir la liste complète des versions de système d’exploitation SAP HANA prises en charge pour le type de machine virtuelle spécifique
>  

1. Créez un groupe de ressources.
1. Créez un réseau virtuel.
1. Créer un groupe à haute disponibilité.
   - Définir le domaine de mise à jour maximal.
1. Créer un équilibrage de charge (interne). Nous vous recommandons [Standard Load Balancer](../../../load-balancer/load-balancer-overview.md).
   - Sélectionnez le réseau virtuel créé à l’étape 2.
1. Créez la machine virtuelle 1.
   - Utilisez une image SLES4SAP de la galerie Azure prise en charge pour SAP HANA sur le type de machine virtuelle que vous avez sélectionné.
   - Sélectionnez le groupe à haute disponibilité créé à l’étape 3.
1. Créez la machine virtuelle 2.
   - Utilisez une image SLES4SAP de la galerie Azure prise en charge pour SAP HANA sur le type de machine virtuelle que vous avez sélectionné.
   - Sélectionnez le groupe à haute disponibilité créé à l’étape 3. 
1. Ajoutez des disques de données.

> [!IMPORTANT]
> Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d'informations, consultez [Limitations relatives à Azure Load Balancer](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.   

> [!Note]
> Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité Internet sortante, sauf si une configuration supplémentaire est effectuée pour autoriser le routage vers des points de terminaison publics. Pour savoir plus en détails comment bénéficier d’une connectivité sortante, voir [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

1. Si vous utilisez Standard Load Balancer, suivez ces étapes de configuration :
   1. Commencez par créer un pool d’adresses IP frontales :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
      1. Entrez le nom du nouveau pool d’adresses IP frontales (par exemple **hana-frontend**).
      1. Définissez l’**affectation** sur **Statique** et entrez l’adresse IP (par exemple **10.0.0.13**).
      1. Sélectionnez **OK**.
      1. Une fois le pool d’adresses IP frontal créé, notez son adresse IP.
   
   1. Créez ensuite un pool principal :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
      1. Entrer le nom du nouveau pool principal (par exemple **hana-backend**).
      1. Sélectionnez **Réseau virtuel**.
      1. Cliquez sur **Ajouter une machine virtuelle**.
      1. Sélectionnez **Machine virtuelle**.
      1. Sélectionnez les machines virtuelles du cluster SAP HANA et leurs adresses IP.
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

1. Si à l’inverse votre scénario exige d’utiliser l’équilibreur de charge de base, suivez ces étapes de configuration :
   1. Commencez par créer un pool d’adresses IP frontales :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
      1. Entrez le nom du nouveau pool d’adresses IP frontales (par exemple **hana-frontend**).
      1. Définissez l’**affectation** sur **Statique** et entrez l’adresse IP (par exemple **10.0.0.13**).
      1. Sélectionnez **OK**.
      1. Une fois le pool d’adresses IP frontal créé, notez son adresse IP.
   
   1. Créez ensuite un pool principal :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
      1. Entrer le nom du nouveau pool principal (par exemple **hana-backend**).
      1. Cliquez sur **Ajouter une machine virtuelle**.
      1. Sélectionnez le groupe à haute disponibilité créé à l’étape 3.
      1. Sélectionnez les machines virtuelles du cluster SAP HANA.
      1. Sélectionnez **OK**.
   
   1. Créez ensuite une sonde d’intégrité :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez les **sondes d’intégrité**, puis cliquez sur **Ajouter**.
      1. Entrez le nom de la nouvelle sonde d’intégrité (par exemple **hana-hp**).
      1. Sélectionnez **TCP** pour le protocole et le port 625 **03**. Consersez la valeur **Intervalle** à 5, et la valeur **Seuil de défaillance** à 2.
      1. Sélectionnez **OK**.
   
   1. Pour SAP HANA 1.0, créez les règles d’équilibrage de charge :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter**.
      1. Entrer le nom de la nouvelle règle d’équilibrage de charge (par exemple hana-lb-3 **03** 15).
      1. Sélectionnez l’adresse IP du serveur frontal, le pool principal et la sonde d’intégrité que vous avez créée précédemment (par exemple, **hana-frontend**).
      1. Conservez le **Protocole** à **TCP**, puis entrez le port 3 **03** 15.
      1. Augmentez le **délai d’inactivité** à 30 minutes.
      1. Veillez à **activer l’IP flottante** .
      1. Sélectionnez **OK**.
      1. Répétez ces étapes pour le port 3 **03** 17.
   
   1. Pour SAP HANA 2.0, créez les règles d’équilibrage de charge pour la base de données du système :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter**.
      1. Entrez le nom de la nouvelle règle d’équilibrage de charge (par exemple hana-lb-3 **03** 13).
      1. Sélectionnez l’adresse IP du serveur frontal, le pool principal et la sonde d’intégrité que vous avez créée précédemment (par exemple, **hana-frontend**).
      1. Conservez le **Protocole** à **TCP**, puis entrez le port 3 **03** 13.
      1. Augmentez le **délai d’inactivité** à 30 minutes.
      1. Veillez à **activer l’IP flottante** .
      1. Sélectionnez **OK**.
      1. Répétez ces étapes pour le port 3 **03** 14.
   
   1. Pour SAP HANA 2.0, créez d’abord les règles d’équilibrage de charge pour la base de données locataire :
   
      1. Ouvrez l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter**.
      1. Entrez le nom de la nouvelle règle d’équilibrage de charge (par exemple hana-lb-3 **03** 40).
      1. Sélectionnez l’adresse IP du serveur frontal, le pool principal et la sonde d’intégrité créés précédemment (par exemple **hana-frontend**).
      1. Conservez le **Protocole** à **TCP**, puis entrez le port 3 **03** 40.
      1. Augmentez le **délai d’inactivité** à 30 minutes.
      1. Veillez à **activer l’IP flottante** .
      1. Sélectionnez **OK**.
      1. Répétez ces étapes pour les ports 3 **03** 41 et 3 **03** 42.

   Pour plus d’informations sur les ports requis pour SAP HANA, consultez le chapitre [Connections to Tenant Databases](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6/latest/en-US/7a9343c9f2a2436faa3cfdb5ca00c052.html) (Connexions aux bases de données locataires) dans le guide [SAP HANA Tenant Databases](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6) (Bases de données locataires SAP HANA) ou la [Note SAP 2388694][2388694].

> [!IMPORTANT]
> N’activez pas les timestamps TCP sur des machines virtuelles Azure placées derrière Azure Load Balancer. L’activation des timestamps TCP entraîne l’échec des sondes d’intégrité. Définissez le paramètre **net.ipv4.tcp_timestamps** sur **0**. Pour plus d’informations, consultez [Load Balancer health probes](../../../load-balancer/load-balancer-custom-probe-overview.md) (Sondes d’intégrité Load Balancer).
> Voir aussi la note SAP [2382421](https://launchpad.support.sap.com/#/notes/2382421). 

## <a name="create-a-pacemaker-cluster"></a>Créez un cluster Pacemaker

Suivez les étapes décrites à la page [Configuration de Pacemaker sur SUSE Linux Enterprise Server dans Azure](high-availability-guide-suse-pacemaker.md) pour créer un cluster Pacemaker de base pour ce serveur HANA. Vous pouvez utiliser le même cluster Pacemaker pour SAP HANA et SAP NetWeaver (A)SCS.

## <a name="install-sap-hana"></a>Installer SAP HANA

Les étapes de cette section utilisent les préfixes suivants :
- **[A]**  : l’étape s’applique à tous les nœuds.
- **[1]**  : l’étape ne s’applique qu’au nœud 1.
- **[2]**  : l’étape s’applique uniquement au nœud 2 du cluster Pacemaker.

1. **[A]** Configurez la disposition du disque : **Logical Volume Manager (LVM)** .

   Nous recommandons d’utiliser LVM pour les volumes qui stockent des données et des fichiers journaux. L’exemple suivant part du principe que les machines virtuelles disposent de quatre disques de données joints qui sont utilisés pour créer deux volumes.

   Répertoriez tous les disques disponibles :

   <pre><code>ls /dev/disk/azure/scsi1/lun*
   </code></pre>

   Exemple de sortie :

   <pre><code>
   /dev/disk/azure/scsi1/lun0  /dev/disk/azure/scsi1/lun1  /dev/disk/azure/scsi1/lun2  /dev/disk/azure/scsi1/lun3
   </code></pre>

   Créez des volumes physiques pour tous les disques que vous souhaitez utiliser :

   <pre><code>sudo pvcreate /dev/disk/azure/scsi1/lun0
   sudo pvcreate /dev/disk/azure/scsi1/lun1
   sudo pvcreate /dev/disk/azure/scsi1/lun2
   sudo pvcreate /dev/disk/azure/scsi1/lun3
   </code></pre>

   Créez un groupe de volume pour les fichiers de données. Utilisez un groupe de volume pour les fichiers journaux et un autre pour le répertoire partagé de SAP HANA :

   <pre><code>sudo vgcreate vg_hana_data_<b>HN1</b> /dev/disk/azure/scsi1/lun0 /dev/disk/azure/scsi1/lun1
   sudo vgcreate vg_hana_log_<b>HN1</b> /dev/disk/azure/scsi1/lun2
   sudo vgcreate vg_hana_shared_<b>HN1</b> /dev/disk/azure/scsi1/lun3
   </code></pre>

   Créez les volumes logiques. Un volume linéaire est créé lorsque vous utilisez `lvcreate` sans le commutateur `-i`. Nous vous suggérons de créer un volume agrégé par bandes pour obtenir de meilleures performances d’E/S, et d’aligner les tailles des bandes sur les valeurs décrites dans [Configurations de stockage de machines virtuelles SAP HANA](./hana-vm-operations-storage.md). L’argument `-i` doit indiquer le nombre de volumes physiques sous-jacents et l’argument `-I` la taille de bande. Dans ce document, deux volumes physiques sont utilisés pour le volume de données. Par conséquent, l’argument de commutateur `-i` est défini sur **2**. La taille de bande pour le volume de données est de **256 Kio**. Un volume physique étant utilisé pour le volume du fichier journal, aucun commutateur `-i` ou `-I` n’est utilisé explicitement pour les commandes de volume du fichier journal.  

   > [!IMPORTANT]
   > Utilisez le commutateur `-i` et définissez sa valeur sur le nombre de volumes physiques sous-jacents lorsque vous utilisez plusieurs volumes physiques pour chaque volume de données, volume de journal ou volume partagé. Utilisez le commutateur `-I` pour spécifier la taille de bande lors de la création d’un volume agrégé par bandes.  
   > Pour connaître les configurations de stockage recommandées, notamment les tailles de bande et le nombre de disques, consultez [Configurations de stockage de machines virtuelles SAP HANA](./hana-vm-operations-storage.md).  

   <pre><code>sudo lvcreate <b>-i 2</b> <b>-I 256</b> -l 100%FREE -n hana_data vg_hana_data_<b>HN1</b>
   sudo lvcreate -l 100%FREE -n hana_log vg_hana_log_<b>HN1</b>
   sudo lvcreate -l 100%FREE -n hana_shared vg_hana_shared_<b>HN1</b>
   sudo mkfs.xfs /dev/vg_hana_data_<b>HN1</b>/hana_data
   sudo mkfs.xfs /dev/vg_hana_log_<b>HN1</b>/hana_log
   sudo mkfs.xfs /dev/vg_hana_shared_<b>HN1</b>/hana_shared
   </code></pre>
  
   Créez les répertoires de montage et copiez l’UUID de tous les volumes logiques :

   <pre><code>sudo mkdir -p /hana/data/<b>HN1</b>
   sudo mkdir -p /hana/log/<b>HN1</b>
   sudo mkdir -p /hana/shared/<b>HN1</b>
   # Write down the ID of /dev/vg_hana_data_<b>HN1</b>/hana_data, /dev/vg_hana_log_<b>HN1</b>/hana_log, and /dev/vg_hana_shared_<b>HN1</b>/hana_shared
   sudo blkid
   </code></pre>

   Créez des entrées `fstab` pour les trois volumes logiques :       

   <pre><code>sudo vi /etc/fstab
   </code></pre>

   Insérez la ligne suivante dans le fichier `/etc/fstab` :      

   <pre><code>/dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_data_<b>HN1</b>-hana_data&gt;</b> /hana/data/<b>HN1</b> xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_log_<b>HN1</b>-hana_log&gt;</b> /hana/log/<b>HN1</b> xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_shared_<b>HN1</b>-hana_shared&gt;</b> /hana/shared/<b>HN1</b> xfs  defaults,nofail  0  2
   </code></pre>

   Montez les nouveaux volumes :

   <pre><code>sudo mount -a
   </code></pre>

1. **[A]** Configurez la disposition du disque : **Disques simples**.

   Pour les systèmes de démonstration, vous pouvez placer vos données et fichiers journaux HANA sur un disque. Créez une partition sur /dev/disk/azure/scsi1/lun0 au format xfs :

   <pre><code>sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun0'
   sudo mkfs.xfs /dev/disk/azure/scsi1/lun0-part1
   
   # Write down the ID of /dev/disk/azure/scsi1/lun0-part1
   sudo /sbin/blkid
   sudo vi /etc/fstab
   </code></pre>

   Insérez cette ligne dans le fichier /etc/fstab :

   <pre><code>/dev/disk/by-uuid/<b>&lt;UUID&gt;</b> /hana xfs  defaults,nofail  0  2
   </code></pre>

   Créez le répertoire cible et montez le disque :

   <pre><code>sudo mkdir /hana
   sudo mount -a
   </code></pre>

1. **[A]** Configurez la résolution de nom d’hôte pour tous les hôtes.

   Vous pouvez utiliser un serveur DNS ou modifier le fichier /etc/hosts sur tous les nœuds. Cet exemple vous montre comment utiliser le fichier /etc/hosts.
   Remplacez l’adresse IP et le nom d’hôte dans les commandes suivantes :

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Insérez les lignes suivantes dans le fichier /etc/hosts. Modifiez l’adresse IP et le nom d’hôte en fonction de votre environnement :

   <pre><code><b>10.0.0.5 hn1-db-0</b>
   <b>10.0.0.6 hn1-db-1</b>
   </code></pre>

1. **[A]** Installez les packages de haute disponibilité SAP HANA :

   <pre><code>sudo zypper install SAPHanaSR
   </code></pre>

Consultez le chapitre 4 de la publication [SAP HANA SR Performance Optimized Scenario guide](https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/) (Scénario à performances optimisées de réplication système de SAP HANA) pour installer la réplication système SAP HANA.

1. **[A]** Exécutez le programme **hdblcm** du DVD HANA. Entrez les valeurs suivantes à l’invite :
   * Choose installation : tapez **1**.
   * Select additional components for installation : tapez **1**.
   * Enter Installation Path [/hana/shared] : Sélectionnez Entrée.
   * Enter Local Host Name [..] : Sélectionnez Entrée.
   * Do you want to add additional hosts to the system? (y/n) [n] : Sélectionnez Entrée.
   * Enter SAP HANA System ID : entrez le SID HANA, par exemple : **HN1**.
   * Enter Instance Number [00] : entrez le numéro d’instance HANA. Entrez **03** si vous avez utilisé le modèle Azure ou si vous avez suivi la section de cet article sur le déploiement manuel.
   * Select Database Mode / Enter Index [1] : Sélectionnez Entrée.
   * Select System Usage / Enter Index [4] : sélectionnez la valeur de l’utilisation du système.
   * Enter Location of Data Volumes [/hana/data/HN1] : Sélectionnez Entrée.
   * Enter Location of Log Volumes [/hana/log/HN1] : Sélectionnez Entrée.
   * Restrict maximum memory allocation? [n] : Sélectionnez Entrée.
   * Enter Certificate Host Name For Host '...' [...] : Sélectionnez Entrée.
   * Enter SAP Host Agent User (sapadm) Password: tapez le mot de passe utilisateur de l’agent hôte.
   * Confirm SAP Host Agent User (sapadm) Password: retapez le mot de passe utilisateur de l’agent hôte pour confirmer.
   * Enter System Administrator (hdbadm) Password: tapez le mot de passe d’administrateur système.
   * Confirm System Administrator (hdbadm) Password: retapez le mot de passe d’administrateur système pour confirmer.
   * Enter System Administrator Home Directory [/usr/sap/HN1/home] : Sélectionnez Entrée.
   * Enter System Administrator Login Shell [/bin/sh] : Sélectionnez Entrée.
   * Enter System Administrator User ID [1001] : Sélectionnez Entrée.
   * Enter ID of User Group (sapsys) [79] : Sélectionnez Entrée.
   * Enter Database User (SYSTEM) Password: tapez le mot de passe d’utilisateur de base de données.
   * Confirm Database User (SYSTEM) Password: retapez le mot de passe d’utilisateur de base de données pour confirmer.
   * Restart system after machine reboot? [n] : Sélectionnez Entrée.
   * Do you want to continue? (y/n) : validez le récapitulatif. Tapez **Y** pour continuer.

1. **[A]** Mettre à niveau l’agent hôte SAP.

   Téléchargez la dernière archive de l’agent hôte SAP à partir du [SAP Software Center][sap-swcenter] et exécutez la commande suivante pour mettre à niveau l’agent. Remplacez le chemin d’accès à l’archive pour pointer vers le fichier que vous avez téléchargé :

   <pre><code>sudo /usr/sap/hostctrl/exe/saphostexec -upgrade -archive &lt;path to SAP Host Agent SAR&gt;
   </code></pre>

## <a name="configure-sap-hana-20-system-replication"></a>Configurer la réplication de système SAP HANA 2.0

Les étapes de cette section utilisent les préfixes suivants :

* **[A]**  : l’étape s’applique à tous les nœuds.
* **[1]**  : l’étape ne s’applique qu’au nœud 1.
* **[2]**  : l’étape s’applique uniquement au nœud 2 du cluster Pacemaker.

1. **[1]** Créer une base de données locataire.

   Si vous utilisez SAP HANA 2.0 ou MDC, créez une base de données locataire pour votre système SAP NetWeaver. Remplacez **NW1** par le SID de votre système SAP.

   Exécutez la commande qui suit en tant que <hanasid\>adm :

   <pre><code>hdbsql -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> -d SYSTEMDB 'CREATE DATABASE <b>NW1</b> SYSTEM USER PASSWORD "<b>passwd</b>"'
   </code></pre>

1. **[1]** Configurez la réplication de système sur le premier nœud :

   Sauvegardez les bases de données en tant que <hanasid\>adm :

   <pre><code>hdbsql -d SYSTEMDB -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupSYS</b>')"
   hdbsql -d <b>HN1</b> -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupHN1</b>')"
   hdbsql -d <b>NW1</b> -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupNW1</b>')"
   </code></pre>

   Copiez les fichiers d’infrastructure à clé publique du système sur le site secondaire :

   <pre><code>scp /usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/data/SSFS_<b>HN1</b>.DAT   <b>hn1-db-1</b>:/usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/data/
   scp /usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/key/SSFS_<b>HN1</b>.KEY  <b>hn1-db-1</b>:/usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/key/
   </code></pre>

   Créez le site principal :

   <pre><code>hdbnsutil -sr_enable --name=<b>SITE1</b>
   </code></pre>

1. **[2]** Configurez la réplication de système sur le second nœud :
    
   Inscrivez le second nœud pour démarrer la réplication de système. Exécutez la commande suivante en tant que <hanasid\>adm :

   <pre><code>sapcontrol -nr <b>03</b> -function StopWait 600 10
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b> 
   </code></pre>

## <a name="configure-sap-hana-10-system-replication"></a>Configurer la réplication de système SAP HANA 1.0

Les étapes de cette section utilisent les préfixes suivants :

* **[A]**  : l’étape s’applique à tous les nœuds.
* **[1]**  : l’étape ne s’applique qu’au nœud 1.
* **[2]**  : l’étape s’applique uniquement au nœud 2 du cluster Pacemaker.

1. **[1]** Créez les utilisateurs requis.

   Exécutez la commande suivante en tant que racine. Veillez à remplacer les chaînes en gras (ID du système HANA **HN1** et numéro d’instance **03**) par les valeurs de votre installation SAP HANA :

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbsql -u system -i <b>03</b> 'CREATE USER <b>hdb</b>hasync PASSWORD "<b>passwd</b>"'
   hdbsql -u system -i <b>03</b> 'GRANT DATA ADMIN TO <b>hdb</b>hasync'
   hdbsql -u system -i <b>03</b> 'ALTER USER <b>hdb</b>hasync DISABLE PASSWORD LIFETIME'
   </code></pre>

1. **[A]** Créez l’entrée keystore.

   Exécutez la commande suivante en tant que racine pour créer une nouvelle entrée de magasin de clés :

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbuserstore SET <b>hdb</b>haloc localhost:3<b>03</b>15 <b>hdb</b>hasync <b>passwd</b>
   </code></pre>

1. **[1]** Sauvegardez la base de données.

   Sauvegardez les bases de données en tant que racine :

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbsql -d SYSTEMDB -u system -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackup</b>')"
   </code></pre>

   Si vous utilisez une installation mutualisée, sauvegardez également la base de données locataire :

   <pre><code>hdbsql -d <b>HN1</b> -u system -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackup</b>')"
   </code></pre>

1. **[1]** Configurez la réplication de système sur le premier nœud.

   Créez le site principal en tant que <hanasid\>adm :

   <pre><code>su - <b>hdb</b>adm
   hdbnsutil -sr_enable –-name=<b>SITE1</b>
   </code></pre>

1. **[2]** Configurez la réplication de système sur le nœud secondaire.

   Inscrivez le site secondaire en tant que <hanasid\>adm :

   <pre><code>sapcontrol -nr <b>03</b> -function StopWait 600 10
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b> 
   </code></pre>

## <a name="create-sap-hana-cluster-resources"></a>Créer les ressources de cluster SAP HANA

Tout d’abord, créez la topologie HANA. Exécutez les commandes suivantes sur l’un des nœuds du cluster Pacemaker :

<pre><code>sudo crm configure property maintenance-mode=true

# Replace the bold string with your instance number and HANA system ID

sudo crm configure primitive rsc_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> ocf:suse:SAPHanaTopology \
  operations \$id="rsc_sap2_<b>HN1</b>_HDB<b>03</b>-operations" \
  op monitor interval="10" timeout="600" \
  op start interval="0" timeout="600" \
  op stop interval="0" timeout="300" \
  params SID="<b>HN1</b>" InstanceNumber="<b>03</b>"

sudo crm configure clone cln_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> rsc_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> \
  meta clone-node-max="1" target-role="Started" interleave="true"
</code></pre>

Ensuite, créez les ressources HANA :

> [!IMPORTANT]
> Des tests récents ont révélé des cas où netcat cessait de répondre aux demandes en raison du backlog et de sa capacité à ne gérer qu’une seule connexion. La ressource netcat cesse d’écouter les demandes d’Azure Load Balancer et l’adresse IP flottante devient indisponible.  
> Pour les clusters Pacemaker existants, nous vous recommandons de remplacer netcat par socat. Actuellement, nous vous recommandons d'utiliser l'agent de ressources azure-lb, qui fait partie du package resource-agents, avec la configuration requise suivante pour la version du package :
> - Pour SLES 12 SP4/SP5, la version minimum est resource-agents-4.3.018.a7fb5035-3.30.1.  
> - Pour SLES 15/15 SP1, la version minimum est resource-agents-4.3.0184.6ee15eb2-4.13.1.  
>
> Notez que la modification nécessitera un bref temps d’arrêt.  
> Pour les clusters Pacemaker existants, si la configuration a déjà été modifiée pour utiliser socat comme décrit à la page [Azure Load-Balancer Detection Hardening](https://www.suse.com/support/kb/doc/?id=7024128), il n’est pas nécessaire de passer immédiatement à l’agent de ressources azure-lb.


> [!NOTE]
> Cet article contient des références aux termes *maître* et *esclave*, termes que Microsoft n’utilise plus. Lorsque ces termes seront supprimés du logiciel, nous les supprimerons de cet article.

<pre><code># Replace the bold string with your instance number, HANA system ID, and the front-end IP address of the Azure load balancer. 

sudo crm configure primitive rsc_SAPHana_<b>HN1</b>_HDB<b>03</b> ocf:suse:SAPHana \
  operations \$id="rsc_sap_<b>HN1</b>_HDB<b>03</b>-operations" \
  op start interval="0" timeout="3600" \
  op stop interval="0" timeout="3600" \
  op promote interval="0" timeout="3600" \
  op monitor interval="60" role="Master" timeout="700" \
  op monitor interval="61" role="Slave" timeout="700" \
  params SID="<b>HN1</b>" InstanceNumber="<b>03</b>" PREFER_SITE_TAKEOVER="true" \
  DUPLICATE_PRIMARY_TIMEOUT="7200" AUTOMATED_REGISTER="false"

sudo crm configure ms msl_SAPHana_<b>HN1</b>_HDB<b>03</b> rsc_SAPHana_<b>HN1</b>_HDB<b>03</b> \
  meta notify="true" clone-max="2" clone-node-max="1" \
  target-role="Started" interleave="true"

sudo crm configure primitive rsc_ip_<b>HN1</b>_HDB<b>03</b> ocf:heartbeat:IPaddr2 \
  meta target-role="Started" \
  operations \$id="rsc_ip_<b>HN1</b>_HDB<b>03</b>-operations" \
  op monitor interval="10s" timeout="20s" \
  params ip="<b>10.0.0.13</b>"

sudo crm configure primitive rsc_nc_<b>HN1</b>_HDB<b>03</b> azure-lb port=625<b>03</b> \
  meta resource-stickiness=0

sudo crm configure group g_ip_<b>HN1</b>_HDB<b>03</b> rsc_ip_<b>HN1</b>_HDB<b>03</b> rsc_nc_<b>HN1</b>_HDB<b>03</b>

sudo crm configure colocation col_saphana_ip_<b>HN1</b>_HDB<b>03</b> 4000: g_ip_<b>HN1</b>_HDB<b>03</b>:Started \
  msl_SAPHana_<b>HN1</b>_HDB<b>03</b>:Master  

sudo crm configure order ord_SAPHana_<b>HN1</b>_HDB<b>03</b> Optional: cln_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> \
  msl_SAPHana_<b>HN1</b>_HDB<b>03</b>

# Clean up the HANA resources. The HANA resources might have failed because of a known issue.
sudo crm resource cleanup rsc_SAPHana_<b>HN1</b>_HDB<b>03</b>

sudo crm configure property maintenance-mode=false
sudo crm configure rsc_defaults resource-stickiness=1000
sudo crm configure rsc_defaults migration-threshold=5000
</code></pre>

Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.

<pre><code>sudo crm_mon -r

# Online: [ hn1-db-0 hn1-db-1 ]
#
# Full list of resources:
#
# stonith-sbd     (stonith:external/sbd): Started hn1-db-0
# Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
#     Started: [ hn1-db-0 hn1-db-1 ]
# Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
#     Masters: [ hn1-db-0 ]
#     Slaves: [ hn1-db-1 ]
# Resource Group: g_ip_HN1_HDB03
#     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
#     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
</code></pre>

## <a name="test-the-cluster-setup"></a>Tester la configuration du cluster

Cette section explique comment vous pouvez tester votre configuration. Chaque test suppose que vous êtes connecté en tant que root et que le nœud principal SAP HANA est en cours d’exécution sur la machine virtuelle **hn1-db-0**.

### <a name="test-the-migration"></a>Tester la migration

Avant de commencer le test, assurez-vous que Pacemaker ne comporte pas d’action ayant échoué (via crm_mon -r), qu’il n’existe pas de contraintes d’emplacement inattendues (par exemple les reliquats d’un test de migration) et que HANA est en état de synchronisation, par exemple avec SAPHanaSR-showAttr :

<pre><code>hn1-db-0:~ # SAPHanaSR-showAttr

Global cib-time
--------------------------------
global Mon Aug 13 11:26:04 2018

Hosts    clone_state lpa_hn1_lpt node_state op_mode   remoteHost    roles                            score site  srmode sync_state version                vhost
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
hn1-db-0 PROMOTED    1534159564  online     logreplay nws-hana-vm-1 4:P:master1:master:worker:master 150   SITE1 sync   PRIM       2.00.030.00.1522209842 nws-hana-vm-0
hn1-db-1 DEMOTED     30          online     logreplay nws-hana-vm-0 4:S:master1:master:worker:master 100   SITE2 sync   SOK        2.00.030.00.1522209842 nws-hana-vm-1
</code></pre>

Vous pouvez migrer le nœud principal SAP HANA en exécutant la commande suivante :

<pre><code>crm resource migrate msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-1</b>
</code></pre>

Si vous définissez `AUTOMATED_REGISTER="false"`, cette séquence de commandes permet de migrer vers hn1-db-1 le nœud principal SAP HANA et le groupe qui contient l’adresse IP virtuelle.

Une fois la migration terminée, la sortie crm_mon -r ressemble à ce qui suit

<pre><code>Online: [ hn1-db-0 hn1-db-1 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started hn1-db-1
 Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     Started: [ hn1-db-0 hn1-db-1 ]
 Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     Masters: [ hn1-db-1 ]
     Stopped: [ hn1-db-0 ]
 Resource Group: g_ip_HN1_HDB03
     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1

Failed Actions:
* rsc_SAPHana_HN1_HDB03_start_0 on hn1-db-0 'not running' (7): call=84, status=complete, exitreason='none',
    last-rc-change='Mon Aug 13 11:31:37 2018', queued=0ms, exec=2095ms
</code></pre>

La ressource SAP HANA sur hn1-db-0 ne peut pas démarrer en tant que ressource secondaire. Dans ce cas, configurez l’instance HANA comme étant secondaire en exécutant cette commande :

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr <b>03</b> -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>
</code></pre>

La migration crée des contraintes d’emplacement qui doivent être de nouveau supprimées :

<pre><code># Switch back to root and clean up the failed state
exit
hn1-db-0:~ # crm resource unmigrate msl_SAPHana_<b>HN1</b>_HDB<b>03</b>
</code></pre>

Vous devez également nettoyer l’état de la ressource du nœud secondaire :

<pre><code>hn1-db-0:~ # crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

Surveillez l’état de la ressource HANA à l’aide de crm_mon -r. Une fois que HANA est lancé sur hn1-db-0, la sortie doit ressembler à ce qui suit

<pre><code>Online: [ hn1-db-0 hn1-db-1 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started hn1-db-1
 Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     Started: [ hn1-db-0 hn1-db-1 ]
 Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     Masters: [ hn1-db-1 ]
     Slaves: [ hn1-db-0 ]
 Resource Group: g_ip_HN1_HDB03
     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
</code></pre>

### <a name="test-the-azure-fencing-agent-not-sbd"></a>Tester l’agent de délimitation Azure (pas SBD)

Vous pouvez tester l’installation de l’agent de délimitation Azure en désactivant l’interface réseau sur le nœud hn1-db-0 :

<pre><code>sudo ifdown eth0
</code></pre>

La machine virtuelle devrait maintenant redémarrer ou s’arrêter en fonction de la configuration de votre cluster.
Si vous désactivez le paramètre `stonith-action`, la machine virtuelle est arrêtée et les ressources sont migrées vers la machine virtuelle en cours d’exécution.

Après avoir redémarré la machine virtuelle, la ressource SAP HANA ne peut pas démarrer en tant que ressource secondaire si vous définissez `AUTOMATED_REGISTER="false"`. Dans ce cas, configurez l’instance HANA comme étant secondaire en exécutant cette commande :

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
sapcontrol -nr <b>03</b> -function StopWait 600 10
hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>

# Switch back to root and clean up the failed state
exit
crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

### <a name="test-sbd-fencing"></a>Tester l’isolation SBD

Vous pouvez tester la configuration SBD en arrêtant le processus inquisiteur.

<pre><code>hn1-db-0:~ # ps aux | grep sbd
root       1912  0.0  0.0  85420 11740 ?        SL   12:25   0:00 sbd: inquisitor
root       1929  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-360014056f268462316e4681b704a9f73 - slot: 0 - uuid: 7b862dba-e7f7-4800-92ed-f76a4e3978c8
root       1930  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-360014059bc9ea4e4bac4b18808299aaf - slot: 0 - uuid: 5813ee04-b75c-482e-805e-3b1e22ba16cd
root       1931  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-36001405b8dddd44eb3647908def6621c - slot: 0 - uuid: 986ed8f8-947d-4396-8aec-b933b75e904c
root       1932  0.0  0.0  90524 16656 ?        SL   12:25   0:00 sbd: watcher: Pacemaker
root       1933  0.0  0.0 102708 28260 ?        SL   12:25   0:00 sbd: watcher: Cluster
root      13877  0.0  0.0   9292  1572 pts/0    S+   12:27   0:00 grep sbd

hn1-db-0:~ # kill -9 1912
</code></pre>

Le nœud de cluster hn1-db-0 doit être redémarré. Il se peut que le service Pacemaker ne démarre pas ensuite. Veillez bien à le redémarrer.

### <a name="test-a-manual-failover"></a>Tester un basculement manuel

Vous pouvez tester un basculement manuel en arrêtant le service `pacemaker` sur le nœud hn1-db-0 :

<pre><code>service pacemaker stop
</code></pre>

Après le basculement, vous pouvez démarrer de nouveau le service. Si vous définissez `AUTOMATED_REGISTER="false"`, la ressource SAP HANA sur le nœud hn1-db-0 ne peut pas démarrer en tant que ressource secondaire. Dans ce cas, configurez l’instance HANA comme étant secondaire en exécutant cette commande :

<pre><code>service pacemaker start
su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
sapcontrol -nr <b>03</b> -function StopWait 600 10
hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b> 

# Switch back to root and clean up the failed state
exit
crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

### <a name="suse-tests"></a>Tests SUSE

> [!IMPORTANT]
> Vérifiez que le système d’exploitation que vous sélectionnez est certifié SAP pour SAP HANA sur les types de machine virtuelle spécifiques que vous utilisez. La liste des types de machine virtuelle certifiés SAP HANA et des systèmes d’exploitation correspondants peut être consultée dans la page [SAP HANA Certified IaaS Platforms](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Cliquez sur les détails du type de machine virtuelle listé pour obtenir la liste complète des versions de système d’exploitation SAP HANA prises en charge pour le type de machine virtuelle spécifique

Exécutez tous les cas de test répertoriés dans le guide « SAP HANA SR Performance Optimized Scenario » (Scénario à performances optimisées AP HANA SR) ou « SAP HANA SR Cost Optimized Scenario » (Scénario à coût optimisé SAP HANA SR), selon votre cas d’utilisation. Vous pouvez trouver les guides sur la [page des meilleures pratiques SLES pour SAP][sles-for-sap-bp].

Les tests suivants sont une copie des tests décrits dans le guide « SAP HANA SR Performance Optimized Scenario SUSE Linux Enterprise Server for SAP Applications 12 SP1 » (Scénario à performances optimisées de réplication système de SAP HANA SUSE Linux Enterprise Server for SAP Applications 12 SP1). Pour obtenir une version actualisée, lisez toujours le guide à proprement parler. Assurez-vous toujours de la synchronisation de HANA avant de commencer le test et vérifiez que la configuration du service Pacemaker est correcte.

Dans les descriptions de test suivantes, nous supposons que PREFER_SITE_TAKEOVER=« true » et AUTOMATED_REGISTER=« false ».
REMARQUE :  Les tests suivants doivent être exécutés de façon séquentielle et dépendent de l’état de sortie des tests précédents.

1. TEST 1 : ARRÊTER LA BASE DE DONNÉES PRIMAIRE SUR LE NŒUD 1

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Exécutez les commandes suivantes en tant que <hanasid\>adm sur le nœud hn1-db-0 :

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker doit détecter l’instance HANA arrêtée et basculer vers l’autre nœud. Une fois le basculement terminé, l’instance HANA sur le nœud hn1-db-0 est arrêtée, car Pacemaker n’enregistre pas automatiquement le nœud comme étant une base de données HANA secondaire.

   Exécutez les commandes suivantes pour enregistrer le nœud hn1-db-0 comme secondaire et nettoyer la ressource ayant échoué.

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

1. TEST 2 : ARRÊTER LA BASE DE DONNÉES PRIMAIRE SUR LE NŒUD 2

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

   Exécutez les commandes suivantes en tant que <hanasid\>adm sur le nœud hn1-db-1 :

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker doit détecter l’instance HANA arrêtée et basculer vers l’autre nœud. Une fois le basculement terminé, l’instance HANA sur le nœud hn1-db-1 est arrêtée, car Pacemaker n’enregistre pas automatiquement le nœud comme étant une base de données HANA secondaire.

   Exécutez les commandes suivantes pour enregistrer le nœud hn1-db-1 comme étant secondaire et nettoyer la ressource ayant échoué.

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. TEST 3 : BLOQUER LA BASE DE DONNÉES PRIMAIRE SUR LE NŒUD

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Exécutez les commandes suivantes en tant que <hanasid\>adm sur le nœud hn1-db-0 :

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>
   
   Pacemaker doit détecter l’instance HANA arrêtée et basculer vers l’autre nœud. Une fois le basculement terminé, l’instance HANA sur le nœud hn1-db-0 est arrêtée, car Pacemaker n’enregistre pas automatiquement le nœud comme étant une base de données HANA secondaire.

   Exécutez les commandes suivantes pour enregistrer le nœud hn1-db-0 comme secondaire et nettoyer la ressource ayant échoué.

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

1. TEST 4 : BLOQUER LA BASE DE DONNÉES PRIMAIRE SUR LE NŒUD 2

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

   Exécutez les commandes suivantes en tant que <hanasid\>adm sur le nœud hn1-db-1 :

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>

   Pacemaker doit détecter l’instance HANA arrêtée et basculer vers l’autre nœud. Une fois le basculement terminé, l’instance HANA sur le nœud hn1-db-1 est arrêtée, car Pacemaker n’enregistre pas automatiquement le nœud comme étant une base de données HANA secondaire.

   Exécutez les commandes suivantes pour enregistrer le nœud hn1-db-1 comme étant secondaire et nettoyer la ressource ayant échoué.

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. TEST 5 : BLOQUER LE NŒUD DU SITE PRINCIPAL (NŒUD 1)

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Exécutez les commandes suivantes en tant qu’utilisateur root sur le nœud hn1-db-0 :

   <pre><code>hn1-db-0:~ #  echo 'b' > /proc/sysrq-trigger
   </code></pre>

   Pacemaker doit détecter le nœud de cluster arrêté et l’isoler. Une fois le nœud isolé, Pacemaker lance une prise de contrôle de l’instance HANA. Pacemaker ne démarre pas automatiquement lorsque le nœud isolé est redémarré.

   Exécutez les commandes suivantes pour démarrer Pacemaker, nettoyer les messages SBD du nœud hn1-db-0, enregistrer le nœud hn1-db-0 comme secondaire, et nettoyer la ressource ayant échoué.

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-0:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-0:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-0 clear
   
   hn1-db-0:~ # systemctl start pacemaker
   
   # run as &lt;hanasid&gt;adm
   hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

1. TEST 6 : BLOQUER LE NŒUD DU SITE SECONDAIRE (NŒUD 2)

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

   Exécutez les commandes suivantes en tant qu’utilisateur root sur le nœud hn1-db-1 :

   <pre><code>hn1-db-1:~ #  echo 'b' > /proc/sysrq-trigger
   </code></pre>

   Pacemaker doit détecter le nœud de cluster arrêté et l’isoler. Une fois le nœud isolé, Pacemaker lance une prise de contrôle de l’instance HANA. Pacemaker ne démarre pas automatiquement lorsque le nœud isolé est redémarré.

   Exécutez les commandes suivantes pour démarrer Pacemaker, nettoyer les messages SBD du nœud hn1-db-1, enregistrer le nœud hn1-db-1 comme secondaire et nettoyer la ressource ayant échoué.

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-1:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-1:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-1 clear
   
   hn1-db-1:~ # systemctl start pacemaker
   
   # run as &lt;hanasid&gt;adm
   hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. TEST 7 : ARRÊTER LA BASE DE DONNÉES SECONDAIRE SUR LE NŒUD 2

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Exécutez les commandes suivantes en tant que <hanasid\>adm sur le nœud hn1-db-1 :

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker détecte l’instance HANA arrêtée et indique que la ressource a échoué sur le nœud hn1-db-1. Pacemaker doit redémarrer automatiquement l’instance HANA. Exécutez la commande suivante pour nettoyer l’état d’échec.

   <pre><code># run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. TEST 8 : BLOQUER LA BASE DE DONNÉES SECONDAIRE SUR LE NŒUD 2

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Exécutez les commandes suivantes en tant que <hanasid\>adm sur le nœud hn1-db-1 :

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>

   Pacemaker détecte l’instance HANA arrêté et indique que la ressource a échoué sur le nœud hn1-db-1. Exécutez la commande suivante pour nettoyer l’état d’échec. Pacemaker doit alors redémarrer automatiquement l’instance HANA.

   <pre><code># run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. TEST 9 : BLOQUER LE NŒUD DU SITE SECONDAIRE (NŒUD 2) EXÉCUTANT LA BASE DE DONNÉES HANA SECONDAIRE

   État des ressources avant le début du test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Exécutez les commandes suivantes en tant qu’utilisateur root sur le nœud hn1-db-1 :

   <pre><code>hn1-db-1:~ # echo b > /proc/sysrq-trigger
   </code></pre>

   Pacemaker doit détecter le nœud de cluster arrêté et l’isoler. Pacemaker ne démarre pas automatiquement lorsque le nœud isolé est redémarré.

   Exécutez les commandes suivantes pour lancer Pacemaker, nettoyer les messages SBD du nœud hn1-db-1 et nettoyer la ressource ayant échoué.

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-1:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-1:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-1 clear
   
   hn1-db-1:~ # systemctl start pacemaker  
   
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   État des ressources après le test :

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

## <a name="next-steps"></a>Étapes suivantes

* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
