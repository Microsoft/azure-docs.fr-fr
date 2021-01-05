---
title: Check-list relative à la planification et au déploiement de la charge de travail SAP | Microsoft Docs
description: Check-list de planification des déploiements de la charge de travail SAP vers Azure et de déploiement des charges de travail
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/10/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 06d20dd47ceb71b51e226d662892eab06b072685
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500935"
---
# <a name="sap-workloads-on-azure-planning-and-deployment-checklist"></a>Check-list relative à la planification et au déploiement de la charge de travail SAP sur Azure

Cette check-list s’adresse aux clients qui migrent des applications SAP NetWeaver, S/4HANA et Hybris vers l’IaaS (infrastructure as a service) Azure. Pendant toute la durée du projet, un client et/ou un partenaire SAP doit examiner la check-list. Il est important de noter que la plupart des vérifications doivent être effectuées en début de projet et durant la phase de planification. À l’issue du déploiement, toute modification apportée directement à l’infrastructure Azure déployée ou aux versions des logiciels SAP peut s’avérer complexe.

Passez en revue la check-list aux étapes les plus importantes de votre projet. Cela vous permet de détecter les problèmes avant qu’ils ne prennent de l’ampleur. Vous aurez également suffisamment de temps pour remanier et tester les modifications nécessaires. Cette check-list n’est pas exhaustive. Selon votre situation, il se peut que vous deviez effectuer beaucoup plus de vérifications.

La check-list proposée n’inclut pas les tâches réalisées indépendamment d’Azure. Par exemple, les interfaces des applications SAP changent lors d’un transfert vers la plateforme Azure ou vers un fournisseur d’hébergement.

Cette check-list peut également être utilisée pour les systèmes déjà déployés. De nouvelles fonctionnalités, telles que l’Accélérateur d’écriture, les Zones de disponibilité et les nouveaux types de machines virtuelles, peuvent avoir été ajoutées depuis le déploiement. Il est donc utile d’examiner régulièrement la check-list pour vous tenir informé des nouvelles fonctionnalités ajoutées à la plateforme Azure.

## <a name="project-preparation-and-planning-phase"></a>Phase de préparation et de planification du projet
Au cours de cette phase, vous planifiez la migration de votre charge de travail SAP vers la plateforme Azure. Au minimum, au cours de cette phase, vous devez créer les documents suivants et définir et aborder les éléments suivants de la migration :

1. Document de conception de haut niveau. Ce document doit contenir :
    - L’inventaire actuel des composants et applications SAP et un inventaire des applications cibles pour Azure.
    - Une matrice d’attribution des responsabilités (RACI) qui définit les responsabilités et les attributions des parties impliquées. Démarrez à un niveau élevé et évoluez vers des niveaux plus granulaires pour la planification et les premiers déploiements.
    - Une architecture de solution de haut niveau.
    - Une décision sur les régions Azure vers lesquelles effectuer le déploiement. Consultez la [liste des régions Azure](https://azure.microsoft.com/global-infrastructure/regions/). Pour savoir quels services sont disponibles dans chaque région, consultez [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/).
    - Une architecture de mise en réseau pour la connexion à Azure à partir d’une instance locale. Commencez par vous familiariser avec le [blueprint Centre de données virtuel pour Azure](/azure/architecture/vdc/).
    - Les principes de sécurité pour l’exécution des données à impact commercial élevé dans Azure. Pour en savoir plus sur la sécurité des données, commencez par la [documentation sur la sécurité Azure](../../../security/index.yml).
2.  Document de conception technique. Ce document doit contenir :
    - Un schéma fonctionnel de la solution.
    - Le dimensionnement des composants de calcul, de stockage et de mise en réseau dans Azure. Pour le dimensionnement SAP des machines virtuelles Azure, consultez [SAP 
    -  note #1928533](https://launchpad.support.sap.com/#/notes/1928533).
    - L’architecture de continuité d’activité et de reprise d’activité.
    - Informations détaillées sur les versions des packs de support SE, DB, noyau et SAP. Toutes les versions de SE prises en charge par SAP NetWeaver ou S/4HANA ne sont pas nécessairement prises en charge sur les machines virtuelles Azure. Il en va de même pour les versions de SGBD. Vérifiez les sources suivantes pour aligner et, si nécessaire, mettre à niveau les versions de SAP, les versions de SGBD et les versions de SE afin de garantir le support SAP et Azure. Vous devez disposer de combinaisons de mises en production prises en charge par SAP et Azure pour bénéficier du support complet de SAP et Microsoft. Si nécessaire, planifiez la mise à niveau de certains composants logiciels. Vous trouverez plus de détails sur les logiciels SAP, SE et SGBD pris en charge ici :
        - [Note de support SAP n° 1928533](https://launchpad.support.sap.com/#/notes/1928533). Cette note définit les versions minimum des SE prises en charge sur les machines virtuelles Azure. Elle définit également les versions minimum requises pour la plupart des bases de données non HANA. Enfin, elle indique le dimensionnement SAP pour les types de machines virtuelles Azure pris en charge par SAP.
        - [Note de support SAP n° 2015553](https://launchpad.support.sap.com/#/notes/2015553). Cette note définit des stratégies de support concernant le stockage Azure et la relation de support nécessaire avec Microsoft.
        - [Note de support SAP n° 2039619](https://launchpad.support.sap.com/#/notes/2039619). Cette note définit la matrice de prise en charge Oracle pour Azure. Windows et Oracle Linux sont les seuls systèmes d’exploitation pris en charge par Oracle en tant que systèmes d’exploitation invités dans les charges de travail Azure pour SAP. Cette instruction s’applique également à la couche Application SAP qui exécute les instances SAP. Toutefois, Oracle ne prend pas en charge la haute disponibilité pour les services SAP Central Services dans Oracle Linux par le biais de Pacemaker. Si vous avez besoin de la haute disponibilité pour ASCS sur Oracle Linux, vous devez utiliser SIOS Protection Suite pour Linux. Pour obtenir des données de certification SAP détaillées, consultez la note de support SAP [n° 1662610 - Détails de prise en charge pour SIOS Protection Suite pour Linux](https://launchpad.support.sap.com/#/notes/1662610). Pour Windows, la solution Clustering de basculement Windows Server prise en charge par SAP pour SAP Central Services est prise en charge conjointement avec Oracle comme couche SGBD.
        - [Note de support SAP n° 2235581](https://launchpad.support.sap.com/#/notes/2235581). Cette note fournit la matrice de prise en charge de SAP HANA sur les différentes versions de SE.
        - Les machines virtuelles Azure prises en charge par SAP HANA et les [Grandes instances HANA](./hana-overview-architecture.md) sont répertoriées sur le [site web de SAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure).
        - [Tableau de disponibilité des produits SAP](https://support.sap.com/en/).
        - [Note de support SAP n° 2555629 - SAP HANA 2.0 Dynamic Tiering – Prise en charge de l’hyperviseur et du cloud](https://launchpad.support.sap.com/#/notes/2555629)
        - [Note de support SAP n° 1662610 - Détails de prise en charge pour SIOS Protection Suite pour Linux](https://launchpad.support.sap.com/#/notes/1662610)
        - Notes SAP pour d’autres produits SAP spécifiques.     
    - Azure prend en charge l’utilisation de configurations de cluster multi-SID pour SAP Central Services sur les systèmes d’exploitation invités Windows, SLES et RHEL. Gardez à l’esprit que plus vous placez d’instances ASCS/SCS sur un tel cluster multi-SID, plus le rayon d’impact peut augmenter. Vous trouverez la documentation du scénario du système d’exploitation invité correspondant dans les articles suivants :
        - [Haute disponibilité multi-SID de l’instance SAP ASCS/SCS avec le clustering de basculement Windows Server et un disque partagé sur Azure](./sap-ascs-ha-multi-sid-wsfc-shared-disk.md)
        - [Haute disponibilité multi-SID de l’instance SAP ASCS/SCS avec le clustering de basculement Windows Server et le partage de fichiers sur Azure](./sap-ascs-ha-multi-sid-wsfc-file-share.md)
        - [Guide de haute disponibilité multi-SID pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse-multi-sid.md)
        - [Guide multi-SID de haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur Red Hat Enterprise Linux for SAP Applications](./high-availability-guide-rhel-multi-sid.md)
    - Architecture de haute disponibilité et de récupération d’urgence.
        - En fonction des objectifs RTO et RPO, définissez l’architecture de haute disponibilité et de récupération d’urgence.
        - Pour une haute disponibilité dans une zone, vérifiez ce que le SGBD souhaité peut offrir dans Azure. La plupart des packages SGBD offrent les méthodes synchrones d’un serveur de secours synchrone, ce que nous recommandons pour les systèmes de production. Consultez aussi la documentation SAP pour les différentes bases de données en commençant par [Facteurs à prendre en compte pour le déploiement SGBD de machines virtuelles Azure pour les charges de travail SAP](./dbms_guide_general.md) et les documents connexes.
           L’utilisation du Clustering de basculement Windows Server avec une configuration de disque partagé pour la couche SGBD comme [décrit pour SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server?view=sql-server-2017), par exemple, n’est pas prise en charge. Utilisez plutôt des solutions comme :
           - [SQL Server AlwaysOn](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups)
           - [Oracle Data Guard](../oracle/configure-oracle-dataguard.md)
           - [Réplication de système HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html)
        - Pour la récupération d’urgence dans les régions d’Azure, passez en revue les solutions offertes par les différents fournisseurs de SGBD. La plupart d’entre eux prennent en charge la réplication asynchrone ou la copie des journaux de transaction.
        - Pour la couche Application SAP, déterminez si vous souhaiterez exécuter vos systèmes de test de régression métier, qui sont idéalement des réplicas de vos déploiements de production, dans la même région Azure ou dans la région de votre récupération d’urgence. Dans le deuxième cas, vous pouvez désigner ce système de régression métier comme cible de la récupération d’urgence pour vos déploiements en production.
        - Si vous décidez de ne pas placer de systèmes hors production sur le site de reprise d’activité, envisagez Azure Site Recovery comme méthode de réplication de la couche Application SAP dans la région Azure de reprise d’activité. Pour plus d’informations, consultez [Configurer la reprise d’activité pour un déploiement d’application SAP NetWeaver multiniveau](../../../site-recovery/site-recovery-sap.md).
        - Si vous décidez d’utiliser une configuration HADR combinée à l’aide de [Zones de disponibilité Azure](../../../availability-zones/az-overview.md), familiarisez-vous avec les régions Azure où les zones de disponibilité sont disponibles. Prenez également en compte les restrictions liées à l’augmentation des latences du réseau entre deux Zones de disponibilité.  
3.  Inventaire de toutes les interfaces SAP (SAP et non SAP).
4.  Conception des services de base. Cette conception doit inclure les éléments suivants :
    - Conception Active Directory et DNS.
    - Topologie de réseau dans Azure et affectation de différents systèmes SAP.
    - Structure de[contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../../role-based-access-control/overview.md) pour les équipes afin d’assurer la gestion de l’infrastructure et des applications SAP dans Azure.
    - Topologie des groupes de ressources.
    - [Stratégie de balisage](../../../azure-resource-manager/management/tag-resources.md#tags-and-billing).
    - Convention d’affectation de noms pour les machines virtuelles, et autres composants d’infrastructure et/ou noms logiques.
5.  Contrat de support Premier ou Professionnel Microsoft. Identifiez votre gestionnaire de compte technique (TAM) Microsoft si vous disposez d’un contrat de support Premier avec Microsoft. Pour connaître les exigences de support SAP, lisez la [note de support SAP n° 2015553](https://launchpad.support.sap.com/#/notes/2015553).
6.  Nombre d’abonnements Azure et quota principal pour les abonnements. Au besoin, [ouvrez des demandes de support pour augmenter les quotas d’abonnements Azure](../../../azure-portal/supportability/resource-manager-core-quotas-request.md).
7.  Plan de réduction et de migration des données pour la migration des données SAP vers Azure. Pour les systèmes SAP NetWeaver, SAP propose des directives afin de limiter les gros volumes de données. Consultez [ce guide SAP](https://wiki.scn.sap.com/wiki/download/attachments/247399467/DVM_%20Guide_7.2.pdf?version=1&modificationDate=1549365516000&api=v2) consacré à la gestion des données dans les systèmes SAP ERP. Certains de ces contenus s’appliquent également aux systèmes NetWeaver et S/4HANA en général.
8.  Une approche de type déploiement automatisé. L’objectif de l’automatisation des déploiements d’infrastructures sur Azure est de déployer de manière déterministe et d’obtenir des résultats déterministes. Beaucoup de clients utilisent des scripts basés sur PowerShell ou sur l’interface CLI. Cependant, différentes technologies open source vous permettent de déployer une infrastructure Azure pour SAP, et même pour installer les logiciels SAP. Vous trouverez des exemples sur GitHub :
    - [Déploiements SAP automatisés dans le cloud Azure](https://github.com/Azure/sap-hana)
    - [Installation de SAP HANA](https://github.com/AzureCAT-GSI/SAP-HANA-ARM)
9.  Définissez une cadence régulière de révision de la conception et du déploiement entre vous, en tant que client, l’intégrateur système, Microsoft et les autres parties concernées.


## <a name="pilot-phase-strongly-recommended"></a>Phase pilote (fortement recommandée)
 
Vous pouvez exécuter un pilote avant ou pendant la planification et la préparation du projet. Vous pouvez également utiliser la phase pilote pour tester les approches et la conception proposées durant la phase de planification et de préparation. Vous pouvez développer la phase pilote pour en faire une véritable preuve de concept.

Nous vous recommandons de configurer et de valider une solution complète HADR et une conception de sécurité lors d’un déploiement pilote. Certains clients effectuent des tests d’extensibilité au cours de cette phase. D’autres clients ont recours aux déploiements de systèmes « bac à sable » SAP pour une phase pilote. Nous supposons donc que vous avez identifié un système que vous souhaitez migrer vers Azure pour le pilote.

1. Optimisez le transfert de données vers Azure. Le choix optimal dépend étroitement du scénario. Le transfert à partir d’un site local via [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/) est le plus rapide si le circuit ExpressRoute dispose d’une bande passante suffisante. Dans d’autres scénarios, le transfert via Internet est plus rapide.
2. Pour une migration de plateforme SAP hétérogène qui implique l’exportation et l’importation des données, testez et optimisez les phases d’exportation et d’importation. Pour les migrations volumineuses dans lesquelles SQL Server est la plateforme de destination, vous pouvez trouver des [recommandations](https://techcommunity.microsoft.com/t5/Running-SAP-Applications-on-the/SAP-OS-DB-Migration-to-SQL-Server-8211-FAQ-v6-2-April-2017/ba-p/368070). Vous pouvez utiliser le moniteur de migration/SWPM si vous n’avez pas besoin d’une mise à niveau de version combinée. Vous pouvez utiliser le processus [SAP DMO](https://blogs.sap.com/2013/11/29/database-migration-option-dmo-of-sum-introduction/) lorsque vous combinez la migration à une mise à niveau de version SAP. Pour ce faire, vous devez répondre à certaines exigences pour la combinaison de plateformes SGBD source et cible. Ce processus est documenté dans l’[option DMO (Database Migration Option) de SUM 2.0 SP03](https://launchpad.support.sap.com/#/notes/2631152).
   1.  Exporter vers la source, Exporter le fichier chargé vers Azure et Performances d’importation. Optimisez le chevauchement entre l’exportation et l’importation.
   2.  Évaluez le volume de la base de données sur les plateformes cible et de destination dans le cadre du dimensionnement de l’infrastructure.
   3.  Validez et optimisez le timing.
1. Validation technique.
   1. Types de machines virtuelles.
        - Passez en revue les ressources dans les notes de support SAP, dans le répertoire matériel SAP HANA et dans SAP PAM. Assurez-vous qu’aucune modification n’est apportée aux machines virtuelles prises en charge pour Azure, aux versions de système d’exploitation prises en charge pour ces types de machines virtuelles et aux versions SAP et de SGBD.
        - Validez à nouveau le dimensionnement de votre application et de l'infrastructure que vous déployez sur Azure. Si vous déplacez des applications existantes, vous pouvez souvent extraire le SAPS nécessaire de l’infrastructure utilisée et de la [page web du benchmark SAP](https://www.sap.com/dmc/exp/2018-benchmark-directory/#/sd) et le comparer avec les numéros de SAPS répertoriés sur la [note de support SAP n° 1928533](https://launchpad.support.sap.com/#/notes/1928533). Consultez également [cet article sur les évaluations SAPS](https://techcommunity.microsoft.com/t5/Running-SAP-Applications-on-the/SAPS-ratings-on-Azure-VMs-8211-where-to-look-and-where-you-can/ba-p/368208).
        - Évaluez et testez le dimensionnement de vos machines virtuelles Azure en fonction du débit de stockage et du débit réseau maximum des types de machines virtuelles que vous avez choisis durant la phase de planification. Les données sont disponibles ici :
           -  [Tailles des machines virtuelles Windows dans Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Il est important de tenir compte du *débit maximum du disque non mis en cache* pour le dimensionnement.
           -  [Tailles des machines virtuelles Linux dans Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Il est important de tenir compte du *débit maximum du disque non mis en cache* pour le dimensionnement.
   2. Stockage.
        - Vérifiez le document [Types de stockage Azure pour une charge de travail SAP](./planning-guide-storage.md).
        - Utilisez au minimum le [stockage Disque SSD Standard Azure](../../disks-types.md#standard-ssd) pour les machines virtuelles représentant des couches Application SAP et pour un déploiement de SGBD non sensibles aux performances.
        - En général, nous déconseillons l’utilisation de [disques HDD Standard Azure](../../disks-types.md#standard-hdd).
        - Utilisez [Stockage Premium Azure](../../disks-types.md#premium-ssd) pour toutes les machines virtuelles SGBD qui sont sensibles aux performances à distance.
        - Utilisez [Azure Disques managés](https://azure.microsoft.com/services/managed-disks/).
        - Utilisez l'Accélérateur d'écriture Azure pour les lecteurs de journaux SGBD avec M-Series. Tenez compte des limites et de l’utilisation de l’Accélérateur d’écriture décrites dans [Accélérateur d’écriture](../../how-to-enable-write-accelerator.md).
        - Pour les différents types de SGBD, consultez la [documentation générique relative au SGBD lié à SAP](./dbms_guide_general.md) et la documentation spécifique au SGBD auxquelles le document générique vous renvoie.
        - Pour plus d’informations sur SAP HANA, consultez [Configurations et opérations de l’infrastructure SAP HANA sur Azure](./hana-vm-operations.md).
        - Ne montez jamais de disques de données Azure dans une machine virtuelle Azure Linux en utilisant l’ID d’appareil. À la place, utilisez l’identificateur unique universel (UUID). Soyez prudent, par exemple lorsque vous utilisez des outils graphiques pour monter des disques de données Azure. Examinez les entrées dans /etc/fstab pour vérifier que l’UUID est utilisé pour le montage des disques. Vous trouverez plus d’informations dans [cet article](../../linux/attach-disk-portal.md#connect-to-the-linux-vm-to-mount-the-new-disk).
   3. Mise en réseau.
        - Testez et évaluez votre infrastructure de réseau virtuel et la distribution de vos applications SAP sur les différents réseaux virtuels Azure.
        -  Évaluez l’approche de l’architecture de réseau virtuel en étoile ou l’approche de la microsegmentation au sein d’un réseau virtuel Azure unique. Basez cette évaluation sur les éléments suivants :
               1. Coûts liés à l’échange de données entre des [réseaux virtuels Azure appairés](../../../virtual-network/virtual-network-peering-overview.md). Pour plus d’informations sur les coûts, consultez [Tarification des réseaux virtuels](https://azure.microsoft.com/pricing/details/virtual-network/).
               2. Avantages d’une déconnexion rapide de l’appairage entre les réseaux virtuels Azure par rapport à un changement de groupe de sécurité réseau pour isoler un sous-réseau dans un réseau virtuel. Cette évaluation concerne les cas où les applications ou les machines virtuelles hébergées dans un sous-réseau du réseau virtuel constituent un risque pour la sécurité.
                3. Journalisation et audit centralisés du trafic réseau entre le site local, le monde extérieur et le centre de données virtuel que vous avez créé dans Azure.
        - Évaluez et testez le chemin des données entre la couche application SAP et la couche SGBD SAP.
            -  Le placement d’[appliances virtuelles réseau Azure](https://azure.microsoft.com/solutions/network-appliances/) sur le chemin de communication entre l’application SAP et la couche SGBD des systèmes SAP basés sur SAP NetWeaver, Hybris ou S/4HANA n’est pas pris en charge.
            -  Le placement des couches Application SAP et SGBD SAP sur différents réseaux virtuels Azure non appairés n’est pas pris en charge.
            -  Vous pouvez utiliser des [règles de groupe de sécurité d’application et de groupe de sécurité réseau](../../../virtual-network/network-security-groups-overview.md) pour définir les itinéraires entre la couche d’application SAP et la couche SGBD SAP.
        - Assurez-vous que la [mise en réseau accélérée Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) est activée sur les machines virtuelles utilisées dans la couche application SAP et la couche SGBD SAP. N'oubliez pas que différents niveaux de SE sont nécessaires pour prendre en charge la mise en réseau accélérée dans Azure :
            - Windows Server 2012 R2 ou ultérieur.
            - SUSE Linux 12 SP3 ou version ultérieure.
            - RHEL 7.4 ou version ultérieure.
            - Oracle Linux 7.5. Si vous utilisez le noyau RHCKL, la version 3.10.0-862.13.1. EL7 est requise. Si vous utilisez le noyau Oracle UEK, la version 5 est requise.
        - Testez et évaluez la latence du réseau entre les machines virtuelles de la couche Application SAP et celles de la couche SGBD, conformément aux notes de support SAP [n° 500235](https://launchpad.support.sap.com/#/notes/500235) et [n° 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). Évaluez les résultats par rapport aux indications de latence du réseau de la [note de support SAP n° 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). La latence du réseau doit être modérée ou bonne. Des exceptions s’appliquent au trafic entre les machines virtuelles et les grandes instances HANA, comme indiqué dans la documentation disponible dans [cet article](./hana-network-architecture.md#networking-architecture-for-hana-large-instance).
        - Assurez-vous que les déploiements ILB sont configurés pour utiliser le retour direct du serveur. Ce paramètre réduira la latence lorsque des ILB Azure sont utilisés pour les configurations à haute disponibilité sur la couche SGBD.
        - Si vous utilisez Azure Load Balancer avec des systèmes d’exploitation invité Linux, vérifiez que le paramètre réseau Linux **net.ipv4.tcp_timestamps** a la valeur **0**. Cette suggestion est incompatible avec les suggestions des anciennes versions de la [note SAP n° 2382421](https://launchpad.support.sap.com/#/notes/2382421). La note SAP mise à jour indique que ce paramètre doit être défini sur **0** pour fonctionner avec les équilibreurs de charge Azure.
        - Envisagez d’utiliser des [groupes de placement de proximité Azure](../../linux/co-location.md) pour bénéficier d’une latence réseau optimale. Pour plus d’informations, consultez [Groupes de placements de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md).
   4. Déploiements de haute disponibilité et de récupération d’urgence.
        - Si vous déployez la couche Application SAP sans définir de zone de disponibilité Azure spécifique, assurez-vous que toutes les machines virtuelles qui exécutent des instances de dialogue SAP ou des instances intergiciel d’un système SAP unique sont déployées dans un [groupe à haute disponibilité](../../manage-availability.md).
        - Si vous n’avez pas besoin de la haute disponibilité pour les services SAP Central Services et le SGBD, vous pouvez déployer ces machines virtuelles dans le même groupe à haute disponibilité que la couche Application SAP.
        - Si vous protégez les services SAP Central Services et la couche SGBD pour bénéficier de la haute disponibilité avec la réplication passive, placez les deux nœuds SAP Central Services dans un groupe à haute disponibilité distinct et les deux nœuds SGBD dans un autre groupe à haute disponibilité.
        - Si vous procédez à des déploiements dans des Zones de disponibilité Azure, vous ne pouvez pas utiliser de groupes à haute disponibilité. Toutefois, vous devez veiller à déployer les nœuds Central Services actifs et passifs dans deux zones de disponibilité différentes. Utilisez les zones de disponibilité ayant la latence la plus faible entre elles.
          N’oubliez pas que vous devez utiliser [Azure Standard Load Balancer](../../../load-balancer/load-balancer-standard-availability-zones.md) si vous devez établir des clusters de basculement Windows ou Pacemaker pour la couche SGBD et SAP Central Services dans les zones de disponibilité. Évitez d’utiliser l’[équilibreur de charge de base](../../../load-balancer/load-balancer-overview.md) pour les déploiements zonaux.
   5. Paramètres liés au délai d’expiration.
        - Consultez les rapports des développeurs de SAP NetWeaver pour les instances SAP pour vous assurer qu’aucune rupture de connexion n’existe entre le serveur de mise en file d’attente et les processus de travail SAP. Vous pouvez éviter ces ruptures de connexion en définissant les deux paramètres de registre suivants :
            - HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\KeepAliveTime = 120000. Pour plus d’informations, consultez [KeepAliveTime](/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10)).
            - HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\KeepAliveInterval = 120000. Pour plus d’informations, consultez [KeepAliveInterval](/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)).
        - Pour éviter les expirations de délai d’attente entre les interfaces GUI et les couches Application SAP locales déployées dans Azure, vérifiez que ces paramètres sont définis dans le fichier default.pfl ou dans le profil de l’instance :
            - rdisp/keepalive_timeout = 3600
            - rdisp/keepalive = 20
        - Pour éviter toute interruption des connexions établies entre le processus de mise en file d’attente SAP et les processus de travaux SAP, vous devez définir le paramètre **enque/encni/set_so_keepalive** sur **true**. Voir aussi la [note SAP n° 2743751](https://launchpad.support.sap.com/#/notes/2743751).  
        - Si vous utilisez une configuration de cluster de basculement Windows, assurez-vous que le temps de réaction relatifs aux nœuds non réactifs est correctement défini pour Azure. L’article [Réglage des seuils réseau des clusters de basculement](https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834) répertorie les paramètres et leur effet sur la sensibilité de basculement. En supposant que les nœuds de cluster sont dans le même sous-réseau, vous devez modifier ces paramètres :
            - SameSubNetDelay = 2000
            - SameSubNetThreshold = 15
            - RoutingHistorylength = 30
    6. Paramètres ou correctifs du système d’exploitation
        - Pour exécuter HANA sur SAP, lisez les notes et les documents suivants :
            -   [Note de support SAP n° 2814271 - La sauvegarde SAP HANA échoue sur Azure avec une erreur de somme de contrôle](https://launchpad.support.sap.com/#/notes/2814271)
            -   [Note de support SAP n° 2753418 - Dégradation potentielle des performances en raison du repli du minuteur](https://launchpad.support.sap.com/#/notes/2753418)
            -   [Note de support SAP n° 2791572 - Dégradation des performances en raison de la prise en charge VDSO manquante pour Hyper-V dans Azure](https://launchpad.support.sap.com/#/notes/2791572)
            -   [Note de support SAP n° 2382421 - Optimisation de la configuration réseau au niveau HANA et du système d’exploitation](https://launchpad.support.sap.com/#/notes/2382421)
            -   [Note de support SAP n° 2694118 - Module complémentaire de haute disponibilité Red Hat Enterprise Linux sur Azure](https://launchpad.support.sap.com/#/notes/2694118)
            -   [Note de support SAP n° 1984787 - SUSE Linux Enterprise Server 12 : notes d’installation](https://launchpad.support.sap.com/#/notes/1984787)
            -   [Note de support SAP #2002167 - Red Hat Enterprise Linux 7.x : installation et mise à niveau](https://launchpad.support.sap.com/#/notes/0002002167)
            -   [Note de support SAP #2292690 - SAP HANA DB : Paramètres de système d’exploitation recommandés pour RHEL 7](https://launchpad.support.sap.com/#/notes/0002292690)
            -   [Note de support SAP n° 2772999 - Red Hat Enterprise Linux 8.x : installation et configuration](https://launchpad.support.sap.com/#/notes/2772999)
            -   [Note de support SAP n° 2777782 - SAP HANA DB : paramètres de système d’exploitation recommandés pour RHEL 8](https://launchpad.support.sap.com/#/notes/2777782)
            -   [Note de support SAP n° 2578899 - SUSE Linux Enterprise Server 15 : note d’installation](https://launchpad.support.sap.com/#/notes/2578899)
            -   [Note de support SAP n° 2455582 – Linux : Exécution d’applications SAP compilées avec GCC 6.x](https://launchpad.support.sap.com/#/notes/0002455582)
            -    [Note de support SAP n° 2729475 - Échec de HWCCT avec une erreur de type « L’hyperviseur n’est pas pris en charge » sur les machines virtuelles Azure certifiées pour SAP HANA](https://launchpad.support.sap.com/#/notes/2729475)
1. Testez vos procédures de haute disponibilité et de récupération d’urgence.
   1. Simulez les situations de basculement en arrêtant les machines virtuelles (systèmes d’exploitation invités Windows) ou en plaçant les systèmes d’exploitation en mode urgence (systèmes d’exploitation invités Linux). Cette étape vous permet de déterminer si vos configurations de basculement fonctionnent comme prévu.
   1. Mesurez le temps nécessaire à l’exécution d’un basculement. Si les délais sont trop longs, tenez compte des éléments suivants :
        - Pour SUSE Linux, utilisez des périphériques SBD plutôt que l’agent Azure Fence afin d’accélérer le basculement.
        - Pour SAP HANA, si le rechargement des données prend trop de temps, songez à augmenter la bande passante de stockage.
   3. Testez votre séquence de sauvegarde/restauration et le minutage, et apportez les corrections adéquates si nécessaire. Vérifiez que les temps de sauvegarde sont suffisants. Vous devez également tester les activités de restauration et de restauration dans le temps. Vérifiez que les temps de restauration sont dans les objectifs RTO de vos contrats SLA, que votre RTO dépende d’une base de données ou d’un processus de restauration de machine virtuelle.
   4. Testez l’architecture et la fonctionnalité de récupération d’urgence inter-régions.
1. Vérifications de sécurité.
   1. Testez la validité de votre architecture de contrôle d’accès en fonction du rôle Azure (Azure RBAC). L’objectif est de séparer et de limiter l’accès et les autorisations des différentes équipes. Par exemple, les membres de l’équipe SAP Basis doivent pouvoir déployer des machines virtuelles et assigner des disques du Stockage Azure à un réseau virtuel Azure donné. Cependant, l’équipe SAP Basis ne doit pas être en mesure de créer ses propres réseaux virtuels ou de modifier les paramètres des réseaux virtuels existants. Les membres de l’équipe réseau ne doivent pas être en mesure de déployer des machines virtuelles sur des réseaux virtuels dans lesquels des machines virtuelles sont exécutées sur les couches Application SAP et SGBD. Les membres de cette équipe ne doivent pas non plus être en mesure de modifier les attributs des machines virtuelles, ni même de supprimer des machines virtuelles ou des disques.  
   1.  Vérifiez que les [règles du groupe de sécurité et les règles ASC](../../../virtual-network/network-security-groups-overview.md) fonctionnent comme prévu et protègent les ressources protégées.
   1.  Assurez-vous que toutes les ressources qui doivent être chiffrées le sont. Définissez et implémentez des processus pour sauvegarder les certificats, les stocker, y accéder et restaurer les entités chiffrées.
   1.  Utilisez [Azure Disk Encryption](../../../security/fundamentals/azure-disk-encryption-vms-vmss.md) pour les disques de système d’exploitation, si possible du point de vue de la prise en charge du système d’exploitation.
   1.  Veillez à ne pas utiliser un nombre excessif de couches de chiffrement. Dans certains cas, il est logique d’utiliser Azure Disk Encryption avec l’une des méthodes TDE (Transparent Data Encryption) SGBD afin de protéger différents disques ou composants sur le même serveur.  Par exemple, sur un serveur SGBD SAP, Azure Disk Encryption (ADE) peut être activé sur le disque de démarrage du système d’exploitation (si le système d’exploitation prend en charge ADE) et sur les disques de données qui ne sont pas utilisés par les fichiers de persistance des données du SGBD.  Vous pouvez notamment utiliser ADE sur le disque contenant les clés de chiffrement TDE SGBD.
1. Tests de performances. Dans SAP, en fonction du suivi et des mesures SAP, effectuez les comparaisons suivantes :
   - Le cas échéant, comparez les 10 premiers rapports en ligne à votre implémentation actuelle.
   - Le cas échéant, comparez les 10 premiers traitements par lots à votre implémentation actuelle.
   - Comparez les transferts de données via des interfaces dans le système SAP. Concentrez-vous sur les interfaces pour lesquelles vous savez que le transfert s’effectue entre différents emplacements, par exemple d’un site local vers Azure.


## <a name="non-production-phase"></a>Phase hors production 
Lors de cette phase, nous supposons qu’après une preuve de concept ou une phase pilote réussie, vous commencez à déployer des systèmes SAP non destinés à la production vers Azure. Intégrez tout ce que vous avez appris et expérimenté pendant la preuve de concept à ce déploiement. Tous les critères et étapes répertoriés pour les preuves de concept s’appliquent également à ce déploiement.

Au cours de cette phase, vous déployez généralement des systèmes de développement, des systèmes de test unitaire et des systèmes de test de régression métier vers Azure. Nous vous recommandons qu’au moins un des systèmes hors production d’une ligne d’applications SAP dispose de la configuration haute disponibilité complète, comme ce sera le cas pour le futur système de production. Voici quelques étapes supplémentaires que vous devez effectuer lors de cette phase :  

1.  Avant de déplacer les systèmes de l’ancienne plateforme vers Azure, recueillez des données sur la consommation des ressources, comme l’utilisation de l’UC, le débit de stockage et les données IOPS. En particulier, collectez ces données non seulement à partir des unités de la couche SGBD, mais aussi à partir des unités de la couche Application. Mesurez également la latence du réseau et du stockage.
2.  Enregistrez les modèles de temps d’utilisation de la disponibilité de vos systèmes. L’objectif est de déterminer si les systèmes hors production doivent être disponibles 24 heures sur 24, 7 jours sur 7 ou si certains d’entre eux peuvent être arrêtés à certains moments de la semaine ou du mois.
3.  Procédez aux tests et déterminez si vous souhaitez créer vos images du système d’exploitation pour vos machines virtuelles dans Azure ou utiliser une image d’Azure Shared Image Gallery. Si vous utilisez une image d’Azure Shared Image Gallery, veillez à utiliser une image qui reflète le mieux le contrat de support souscrit auprès de votre fournisseur de SE. Pour certains fournisseurs de systèmes d’exploitation, Azure Shared Image Gallery vous permet d’intégrer vos images BYOL (apportez votre propre licence). Pour les autres images de SE, le support est compris dans le prix indiqué par Azure. Si vous décidez de créer vos propres images de SE, vous trouverez la documentation dans les articles suivants :
    -   [Créer une image managée d’une machine virtuelle Windows généralisée dans Azure](../../windows/capture-image-resource.md)
    -   [Créer une image managée d’une machine virtuelle Linux généralisée dans Azure](../../linux/capture-image.md)
3.  Si vous utilisez SUSE et des images Red Hat Linux de la Shared Image Gallery, vous devez utiliser les images pour SAP fournies par les fournisseurs de Linux dans cette galerie.
4.  Assurez-vous de remplir les conditions de support requises par SAP en matière de contrats de support Microsoft. Consultez la [Note de support SAP n° 2015553](https://launchpad.support.sap.com/#/notes/2015553). Pour les Grandes instances HANA, consultez les [Conditions d’intégration](./hana-onboarding-requirements.md).
4.  Veillez à ce que les personnes adéquates reçoivent les [notifications de maintenance planifiée](https://azure.microsoft.com/blog/a-new-planned-maintenance-experience-for-your-virtual-machines/) afin de pouvoir choisir les temps d’arrêt les plus appropriés.
5.  Consultez régulièrement les présentations Microsoft sur les canaux tels que [Channel 9](https://channel9.msdn.com/) pour connaître les nouvelles fonctionnalités susceptibles de s’appliquer à vos déploiements.
6.  Consultez les notes SAP relatives à Azure, comme la [note de support n° 1928533](https://launchpad.support.sap.com/#/notes/1928533) pour connaître les nouvelles références SKU de machines virtuelles et les nouvelles versions de SE et de SGBD prises en charge. Comparez les tarifs des anciens et nouveaux types de machines virtuelles afin de pouvoir déployer les machines virtuelles qui présentent le meilleur rapport qualité-prix.
7.  Vérifiez à nouveau les notes de support SAP, le répertoire matériel SAP HANA et SAP PAM. Assurez-vous qu’aucune modification n’est apportée aux machines virtuelles prises en charge pour Azure, aux versions de système d’exploitation prises en charge pour ces machines virtuelles et aux versions SAP et de SGBD.
8.  Consultez le [site web SAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) pour connaître les nouvelles références SKU certifiées HANA dans Azure. Comparez la tarification des nouvelles références SKU à celles que vous avez prévues. Enfin, apportez les modifications nécessaires pour utiliser celles qui ont le meilleur rapport prix/performances.
9.  Adaptez vos scripts de déploiement pour utiliser des nouveaux types de machines virtuelles et incorporer les nouvelles fonctionnalités Azure que vous souhaitez utiliser.
10. À l’issue du déploiement de l’infrastructure, testez et évaluez la latence du réseau entre les machine virtuelles de la couche Application SAP et celles de la couche SGBD, conformément aux notes de support SAP [n° 500235](https://launchpad.support.sap.com/#/notes/500235) et [n° 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). Évaluez les résultats par rapport aux indications de latence du réseau de la [note de support SAP n° 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). La latence du réseau doit être modérée ou bonne. Des exceptions s’appliquent au trafic entre les machines virtuelles et les grandes instances HANA, comme indiqué dans la documentation disponible dans [cet article](./hana-network-architecture.md#networking-architecture-for-hana-large-instance). Assurez-vous qu’aucune des restrictions mentionnées dans [Facteurs à prendre en compte dans le cadre du déploiement SGBD des machines virtuelles Azure pour les charges de travail SAP](./dbms_guide_general.md#azure-network-considerations) et dans [Configurations et opérations de l’infrastructure SAP HANA sur Azure](./hana-vm-operations.md) ne s’applique à votre déploiement.
11. Assurez-vous que vos machines virtuelles sont déployées avec le [groupe de placements de proximité Azure](../../linux/co-location.md) adéquat, comme décrit dans [Groupes de placements de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md).
11. Effectuez toutes les autres vérifications de la preuve de concept avant d’appliquer la charge de travail.
12. Au fur et à mesure que la charge de travail s’applique, enregistrez la consommation de ressources des systèmes dans Azure. Comparez cette consommation avec les enregistrements de votre ancienne plateforme. Ajustez le dimensionnement des machines virtuelles des futurs déploiements si vous constatez des différences importantes. Gardez à l’esprit que, quand vous réduisez la taille, le stockage et la bande passante réseau des machines virtuelles sont également réduits.
    - [Tailles des machines virtuelles Windows dans Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
    - [Tailles des machines virtuelles Linux dans Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 
13. Expérimentez la fonctionnalité et les processus de copie du système. L’objectif est de faciliter la copie d’un système de développement ou d’un système de test afin que les équipes de projet bénéficient rapidement de nouveaux systèmes. 
14. Vous pouvez optimiser et affiner les accès, les autorisations et les processus basés sur les rôles Azure de votre équipe afin d’être sûr de bénéficier de la séparation des tâches. En même temps, assurez-vous que toutes les équipes peuvent effectuer leurs tâches dans l’infrastructure Azure.
15. Testez les procédures de haute disponibilité et de récupération d’urgence pour permettre à votre personnel d’exécuter ces tâches. Identifiez les défauts et adaptez les nouvelles fonctionnalités Azure que vous intégrez à vos déploiements.


## <a name="production-preparation-phase"></a>Phase de préparation de la production 
Au cours de cette phase, recueillez ce que vous avez appris dans vos déploiements hors production et appliquez-le aux futurs déploiements de production. Vous devez également préparer le travail de transfert de données entre votre emplacement d’hébergement actuel et Azure.

1.  Effectuez les mises à niveau nécessaires des versions SAP de vos systèmes de production avant de passer à Azure.
1.  Contactez les membres de la direction pour planifier les tests de fonctionnement et les tests commerciaux à réaliser après la migration du système de production.
1.  Veillez à ce que ces tests soient effectués à l’emplacement d’hébergement actuel, avec les systèmes source. Évitez d’effectuer des essais pour la première fois après que le système a été déplacé vers Azure.
1.  Testez le processus de migration des systèmes de production vers Azure. Si vous ne déplacez pas tous les systèmes de production vers Azure dans le même laps de temps, créez des groupes de systèmes de production qui doivent se trouver au même emplacement d’hébergement. Testez la migration des données. Voici quelques méthodes courantes :
    - Utilisez des méthodes SGBD telles que la sauvegarde/restauration en combinaison avec SQL Server Always On, HANA System Replication ou la copie des journaux de transaction pour alimenter et synchroniser le contenu de la base de données dans Azure.
    - Utilisez la sauvegarde/restauration pour les bases de données de plus petite taille.
    - Utilisez le Moniteur de migration SAP intégré dans SAP SWPM pour effectuer des migrations hétérogènes.
    - Utilisez le processus [SAP DMO](https://blogs.sap.com/2013/11/29/database-migration-option-dmo-of-sum-introduction/) si vous avez besoin de combiner votre migration avec une mise à niveau de version SAP. N’oubliez pas que certaines combinaisons entre les SGBD source et cible ne sont pas prises en charge. Pour plus d’informations, reportez-vous aux notes de support SAP spécifiques aux différentes versions de DMO. Par exemple, [Outil Database Migration Option (DMO) de SUM 2.0 SP04](https://launchpad.support.sap.com/#/notes/2644872).
    - Vérifiez si le débit de transfert de données est meilleur via Internet ou via ExpressRoute, au cas où vous auriez besoin de déplacer des sauvegardes ou des fichiers d’exportation SAP. Si vous déplacez des données via Internet, vous devrez peut-être modifier certaines des règles de votre groupe de sécurité réseau/groupe de sécurité d’application à mettre en place pour les futurs systèmes de production.
1.  Avant de migrer des systèmes de votre ancienne plateforme vers Azure, collectez les données de consommation des ressources. Les données utiles incluent l’utilisation du processeur, le débit de stockage et les données IOPS. En particulier, collectez ces données non seulement à partir des unités de la couche SGBD, mais aussi à partir des unités de la couche Application. Mesurez également la latence du réseau et du stockage.
1.  Vérifiez à nouveau les notes de support SAP et les paramètres de système d’exploitation requis, le répertoire matériel SAP HANA et SAP PAM. Assurez-vous qu’aucune modification n’est apportée aux machines virtuelles prises en charge pour Azure, aux versions de système d’exploitation prises en charge dans ces machines virtuelles et aux versions SAP et de SGBD.
1.  Mettez à jour les scripts de déploiement pour prendre en compte les dernières décisions que vous avez prises sur les types de machines virtuelles et les fonctionnalités Azure.
1.  Après le déploiement de l’infrastructure et des applications, vérifiez que :
    - Les types de machines virtuelles appropriés ont été déployés avec les attributs et les tailles de stockage qui conviennent.
    - Les machines virtuelles utilisent les versions et correctifs appropriés et souhaités du système d’exploitation et assurez-vous qu’ils sont uniformes.
    - Les machines virtuelles sont renforcées selon les besoins et de façon uniforme.
    - Les versions et correctifs appropriés ont été installés et déployés pour l’application.
    - Les machines virtuelles ont été déployées comme prévu dans les groupes à haute disponibilité Azure.
    - Le Stockage Premium Azure est utilisé pour les disques sensibles à la latence ou lorsque le [Contrat de niveau de service garantissant une disponibilité de 99,9 % des machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/) est requis.
    - L’Accélérateur d’écriture Azure est déployé correctement.
        - Assurez-vous que, dans les machines virtuelles, les espaces de stockage ou les jeux d’agrégats par bandes ont été correctement générés sur les disques nécessitant l’accélérateur d’écriture.
        - Vérifiez la [configuration du logiciel RAID sur Linux](/previous-versions/azure/virtual-machines/linux/configure-raid).
        - Vérifiez la [configuration de LVM sur les machines virtuelles Linux dans Azure](/previous-versions/azure/virtual-machines/linux/configure-lvm).
    - Seuls des [disques managés Azure](https://azure.microsoft.com/services/managed-disks/) sont utilisés.
    - Les machines virtuelles ont été déployées dans les groupes à haute disponibilité et dans les Zones de disponibilité appropriés.
    - La [mise en réseau accélérée Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) est activée sur les machines virtuelles utilisées dans la couche Application SAP et la couche SGBD SAP.
    - Aucune [appliance virtuelle réseau Azure](https://azure.microsoft.com/solutions/network-appliances/) ne réside dans le chemin de communication entre l’application SAP et la couche SGBD des systèmes SAP basés sur SAP NetWeaver, Hybris ou S/4HANA.
    - Les règles de groupe de sécurité d’application et de groupe de sécurité réseau permettent la communication souhaitée et planifiée, et bloquent celle-ci en cas de besoin.
    - Les paramètres liés au délai d’expiration ont été correctement définis, décrits précédemment.
    - Les machines virtuelles sont déployées avec le bon [groupe de placements de proximité Azure](../../linux/co-location.md), comme décrit dans l’article [Groupes de placements de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md).
    - La latence du réseau entre les machines virtuelles de la couche Application SAP et les machines virtuelles de SGBD est testée et validée comme décrit dans les notes de support SAP [n° 500235](https://launchpad.support.sap.com/#/notes/500235) et [n° 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). Évaluez les résultats par rapport aux indications de latence du réseau de la [note de support SAP n° 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). La latence du réseau doit être modérée ou bonne. Des exceptions s’appliquent au trafic entre les machines virtuelles et les grandes instances HANA, comme indiqué dans la documentation disponible dans [cet article](./hana-network-architecture.md#networking-architecture-for-hana-large-instance).
    - Le chiffrement a été mis en œuvre là où cela était nécessaire et avec la méthode de chiffrement appropriée.
    - Les interfaces et autres applications sont en mesure de connecter l’infrastructure nouvellement déployée.
1.  Créez un playbook pour réagir à la maintenance Azure planifiée. Déterminez l’ordre dans lequel les systèmes et les machines virtuelles doivent être redémarrés pour une maintenance planifiée.
    

## <a name="go-live-phase"></a>Phase de mise en production
Lors de la phase de mise en service, veillez à suivre les playbooks que vous avez développés au cours des phases précédentes. Exécutez les étapes que vous avez testées et expérimentées. N’acceptez pas les changements de dernière minute pour les configurations et processus. Effectuez également cette procédure :

1. Vérifiez que la supervision du portail Azure et les autres outils de supervision fonctionnent. Nous recommandons d’utiliser l’utilitaire Analyseur de performances (perfmon) pour Windows et SAR pour Linux.
    - Compteurs UC.
        - Utilisation moyenne de l’UC, total (toutes les UC)
        - Utilisation moyenne de l’UC pour chaque processeur individuel (128 processeurs sur des machines virtuelles M128)
        - Temps noyau UC pour chaque processeur individuel
        - Temps utilisateur UC pour chaque processeur individuel
    - Mémoire.
        - Mémoire disponible
        - Page mémoire entrante/seconde
        - Page mémoire sortante/seconde
    - disque.
        - Lectures disque en Ko/seconde, par disque individuel
        - Lectures disque/seconde, par disque individuel
        - Lectures disque en microsecondes/lecture, par disque individuel
        - Écritures disque en Ko/seconde, par disque individuel
        - Écritures disque/seconde, par disque individuel
        - Écritures disque en microsecondes/lecture, par disque individuel
    - Réseau.
        - Paquets réseau entrants/seconde
        - Paquets réseau sortants/seconde
        - Ko réseau entrants/seconde
        - Ko réseau sortants/seconde
1.  Après la migration des données, effectuez tous les tests de validation, comme convenu avec les membres de la direction. Acceptez uniquement les résultats des tests de validation lorsque vous disposez des résultats des systèmes source d’origine.
1.  Vérifiez que les interfaces fonctionnent et que d’autres applications peuvent communiquer avec les systèmes de production nouvellement déployés.
1.  Vérifiez le système de transport et de correction via la transaction SAP STMS.
1.  Effectuez des sauvegardes des bases de données une fois le système mis en production.
1.  Effectuez des sauvegardes des machines virtuelles de la couche Application SAP après que le système est mis en production.
1.  Pour les systèmes SAP ne faisant pas partie de la phase de mise en service actuelle, mais qui communiquent avec les systèmes SAP que vous avez transférés vers Azure durant cette phase, vous devez réinitialiser le tampon de noms d’hôte dans SM51. Cela supprimera les anciennes adresses IP mises en cache associées aux noms des instances de l’application que vous avez déplacées vers Azure.  


## <a name="post-production"></a>Post-production
Cette phase consiste à surveiller, exploiter et administrer le système. D’un point de vue SAP, les tâches que vous deviez habituellement exécuter avec votre ancien emplacement d’hébergement s’appliquent. Effectuez également ces tâches spécifiques à Azure :

1. Passez en revue les factures Azure pour les systèmes à facturation élevée.
2. Optimisez le rapport qualité-prix côté machines virtuelles et côté stockage.
3. Optimisez les moments où vous pouvez arrêter les systèmes.  


## <a name="next-steps"></a>Étapes suivantes
Reportez-vous aux articles suivants :

- [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver](./planning-guide.md)
- [Déploiement de machines virtuelles Azure pour SAP NetWeaver](./deployment-guide.md)
- [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](./dbms_guide_general.md)