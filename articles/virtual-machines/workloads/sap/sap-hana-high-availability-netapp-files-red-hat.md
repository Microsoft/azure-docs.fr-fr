---
title: Haute disponibilité du scale-up SAP HANA avec ANF sur RHEL | Microsoft Docs
description: Créer une haute disponibilité de SAP HANA avec ANF sur des machines virtuelles Azure.
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
ms.openlocfilehash: cfa68bde2462cefd6f690247cfd1e3bd2e3dbc74
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489205"
---
# <a name="high-availability-of-sap-hana-scale-up-with-azure-netapp-files-on-red-hat-enterprise-linux"></a>Haute disponibilité du scale-up SAP HANA avec Azure NetApp Files sur Red Hat Enterprise Linux

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

Cet article explique comment configurer la réplication du système SAP HANA dans le cadre d’un déploiement de scale-up lorsque les systèmes de fichiers HANA sont montés via NFS à l’aide d’Azure NetApp Files (ANF). Les exemples de configuration et les commandes d’installation utilisent le numéro d’instance **03** et l’ID système HANA **HN1**. La réplication SAP HANA se compose d’un nœud principal et d’au moins un nœud secondaire.

Lorsque les étapes de ce document sont marquées des préfixes suivants, la signification est la suivante :

- **[A]**  : l’étape s’applique à tous les nœuds.
- **[1]**  : l’étape ne s’applique qu’à node1.
- **[2]**  : l’étape ne s’applique qu’à node2.
 
Commencez par lire les notes et publications SAP suivantes :

- Note SAP [1928533](https://launchpad.support.sap.com/#/notes/1928533), qui contient :
    - La liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP.
    - Des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure.
    - Les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données.
    - La version du noyau SAP requise pour Windows et Linux sur Microsoft Azure.
- La note SAP [2015553](https://launchpad.support.sap.com/#/notes/2015553) répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure.
- La note SAP [405827](https://launchpad.support.sap.com/#/notes/405827) répertorie les systèmes de fichiers recommandés pour l’environnement HANA.
- La note SAP [2002167](https://launchpad.support.sap.com/#/notes/2002167) recommande des paramètres de système d’exploitation pour Red Hat Enterprise Linux.
- La note SAP [2009879](https://launchpad.support.sap.com/#/notes/2009879) conseille sur SAP HANA pour Red Hat Enterprise Linux.
- La note SAP [2178632](https://launchpad.support.sap.com/#/notes/2178632) contient des informations détaillées sur toutes les métriques de surveillance rapportées pour SAP sur Azure.
- La note SAP [2191498](https://launchpad.support.sap.com/#/notes/2191498) contient la version requise de l’agent hôte SAP pour Linux sur Azure.
- La note SAP [2243692](https://launchpad.support.sap.com/#/notes/2243692) contient des informations sur les licences SAP sur Linux dans Azure.
- La note SAP [1999351](https://launchpad.support.sap.com/#/notes/1999351) contient des informations de dépannage supplémentaires pour l’extension d’analyse Azure améliorée pour SAP.
- Le [wiki de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contient toutes les notes SAP nécessaires pour Linux.
- [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide]
- [Déploiement de machines virtuelles Azure pour SAP sur Linux][deployment-guide]
- [Déploiement SGBD de machines virtuelles Azure pour SAP sur Linux][dbms-guide]
- [Réplication du système SAP HANA dans un cluster Pacemaker](https://access.redhat.com/articles/3004101)
- Documentation RHEL générale
    - [Vue d’ensemble des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
    - [Administration des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
    - [Référence des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
    - [Configurer la réplication du système SAP HANA en scale-up dans un cluster Pacemaker lorsque les systèmes de fichiers HANA se trouvent sur des partages NFS](https://access.redhat.com/solutions/5156571)
- Documentation RHEL spécifique à Azure :
    - [Stratégies de prise en charge des clusters à haute disponibilité RHEL – Machines virtuelles Microsoft Azure en tant que membres du cluster](https://access.redhat.com/articles/3131341)
    - [Installation et configuration d’un cluster à haute disponibilité Red Hat Enterprise Linux 7.4 (et versions ultérieures) sur Microsoft Azure](https://access.redhat.com/articles/3252491)
    - [Installation de SAP HANA sur Red Hat Enterprise Linux pour une utilisation dans Microsoft Azure](https://access.redhat.com/solutions/3193782)
    - [Configurer la réplication du système SAP HANA en scale-up dans un cluster Pacemaker lorsque les systèmes de fichiers HANA se trouvent sur des partages NFS](https://access.redhat.com/solutions/5156571)
- [Applications SAP NetApp su Microsoft Azure avec Azure NetApp Files](https://www.netapp.com/us/media/tr-4746.pdf)

## <a name="overview"></a>Vue d’ensemble

Traditionnellement, dans un environnement de scale-up, tous les systèmes de fichiers pour SAP HANA sont montés à partir du stockage local. La configuration de la haute disponibilité pour la réplication du système SAP HANA sur Red Hat Enterprise Linux est publiée dans le guide [Configurer la réplication du système SAP HANA sur RHEL](./sap-hana-high-availability-rhel.md).

Afin d’atteindre la haute disponibilité SAP HANA du système de scale-up sur les partages NFS d’[Azure NetApp Files](../../../azure-netapp-files/index.yml), nous avons besoin d’une configuration supplémentaire des ressources dans le cluster, afin que les ressources HANA soient récupérées lorsqu’un nœud perd l’accès aux partages NFS sur ANF.  Le cluster gère les montages NFS, ce qui lui permet de surveiller l’intégrité des ressources. Les dépendances entre les montages de système de fichiers et les ressources SAP HANA sont appliquées.  

![Scale-up de la haute disponibilité SAP HANA sur ANF](./media/sap-hana-high-availability-rhel/sap-hana-scale-up-netapp-files-red-hat.png)

Les systèmes de fichiers SAP HANA sont montés sur des partages NFS à l’aide d’Azure NetApp Files sur chaque nœud. Les systèmes de fichiers /hana/data, /hana/log et /hana/shared sont uniques à chaque nœud. 

Monté sur node1 (**hanadb1**)

- 10.32.2.4:/**hanadb1**-data-mnt00001 sur /hana/data
- 10.32.2.4:/**hanadb1**-log-mnt00001 sur /hana/log
- 10.32.2.4:/**hanadb1**-shared-mnt00001 sur /hana/shared

Monté sur node2 (**hanadb2**)

- 10.32.2.4:/**hanadb2**-data-mnt00001 sur /hana/data
- 10.32.2.4:/**hanadb2**-log-mnt00001 sur /hana/log
- 10.32.2.4:/**hanadb2**-shared-mnt00001 sur /hana/shared

> [!NOTE]
> Les systèmes de fichiers /hana/shared, /hana/data et /hana/log ne sont pas partagés entre les deux nœuds. Chaque nœud de cluster a ses propres systèmes de fichiers distincts.   

La configuration de la réplication du système SAP HANA utilise un nom d’hôte virtuel dédié et des adresses IP virtuelles. Sur Azure, un équilibreur de charge est nécessaire pour utiliser une adresse IP virtuelle. La liste suivante illustre la configuration de l’équilibreur de charge :

- Configuration front-end : Adresse IP 10.32.0.10 pour hn1-db
- Configuration back-end : Connecté aux interfaces réseau principales de toutes les machines virtuelles qui doivent faire partie de la réplication de système HANA
- Port de la sonde : Port 62503
- Règles d’équilibrage de charge : 30313 TCP, 30315 TCP, 30317 TCP, 30340 TCP, 30341 TCP, 30342 TCP (si vous utilisez Basic Azure Load Balancer)  

## <a name="set-up-the-azure-netapp-file-infrastructure"></a>Configurer l’infrastructure Azure NetApp Files

Avant de poursuivre la configuration de l’infrastructure Azure NetApp Files, familiarisez-vous avec la [documentation Azure NetApp Files](../../../azure-netapp-files/index.yml).

Azure NetApp Files est disponible dans plusieurs [régions Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Vérifiez si la région Azure que vous avez sélectionnée est compatible avec Azure NetApp Files.

Pour plus d’informations sur la disponibilité d’Azure NetApp Files par région Azure, consultez [Disponibilité d’Azure NetApp Files par région Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all).

Avant de déployer Azure NetApp Files, demandez l’intégration à Azure NetApp Files en accédant à [Instructions d’inscription à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).

### <a name="deploy-azure-netapp-files-resources"></a>Déployer des ressources Azure NetApp Files

Les instructions suivantes supposent que vous avez déjà déployé votre [réseau virtuel Azure](../../../virtual-network/virtual-networks-overview.md). Les ressources Azure NetApp Files et les machines virtuelles, où les ressources Azure NetApp Files seront montées, doivent être déployées dans le même réseau virtuel Azure ou dans des réseaux virtuels Azure appairés.

1. Si vous n’avez pas encore déployé les ressources, demandez l’[intégration à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).

2. Créez un compte NetApp dans la région Azure sélectionnée en suivant les instructions de la page [Création d’un compte NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).

3.  Configurez un pool de capacité Azure NetApp Files en suivant les instructions de la page [Configuration d’un pool de capacité Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).

    L’architecture HANA présentée dans cet article utilise un seul pool de capacité Azure NetApp Files au niveau de service *Ultra*. Pour les charges de travail HANA sur Azure, nous vous recommandons d’utiliser un [niveau de service](../../../azure-netapp-files/azure-netapp-files-service-levels.md)*Ultra* ou *Premium* d’Azure NetApp Files.

4.  Déléguez un sous-réseau à Azure NetApp Files, comme décrit dans les instructions de la page [Déléguer un sous-réseau à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).

5.  Déployez des volumes Azure NetApp Files en suivant les instructions de la page [Créer un volume NFS pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).

    Lors du déploiement des volumes, veillez à sélectionner la version NFSv4.1. Déployez les volumes dans le sous-réseau Azure NetApp Files désigné. Les adresses IP des volumes Azure NetApp sont attribuées automatiquement.

    Gardez à l’esprit que les ressources Azure NetApp Files et les machines virtuelles Azure doivent être dans le même réseau virtuel Azure ou dans des réseaux virtuels Azure appairés. Par exemple, hanadb1-data-mnt00001, hanadb1-log-mnt00001 et ainsi de suite sont les noms de volume, et nfs://10.32.2.4/hanadb1-data-mnt00001, nfs://10.32.2.4/hanadb1-log-mnt00001 et ainsi de suite sont les chemin d’accès de fichiers pour les volumes Azure NetApp Files.
    
    Sur **hanadb1**

    - Volume hanadb1-data-mnt00001 (nfs://10.32.2.4:/hanadb1-data-mnt00001) 
    - Volume hanadb1-log-mnt00001 (nfs://10.32.2.4:/hanadb1-log-mnt00001)
    - Volume hanadb1-shared-mnt00001 (nfs://10.32.2.4:/hanadb1-shared-mnt00001)
    
    Sur **hanadb2**

    - Volume hanadb2-data-mnt00001 (nfs://10.32.2.4:/hanadb2-data-mnt00001)
    - Volume hanadb2-log-mnt00001 (nfs://10.32.2.4:/hanadb2-log-mnt00001)
    - Volume hanadb2-shared-mnt00001 (nfs://10.32.2.4:/hanadb2-shared-mnt00001)

### <a name="important-considerations"></a>Points importants à prendre en compte

Au fur et à mesure que vous créez votre instance Azure NetApp Files pour les systèmes de scale-up SAP HANA, tenez compte des points suivants :

- La taille de pool de capacité minimale est de 4 tébioctets (Tio).
- La taille de volume minimale est de 100 gibioctets (Gio).
- Azure NetApp Files et toutes les machines virtuelles où les volumes Azure NetApp Files seront montés doivent être déployés dans le même réseau virtuel Azure ou dans des [réseaux virtuels appairés](../../../virtual-network/virtual-network-peering-overview.md) dans la même région.
- Le réseau virtuel sélectionné doit avoir un sous-réseau délégué à Azure NetApp Files.
- Le débit d’un volume Azure NetApp Files est une fonction du quota de volume et du niveau de service, comme décrit dans [Niveau de service pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Lorsque vous dimensionnez les volumes HANA Azure NetApp, assurez-vous que le débit obtenu répond à la configuration requise pour HANA.
- Grâce à la [stratégie d’exportation](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md) d’Azure NetApp Files, vous pouvez contrôler les clients autorisés, le type d’accès (lecture et écriture, lecture seule, etc.).
- La fonctionnalité Azure NetApp Files ne tient pas encore compte des zones. Actuellement, la fonctionnalité n’est pas déployée dans toutes les zones de disponibilité d’une région Azure. Méfiez-vous de l’impact potentiel sur les temps de latence dans certaines régions Azure.

> [!IMPORTANT]
> Pour les charges de travail SAP HANA, une latence faible est critique. Collaborez avec votre représentant Microsoft pour vous assurer que les machines virtuelles et les volumes Azure NetApp Files sont déployés à proximité.

### <a name="sizing-of-hana-database-on-azure-netapp-files"></a>Dimensionnement de la base de données HANA sur Azure NetApp Files

Le débit d’un volume NetApp Azure Files est une fonction de la taille de volume et du niveau de service, comme décrit dans [Niveau de service pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md).

Lorsque vous concevez l’infrastructure pour SAP dans Azure, tenez compte des besoins de stockage minimaux de SAP, qui se traduisent par des caractéristiques de débit minimales :

- Lecture-écriture sur /hana/log de 250 mégaoctets par seconde (Mo/s) avec des tailles d’E/S de 1 Mo.
- Activité de lecture à un débit de 400 Mo/s au minimum pour /hana/data pour des tailles d’E/S de 16 Mo et 64 Mo.
- Activité d’écriture à un débit de 250 Mo/s au minimum pour /hana/data avec des tailles d’E/S de 16 Mo et 64 Mo.

Les [limites de débit Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md) pour un quota de volume de 1 Tio sont les suivantes :

- Niveau de stockage Premium – 64 Mio/s.
- Niveau de stockage Ultra – 128 Mio/s.

Pour respecter les exigences de débit minimal SAP pour /hana/data et /hana/log ainsi que les instructions pour /hana/shared, les tailles recommandées sont les suivantes :

|    Volume    | Taille du niveau de stockage Premium | Taille du niveau de stockage Ultra | Protocole NFS pris en charge |
| :----------: | :--------------------------: | :------------------------: | :--------------------: |
|  /hana/log   |            4 Tio             |           2 Tio            |          v4.1          |
|  /hana/data  |           6,3 Tio            |          3,2 Tio           |          v4.1          |
| /hana/shared |           1 x RAM            |          1 x RAM           |          v3 ou v4.1    |


> [!NOTE]
> Les recommandations relatives aux tailles Azure NetApp Files qui sont indiquées ici ciblent une configuration minimale que le protocole SAP recommande pour ses fournisseurs d’infrastructure. Dans les déploiements de clients et les scénarios de charge de travail réels, cela peut être insuffisant. Prenez ces recommandations en tant que point de départ et adaptez-les en fonction des exigences de votre charge de travail spécifique.

> [!TIP]
> Vous pouvez redimensionner les volumes Azure NetApp Files de manière dynamique, sans avoir à *démonter* les volumes, à arrêter les machines virtuelles ou à arrêter SAP HANA. Cette approche permet de répondre aux demandes de débit à la fois attendues et imprévues de votre application.

> [!NOTE]
> Toutes les commandes permettant de monter /hana/shared dans cet article sont présentées pour les volumes /hana/shared NFSv4.1.
> Si vous avez déployé les volumes /hana/shared en tant que volumes NFSv3, n’oubliez pas d’ajuster les commandes de montage pour /hana/shared en conséquence.


## <a name="deploy-linux-virtual-machine-via-azure-portal"></a>Déployer une machine virtuelle Linux via Portail Azure 

Vous devez d’abord créer les volumes Azure NetApp Files. Effectuez ensuite les étapes suivantes :

1.  Créez un groupe de ressources.
2.  Créez un réseau virtuel.
3.  Créer un groupe à haute disponibilité. Définir le domaine de mise à jour maximal.
4.  Créer un équilibrage de charge (interne). Nous vous recommandons Standard Load Balancer.
    Sélectionnez le réseau virtuel créé à l’étape 2.
5.  Créez la machine virtuelle 1 (**hanadb1**). 
6.  Créez la machine virtuelle 2 (**hanadb2**).  
7.  Lors de la création de la machine virtuelle, nous n’ajouterons aucun disque, car tous nos points de montage se trouveront sur des partages NFS d’Azure NetApp Files. 

> [!IMPORTANT]
> Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d’informations, consultez [Limitations d’équilibreur de charge Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.    

> [!NOTE] 
> Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité Internet sortante, sauf si une configuration supplémentaire est effectuée pour autoriser le routage vers des points de terminaison publics. Pour savoir plus en détails comment bénéficier d’une connectivité sortante, voir [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).

8.  Si vous utilisez Standard Load Balancer, suivez ces étapes de configuration :
    1.  Commencez par créer un pool d’adresses IP frontales :
        1.  Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
        1.  Entrez le nom du nouveau pool d’adresses IP frontales (par exemple **hana-frontend**).
        1.  Définissez l’**affectation** sur **Statique** et entrez l’adresse IP (par exemple, **10.32.0.10**).
        1.  Sélectionnez **OK**.
        1.  Une fois le pool d’adresses IP frontal créé, notez son adresse IP.
    1.  Créez ensuite un pool principal :
        1.  Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
        1.  Entrer le nom du nouveau pool principal (par exemple **hana-backend**).
        1.  Cliquez sur **Ajouter une machine virtuelle**.
        1.  Sélectionnez **Machine virtuelle**.
        1.  Sélectionnez les machines virtuelles du cluster SAP HANA et leurs adresses IP.
        1.  Sélectionnez **Ajouter**.
    1.  Créez ensuite une sonde d’intégrité :
        1.  Ouvrez l’équilibrage de charge, sélectionnez les **sondes d’intégrité**, puis cliquez sur **Ajouter**.
        1.  Entrez le nom de la nouvelle sonde d’intégrité (par exemple **hana-hp**).
        1.  Sélectionnez TCP pour le protocole et le port 625 **03**. Consersez la valeur **Intervalle** à 5, et la valeur **Seuil de défaillance** à 2.
        1.  Sélectionnez **OK**.
    1.  Ensuite, créez les règles d’équilibrage de charge :
        1.  Ouvrez l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter**.
        1.  Entrez le nom de la nouvelle règle d’équilibrage de charge (par exemple, **hana-lb**).
        1.  Sélectionnez l’adresse IP frontale, le pool principal et la sonde d’intégrité que vous avez créés (par exemple,**hana-frontend**, **hana-backend** et **hana-hp**).
        1.  Sélectionnez **Ports HA**.
        1.  Augmentez le **délai d’inactivité** à 30 minutes.
        1.  Veillez à **activer l’IP flottante** .
        1.  Sélectionnez **OK**.


9. Si à l’inverse votre scénario exige d’utiliser l’équilibreur de charge de base, suivez ces étapes de configuration :
    1.  Configurez l’équilibrage de charge. Commencez par créer un pool d’adresses IP frontales :
        1.  Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
        1.  Entrez le nom du nouveau pool d’adresses IP frontales (par exemple **hana-frontend**).
        1.  Définissez l’**affectation** sur **Statique** et entrez l’adresse IP (par exemple, **10.32.0.10**).
        1.  Sélectionnez **OK**.
        1.  Une fois le pool d’adresses IP frontal créé, notez son adresse IP.
    1.  Créez ensuite un pool principal :
        1.  Ouvrez l’équilibrage de charge, sélectionnez le **Pool d’adresses IP frontales**, puis cliquez sur **Ajouter**.
        1.  Entrer le nom du nouveau pool principal (par exemple **hana-backend**).
        1.  Cliquez sur **Ajouter une machine virtuelle**.
        1.  Sélectionnez le groupe à haute disponibilité créé à l’étape 3.
        1.  Sélectionnez les machines virtuelles du cluster SAP HANA.
        1.  Sélectionnez **OK**.
    1.  Créez ensuite une sonde d’intégrité :
        1.  Ouvrez l’équilibrage de charge, sélectionnez les **sondes d’intégrité**, puis cliquez sur **Ajouter**.
        1.  Entrez le nom de la nouvelle sonde d’intégrité (par exemple **hana-hp**).
        1.  Sélectionnez **TCP** pour le protocole et le port 625 **03**. Consersez la valeur **Intervalle** à 5, et la valeur **Seuil de défaillance** à 2.
        1.  Sélectionnez **OK**.
    1.  Pour SAP HANA 1.0, créez les règles d’équilibrage de charge :
        1.  Ouvrez l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter**.
        1.  Entrer le nom de la nouvelle règle d’équilibrage de charge (par exemple hana-lb-3 **03** 15).
        1.  Sélectionnez l’adresse IP du serveur frontal, le pool principal et la sonde d’intégrité que vous avez créée précédemment (par exemple, **hana-frontend**).
        1.  Conservez le **Protocole** à **TCP**, puis entrez le port 3 **03** 15.
        1.  Augmentez le **délai d’inactivité** à 30 minutes.
        1.  Veillez à **activer l’IP flottante** .
        1.  Sélectionnez **OK**.
        1.  Répétez ces étapes pour le port 3 **03** 17.
    1.  Pour SAP HANA 2.0, créez les règles d’équilibrage de charge pour la base de données du système :
        1.  Ouvrez l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter**.
        1.  Entrez le nom de la nouvelle règle d’équilibrage de charge (par exemple hana-lb-3 **03** 13).
        1.  Sélectionnez l’adresse IP du serveur frontal, le pool principal et la sonde d’intégrité que vous avez créée précédemment (par exemple, **hana-frontend**).
        1.  Conservez le **Protocole** à **TCP**, puis entrez le port 3 **03** 13.
        1.  Augmentez le **délai d’inactivité** à 30 minutes.
        1.  Veillez à **activer l’IP flottante** .
        1.  Sélectionnez **OK**.
        1.  Répétez ces étapes pour le port 3 **03** 14.
    1.  Pour SAP HANA 2.0, créez d’abord les règles d’équilibrage de charge pour la base de données locataire :
        1.  Ouvrez l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter**.
        1.  Entrez le nom de la nouvelle règle d’équilibrage de charge (par exemple hana-lb-3 **03** 40).
        1.  Sélectionnez l’adresse IP du serveur frontal, le pool principal et la sonde d’intégrité créés précédemment (par exemple **hana-frontend**).
        1.  Conservez le **Protocole** à **TCP**, puis entrez le port 3 **03** 40.
        1.  Augmentez le **délai d’inactivité** à 30 minutes.
        1.  Veillez à **activer l’IP flottante** .
        1.  Sélectionnez **OK**.
        1.  Répétez ces étapes pour les ports 3 **03** 41 et 3 **03** 42.

Pour plus d’informations sur les ports requis pour SAP HANA, consultez le chapitre [Connections to Tenant Databases](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6/latest/en-US/7a9343c9f2a2436faa3cfdb5ca00c052.html) (Connexions aux bases de données locataires) dans le guide [SAP HANA Tenant Databases](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6) (Bases de données locataires SAP HANA) ou la note SAP [2388694](https://launchpad.support.sap.com/#/notes/2388694).

> [!IMPORTANT]
> N’activez pas les timestamps TCP sur des machines virtuelles Azure placées derrière Azure Load Balancer. L’activation des timestamps TCP entraîne l’échec des sondes d’intégrité. Définissez le paramètre **net.ipv4.tcp_timestamps** sur **0**. Pour plus d’informations, consultez [Load Balancer health probes](../../../load-balancer/load-balancer-custom-probe-overview.md) (Sondes d’intégrité Load Balancer). Voir aussi la note SAP [2382421](https://launchpad.support.sap.com/#/notes/2382421).

## <a name="mount-the-azure-netapp-files-volume"></a>Monter le volume Azure NetApp Files

1. **[A]** Créez des points de montage pour les volumes de base de données HANA. 

    ```
    mkdir -p /hana/data
    mkdir -p /hana/log
    mkdir -p /hana/shared
    ```

2. **[A]** Vérifiez le paramètre de domaine NFS. Assurez-vous que le domaine est configuré en tant que domaine Azure NetApp Files par défaut, par exemple **defaultv4iddomain.com**, et que le mappage est défini sur **nobody**.

    ```
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = defaultv4iddomain.com
    [Mapping]
    Nobody-User = nobody
    Nobody-Group = nobody
    ```

    > [!IMPORTANT]    
    > Veillez à définir le domaine NFS dans /etc/idmapd.conf sur la machine virtuelle pour qu’il corresponde à la configuration de domaine par défaut sur Azure NetApp Files : **defaultv4iddomain.com**. En cas d’incompatibilité entre la configuration de domaine sur le client NFS (c’est-à-dire, la machine virtuelle) et le serveur NFS, par exemple la configuration Azure NetApp, les autorisations pour les fichiers sur les volumes Azure NetApp montés sur les machines virtuelles s’affichent en tant que « nobody ».
    

3. **[1]** Montez les volumes spécifiques aux nœuds sur node1 (**hanadb1**). 

    ```
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-shared-mnt00001 /hana/shared
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-log-mnt00001 /hana/log
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-data-mnt00001 /hana/data
    ```
    
4.  **[2]** Montez les volumes spécifiques aux nœuds sur node2 (**hanadb2**).
    
    ```
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-shared-mnt00001 /hana/shared
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-log-mnt00001 /hana/log
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-data-mnt00001 /hana/data
    ```

5. **[A]** Vérifiez que tous les volumes HANA sont montés avec la version NFSv4 du protocole NFS.

    ```
    sudo nfsstat -m
    
    # Verify that flag vers is set to 4.1 
    # Example from hanadb1
    
    /hana/log from 10.32.2.4:/hanadb1-log-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    /hana/data from 10.32.2.4:/hanadb1-data-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    /hana/shared from 10.32.2.4:/hanadb1-shared-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    ```

6. **[A]** Vérifiez **nfs4_disable_idmapping**. Cette option doit avoir la valeur **Y**. Pour créer la structure de répertoire où se trouve **nfs4_disable_idmapping**, exécutez la commande de montage. Vous ne serez pas en mesure de créer manuellement le répertoire sous /sys/modules, car l’accès est réservé pour le noyau/les pilotes.

    ```
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    
    # If you need to set nfs4_disable_idmapping to Y
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    ```

   Pour plus d’informations sur la modification du paramètre **nfs_disable_idmapping**, consultez [https://access.redhat.com/solutions/1749883](https://access.redhat.com/solutions/1749883). 


## <a name="sap-hana-installation"></a>Installation de SAP HANA

1. **[A]** Configurez la résolution de nom d’hôte pour tous les hôtes.

   Vous pouvez utiliser un serveur DNS ou modifier le fichier /etc/hosts sur tous les nœuds. Cet exemple vous montre comment utiliser le fichier /etc/hosts. Remplacez l’adresse IP et le nom d’hôte dans les commandes suivantes :

   ```
   sudo vi /etc/hosts
   # Insert the following lines in the /etc/hosts file. Change the IP address and hostname to match your environment  
   10.32.0.4   hanadb1
   10.32.0.5   hanadb2
   ```

2. **[A]** Configurez RHEL pour HANA

   Configurez RHEL comme décrit dans la note SAP ci-dessous en fonction de votre version de RHEL

   - [2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/2292690) (SAP HANA DB : Paramètres de système d’exploitation recommandés pour RHEL 7)
   - [2777782 – Base de données SAP HANA : paramètres de système d’exploitation recommandés pour RHEL 8](https://launchpad.support.sap.com/#/notes/2777782)
   - [2455582 – Linux : Exécution d’applications SAP compilées avec GCC 6.x](https://launchpad.support.sap.com/#/notes/2455582)
   - [2593824 – Linux : Exécution d’applications SAP compilées avec GCC 7.x](https://launchpad.support.sap.com/#/notes/2593824) 
   - [2886607 – Linux : Exécution d’applications SAP compilées avec GCC 9.x](https://launchpad.support.sap.com/#/notes/2886607)

3. **[A]** Installer SAP HANA

   À partir de HANA 2.0 SPS 01, MDC est l’option par défaut. Quand vous installez le système HANA, SYSTEMDB et un locataire avec le même SID sont créés ensemble. Dans certains cas, vous ne souhaitez pas utiliser le locataire par défaut. Si vous ne souhaitez pas créer de locataire initial pendant l’installation, vous pouvez suivre la note SAP [2629711](https://launchpad.support.sap.com/#/notes/2629711).

    Exécutez le programme **hdblcm** depuis le DVD HANA. Entrez les valeurs suivantes à l’invite :  
    Choose installation : Entrer **1** (pour l’installation)  
    Select additional components for installation : tapez **1**.  
    Enter Installation Path [/hana/shared] : appuyez sur Entrée pour accepter la valeur par défaut  
    Enter Local Host Name [..] : Appuyer sur Entrée pour utiliser la valeur par défaut  
    Do you want to add additional hosts to the system? (y/n) [n] : **n**  
    Enter SAP HANA System ID : entrez **HN1**  
    Enter Instance Number [00] : Entrer **03**  
    Select Database Mode / Enter Index [1] : appuyez sur Entrée pour accepter la valeur par défaut  
    Select System Usage / Enter Index [4] : entrez **4** (pour personnalisé)  
    Enter Location of Data Volumes [/hana/data] : appuyez sur Entrée pour accepter la valeur par défaut  
    Enter Location of Log Volumes [/hana/log] : appuyez sur Entrée pour accepter la valeur par défaut  
    Restrict maximum memory allocation? [n] : appuyez sur Entrée pour utiliser la valeur par défaut  
    Enter Certificate Host Name For Host '...' [...] : appuyez sur Entrée pour accepter la valeur par défaut  
    Enter SAP Host Agent User (sapadm) Password: entrez le mot de passe utilisateur de l’agent hôte  
    Confirm SAP Host Agent User (sapadm) Password: entrez à nouveau le mot de passe utilisateur de l’agent hôte pour confirmer  
    Entrer le mot de passe de l’administrateur système (hn1adm) : entrez le mot de passe d’administrateur système  
    Confirm System Administrator (hn1adm) Password : entrez à nouveau le mot de passe d’administrateur système pour confirmer  
    Enter System Administrator Home Directory [/usr/sap/HN1/home] : appuyez sur Entrée pour accepter la valeur par défaut  
    Enter System Administrator Login Shell [/bin/sh] : appuyez sur Entrée pour accepter la valeur par défaut  
    Enter System Administrator User ID [1001] : appuyez sur Entrée pour accepter la valeur par défaut  
    Enter ID of User Group (sapsys) [79] : appuyez sur Entrée pour accepter la valeur par défaut  
    Enter Database User (SYSTEM) Password: entrez le mot de passe utilisateur de la base de données  
    Confirm Database User (SYSTEM) Password: entrez à nouveau le mot de passe utilisateur de la base de données pour confirmer  
    Restart system after machine reboot? [n] : appuyez sur Entrée pour utiliser la valeur par défaut  
    Do you want to continue? (y/n) : validez le récapitulatif. Entrez **y** pour continuer  

4. **[A]** Mettre à niveau l’agent hôte SAP

   Téléchargez la dernière archive de l’agent hôte SAP à partir du [SAP Software Center](https://launchpad.support.sap.com/#/softwarecenter) et exécutez la commande suivante pour mettre à niveau l’agent. Remplacez le chemin d’accès à l’archive pour pointer vers le fichier que vous avez téléchargé :

   ```
   sudo /usr/sap/hostctrl/exe/saphostexec -upgrade -archive <path to SAP Host Agent SAR>
   ```

5. **[A]** Configurer le pare-feu

   Créez la règle de pare-feu pour le port d’analyse de l’équilibreur de charge Azure.

   ```
   sudo firewall-cmd --zone=public --add-port=62503/tcp
   sudo firewall-cmd --zone=public --add-port=62503/tcp –permanent
   ```

## <a name="configure-sap-hana-system-replication"></a>Configurer la réplication de système SAP HANA

Suivez les étapes décrites dans [Configurer la réplication du système SAP HANA](./sap-hana-high-availability-rhel.md#configure-sap-hana-20-system-replication) pour configurer la réplication du système SAP HANA. 

## <a name="cluster-configuration"></a>Configuration de clusters

Cette section décrit les étapes nécessaires au bon fonctionnement du cluster lorsque SAP HANA est installé sur des partages NFS à l’aide d’Azure NetApp Files. 

### <a name="create-a-pacemaker-cluster"></a>Créez un cluster Pacemaker

Suivez les étapes décrites sur la page [Configurer Pacemaker sur Red Hat Enterprise Linux](./high-availability-guide-rhel-pacemaker.md) dans Azure pour créer un cluster Pacemaker de base pour ce serveur HANA.

### <a name="configure-filesystem-resources"></a>Configurer les ressources du système de fichiers

Dans cet exemple, chaque nœud de cluster possède ses propres systèmes de fichiers NFS HANA : /hana/shared, /hana/data et /hana/log.   

1. **[1]** Mettez le cluster en mode maintenance.

   ```
   pcs property set maintenance-mode=true
   ```

2. **[1]** Créez les ressources du système de fichiers pour les montages **hanadb1**.

    ```
    pcs resource create hana_data1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-data-mnt00001 directory=/hana/data fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    pcs resource create hana_log1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-log-mnt00001 directory=/hana/log fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    pcs resource create hana_shared1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-shared-mnt00001 directory=/hana/shared fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    ```

3. **[2]** Créez les ressources du système de fichiers pour les montages **hanadb2**.

    ```
    pcs resource create hana_data2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-data-mnt00001 directory=/hana/data fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    pcs resource create hana_log2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-log-mnt00001 directory=/hana/log fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    pcs resource create hana_shared2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-shared-mnt00001 directory=/hana/shared fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    ```

    L’attribut `OCF_CHECK_LEVEL=20` est ajouté à l’opération d’analyse afin que chaque analyse effectue un test en lecture/écriture sur le système de fichiers. Sans cet attribut, l’opération d’analyse vérifie uniquement que le système de fichiers est monté. Cela peut être un problème, car, en cas de perte de connectivité, le système de fichiers peut rester monté bien qu’il soit inaccessible.

    L’attribut `on-fail=fence` est également ajouté à l’opération d’analyse. Avec cette option, si l’opération d’analyse échoue sur un nœud, ce dernier est immédiatement isolé. Sans cette option, le comportement par défaut consiste à arrêter toutes les ressources qui dépendent de la ressource défaillante, à redémarrer cette dernière, puis à démarrer toutes les ressources en question. Non seulement ce comportement peut prendre beaucoup de temps lorsqu’une ressource SAP HANA dépend de la ressource qui a échoué, mais il peut aussi échouer complètement. La ressource SAP HANA ne peut pas s’arrêter correctement si le serveur NFS contenant les exécutables HANA est inaccessible.

4. **[1]** Configurez des contraintes d’emplacement.

   Configurez des contraintes d’emplacement pour vous assurer que les ressources qui gèrent les montages uniques hanadb1 ne peuvent jamais s’exécuter sur hanadb2, et vice-versa.

    ```
    pcs constraint location hanadb1_nfs rule score=-INFINITY resource-discovery=never \#uname eq hanadb2
    pcs constraint location hanadb2_nfs rule score=-INFINITY resource-discovery=never \#uname eq hanadb1
    ```

    L’option `resource-discovery=never` est définie, car les montages uniques de chaque nœud partagent le même point de montage. Par exemple, `hana_data1` utilise le point de montage `/hana/data`, et `hana_data2` utilise également le point de montage `/hana/data`. Cela peut provoquer un faux positif pour une opération de sondage, lorsque l’état de la ressource est vérifié au démarrage du cluster, et cela peut à son tour entraîner un comportement de récupération inutile. Cela peut être évité en définissant `resource-discovery=never`.

5. **[1]** Configurez des ressources d’attribut.

   Configurez des ressources d’attribut. Ces attributs sont définis sur true si tous les montages NFS d’un nœud (/hana/data, /hana/log et /hana/data) sont montés ; ils sont définis sur false dans le cas contraire.

   ```
   pcs resource create hana_nfs1_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs1_active
   pcs resource create hana_nfs2_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs2_active
   ```

6. **[1]** Configurez des contraintes d’emplacement.

   Configurez des contraintes d’emplacement pour vous assurer que la ressource d’attribut de hanadb1 ne s’exécute jamais sur hanadb2, et vice-versa.

   ```
   pcs constraint location hana_nfs1_active avoids hanadb2
   pcs constraint location hana_nfs2_active avoids hanadb1
   ```

7. **[1]** Créez des contraintes de classement

   Configurez des contraintes de classement afin que les ressources d’attribut d’un nœud démarrent uniquement une fois que tous les montages NFS du nœud ont été montés.
   
    ```
    pcs constraint order hanadb1_nfs then hana_nfs1_active
    pcs constraint order hanadb2_nfs then hana_nfs2_active
    ```

   > [!TIP]
   > Si votre configuration inclut des systèmes de fichiers, en dehors du groupe `hanadb1_nfs` ou `hanadb2_nfs`, incluez l’option `sequential=false` afin qu’il n’y ait aucune dépendance de classement entre les systèmes de fichiers. Tous les systèmes de fichiers doivent démarrer avant `hana_nfs1_active`, mais ils n’ont pas besoin de démarrer dans un ordre particulier les uns par rapport aux autres. Pour plus d’informations, consultez [Comment configurer la réplication du système SAP HANA en scale-up dans un cluster Pacemaker lorsque les systèmes de fichiers HANA se trouvent sur des partages NFS](https://access.redhat.com/solutions/5156571).

### <a name="configure-sap-hana-cluster-resources"></a>Configurer les ressources de cluster SAP HANA

1. Suivez les étapes décrites dans [Créer les ressources de cluster SAP HANA](./sap-hana-high-availability-rhel.md#create-sap-hana-cluster-resources) pour créer les ressources SAP HANA dans le cluster. Une fois que les ressources SAP HANA sont créées, nous devons créer une contrainte de règle d’emplacement entre les ressources SAP HANA et les systèmes de fichiers (montages NFS).

2. **[1]** Configurez des contraintes entre les ressources SAP HANA et les montages NFS.

   Les contraintes de règle d’emplacement sont définies de sorte que les ressources SAP HANA peuvent s’exécuter sur un nœud uniquement si tous les montages NFS du nœud sont montés.

    ```
    pcs constraint location SAPHanaTopology_HN1_03-clone rule score=-INFINITY hana_nfs1_active ne true and hana_nfs2_active ne true
    # On RHEL 7.x
    pcs constraint location SAPHana_HN1_03-master rule score=-INFINITY hana_nfs1_active ne true and hana_nfs2_active ne true
    # On RHEL 8.x
    pcs constraint location SAPHana_HN1_03-clone rule score=-INFINITY hana_nfs1_active ne true and hana_nfs2_active ne true
    # Take the cluster out of maintenance mode
    sudo pcs property set maintenance-mode=false
    ```

   Vérifiez l’état du cluster et celui de toutes les ressources.
   > [!NOTE]
   > Cet article contient des références au terme *esclave*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.
   
    ```
    sudo pcs status
    
    Online: [ hanadb1 hanadb2 ]
    
    Full list of resources:
    
    rsc_hdb_azr_agt(stonith:fence_azure_arm):  Started hanadb1
    
    Resource Group: hanadb1_nfs
    hana_data1 (ocf::heartbeat:Filesystem):Started hanadb1
    hana_log1  (ocf::heartbeat:Filesystem):Started hanadb1
    hana_shared1   (ocf::heartbeat:Filesystem):Started hanadb1
    
    Resource Group: hanadb2_nfs
    hana_data2 (ocf::heartbeat:Filesystem):Started hanadb2
    hana_log2  (ocf::heartbeat:Filesystem):Started hanadb2
    hana_shared2   (ocf::heartbeat:Filesystem):Started hanadb2
    
    hana_nfs1_active   (ocf::pacemaker:attribute): Started hanadb1
    hana_nfs2_active   (ocf::pacemaker:attribute): Started hanadb2
    
    Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hanadb1 hanadb2 ]
    
    Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hanadb1 ]
    Slaves: [ hanadb2 ]
    
    Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):  Started hanadb1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):   Started hanadb1
    ```

## <a name="test-the-cluster-setup"></a>Tester la configuration du cluster

Cette section explique comment vous pouvez tester votre configuration. 

1. Avant de commencer le test, assurez-vous que Pacemaker ne comporte aucune action ayant échoué (via pcs status), qu’il n’existe aucune contrainte d’emplacement inattendue (par exemple, les reliquats d’un test de migration) et que la réplication du système HANA est en état de synchronisation, par exemple avec systemReplicationStatus :

    ```
    sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
    ```

2. Vérifiez la configuration du cluster pour un scénario d’échec lorsqu’un nœud perd l’accès au partage NFS (/hana/shared).

   Les agents de ressource SAP HANA dépendent de binaires, qui sont stockés sur `/hana/shared` pour effectuer des opérations pendant le basculement. Le système de fichiers `/hana/shared` est monté sur NFS dans le scénario présenté.  
   Il est difficile de simuler une panne pendant laquelle l’un des serveurs perd l’accès au partage NFS. Un test possible consiste à remonter le système de fichiers en lecture seule.
   Cette approche valide la capacité du cluster à basculer en cas de perte de l’accès à `/hana/shared` sur le nœud actif.     


   **Résultat attendu :** En faisant de `/hana/shared` un système de fichiers en lecture seule, l’attribut `OCF_CHECK_LEVEL` de la ressource `hana_shared1` qui effectue une opération de lecture/d’écriture sur le système de fichiers échouera, car il n’est pas en mesure d’écrire quoi que ce soit sur le système de fichiers, et il effectuera le basculement de la ressource HANA.  Le même résultat est attendu lorsque votre nœud HANA perd l’accès aux partages NFS. 

   État des ressources avant le début du test :

    ```
    sudo pcs status
    # Example output
    Full list of resources:
     rsc_hdb_azr_agt        (stonith:fence_azure_arm):      Started hanadb1

     Resource Group: hanadb1_nfs
         hana_data1 (ocf::heartbeat:Filesystem):    Started hanadb1
         hana_log1  (ocf::heartbeat:Filesystem):    Started hanadb1
         hana_shared1       (ocf::heartbeat:Filesystem):    Started hanadb1

    Resource Group: hanadb2_nfs
         hana_data2 (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_log2  (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_shared2       (ocf::heartbeat:Filesystem):    Started hanadb2

     hana_nfs1_active       (ocf::pacemaker:attribute):     Started hanadb1
     hana_nfs2_active       (ocf::pacemaker:attribute):     Started hanadb2

     Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
         Started: [ hanadb1 hanadb2 ]

     Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
         Masters: [ hanadb1 ]
         Slaves: [ hanadb2 ]

     Resource Group: g_ip_HN1_03
         nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hanadb1
         vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hanadb1
    ```

   Vous pouvez placer /hana/shared en mode lecture seule sur le nœud de cluster actif à l’aide de la commande suivante :

    ```
    sudo mount -o ro 10.32.2.4:/hanadb1-shared-mnt00001 /hana/shared
    ```

   hanadb1 redémarre ou s’éteint en fonction de l’action définie sur stonith (`pcs property show stonith-action`).  Une fois que le serveur (hanadb1) est arrêté, la ressource HANA est déplacée vers hanadb2. Vous pouvez vérifier l’état du cluster à partir de hanadb2.

    ```
    pcs status

    Full list of resources:

     rsc_hdb_azr_agt        (stonith:fence_azure_arm):      Started hanadb2

     Resource Group: hanadb1_nfs
         hana_data1 (ocf::heartbeat:Filesystem):    Stopped
         hana_log1  (ocf::heartbeat:Filesystem):    Stopped
         hana_shared1       (ocf::heartbeat:Filesystem):    Stopped

     Resource Group: hanadb2_nfs
         hana_data2 (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_log2  (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_shared2       (ocf::heartbeat:Filesystem):    Started hanadb2

     hana_nfs1_active       (ocf::pacemaker:attribute):     Stopped
     hana_nfs2_active       (ocf::pacemaker:attribute):     Started hanadb2

     Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
         Started: [ hanadb2 ]
         Stopped: [ hanadb1 ]

     Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
         Masters: [ hanadb2 ]
         Stopped: [ hanadb1 ]

     Resource Group: g_ip_HN1_03
         nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hanadb2
         vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hanadb2
    ```

   Nous vous recommandons de tester minutieusement la configuration des clusters SAP HANA, en effectuant également les tests décrits dans [Configurer la réplication du système SAP HANA sur RHEL](./sap-hana-high-availability-rhel.md#test-the-cluster-setup).