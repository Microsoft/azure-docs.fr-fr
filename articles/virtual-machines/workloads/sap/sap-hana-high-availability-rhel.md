---
title: Haute disponibilité de SAP HANA sur les machines virtuelles Azure qui exécutent RHEL | Microsoft Docs
description: Créer une haute disponibilité de SAP HANA sur des machines virtuelles Azure.
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
ms.openlocfilehash: 277ed8ad5f9888daa911cb3b5c7dcf00fd285bf4
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489154"
---
# <a name="high-availability-of-sap-hana-on-azure-vms-on-red-hat-enterprise-linux"></a>Haute disponibilité de SAP HANA sur les machines virtuelles Azure dans le système Red Hat Enterprise Linux

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
[2292690]:https://launchpad.support.sap.com/#/notes/2292690
[2455582]:https://launchpad.support.sap.com/#/notes/2455582
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879

[sap-swcenter]:https://launchpad.support.sap.com/#/softwarecenter
[template-multisid-db]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json

En développement local, vous pouvez utiliser soit la réplication système HANA soit un stockage partagé pour établir une haute disponibilité pour SAP HANA.
Sur les machines virtuelles Azure, la réplication système SAP HANA dans Azure est la seule fonction de haute disponibilité prise en charge actuellement.
La réplication SAP HANA se compose d’un nœud principal et d’au moins un nœud secondaire. Les modifications apportées aux données sur le nœud principal sont répliquées vers les nœuds secondaires de manière synchrone ou asynchrone.

Cet article décrit comment déployer et configurer les machines virtuelles, installer l’infrastructure de cluster, et installer et configurer la réplication système SAP HANA.
Les exemples de configuration et les commandes d’installation utilisent le numéro d’instance **03** et l’ID système HANA **HN1**.

Commencez par lire les notes et publications SAP suivantes :

* Note SAP [1928533], qui contient :
  * La liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP.
  * Des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure.
  * Les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données.
  * La version du noyau SAP requise pour Windows et Linux sur Microsoft Azure.
* La note SAP [2015553] répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure.
* La note SAP [2002167] recommande des paramètres de système d’exploitation pour Red Hat Enterprise Linux
* La note SAP [2009879] conseille sur SAP HANA pour Red Hat Enterprise Linux
* La note SAP [2178632] contient des informations détaillées sur toutes les métriques de surveillance rapportées pour SAP sur Azure.
* La note SAP [2191498] contient la version requise de l’agent hôte SAP pour Linux sur Azure.
* La note SAP [2243692] contient des informations sur les licences SAP sur Linux dans Azure.
* La note SAP [1999351] contient des informations de dépannage supplémentaires pour l’extension d’analyse Azure améliorée pour SAP.
* Le [WIKI de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contient toutes les notes SAP requises pour Linux.
* [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP sur Linux (cet article)][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP sur Linux][dbms-guide]
* [Réplication de système SAP HANA dans un cluster Pacemaker](https://access.redhat.com/articles/3004101)
* Documentation RHEL générale
  * [Vue d’ensemble des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administration des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Référence des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
* Documentation RHEL spécifique à Azure :
  * [Stratégies de prise en charge des clusters à haute disponibilité RHEL - Machines virtuelles Microsoft Azure en tant que membres du cluster](https://access.redhat.com/articles/3131341)
  * [Installation et configuration d’un cluster à haute disponibilité Red Hat Enterprise Linux 7.4 (et versions ultérieures) sur Microsoft Azure](https://access.redhat.com/articles/3252491)
  * [Installation de SAP HANA sur Red Hat Enterprise Linux pour une utilisation dans Microsoft Azure](https://access.redhat.com/solutions/3193782)

## <a name="overview"></a>Vue d’ensemble

Pour atteindre la haute disponibilité, SAP HANA est installé sur deux machines virtuelles. Les données sont répliquées à l’aide de la réplication de système HANA.

![Présentation de la haute disponibilité de SAP HANA](./media/sap-hana-high-availability-rhel/ha-hana.png)

La configuration de la réplication du système SAP HANA utilise un nom d’hôte virtuel dédié et des adresses IP virtuelles. Sur Azure, un équilibreur de charge est nécessaire pour utiliser une adresse IP virtuelle. La liste suivante illustre la configuration de l’équilibreur de charge :

* Configuration front-end : Adresse IP 10.0.0.13 pour hn1-db
* Configuration back-end : Connecté aux interfaces réseau principales de toutes les machines virtuelles qui doivent faire partie de la réplication de système HANA
* Port de la sonde : Port 62503
* Règles d’équilibrage de charge : 30313 TCP, 30315 TCP, 30317 TCP, 30340 TCP, 30341 TCP, 30342 TCP

## <a name="deploy-for-linux"></a>Déploiement pour Linux

La Place de marché Azure contient une image de Red Hat Enterprise Linux 7.4 for SAP HANA que vous pouvez utiliser pour déployer de nouvelles machines virtuelles.

### <a name="deploy-with-a-template"></a>Déployer les mises à niveau avec un modèle

Vous pouvez utiliser un des modèles de démarrage rapide disponibles sur GitHub pour déployer toutes les ressources nécessaires. Le modèle déploie les machines virtuelles, l’équilibrage de charge, le groupe à haute disponibilité, etc.
Suivez ces étapes pour déployer le modèle :

1. Ouvrez le [modèle de base de données][template-multisid-db] dans le portail Microsoft Azure.
1. Entrez les paramètres suivants :
    * **ID du système SAP** : Entrez l’ID du système SAP que vous souhaitez installer. Cet ID est utilisé comme préfixe pour les ressources déployées.
    * **Type de système d’exploitation** : Sélectionnez l’une des distributions Linux. Dans cet exemple, sélectionnez **RHEL 7**.
    * **Type de base de données** : sélectionnez **HANA**.
    * **Taille du système SAP** : entrez le nombre de SAP que le nouveau système va fournir. Si vous ne savez pas combien de SAP sont exigés par le système, demandez à votre partenaire technologique SAP ou un intégrateur système.
    * **Disponibilité du système** : Sélectionnez la haute disponibilité **(HA)** .
    * **Nom d’utilisateur administrateur, mot de passe d’administrateur ou clé SSH** : Un utilisateur pouvant être utilisé pour se connecter à la machine est créé.
    * **ID de sous-réseau** : Si vous voulez déployer la machine virtuelle dans un réseau virtuel existant où vous avez défini un sous-réseau auquel la machine virtuelle doit être attribuée, nommez l’ID de ce sous-réseau spécifique. L’identifiant se présente généralement sous la forme **/subscriptions/\<subscription ID>/resourceGroups/\<resource group name>/providers/Microsoft.Network/virtualNetworks/\<virtual network name>/subnets/\<subnet name>** . Laissez-le vide si vous souhaitez créer un réseau virtuel.

### <a name="manual-deployment"></a>Déploiement manuel

1. Créez un groupe de ressources.
1. Créez un réseau virtuel.
1. Créer un groupe à haute disponibilité.  
   Définir le domaine de mise à jour maximal.
1. Créer un équilibrage de charge (interne). Nous vous recommandons [Standard Load Balancer](../../../load-balancer/load-balancer-overview.md).
   * Sélectionnez le réseau virtuel créé à l’étape 2.
1. Créez la machine virtuelle 1.  
   À utiliser pour Red Hat Enterprise Linux for SAP HANA, version 7.4 au moins. Cet exemple utilise l’image Red Hat Enterprise Linux 7.4 for SAP HANA <https://portal.azure.com/#create/RedHat.RedHatEnterpriseLinux75forSAP-ARM> Sélectionnez le groupe à haute disponibilité créé à l’étape 3.
1. Créez la machine virtuelle 2.  
   À utiliser pour Red Hat Enterprise Linux for SAP HANA, version 7.4 au moins. Cet exemple utilise l’image Red Hat Enterprise Linux 7.4 for SAP HANA <https://portal.azure.com/#create/RedHat.RedHatEnterpriseLinux75forSAP-ARM> Sélectionnez le groupe à haute disponibilité créé à l’étape 3.
1. Ajoutez des disques de données.

> [!IMPORTANT]
> Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d’informations, consultez [Limitations d’équilibreur de charge Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.    

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
   1. Configurez l’équilibrage de charge. Commencez par créer un pool d’adresses IP frontales :

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

## <a name="install-sap-hana"></a>Installer SAP HANA

Les étapes de cette section utilisent les préfixes suivants :

* **[A]**  : l’étape s’applique à tous les nœuds.
* **[1]**  : l’étape ne s’applique qu’au nœud 1.
* **[2]**  : l’étape s’applique uniquement au nœud 2 du cluster Pacemaker.

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

1. **[A]** RHEL pour la configuration HANA

   Configurez RHEL tel que décrit dans <https://access.redhat.com/solutions/2447641> et dans les notes SAP suivantes :  
   - [2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/2292690) (SAP HANA DB : Paramètres de système d’exploitation recommandés pour RHEL 7)
   - [2777782 – Base de données SAP HANA : paramètres de système d’exploitation recommandés pour RHEL 8](https://launchpad.support.sap.com/#/notes/2777782)
   - [2455582 – Linux : Exécution d’applications SAP compilées avec GCC 6.x](https://launchpad.support.sap.com/#/notes/2455582)
   - [2593824 – Linux : Exécution d’applications SAP compilées avec GCC 7.x](https://launchpad.support.sap.com/#/notes/2593824) 
   - [2886607 – Linux : Exécution d’applications SAP compilées avec GCC 9.x](https://launchpad.support.sap.com/#/notes/2886607)

1. **[A]** Installer SAP HANA

   Pour installer la réplication de système SAP HANA, suivez les étapes de <https://access.redhat.com/articles/3004101>.

   * Exécutez le programme **hdblcm** depuis le DVD HANA. Entrez les valeurs suivantes à l’invite :
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

1. **[A]** Configurer le pare-feu

   Créez la règle de pare-feu pour le port d’analyse de l’équilibreur de charge Azure.

   <pre><code>sudo firewall-cmd --zone=public --add-port=625<b>03</b>/tcp
   sudo firewall-cmd --zone=public --add-port=625<b>03</b>/tcp --permanent
   </code></pre>

## <a name="configure-sap-hana-20-system-replication"></a>Configurer la réplication de système SAP HANA 2.0

Les étapes de cette section utilisent les préfixes suivants :

* **[A]**  : l’étape s’applique à tous les nœuds.
* **[1]**  : l’étape ne s’applique qu’au nœud 1.
* **[2]**  : l’étape s’applique uniquement au nœud 2 du cluster Pacemaker.

1. **[A]** Configurer le pare-feu

   Créez des règles de pare-feu pour autoriser le trafic de réplication de système HANA et client. Les ports indispensables sont répertoriés ici : [Ports TCP/IP de tous les produits SAP](https://help.sap.com/viewer/ports). Les commandes suivantes sont simplement un exemple pour autoriser le trafic de réplication de système HANA 2.0 et client vers la base de données SYSTEMDB, HN1 et NW1.

   <pre><code>sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40302/tcp
   sudo firewall-cmd --zone=public --add-port=40301/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40301/tcp
   sudo firewall-cmd --zone=public --add-port=40307/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40307/tcp
   sudo firewall-cmd --zone=public --add-port=40303/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40303/tcp
   sudo firewall-cmd --zone=public --add-port=40340/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40340/tcp
   sudo firewall-cmd --zone=public --add-port=30340/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30340/tcp
   sudo firewall-cmd --zone=public --add-port=30341/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30341/tcp
   sudo firewall-cmd --zone=public --add-port=30342/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30342/tcp
   </code></pre>

1. **[1]** Créer une base de données locataire.

   Si vous utilisez SAP HANA 2.0 ou MDC, créez une base de données locataire pour votre système SAP NetWeaver. Remplacez **NW1** par le SID de votre système SAP.

   Exécuter en tant que < hanasid\>adm la commande suivante :

   <pre><code>hdbsql -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> -d SYSTEMDB 'CREATE DATABASE <b>NW1</b> SYSTEM USER PASSWORD "<b>passwd</b>"'
   </code></pre>

1. **[1]** Configurez la réplication de système sur le premier nœud :

   Sauvegardez les bases de données en tant que < hanasid\>adm :

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
    
   Inscrivez le second nœud pour démarrer la réplication de système. Exécutez la commande suivante en tant que <hanasi\>adm :

   <pre><code>sapcontrol -nr <b>03</b> -function StopWait 600 10
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>
   </code></pre>

1. **[1]** Vérifier l’état de la réplication

   Vérifiez l’état de la réplication et attendez que toutes les bases de données soient synchronisées. Si l’état demeure INCONNU, vérifiez vos paramètres de pare-feu.

   <pre><code>sudo su - <b>hn1</b>adm -c "python /usr/sap/<b>HN1</b>/HDB<b>03</b>/exe/python_support/systemReplicationStatus.py"
   # | Database | Host     | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary | Secondary | Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
   # |          |          |       |              |           |         |           | Host      | Port      | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
   # | -------- | -------- | ----- | ------------ | --------- | ------- | --------- | --------- | --------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
   # | SYSTEMDB | <b>hn1-db-0</b> | 30301 | nameserver   |         1 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30301 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>HN1</b>      | <b>hn1-db-0</b> | 30307 | xsengine     |         2 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30307 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>NW1</b>      | <b>hn1-db-0</b> | 30340 | indexserver  |         2 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30340 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>HN1</b>      | <b>hn1-db-0</b> | 30303 | indexserver  |         3 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30303 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   #
   # status system replication site "2": ACTIVE
   # overall system replication status: ACTIVE
   #
   # Local System Replication State
   # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   #
   # mode: PRIMARY
   # site id: 1
   # site name: <b>SITE1</b>
   </code></pre>

## <a name="configure-sap-hana-10-system-replication"></a>Configurer la réplication de système SAP HANA 1.0

Les étapes de cette section utilisent les préfixes suivants :

* **[A]**  : l’étape s’applique à tous les nœuds.
* **[1]**  : l’étape ne s’applique qu’au nœud 1.
* **[2]**  : l’étape s’applique uniquement au nœud 2 du cluster Pacemaker.

1. **[A]** Configurer le pare-feu

   Créez des règles de pare-feu pour autoriser le trafic de réplication de système HANA et client. Les ports indispensables sont répertoriés ici : [Ports TCP/IP de tous les produits SAP](https://help.sap.com/viewer/ports). Les commandes suivantes sont un exemple pour autoriser la réplication de système HANA 2.0. Adaptez-le à votre installation de SAP HANA 1.0.

   <pre><code>sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40302/tcp
   </code></pre>

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

   Créez le site principal en tant que <hanasid\>adm :

   <pre><code>su - <b>hdb</b>adm
   hdbnsutil -sr_enable –-name=<b>SITE1</b>
   </code></pre>

1. **[2]** Configurez la réplication de système sur le nœud secondaire.

   Inscrivez le site secondaire en tant que <hanasid\>adm :

   <pre><code>HDB stop
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>
   HDB start
   </code></pre>

## <a name="create-a-pacemaker-cluster"></a>Créez un cluster Pacemaker

Suivez les étapes décrites sur la page [Configurer Pacemaker sur Red Hat Enterprise Linux dans Azure](high-availability-guide-rhel-pacemaker.md) pour créer un cluster Pacemaker de base pour ce serveur HANA.

## <a name="create-sap-hana-cluster-resources"></a>Créer les ressources de cluster SAP HANA

Installez les agents de la ressource SAP HANA sur **tous les nœuds**. N’oubliez pas d’activer un référentiel qui contient le package. Vous ne devez pas ajouter des répertoires supplémentaires si vous utilisez l’image en haute disponibilité RHEL 8.x.  

<pre><code># Enable repository that contains SAP HANA resource agents
sudo subscription-manager repos --enable="rhel-sap-hana-for-rhel-7-server-rpms"
   
sudo yum install -y resource-agents-sap-hana
</code></pre>

Tout d’abord, créez la topologie HANA. Exécutez les commandes suivantes sur l’un des nœuds du cluster Pacemaker :

<pre><code>sudo pcs property set maintenance-mode=true

# Replace the bold string with your instance number and HANA system ID
sudo pcs resource create SAPHanaTopology_<b>HN1</b>_<b>03</b> SAPHanaTopology SID=<b>HN1</b> InstanceNumber=<b>03</b> \
op start timeout=600 op stop timeout=300 op monitor interval=10 timeout=600 \
clone clone-max=2 clone-node-max=1 interleave=true
</code></pre>

Ensuite, créez les ressources HANA.

> [!NOTE]
> Cet article contient des références au terme *esclave*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

Si vous créez un cluster sur **RHEL 7.x**, utilisez les commandes suivantes :  

<pre><code># Replace the bold string with your instance number, HANA system ID, and the front-end IP address of the Azure load balancer.
#
sudo pcs resource create SAPHana_<b>HN1</b>_<b>03</b> SAPHana SID=<b>HN1</b> InstanceNumber=<b>03</b> PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
op start timeout=3600 op stop timeout=3600 \
op monitor interval=61 role="Slave" timeout=700 \
op monitor interval=59 role="Master" timeout=700 \
op promote timeout=3600 op demote timeout=3600 \
master notify=true clone-max=2 clone-node-max=1 interleave=true

sudo pcs resource create vip_<b>HN1</b>_<b>03</b> IPaddr2 ip="<b>10.0.0.13</b>"
sudo pcs resource create nc_<b>HN1</b>_<b>03</b> azure-lb port=625<b>03</b>
sudo pcs resource group add g_ip_<b>HN1</b>_<b>03</b> nc_<b>HN1</b>_<b>03</b> vip_<b>HN1</b>_<b>03</b>

sudo pcs constraint order SAPHanaTopology_<b>HN1</b>_<b>03</b>-clone then SAPHana_<b>HN1</b>_<b>03</b>-master symmetrical=false
sudo pcs constraint colocation add g_ip_<b>HN1</b>_<b>03</b> with master SAPHana_<b>HN1</b>_<b>03</b>-master 4000

sudo pcs property set maintenance-mode=false
</code></pre>

Si vous créez un cluster sur **RHEL 8.x**, utilisez les commandes suivantes :  

<pre><code># Replace the bold string with your instance number, HANA system ID, and the front-end IP address of the Azure load balancer.
#
sudo pcs resource create SAPHana_<b>HN1</b>_<b>03</b> SAPHana SID=<b>HN1</b> InstanceNumber=<b>03</b> PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
op start timeout=3600 op stop timeout=3600 \
op monitor interval=61 role="Slave" timeout=700 \
op monitor interval=59 role="Master" timeout=700 \
op promote timeout=3600 op demote timeout=3600 \
promotable meta notify=true clone-max=2 clone-node-max=1 interleave=true

sudo pcs resource create vip_<b>HN1</b>_<b>03</b> IPaddr2 ip="<b>10.0.0.13</b>"
sudo pcs resource create nc_<b>HN1</b>_<b>03</b> azure-lb port=625<b>03</b>
sudo pcs resource group add g_ip_<b>HN1</b>_<b>03</b> nc_<b>HN1</b>_<b>03</b> vip_<b>HN1</b>_<b>03</b>

sudo pcs constraint order SAPHanaTopology_<b>HN1</b>_<b>03</b>-clone then SAPHana_<b>HN1</b>_<b>03</b>-clone symmetrical=false
sudo pcs constraint colocation add g_ip_<b>HN1</b>_<b>03</b> with master SAPHana_<b>HN1</b>_<b>03</b>-clone 4000

sudo pcs property set maintenance-mode=false
</code></pre>

Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.

> [!NOTE]
> Les délais d’expiration de la configuration ci-dessus sont simplement des exemples et doivent être adaptés à la configuration HANA spécifique. Par exemple, vous devrez peut-être augmenter le délai de démarrage, si le démarrage de la base de données SAP HANA prend plus de temps.  

<pre><code>sudo pcs status

# Online: [ hn1-db-0 hn1-db-1 ]
#
# Full list of resources:
#
# azure_fence     (stonith:fence_azure_arm):      Started hn1-db-0
#  Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
#      Started: [ hn1-db-0 hn1-db-1 ]
#  Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
#      Masters: [ hn1-db-0 ]
#      Slaves: [ hn1-db-1 ]
#  Resource Group: g_ip_HN1_03
#      nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
#      vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

## <a name="test-the-cluster-setup"></a>Tester la configuration du cluster

Cette section explique comment vous pouvez tester votre configuration. Avant de commencer le test, assurez-vous que Pacemaker ne comporte aucune action ayant échoué (via pcs status), qu’il n’existe aucune contrainte d’emplacement inattendue (par exemple, les reliquats d’un test de migration) et que HANA est en état de synchronisation, par exemple avec systemReplicationStatus :

<pre><code>[root@hn1-db-0 ~]# sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
</code></pre>

### <a name="test-the-migration"></a>Tester la migration

État des ressources avant le début du test :

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Vous pouvez migrer le nœud principal SAP HANA en exécutant la commande suivante :

<pre><code># On RHEL <b>7.x</b> 
[root@hn1-db-0 ~]# pcs resource move SAPHana_HN1_03-master
# On RHEL <b>8.x</b>
[root@hn1-db-0 ~]# pcs resource move SAPHana_HN1_03-clone --master
</code></pre>

Si vous définissez `AUTOMATED_REGISTER="false"`, cette commande permet de migrer vers hn1-db-1 le nœud principal SAP HANA et le groupe contenant l’adresse IP virtuelle.

Une fois la migration terminée, la sortie « sudo pcs status » ressemble à ce qui suit.

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Stopped: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

La ressource SAP HANA sur hn1-db-0 est arrêtée. Dans ce cas, configurez l’instance HANA comme étant secondaire en exécutant cette commande :

<pre><code>[root@hn1-db-0 ~]# su - hn1adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr 03 -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMod
e=sync --name=SITE1
</code></pre>

La migration crée des contraintes d’emplacement qui doivent être de nouveau supprimées :

<pre><code># Switch back to root
exit
[root@hn1-db-0 ~]# pcs resource clear SAPHana_HN1_03-master
</code></pre>

Surveillez l’état de la ressource HANA à l’aide de « pcs status ». Une fois que HANA est lancé sur hn1-db-0, la sortie doit ressembler à ce qui suit

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

### <a name="test-the-azure-fencing-agent"></a>Tester l’agent de délimitation Azure

> [!NOTE]
> Cet article contient des références au terme *esclave*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.  

État des ressources avant le début du test :

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

Vous pouvez tester l’installation de l’agent de délimitation Azure en désactivant l’interface réseau sur le nœud sur lequel SAP HANA s’exécute en tant que nœud principal.
Consultez [l’article de la base de connaissances Red Hat 79523](https://access.redhat.com/solutions/79523) pour savoir comment simuler une panne réseau. Dans cet exemple, nous utilisons le script net_breaker pour bloquer tout accès au réseau.

<pre><code>[root@hn1-db-1 ~]# sh ./net_breaker.sh BreakCommCmd 10.0.0.6
</code></pre>

La machine virtuelle devrait maintenant redémarrer ou s’arrêter en fonction de la configuration de votre cluster.
Si vous désactivez le paramètre `stonith-action`, la machine virtuelle est arrêtée et les ressources sont migrées vers la machine virtuelle en cours d’exécution.

Après avoir redémarré la machine virtuelle, la ressource SAP HANA ne peut pas démarrer en tant que ressource secondaire si vous définissez `AUTOMATED_REGISTER="false"`. Dans ce cas, configurez l’instance HANA comme étant secondaire en exécutant cette commande :

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> sapcontrol -nr <b>03</b> -function StopWait 600 10
hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>

# Switch back to root and clean up the failed state
exit
# On RHEL <b>7.x</b>
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03-master
# On RHEL <b>8.x</b>
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03 node=&lt;hostname on which the resource needs to be cleaned&gt;
</code></pre>

État des ressources après le test :

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

### <a name="test-a-manual-failover"></a>Tester un basculement manuel

État des ressources avant le début du test :

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Vous pouvez tester un basculement manuel en arrêtant le cluster sur le nœud hn1-db-0 :

<pre><code>[root@hn1-db-0 ~]# pcs cluster stop
</code></pre>

Après le basculement, redémarrez le cluster. Si vous définissez `AUTOMATED_REGISTER="false"`, la ressource SAP HANA sur le nœud hn1-db-0 ne peut pas démarrer en tant que ressource secondaire. Dans ce cas, configurez l’instance HANA comme étant secondaire en exécutant cette commande :

<pre><code>[root@hn1-db-0 ~]# pcs cluster start
[root@hn1-db-0 ~]# su - hn1adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr 03 -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>

# Switch back to root and clean up the failed state
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> exit
# On RHEL <b>7.x</b>
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03-master
# On RHEL <b>8.x</b>
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03 node=&lt;hostname on which the resource needs to be cleaned&gt;
</code></pre>

État des ressources après le test :

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
     Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

### <a name="test-a-manual-failover"></a>Tester un basculement manuel

État des ressources avant le début du test :

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Vous pouvez tester un basculement manuel en arrêtant le cluster sur le nœud hn1-db-0 :

<pre><code>[root@hn1-db-0 ~]# pcs cluster stop
</code></pre>


## <a name="next-steps"></a>Étapes suivantes

* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* [Configurations de stockage de machines virtuelles SAP HANA](./hana-vm-operations-storage.md)