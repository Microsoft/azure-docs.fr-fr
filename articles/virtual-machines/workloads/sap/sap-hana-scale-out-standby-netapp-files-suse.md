---
title: Scale-out de SAP HANA avec nœud de secours à l'aide d'Azure NetApp Files sur SLES | Microsoft Docs
description: Découvrez comment déployer un système Scale-out SAP HANA avec nœud de secours sur des machines virtuelles Azure à l’aide d’Azure NetApp Files sur SUSE Linux Enterprise Server.
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
ms.date: 04/24/2020
ms.author: radeltch
ms.openlocfilehash: 13644872fca06ad8fc5806326736aea23e504520
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608654"
---
# <a name="deploy-a-sap-hana-scale-out-system-with-standby-node-on-azure-vms-by-using-azure-netapp-files-on-suse-linux-enterprise-server"></a>Déployer un système Scale-out SAP HANA avec le nœud de secours sur des machines virtuelles Azure à l’aide d’Azure NetApp Files sur SUSE Linux Enterprise Server 

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

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md


Cet article explique comment déployer un système SAP HANA à haut niveau de disponibilité dans une configuration Scale-out avec nœud de secours sur des machines virtuelles Azure en utilisant [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) pour les volumes de stockage partagés.  

Dans les exemples de configuration, de commandes d’installation, etc., l’instance HANA est **03** et l’ID de système HANA est **HN1**. Les exemples sont basés sur HANA 2.0 SP4 et SUSE Linux Enterprise Server pour SAP 12 SP4. 

Avant de commencer, reportez-vous aux notes et documents SAP suivants :

* [Documentation Azure NetApp Files][anf-azure-doc] 
* La note SAP [1928533] comprend :  
  * une liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP
  * des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure
  * les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données
  * la version du noyau SAP requise pour Windows et Linux sur Microsoft Azure
* Note SAP [2015553] : répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure
* Note SAP [2205917] : contient les paramètres de système d’exploitation recommandés pour SUSE Linux Enterprise Server pour les applications SAP
* Note SAP [1944799] : contient des instructions SAP pour SUSE Linux Enterprise Server pour les applications SAP
* Note SAP [2178632] : contient des informations détaillées sur toutes les métriques de supervision rapportées pour SAP dans Azure
* Note SAP [2191498] : contient la version requise de l’agent hôte SAP pour Linux dans Azure
* Note SAP [2243692] : contient des informations sur les licences SAP sur Linux dans Azure
* Note SAP [1984787] : contient des informations générales sur SUSE Linux Enterprise Server 12
* Note SAP [1999351] : contient des informations supplémentaires relatives à la résolution des problèmes pour l’extension d’analyse Azure améliorée pour SAP
* Note SAP [1900823] : contient des informations sur les exigences de stockage SAP HANA
* [Wiki de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) : contient toutes les notes SAP requises pour Linux
* [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP sur Linux][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP sur Linux][dbms-guide]
* [Guides sur les meilleures pratiques de haute disponibilité SAP pour SUSE][suse-ha-guide] : contient toutes les informations nécessaires pour configurer la haute disponibilité NetWeaver et la réplication de système SAP HANA localement (à utiliser comme ligne de base générale ; elles fournissent des informations beaucoup plus détaillées)
* [Notes de publication de SUSE High Availability Extension 12 SP3][suse-ha-12sp3-relnotes]
* [Applications SAP NetApp su Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure]


## <a name="overview"></a>Vue d’ensemble

L’une des méthodes permettant d’obtenir une haute disponibilité HANA consiste à configurer le basculement automatique de l’hôte. Pour configurer le basculement automatique de l’hôte, vous devez ajouter une ou plusieurs machines virtuelles au système HANA et les configurer en tant que nœuds de secours. En cas de défaillance d’un nœud actif, un nœud de secours prend automatiquement le relais. Dans la configuration présentée avec les machines virtuelles Azure, vous obtenez un basculement automatique à l’aide de [NFS sur Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).  

> [!NOTE]
> Le nœud de secours a besoin d’accéder à tous les volumes de base de données. Les volumes HANA doivent être montés en tant que volumes NFSv4. Le mécanisme amélioré de verrouillage basé sur les baux de fichiers dans le protocole NFSv4 est utilisé pour la délimitation `I/O`. 

> [!IMPORTANT]
> Pour générer la configuration prise en charge, vous devez déployer les volumes de données et de journaux HANA en tant que volumes NFSv4.1 et les monter à l’aide du protocole NFSv4.1. La configuration du basculement automatique de l’hôte HANA avec le nœud de secours n’est pas prise en charge avec NFSv3.

![Vue d’ensemble de la haute disponibilité SAP NetWeaver](./media/high-availability-guide-suse-anf/sap-hana-scale-out-standby-netapp-files-suse.png)

Dans le diagramme ci-dessus, qui suit les recommandations de réseau SAP HANA, trois sous-réseaux sont représentés au sein d’un réseau virtuel Azure : 
* Pour la communication client
* Pour la communication avec le système de stockage
* Pour la communication interne HANA entre les nœuds

Les volumes Azure NetApp se trouvent dans un sous-réseau distinct, [délégué à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

Pour cet exemple de configuration, les sous-réseaux sont les suivants :  

  - `client` 10.23.0.0/24  
  - `storage` 10.23.2.0/24  
  - `hana` 10.23.3.0/24  
  - `anf` 10.23.1.0/26  

## <a name="set-up-the-azure-netapp-files-infrastructure"></a>Configurer l’infrastructure Azure NetApp Files 

Avant de poursuivre la configuration de l’infrastructure Azure NetApp Files, familiarisez-vous avec la [documentation Azure NetApp Files][anf-azure-doc]. 

Azure NetApp Files est disponible dans plusieurs [régions Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Vérifiez si la région Azure que vous avez sélectionnée est compatible avec Azure NetApp Files.  

Pour plus d’informations sur la disponibilité d’Azure NetApp Files par région Azure, consultez [Disponibilité d’Azure NetApp Files par région Azure][anf-avail-matrix].  

Avant de déployer Azure NetApp Files, demandez l’intégration à Azure NetApp Files en accédant à [Instructions d’inscription à Azure NetApp Files][anf-register]. 

### <a name="deploy-azure-netapp-files-resources"></a>Déployer des ressources Azure NetApp Files  

Les instructions suivantes supposent que vous avez déjà déployé votre [réseau virtuel Azure](../../../virtual-network/virtual-networks-overview.md). Les ressources Azure NetApp Files et les machines virtuelles, où les ressources Azure NetApp Files seront montées, doivent être déployées dans le même réseau virtuel Azure ou dans des réseaux virtuels Azure appairés.  

1. Si vous n’avez pas encore déployé les ressources, demandez l’[intégration à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).  

2. Créez un compte NetApp dans la région Azure sélectionnée en suivant les instructions de la page [Création d’un compte NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).  

3. Configurez un pool de capacité Azure NetApp Files en suivant les instructions de la page [Configuration d’un pool de capacité Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).  

   L’architecture HANA présentée dans cet article utilise un seul pool de capacité Azure NetApp Files au niveau de *service Ultra*. Pour les charges de travail HANA sur Azure, nous vous recommandons d’utiliser un [niveau de service](../../../azure-netapp-files/azure-netapp-files-service-levels.md)*Ultra* ou *Premium* d’Azure NetApp Files.  

4. Déléguez un sous-réseau à Azure NetApp Files, comme décrit dans les instructions de la page [Déléguer un sous-réseau à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

5. Déployez des volumes Azure NetApp Files en suivant les instructions de la page [Créer un volume NFS pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  

   Lors du déploiement des volumes, veillez à sélectionner la version **NFSv4.1**, . Actuellement, l’accès au NFSv4.1 nécessite d’être ajouté à une liste verte. Déployez les volumes dans le [sous-réseau](/rest/api/virtualnetwork/subnets) Azure NetApp Files désigné. Les adresses IP des volumes Azure NetApp sont attribuées automatiquement. 
   
   Gardez à l’esprit que les ressources Azure NetApp Files et les machines virtuelles Azure doivent être dans le même réseau virtuel Azure ou dans des réseaux virtuels Azure appairés. Par exemple, **HN1**-data-mnt00001, **HN1**-log-mnt00001 et ainsi de suite sont les noms de volume et nfs://10.23.1.5/**HN1**-data-mnt00001, nfs://10.23.1.4/**HN1**-log-mnt00001 et ainsi de suite sont les chemin d’accès de fichiers pour les volumes Azure NetApp Files.  

   * volume **HN1**-data-mnt00001 (nfs://10.23.1.5/**HN1**-data-mnt00001)
   * volume **HN1**-data-mnt00002 (nfs://10.23.1.6/**HN1**-data-mnt00002)
   * volume **HN1**-log-mnt00001 (nfs://10.23.1.4/**HN1**-log-mnt00001)
   * volume **HN1**-log-mnt00002 (nfs://10.23.1.6/**HN1**-log-mnt00002)
   * volume **HN1**-shared (nfs://10.23.1.4/**HN1**-shared)
   
   Dans cet exemple, nous avons utilisé un volume Azure NetApp Files distinct pour chaque volume de données et de journaux HANA. Pour une configuration plus économique sur des systèmes plus petits ou non productifs, il est possible de placer tous les montages de données et tous les montages de journaux sur un volume unique.  

### <a name="important-considerations"></a>Points importants à prendre en compte

Lorsque vous créez votre Azure NetApp Files pour SAP NetWeaver sur une architecture à haute disponibilité SUSE, tenez compte des considérations importantes suivantes :

- La taille de pool de capacité minimale est de 4 tébioctets (Tio).  
- La taille de volume minimale est de 100 gibioctets (Gio).
- Azure NetApp Files et toutes les machines virtuelles où les volumes Azure NetApp Files seront montés doivent être déployés dans le même réseau virtuel Azure ou dans des [réseaux virtuels appairés](../../../virtual-network/virtual-network-peering-overview.md) dans la même région.  
- Le réseau virtuel sélectionné doit avoir un sous-réseau délégué à Azure NetApp Files.
- Le débit d’un volume Azure NetApp Files est une fonction du quota de volume et du niveau de service, comme décrit dans [Niveau de service pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Lorsque vous dimensionnez les volumes HANA Azure NetApp, assurez-vous que le débit obtenu répond à la configuration ²requise pour HANA.  
- Grâce à la [stratégie d’exportation](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md) d’Azure NetApp Files, vous pouvez contrôler les clients autorisés, le type d’accès (lecture et écriture, lecture seule, etc.). 
- La fonctionnalité Azure NetApp Files ne tient pas encore compte des zones. Actuellement, la fonctionnalité n’est pas déployée dans toutes les zones de disponibilité d’une région Azure. Méfiez-vous de l’impact potentiel sur les temps de latence dans certaines régions Azure.  
-  

> [!IMPORTANT]
> Pour les charges de travail SAP HANA, une latence faible est critique. Collaborez avec votre représentant Microsoft pour vous assurer que les machines virtuelles et les volumes Azure NetApp Files soient déployés à proximité.  

### <a name="sizing-for-hana-database-on-azure-netapp-files"></a>Dimensionnement de la base de données HANA sur Azure NetApp Files

Le débit d’un volume NetApp Azure Files est une fonction de la taille de volume et du niveau de service, comme décrit dans [Niveau de service pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). 

Lorsque vous concevez l’infrastructure pour SAP dans Azure, tenez compte des besoins de stockage minimaux de SAP, qui se traduisent par des caractéristiques de débit minimales :

- Activez la lecture-écriture sur /hana/log de 250 mégaoctets par seconde (Mo/s) avec des tailles d’E/S de 1 Mo.  
- Activez l’activité de lecture à un débit de 400 Mo/s au minimum pour /hana/data pour des tailles d’E/S de 16 Mo et 64 Mo.  
- Activez l’activité d’écriture à un débit de 250 Mo/s au minimum pour /hana/data avec des tailles d’E/S de 16 Mo et 64 Mo. 

Les [limites de débit Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md) pour un quota de volume de 1 Tio sont les suivantes :
- Niveau de stockage Premium – 64 Mio/s  
- Niveau de stockage Ultra – 128 Mio/s  

Pour respecter les exigences de débit minimal SAP pour les données et le journal ainsi que les instructions pour /hana/log, les tailles recommandées sont les suivantes :

| Volume | Taille de<br>Niveau de stockage Premium | Taille de<br>Niveau de stockage Ultra | Protocole NFS pris en charge |
| --- | --- | --- | --- |
| /hana/log/ | 4 Tio | 2 Tio | v4.1 |
| /hana/data | 6,3 Tio | 3,2 Tio | v4.1 |
| /hana/shared | Max (512 Go, 1xRAM) par 4 nœuds Worker | Max (512 Go, 1xRAM) par 4 nœuds Worker | v3 ou v4.1 |

La configuration SAP HANA pour la disposition présentée dans cet article, à l’aide du niveau de stockage Ultra d’Azure NetApp Files, serait la suivante :

| Volume | Taille de<br>Niveau de stockage Ultra | Protocole NFS pris en charge |
| --- | --- | --- |
| /hana/log/mnt00001 | 2 Tio | v4.1 |
| /hana/log/mnt00002 | 2 Tio | v4.1 |
| /hana/data/mnt00001 | 3,2 Tio | v4.1 |
| /hana/data/mnt00002 | 3,2 Tio | v4.1 |
| /hana/shared | 2 Tio | v3 ou v4.1 |

> [!NOTE]
> Les recommandations relatives aux tailles Azure NetApp Files qui sont indiquées ici ciblent une configuration minimale que le protocole SAP recommande pour ses fournisseurs d’infrastructure. Dans les déploiements de clients et les scénarios de charge de travail réels, cela peut être insuffisant. Prenez ces recommandations en tant que point de départ et adaptez-les en fonction des exigences de votre charge de travail spécifique.  

> [!TIP]
> Vous pouvez redimensionner les volumes Azure NetApp Files de manière dynamique, sans avoir à *démonter* les volumes, à arrêter les machines virtuelles ou à arrêter SAP HANA. Cette approche permet de répondre aux demandes de débit à la fois attendues et imprévues de votre application.

## <a name="deploy-linux-virtual-machines-via-the-azure-portal"></a>Déployer des machines virtuelles Linux via le Portail Azure

Vous devez d’abord créer les volumes Azure NetApp Files. Effectuez ensuite les étapes suivantes :
1. Créez les [sous-réseaux du réseau virtuel Azure](../../../virtual-network/virtual-network-manage-subnet.md) dans votre [réseau virtuel Azure](../../../virtual-network/virtual-networks-overview.md). 
1. Déployez les machines virtuelles. 
1. Créez des interfaces réseau supplémentaires et attachez-les aux machines virtuelles correspondantes.  

   Chaque machine virtuelle a trois interfaces réseau, qui correspondent aux trois sous-réseaux de réseau virtuel Azure (`client`, `storage` et `hana`). 

   Pour plus d’informations, consultez la documentation [Créer une machine virtuelle Linux dans Azure avec plusieurs cartes d’interface réseau](../../linux/multiple-nics.md).  

> [!IMPORTANT]
> Pour les charges de travail SAP HANA, une latence faible est critique. Pour atteindre une faible latence, collaborez avec votre représentant Microsoft pour vous assurer que les machines virtuelles et les volumes Azure NetApp Files soient déployés à proximité. Soumettez les informations nécessaires lorsque vous [intégrez le nouveau système SAP HANA](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxjSlHBUxkJBjmARn57skvdUQlJaV0ZBOE1PUkhOVk40WjZZQVJXRzI2RC4u) qui utilise SAP HANA Azure NetApp Files. 
 
Les instructions suivantes supposent que vous avez déjà créé le groupe de ressources, le réseau virtuel Azure et les 3 sous-réseaux de réseau virtuel Azure : `client`, `storage` et `hana`. Lorsque vous déployez les machines virtuelles, sélectionnez le sous-réseau client, afin que l’interface réseau cliente soit l’interface principale sur les machines virtuelles. Vous devez également configurer un itinéraire explicite vers le sous-réseau délégué Azure NetApp Files via la passerelle de sous-réseau de stockage. 

> [!IMPORTANT]
> Vérifiez que le système d’exploitation que vous sélectionnez est certifié SAP pour SAP HANA sur les types de machine virtuelle spécifiques que vous utilisez. Pour obtenir la liste des types de machines virtuelles certifiés SAP HANA et des versions de système d’exploitation correspondants, accédez au site [SAP HANA Certified IaaS platforms](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Cliquez sur les détails du type de machine virtuelle répertorié pour obtenir la liste complète des versions de système d’exploitation prises en charge par SAP HANA pour ce type.  

1. Créez un groupe à haute disponibilité pour SAP HANA. Veillez à définir un domaine de mise à jour maximal.  

2. Créez 3 machines virtuelles (**hanadb1**, **hanadb2** et **hanadb3**) en procédant comme suit :  

   a. Utilisez une image SLES4SAP de la galerie Azure prise en charge pour SAP HANA. Dans cet exemple, nous avons utilisé une image SLES4SAP 12 SP4.  

   b. Sélectionnez le groupe à haute disponibilité créé précédemment pour SAP HANA.  

   c. Sélectionnez le sous-réseau de réseau virtuel Azure client. Sélectionnez [Mise en réseau accélérée](../../../virtual-network/create-vm-accelerated-networking-cli.md).  

   Lorsque vous déployez les machines virtuelles, le nom de l’interface réseau est généré automatiquement. Dans ces instructions, par soucis de simplicité, nous allons faire référence aux interfaces réseau générées automatiquement, qui sont attachées au sous-réseau de réseau virtuel Azure client, en tant que **hanadb1-client**, **hanadb2-client** et **hanadb3-client**. 

3. Créez 3 interfaces réseau, une pour chaque machine virtuelle, pour le sous-réseau de réseau virtuel `storage` (dans cet exemple **hanadb1-storage**, **hanadb2-storage** et **hanadb3-storage**).  

4. Créez trois interfaces réseau, une pour chaque machine virtuelle, pour le sous-réseau du réseau virtuel `hana` (dans cet exemple, **hanadb1-hana**, **hanadb2-hana** et **hanadb3-hana**).  

5. Attachez les interfaces réseau virtuelles nouvellement créées aux machines virtuelles correspondantes en procédant comme suit :  

    a. Accédez à la machine virtuelle sur le [Portail Azure](https://portal.azure.com/#home).  

    b. Dans le volet gauche, sélectionnez **Machines virtuelles**. Filtrez sur le nom de la machine virtuelle (par exemple, **hanadb1**), puis sélectionnez la machine virtuelle.  

    c. Dans le volet **Vue d’ensemble**, sélectionnez **Arrêter** pour libérer la machine virtuelle.  

    d. Sélectionnez **Mise en réseau**, puis attachez l’interface réseau. Dans la liste déroulante sous **Attacher l’interface réseau**, sélectionnez les interfaces réseau déjà créées pour les sous-réseaux `storage` et `hana`.  
    
    e. Sélectionnez **Enregistrer**. 
 
    f. Répétez les étapes b à e pour les machines virtuelles restantes (dans notre exemple, **hanadb2** et **hanadb3**).
 
    g. Laissez les machines virtuelles dans l’état arrêté pour l’instant. Ensuite, nous allons activer [Mise en réseau accélérée](../../../virtual-network/create-vm-accelerated-networking-cli.md) pour toutes les interfaces réseau qui viennent d’être attachées.  

6. Activez la mise en réseau accélérée pour les interfaces réseau supplémentaires pour les sous-réseaux `storage` et `hana` en procédant comme suit :  

    a. Ouvrez [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) dans le [Portail Azure](https://portal.azure.com/#home).  

    b. Exécutez les commandes suivantes pour activer la mise en réseau accélérée pour les interfaces réseau supplémentaires, qui sont attachées aux sous-réseaux `storage` et `hana`.  

    <pre><code>
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb1-storage</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb2-storage</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb3-storage</b> --accelerated-networking true
    
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb1-hana</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb2-hana</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb3-hana</b> --accelerated-networking true

    </code></pre>

7. Démarrez les machines virtuelles en procédant comme suit :  

    a. Dans le volet gauche, sélectionnez **Machines virtuelles**. Filtrez sur le nom de la machine virtuelle (par exemple, **hanadb1**), puis sélectionnez-le.  

    b. Dans le volet **Vue d’ensemble**, sélectionnez **Démarrer**.  

## <a name="operating-system-configuration-and-preparation"></a>Configuration et préparation du système d’exploitation

Les instructions des sections suivantes sont précédées de l’un des éléments suivants :
* **[A]**  : applicable à tous les nœuds
* **[1]**  : applicable uniquement au nœud 1
* **[2]**  : applicable uniquement au nœud 2
* **[3]**  : applicable uniquement au nœud 3

Configurez et préparez votre système d’exploitation en procédant comme suit :

1. **[A]** Conservez les fichiers hôtes sur les machines virtuelles. Incluez des entrées pour tous les sous-réseaux. Les entrées suivantes ont été ajoutées à `/etc/hosts` pour cet exemple.  

    <pre><code>
    # Storage
    10.23.2.4   hanadb1-storage
    10.23.2.5   hanadb2-storage
    10.23.2.6   hanadb3-storage
    # Client
    10.23.0.5   hanadb1
    10.23.0.6   hanadb2
    10.23.0.7   hanadb3
    # Hana
    10.23.3.4   hanadb1-hana
    10.23.3.5   hanadb2-hana
    10.23.3.6   hanadb3-hana
    </code></pre>

2. **[A]** Modifiez les paramètres de configuration DHCP et de cloud pour l’interface réseau du stockage afin d’éviter les modifications involontaires du nom d’hôte.  

    Les instructions suivantes supposent que `eth1` est l’interface réseau de stockage. 

    <pre><code>
    vi /etc/sysconfig/network/dhcp
    # Change the following DHCP setting to "no"
    DHCLIENT_SET_HOSTNAME="no"
    vi /etc/sysconfig/network/ifcfg-<b>eth1</b>
    # Edit ifcfg-eth1 
    #Change CLOUD_NETCONFIG_MANAGE='yes' to "no"
    CLOUD_NETCONFIG_MANAGE='no'
    </code></pre>

2. **[A]** Ajoutez un itinéraire réseau, de sorte que la communication avec Azure NetApp Files passe par l’interface réseau de stockage.  

    Les instructions suivantes supposent que `eth1` est l’interface réseau de stockage.  

    <pre><code>
    vi /etc/sysconfig/network/ifroute-<b>eth1</b>
    # Add the following routes 
    # RouterIPforStorageNetwork - - -
    # ANFNetwork/cidr RouterIPforStorageNetwork - -
    <b>10.23.2.1</b> - - -
    <b>10.23.1.0/26</b> <b>10.23.2.1</b> - -
    </code></pre>

    Redémarrez la machine virtuelle pour activer les modifications.  

3. **[A]** Préparez le système d’exploitation pour l’exécution de SAP HANA sur NetApp Systems avec NFS, comme décrit dans [Applications SAP NetApp sur Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure]. Créez un fichier de configuration */etc/sysctl.d/netapp-hana.conf* pour les paramètres de configuration de NetApp.  

    <pre><code>
    vi /etc/sysctl.d/netapp-hana.conf
    # Add the following entries in the configuration file
    net.core.rmem_max = 16777216
    net.core.wmem_max = 16777216
    net.core.rmem_default = 16777216
    net.core.wmem_default = 16777216
    net.core.optmem_max = 16777216
    net.ipv4.tcp_rmem = 65536 16777216 16777216
    net.ipv4.tcp_wmem = 65536 16777216 16777216
    net.core.netdev_max_backlog = 300000
    net.ipv4.tcp_slow_start_after_idle=0
    net.ipv4.tcp_no_metrics_save = 1
    net.ipv4.tcp_moderate_rcvbuf = 1
    net.ipv4.tcp_window_scaling = 1
    net.ipv4.tcp_timestamps = 1
    net.ipv4.tcp_sack = 1
    </code></pre>

4. **[A]** Créez un fichier de configuration */etc/sysctl.d/ms-az.conf* avec les paramètres de configuration Microsoft Azure.  

    <pre><code>
    vi /etc/sysctl.d/ms-az.conf
    # Add the following entries in the configuration file
    ipv6.conf.all.disable_ipv6 = 1
    net.ipv4.tcp_max_syn_backlog = 16348
    net.ipv4.ip_local_port_range = 40000 65300
    net.ipv4.conf.all.rp_filter = 0
    sunrpc.tcp_slot_table_entries = 128
    vm.swappiness=10
    </code></pre>

4. **[A]** Ajuster les paramètres sunrpc, comme recommandé dans les [applications SAP NetApp sur Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure].  

    <pre><code>
    vi /etc/modprobe.d/sunrpc.conf
    # Insert the following line
    options sunrpc tcp_max_slot_table_entries=128
    </code></pre>

## <a name="mount-the-azure-netapp-files-volumes"></a>Monter les volumes Azure NetApp Files

1. **[A]** Créez des points de montage pour les volumes de base de données HANA.  

    <pre><code>
    mkdir -p /hana/data/<b>HN1</b>/mnt00001
    mkdir -p /hana/data/<b>HN1</b>/mnt00002
    mkdir -p /hana/log/<b>HN1</b>/mnt00001
    mkdir -p /hana/log/<b>HN1</b>/mnt00002
    mkdir -p /hana/shared
    mkdir -p /usr/sap/<b>HN1</b>
    </code></pre>

2. **[1]** Créez des répertoires spécifiques aux nœuds pour /usr/sap sur partages **HN1**.  

    <pre><code>
    # Create a temporary directory to mount <b>HN1</b>-shared
    mkdir /mnt/tmp
    # if using NFSv3 for this volume, mount with the following command
    mount <b>10.23.1.4</b>:/<b>HN1</b>-shared /mnt/tmp
    # if using NFSv4.1 for this volume, mount with the following command
    mount -t nfs -o sec=sys,vers=4.1 <b>10.23.1.4</b>:/<b>HN1</b>-shared /mnt/tmp
    cd /mnt/tmp
    mkdir shared usr-sap-<b>hanadb1</b> usr-sap-<b>hanadb2</b> usr-sap-<b>hanadb3</b>
    # unmount /hana/shared
    cd
    umount /mnt/tmp
    </code></pre>

3. **[A]** Vérifiez le paramètre de domaine NFS. Assurez-vous que le domaine est configuré en tant que domaine par défaut Azure NetApp Files, par exemple **`defaultv4iddomain.com`** et que le mappage est défini sur **nobody**.  

    > [!IMPORTANT]
    > Veillez à définir le domaine NFS dans `/etc/idmapd.conf` sur la machine virtuelle pour qu’elle corresponde à la configuration de domaine par défaut sur Azure NetApp Files : **`defaultv4iddomain.com`** . En cas d’incompatibilité entre la configuration de domaine sur le client NFS (c’est-à-dire la machine virtuelle) et le serveur NFS, par exemple la configuration Azure NetApp, les autorisations pour les fichiers sur les volumes Azure NetApp montés sur les machines virtuelles s’affichent en tant que `nobody`.  

    <pre><code>
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Verbosity = 0
    Pipefs-Directory = /var/lib/nfs/rpc_pipefs
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
    mount 10.23.1.4:/HN1-shared /mnt/tmp
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    </code></pre>

5. **[A]** Créez manuellement le groupe et l’utilisateur SAP HANA. Les ID pour le groupe sapsys et l’utilisateur **hn1** adm doivent être définis sur les mêmes ID que ceux fournis lors de l’intégration. (Dans cet exemple, les ID sont définis sur **1001**.) Si les ID ne sont pas définis correctement, vous ne pourrez pas accéder aux volumes. Les ID pour le groupe sapsys et les comptes d’utilisateurs **hn1** adm et sapadm doivent être identiques sur toutes les machines virtuelles.  

    <pre><code>
    # Create user group 
    sudo groupadd -g 1001 sapsys
    # Create  users 
    sudo useradd <b>hn1</b>adm -u 1001 -g 1001 -d /usr/sap/<b>HN1</b>/home -c "SAP HANA Database System" -s /bin/sh
    sudo useradd sapadm -u 1002 -g 1001 -d /home/sapadm -c "SAP Local Administrator" -s /bin/sh
    # Set the password  for both user ids
    sudo passwd hn1adm
    sudo passwd sapadm
    </code></pre>

6. **[A]** Montez les volumes Azure NetApp Files partagés.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.5:/<b>HN1</b>-data-mnt00001 /hana/data/<b>HN1</b>/mnt00001  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.23.1.6:/<b>HN1</b>-data-mnt00002 /hana/data/<b>HN1</b>/mnt00002  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.23.1.4:/<b>HN1</b>-log-mnt00001 /hana/log/<b>HN1</b>/mnt00001  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.23.1.6:/<b>HN1</b>-log-mnt00002 /hana/log/HN1/mnt00002  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.23.1.4:/<b>HN1</b>-shared/shared /hana/shared  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount all volumes
    sudo mount -a 
    </code></pre>

7. **[1]** Montez les volumes spécifiques aux nœuds sur **hanadb1**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb1</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

8. **[2]** Montez les volumes spécifiques aux nœuds sur **hanadb2**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb2</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

9. **[3]** Montez les volumes spécifiques aux nœuds sur **hanadb3**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb3</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

10. **[A]** Vérifiez que tous les volumes HANA sont montés avec la version de protocole NFS **NFSv4**.  

    <pre><code>
    sudo nfsstat -m
    # Verify that flag vers is set to <b>4.1</b> 
    # Example from <b>hanadb1</b>
    /hana/data/<b>HN1</b>/mnt00001 from 10.23.1.5:/<b>HN1</b>-data-mnt00001
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.5
    /hana/log/<b>HN1</b>/mnt00002 from 10.23.1.6:/<b>HN1</b>-log-mnt00002
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.6
    /hana/data/<b>HN1</b>/mnt00002 from 10.23.1.6:/<b>HN1</b>-data-mnt00002
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.6
    /hana/log/<b>HN1</b>/mnt00001 from 10.23.1.4:/<b>HN1</b>-log-mnt00001
    Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.4
    /usr/sap/<b>HN1</b> from 10.23.1.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb1</b>
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.4
    /hana/shared from 10.23.1.4:/<b>HN1</b>-shared/shared
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.4
    </code></pre>

## <a name="installation"></a>Installation  

Dans cet exemple, pour le déploiement de SAP HANA dans une configuration Scale-out avec un nœud de secours à l’aide d’Azure, nous avons utilisé HANA 2.0 SP4.  

### <a name="prepare-for-hana-installation"></a>Préparer l’installation de HANA

1. **[A]** Avant l’installation de HANA, définissez le mot de passe racine. Vous pouvez désactiver le mot de passe racine une fois l’installation terminée. Exécutez en tant que commande `root``passwd`.  

2. **[1]** Vérifiez que vous pouvez vous connecter via SSH sur **hanadb2** et **hanadb3** sans être invité à entrer un mot de passe.  

    <pre><code>
    ssh root@<b>hanadb2</b>
    ssh root@<b>hanadb3</b>
    </code></pre>

3. **[A]** Installez des packages supplémentaires, qui sont nécessaires pour HANA 2.0 SP4. Pour plus d’informations, consultez la note SAP [2593824](https://launchpad.support.sap.com/#/notes/2593824). 

    <pre><code>
    sudo zypper install libgcc_s1 libstdc++6 libatomic1 
    </code></pre>

4. **[2], [3]** Modifiez la propriété de SAP HANA des répertoires `data` et `log` pour **hn1** adm.   

    <pre><code>
    # Execute as root
    sudo chown hn1adm:sapsys /hana/data/<b>HN1</b>
    sudo chown hn1adm:sapsys /hana/log/<b>HN1</b>
    </code></pre>

### <a name="hana-installation"></a>Installation de HANA

1. **[1]** Installez SAP HANA en suivant les instructions fournies dans le [Guide d’installation et de mise à jour de SAP HANA 2.0](https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.04/en-US/7eb0167eb35e4e2885415205b8383584.html). Dans cet exemple, nous installons le Scale-out SAP HANA avec un maître, un Worker et un nœud de secours.  

   a. Démarrez le programme **hdblcm** à partir du répertoire du logiciel d’installation HANA. Utilisez le paramètre `internal_network` et transmettez l’espace d’adressage pour le sous-réseau, qui est utilisé pour la communication interne HANA entre les nœuds.  

    <pre><code>
    ./hdblcm --internal_network=10.23.3.0/24
    </code></pre>

   b. À l’invite, entrez les valeurs suivantes :

     * Pour **Choisir une action** : entrer **1** (pour l’installation)
     * Pour **Composants supplémentaires pour l’installation** : entrer **2, 3**
     * Pour le chemin d’accès de l’installation : appuyer sur Entrée (utilise par défaut /hana/shared)
     * Pour **Nom d’hôte local** : appuyer sur Entrée pour accepter la valeur par défaut
     * Sous **Voulez-vous ajouter des hôtes au système ?**  : entrer **y**
     * Pour **Noms d’hôte séparés par un virgule à ajouter** : entrer **hanadb2, hanadb3**
     * Pour **Nom d’utilisateur racine** [racine] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Mot de passe de l’utilisateur racine** : entrer le mot de passe de l’utilisateur racine
     * Pour les rôles pour l’hôte hanadb2 : entrer **1** (pour Worker)
     * Pour **Groupe de basculement hôte** pour l’hôte hanadb2 [par défaut] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Numéro de partition de stockage** pour l’hôte hanadb2 [<<assign automatically>>] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Groupe Worker** pour l’hôte hanadb2 [par défaut] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Sélectionner des rôles** pour l’hôte hanadb3 : entrer **2** (pour le serveur de secours)
     * Pour **Groupe de basculement hôte** pour l’hôte hanadb3 [par défaut] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Groupe Worker** pour l’hôte hanadb3 [par défaut] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **ID système SAP HANA** : entrer **HN1**
     * Pour **Numéro d’instance** [00] : entrer **03**
     * Pour **Groupe Worker hôte local** [par défaut] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Sélectionner Utilisation du système/Entrer l’index [4]**  : entrer **4** (pour personnalisé)
     * Pour **Emplacement des volumes de données** [/hana/data/HN1] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Emplacement des volumes de journaux** [/hana/log/HN1] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Restreindre l’allocation de mémoire maximale ?** [n] : entrer **n**
     * Pour **Nom d’hôte du certificat pour l’hôte hanadb1** [hanadb1] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Nom d’hôte du certificat pour l’hôte hanadb2** [hanadb2] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Nom d’hôte du certificat pour l’hôte hanadb3** [hanadb3] : appuyer sur Entrée pour accepter la valeur par défaut
     * Pour **Mot de passe de l’administrateur système (hn1adm)**  : entrer le mot de passe
     * Pour **Mot de passe utilisateur de la base de données système (système)**  : entrer le mot de passe du système
     * Pour **Confirmer le mot de passe utilisateur de la base de données système (système)**  : entrer le mot de passe du système
     * Pour **Redémarrer le système après le redémarrage de la machine ?** [n] : entrer **n** 
     * Pour **Voulez-vous continuer (y/n)**  : valider le résumé et si tout semble correct, entrer **y**


2. **[1]** vérifier global. ini  

   Affichez global.ini et assurez-vous que la configuration de la communication interne SAP HANA entre les nœuds est en place. Vérifiez la section **communication**. Elle doit comporter un espace d’adressage pour le sous-réseau `hana` et `listeninterface` doit être défini sur `.internal`. Vérifiez la section **internal_hostname_resolution**. Elle doit comporter les adresses IP des machines virtuelles HANA qui appartiennent au sous-réseau `hana`.  

   <pre><code>
    sudo cat /usr/sap/<b>HN1</b>/SYS/global/hdb/custom/config/global.ini
    # Example 
    #global.ini last modified 2019-09-10 00:12:45.192808 by hdbnameserve
    [communication]
    internal_network = <b>10.23.3/24</b>
    listeninterface = .internal
    [internal_hostname_resolution]
    <b>10.23.3.4</b> = <b>hanadb1</b>
    <b>10.23.3.5</b> = <b>hanadb2</b>
    <b>10.23.3.6</b> = <b>hanadb3</b>
   </code></pre>

3. **[1]** Ajoutez le mappage de l’hôte pour vous assurer que les adresses IP du client sont utilisées pour les communications des clients. Ajoutez la section `public_host_resolution`, puis les adresses IP correspondantes à partir du sous-réseau client.  

   <pre><code>
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    #Add the section
    [public_hostname_resolution]
    map_<b>hanadb1</b> = <b>10.23.0.5</b>
    map_<b>hanadb2</b> = <b>10.23.0.6</b>
    map_<b>hanadb3</b> = <b>10.23.0.7</b>
   </code></pre>

4. **[1]** Redémarrez SAP HANA pour activer les modifications.  

   <pre><code>
    sudo -u <b>hn1</b>adm /usr/sap/hostctrl/exe/sapcontrol -nr <b>03</b> -function StopSystem HDB
    sudo -u <b>hn1</b>adm /usr/sap/hostctrl/exe/sapcontrol -nr <b>03</b> -function StartSystem HDB
   </code></pre>

5. **[1]** Vérifiez que l’interface client utilise les adresses IP du sous-réseau `client` pour les communications.  

   <pre><code>
    sudo -u hn1adm /usr/sap/HN1/HDB03/exe/hdbsql -u SYSTEM -p "<b>password</b>" -i 03 -d SYSTEMDB 'select * from SYS.M_HOST_INFORMATION'|grep net_publicname
    # Expected result
    "<b>hanadb3</b>","net_publicname","<b>10.23.0.7</b>"
    "<b>hanadb2</b>","net_publicname","<b>10.23.0.6</b>"
    "<b>hanadb1</b>","net_publicname","<b>10.23.0.5</b>"
   </code></pre>

   Pour plus d’informations sur la vérification de la configuration, consultez la note SAP [2183363 – Configuration du réseau interne SAP HANA](https://launchpad.support.sap.com/#/notes/2183363).  

6. Afin d’optimiser SAP HANA pour le stockage Azure NetApp Files sous-jacent, définissez les paramètres SAP HANA suivants :

   - `max_parallel_io_requests` **128**
   - `async_read_submit`**on**
   - `async_write_submit_active`**on**
   - `async_write_submit_blocks` **all**

   Pour plus d’informations, consultez [Applications SAP NetApp sur Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure]. 

   À partir des systèmes SAP HANA 2.0, vous pouvez définir les paramètres dans `global.ini`. Pour plus d’informations, consultez la note SAP [1999930](https://launchpad.support.sap.com/#/notes/1999930).  
   
   Pour les systèmes SAP HANA 1.0, versions SPS12 et antérieures, ces paramètres peuvent être définis lors de l’installation, comme décrit dans la note SAP [2267798](https://launchpad.support.sap.com/#/notes/2267798).  

7. Le stockage utilisé par Azure NetApp Files présente une taille de fichier limitée à 16 To. SAP HANA n’est pas implicitement au courant de la limite de stockage et ne crée pas automatiquement de nouveau fichier de données lorsque la limite de taille de fichier de 16 To est atteinte. Comme SAP HANA tente d’augmenter le fichier au-delà de 16 To, cette tentative génère des erreurs et finit par provoquer un incident sur le serveur d’index. 

   > [!IMPORTANT]
   > Pour empêcher SAP HANA d’augmenter la taille des fichiers de données au-delà de la [limite des 16 To](../../../azure-netapp-files/azure-netapp-files-resource-limits.md) du sous-système de stockage, définissez les paramètres suivants dans `global.ini`.  
   > - datavolume_striping = true
   > - datavolume_striping_size_gb = 15000 Pour plus d’informations, consultez la note SAP [2400005](https://launchpad.support.sap.com/#/notes/2400005).
   > Tenez compte de la note SAP [2631285](https://launchpad.support.sap.com/#/notes/2631285). 

## <a name="test-sap-hana-failover"></a>Test de basculement SAP HANA 

> [!NOTE]
> Cet article contient des références aux termes *maître* et *esclave*, termes que Microsoft n’utilise plus. Lorsque ces termes seront supprimés du logiciel, nous les supprimerons de cet article.

1. Simulez un incident de nœud sur un nœud Worker SAP HANA. Effectuez les actions suivantes : 

   a. Avant de simuler l’incident du nœud, exécutez les commandes suivantes en tant que **hn1** adm pour capturer l’état de l’environnement :  

   <pre><code>
    # Check the landscape status
    python /usr/sap/<b>HN1</b>/HDB<b>03</b>/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | yes    | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
    # Check the instance status
    sapcontrol -nr <b>03</b>  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
   </code></pre>

   b. Pour simuler un incident de nœud, exécutez la commande suivante en tant que racine sur le nœud Worker, en l’occurrence **hanadb2** :  
   
   <pre><code>
    echo b > /proc/sysrq-trigger
   </code></pre>

   c. Surveillez le système pour la fin du basculement. Une fois le basculement terminé, capturez l’état, lequel doit ressembler à ce qui suit :  

    <pre><code>
    # Check the instance status
    sapcontrol -nr <b>03</b>  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GRAY
    # Check the landscape status
    /usr/sap/HN1/HDB03/exe/python_support> python landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | no     | info   |          |        |         2 |         0 | default  | default  | master 2   | slave      | worker      | standby     | worker  | standby | default | -       |
    | hanadb3 | yes    | info   |          |        |         0 |         2 | default  | default  | master 3   | slave      | standby     | slave       | standby | worker  | default | default |
   </code></pre>

   > [!IMPORTANT]
   > Lorsqu’un nœud subit une panique du noyau, évitez les retards de basculement SAP HANA en paramétrant `kernel.panic` sur 20 secondes sur *toutes* les machines virtuelles HANA. La configuration est effectuée dans `/etc/sysctl`. Redémarrez les machines virtuelles pour activer la modification. Si cette modification n’est pas effectuée, le basculement peut prendre 10 minutes ou plus lorsqu’un nœud subit une panique du noyau.  

2. Tuez le serveur de noms en procédant comme suit :

   a. Avant le test, vérifiez l’état de l’environnement en exécutant les commandes suivantes en tant que **hn1** adm :  

   <pre><code>
    #Landscape status 
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | no     | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GRAY
   </code></pre>

   b. Exécutez les commandes suivantes en tant que **hn1** adm sur le nœud principal actif, en l’occurrence **hanadb1** :  

    <pre><code>
        hn1adm@hanadb1:/usr/sap/HN1/HDB03> HDB kill
    </code></pre>
    
    Le nœud de secours **hanadb3** prend la forme d’un nœud principal. Voici l’état des ressources après la fin du test de basculement :  

    <pre><code>
        # Check the instance status
        sapcontrol -nr 03 -function GetSystemInstanceList
        GetSystemInstanceList
        OK
        hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
        hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
        hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GRAY
        hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
        # Check the landscape status
        python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
        | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
        |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
        |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
        | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
        | hanadb1 | no     | info   |          |        |         1 |         0 | default  | default  | master 1   | slave      | worker      | standby     | worker  | standby | default | -       |
        | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
        | hanadb3 | yes    | info   |          |        |         0 |         1 | default  | default  | master 3   | master     | standby     | master      | standby | worker  | default | default |
    </code></pre>

   c. Redémarrez l’instance HANA sur **hanadb1** (c’est-à-dire sur la même machine virtuelle où le serveur de noms a été tué). Le nœud **hanadb1** rejoindra l’environnement et conservera son rôle de secours.  

   <pre><code>
    hn1adm@hanadb1:/usr/sap/HN1/HDB03> HDB start
   </code></pre>

   Une fois que SAP HANA a démarré sur **hanadb1**, attendez-vous à l’état suivant :  

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03 -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | info   |          |        |         1 |         0 | default  | default  | master 1   | slave      | worker      | standby     | worker  | standby | default | -       |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | yes    | info   |          |        |         0 |         1 | default  | default  | master 3   | master     | standby     | master      | standby | worker  | default | default |
   </code></pre>

   d. Tuez à nouveau le serveur de noms sur le nœud principal actuellement actif (autrement dit, sur le nœud **hanadb3**).  
   
   <pre><code>
    hn1adm@hanadb3:/usr/sap/HN1/HDB03> HDB kill
   </code></pre>

   Le nœud **hanadb1** va reprendre le rôle du nœud principal. Une fois le test de basculement terminé, l’état se présente comme suit :

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList & python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GRAY
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | no     | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
   </code></pre>

   e. Démarrez SAP HANA sur **hanadb3**, lequel sera prêt à faire office de nœud de secours.  

   <pre><code>
    hn1adm@hanadb3:/usr/sap/HN1/HDB03> HDB start
   </code></pre>

   Une fois que SAP HANA a démarré sur **hanadb3**, l’état ressemble à ce qui suit :  

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList & python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GRAY
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | no     | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
   </code></pre>

## <a name="next-steps"></a>Étapes suivantes

* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur des machines virtuelles Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha].
