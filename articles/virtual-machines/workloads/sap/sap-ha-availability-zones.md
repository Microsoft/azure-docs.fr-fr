---
title: Configurations de la charge de travail SAP avec des zones de disponibilité Azure | Microsoft Docs
description: Scénarios et architecture de haute disponibilité pour SAP NetWeaver avec des zones de disponibilité Azure
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 887caaec-02ba-4711-bd4d-204a7d16b32b
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/05/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ad9820cc3227c788582a9c2a452abe1886a4a2d3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019325"
---
# <a name="sap-workload-configurations-with-azure-availability-zones"></a>Configurations de la charge de travail SAP avec des zones de disponibilité Azure
Les [zones de disponibilité Azure](../../../availability-zones/az-overview.md) correspondent à l’une des fonctionnalités de haute disponibilité fournies par Azure. L’utilisation de zones de disponibilité améliore la disponibilité globale des charges de travail SAP dans Azure. Cette fonctionnalité est déjà disponible dans certaines [régions Azure](https://azure.microsoft.com/global-infrastructure/regions/). Elle le sera demain dans d’autres régions.

Ce graphique montre l’architecture de base de la haute disponibilité SAP :

![Configuration de la haute disponibilité standard](./media/sap-ha-availability-zones/standard-ha-config.png)

La couche Application SAP est déployée sur un seul [groupe à haute disponibilité](../../manage-availability.md) Azure. Pour que SAP Central Services soit hautement disponible, vous pouvez déployer deux machines virtuelles dans un autre groupe à haute disponibilité. En cas de problème lié au framework ou aux logiciels, servez-vous du clustering de basculement Windows Server ou de Pacemaker (Linux) en tant que framework de haute disponibilité avec basculement automatique. Pour en savoir plus sur ces déploiements, consultez :

- [Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster](./sap-high-availability-guide-wsfc-shared-disk.md)
- [Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un partage de fichiers](./sap-high-availability-guide-wsfc-file-share.md)
- [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md)
- [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux](./high-availability-guide-rhel.md)

Une architecture similaire s’applique à la couche SGBD des systèmes SAP NetWeaver, S/4HANA ou Hybris. Vous déployez la couche SGBD en mode actif/passif avec une solution de cluster de basculement afin d’éviter les défaillances liées au framework ou aux logiciels. La solution de cluster de basculement peut être un framework de basculement SGBD, le clustering de basculement Windows Server ou Pacemaker.

Pour déployer la même architecture à l’aide de zones de disponibilité Azure, vous devez apporter certaines modifications à l’architecture présentée précédemment. Cet article décrit ces modifications.

## <a name="considerations-for-deploying-across-availability-zones"></a>Considérations liées au déploiement dans des zones de disponibilité


Tenez compte des points suivants quand vous utilisez des zones de disponibilité :

- Il n’existe aucune garantie concernant les distances entre les différentes zones de disponibilité au sein d’une région Azure.
- Les zones de disponibilité ne constituent pas une solution de reprise d’activité idéale. Les catastrophes naturelles peuvent provoquer des dommages à grande échelle dans certaines régions du monde, y compris de lourds dommages au niveau des infrastructures d’alimentation. Les distances entre les différentes zones ne sont peut-être pas suffisantes pour constituer une solution de reprise d’activité appropriée.
- La latence réseau entre les zones de disponibilité n’est pas le même dans toutes les régions Azure. Dans certains cas, vous pouvez déployer et exécuter la couche Application SAP sur différentes zones, car la latence réseau entre une zone et la machine virtuelle SGBD active est acceptable. Mais dans certaines régions Azure, la latence entre la machine virtuelle SGBD active et l’instance d’application SAP peut être inacceptable pour les processus métier SAP, quand ces dernières sont déployées dans des zones différentes. Le cas échéant, l’architecture de déploiement doit être différente, avec une architecture active/active pour l’application, ou une architecture active/passive là où la latence réseau entre les zones est trop importante.
- Pour déterminer où utiliser des zones de disponibilité, tenez compte de la latence réseau entre les zones. La latence réseau joue un rôle important dans deux domaines :
    - La latence entre les deux instances SGBD qui ont besoin d’une réplication synchrone. Plus la latence réseau est élevée, plus la scalabilité de votre charge de travail a des chances d’être affectée.
    - La différence de latence réseau entre une machine virtuelle exécutant une instance de dialogue SAP dans la même zone que l’instance SGBD active et une machine virtuelle similaire dans une autre zone. Quand cette différence augmente, l’influence qu’elle exerce sur la durée d’exécution des processus métier et des programmes de traitement par lots augmente également, selon qu’ils s’exécutent dans la zone du SGBD ou dans une autre zone.

Quand vous déployez des machines virtuelles Azure sur des zones de disponibilité et établissez des solutions de basculement au sein d’une même région Azure, certaines restrictions s’appliquent :

- Vous devez utiliser [Azure Disques managés](https://azure.microsoft.com/services/managed-disks/) pour un déploiement sur des zones de disponibilité Azure. 
- Le mappage des énumérations de zones aux zones physiques est fixe sur la base d’un abonnement Azure. Si vous utilisez différents abonnements pour déployer vos systèmes SAP, vous devez définir les zones idéales pour chaque abonnement.
- Vous ne pouvez pas déployer de groupes à haute disponibilité Azure au sein d’une zone de disponibilité Azure, sauf si vous utilisez un [groupe de placements de proximité Azure](../../linux/co-location.md). La façon dont vous pouvez déployer la couche SGBD SAP et les services centraux entre les zones et, en même temps, déployer la couche d’application SAP à l’aide de groupes à haute disponibilité tout en continuant à proposer une proximité des machines virtuelles est décrite dans l’article [Groupes de placements de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md). Si vous ne tirez pas parti des groupes de placements de proximité Azure, vous devez choisir l’un ou l’autre comme framework de déploiement pour les machines virtuelles.
- Vous ne pouvez pas utiliser un [équilibreur de charge de base Azure](../../../load-balancer/load-balancer-overview.md) pour créer des solutions de cluster de basculement basées sur le clustering de basculement Windows Server ou Linux Pacemaker. Vous devez plutôt utiliser la [référence SKU Standard Load Balancer Azure](../../../load-balancer/load-balancer-standard-availability-zones.md).



## <a name="the-ideal-availability-zones-combination"></a>Combinaison de zones de disponibilité idéale
Avant de déterminer comment utiliser des zones de disponibilité, identifiez ce qui suit :

- La latence réseau entre les trois zones d’une région Azure. Vous pouvez ainsi choisir les zones qui ont la latence réseau la plus faible dans le trafic réseau interzone.
- La différence entre la latence de machine virtuelle à machine virtuelle au sein de l’une des zones de votre choix et la latence réseau entre deux zones de votre choix.
- La disponibilité des types de machine virtuelle que vous avez besoin de déployer dans les deux zones que vous avez sélectionnées. Avec certaines machines virtuelles, en particulier celles de la série M, vous risquez de vous retrouver dans des situations où certaines références SKU sont disponibles uniquement dans deux des trois zones.

## <a name="network-latency-between-and-within-zones"></a>Latence réseau entre les zones et au sein des zones
Pour déterminer la latence entre les différentes zones, vous avez besoin d’effectuer les étapes suivantes :

- Déployer la référence (SKU) de la machine virtuelle à utiliser pour votre instance de SGBD dans les trois zones. Assurez-vous que l’[accélération réseau Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) est activée quand vous prenez cette mesure.
- Une fois que vous avez trouvé les deux zones avec la latence réseau la plus faible, déployez trois autres machines virtuelles de la référence SKU de machine virtuelle que vous voulez utiliser comme machine virtuelle de la couche Application sur les trois zones de disponibilité. Mesurez la latence réseau par rapport aux deux machines virtuelles SGBD dans les deux zones SGBD que vous avez sélectionnées. 
- Utilisez **niping** en tant qu’outil de mesure. Cet outil, fourni par SAP, est décrit dans les notes de support SAP [#500235](https://launchpad.support.sap.com/#/notes/500235) et [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E). Concentrez-vous sur les commandes SAP documentées pour les mesures de latence. Étant donné que **ping** ne fonctionne pas par les chemins de code d’accélération réseau Azure, nous vous recommandons de ne pas l’utiliser.

Vous n’avez pas besoin d’effectuer ces tests manuellement. La procédure PowerShell [Test de latence de la zone de disponibilité](https://github.com/Azure/SAP-on-Azure-Scripts-and-Utilities/tree/master/AvZone-Latency-Test) permet d’automatiser les tests de latence décrits. 

En fonction de vos mesures et de la disponibilité de vos références SKU de machine virtuelle dans les zones de disponibilité, vous devez prendre des décisions :

- Définissez les zones idéales pour la couche SGBD.
- En fonction des différences de latence réseau au sein d’une zone ou entre plusieurs zones, déterminez si vous voulez distribuer votre couche Application SAP active sur une, deux ou trois zones.
- Déterminez si vous voulez déployer une configuration active/passive ou une configuration active/active du point de vue de l’application. (Ces configurations sont expliquées plus loin dans cet article.)

Pour prendre ces décisions, tenez également compte des recommandations de SAP quant à la latence réseau, comme décrit dans la note SAP [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E).

> [!IMPORTANT]
> Les mesures et les décisions que vous prenez sont valides pour l’abonnement Azure que vous avez utilisé quand vous avez pris les mesures. Si vous utilisez un autre abonnement Azure, vous devez reprendre les mesures. Le mappage des zones énumérées peut être différent pour un autre abonnement Azure.


> [!IMPORTANT]
> Il est probable que les mesures décrites précédemment fournissent des résultats différents dans chaque région Azure qui prend en charge les [zones de disponibilité](../../../availability-zones/az-overview.md). Même si vos besoins en latence réseau ne changent pas, il peut être nécessaire d’adopter différentes stratégies de déploiement dans les différentes régions Azure, car la latence réseau entre les zones peut varier. Dans certaines régions Azure, la latence réseau entre les trois différentes zones peut varier considérablement. Dans d’autres régions, la latence réseau entre les trois différentes zones peut être plus uniforme. L’idée qu’il existe toujours une latence réseau comprise entre 1 et 2 millisecondes est fausse. La latence réseau entre les zones de disponibilité dans les régions Azure ne peut pas être généralisée.

## <a name="activeactive-deployment"></a>Déploiement actif/actif
Cette architecture de déploiement est appelée active/active, car vous déployez vos serveurs d’applications SAP actifs sur deux ou trois zones. L’instance SAP Central Services qui utilise la réplication de la mise en file d’attente sera déployé entre deux zones. Ceci vaut également pour la couche SGBD, qui sera déployée sur les mêmes zones que SAP Central Service.

Quand vous envisagez cette configuration, vous devez rechercher les deux zones de disponibilité dans votre région qui offrent une latence réseau interzone acceptable pour votre charge de travail et pour la réplication synchrone de votre SGBD. Vous devez également vous assurer que le delta entre la latence réseau au sein des zones que vous avez sélectionnées et la latence réseau interzone n’est pas trop important. En effet, vous ne voulez pas de grandes variations dans les durées d’exécution de vos processus métier ou programmes de traitement par lots, selon qu’un travail s’exécute dans la zone avec le serveur SGBD ou entre les zones. Certaines variations sont acceptables, mais pas des différences importantes.

Voici un schéma simplifié d’un déploiement actif/actif sur deux zones :

![Déploiement de zone actif/actif](./media/sap-ha-availability-zones/active_active_zones_deployment.png)

Les considérations suivantes s’appliquent pour cette configuration :

- Si vous n’utilisez pas de [groupe de placements de proximité](../../linux/co-location.md), vous traitez les zones de disponibilité Azure comme des domaines d’erreur et de mise à jour pour toutes les machines virtuelles, car les groupes à haute disponibilité ne peuvent pas être déployés dans des zones de disponibilité Azure.
- Si vous souhaitez combiner des déploiements zonaux pour la couche SGBD et les services centraux, mais que vous souhaitez utiliser des groupes à haute disponibilité Azure pour la couche Application, vous devez utiliser des groupes de proximité Azure, comme décrit dans l’article [Groupes de placements de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md).
- Pour les équilibreurs de charge des clusters de basculement de SAP Central Services et la couche SGBD, vous devez utiliser le service [Azure Load Balancer de la référence SKU standard](../../../load-balancer/load-balancer-standard-availability-zones.md). L’équilibreur de charge de base ne fonctionne pas entre des zones.
- Le réseau virtuel Azure que vous avez déployé pour héberger le système SAP, ainsi que ses sous-réseaux, est réparti sur plusieurs zones. Vous n’avez pas besoin de réseaux virtuels distincts pour chaque zone.
- Pour toutes les machines virtuelles que vous déployez, vous devez utiliser [Azure Disques managés](https://azure.microsoft.com/services/managed-disks/). Les disques non managés ne sont pas pris en charge pour les déploiements sur des zones.
- Le stockage Premium Azure et le [stockage SSD Ultra](../../disks-types.md#ultra-disk) ne prennent en charge aucun type de réplication de stockage entre des zones. L’application (SGBD ou SAP Central Services) doit répliquer des données importantes.
- Il en va de même pour le répertoire partagé sapmnt, qui est un disque partagé (Windows), un partage CIFS (Windows) ou un partage NFS (Linux). Vous devez utiliser une technologie qui réplique ces disques partagés ou ces partages entre les zones. Ces technologies sont prises en charge :
  - Pour Windows, une solution de cluster qui utilise SIOS Datakeeper, comme décrit dans [Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster dans Azure](./sap-high-availability-guide-wsfc-shared-disk.md).
  - Pour SUSE Linux, un partage NFS créé comme décrit dans [Haute disponibilité pour NFS sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md).
    
    Actuellement, la solution qui utilise le serveur de fichiers avec montée en puissance parallèle Microsoft, comme décrit dans [Préparation d’infrastructure Azure pour la haute disponibilité SAP à l’aide de cluster de basculement Windows et de partage de fichiers pour une instance SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md), n’est pas prise en charge entre plusieurs zones.
- La troisième zone est utilisée pour héberger le périphérique SBD au cas où vous créez un [cluster Pacemaker SUSE Linux](./high-availability-guide-suse-pacemaker.md#create-azure-fence-agent-stonith-device) ou d’autres instances d’application.
- Pour obtenir la cohérence d’exécution des processus métier critiques, vous pouvez essayer de diriger certains utilisateurs et tâches de traitement par lots vers des instances d’application qui se trouvent dans la même zone que l’instance SGBD active, à l’aide de groupes de serveurs par lots SAP, de groupes de connexions SAP ou de groupes RFC. Cependant, en cas de basculement entre zones, vous devez déplacer manuellement ces groupes vers les instances qui sont exécutées sur les machines virtuelles qui se trouvent dans la même zone que la machine virtuelle de la base de données active.  
- Vous pouvez éventuellement déployer des instances de dialogue dormantes dans chacune des zones. Celles-ci permettent un retour immédiat à la capacité de ressources antérieure si une zone utilisée par une partie de vos instances d’application est hors service.

> [!IMPORTANT]
> Dans ce scénario actif/actif, des frais supplémentaires sont annoncés par Microsoft à partir du 01/04/2020 pour la bande passante. Consultez le document [Détails sur la tarification de la bande passante](https://azure.microsoft.com/pricing/details/bandwidth/). Le transfert de données entre la couche d'application SAP et la couche SGBD SAP est assez intensif. Par conséquent, le scénario actif/actif peut en partie contribuer aux coûts. Poursuivez la lecture de cet article pour connaître les coûts exacts. 


## <a name="activepassive-deployment"></a>Déploiement actif/passif
Si vous ne trouvez pas un delta acceptable entre la latence réseau au sein d’une même zone et la latence du trafic réseau interzone, vous pouvez déployer une architecture qui a un caractère actif/passif du point de vue de la couche Application SAP. Vous définissez une zone *active*, à savoir la zone dans laquelle vous déployez la couche Application complète et où vous essayez d’exécuter à la fois l’instance SGBD active et l’instance SAP Central Services. Avec une telle configuration, vous devez éviter les variations importantes au niveau des durées d’exécution des transactions métier et des programmes de traitement par lots, selon qu’un travail est exécuté dans la même zone que l’instance SGBD active.

La disposition de base de l’architecture ressemble à ceci :

![Déploiement de zone actif/passif](./media/sap-ha-availability-zones/active_passive_zones_deployment.png)

Les considérations suivantes s’appliquent pour cette configuration :

- Vous ne pouvez pas déployer des groupes à haute disponibilité dans des zones de disponibilité Azure. Pour compenser cela, vous pouvez utiliser des groupes de placement de proximité Azure, tel que décrit dans l’article [Groupes de placements de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md).
- Quand vous utilisez cette architecture, vous devez superviser l’état attentivement et essayer de maintenir actives les instances SGDB et SAP Central Services dans la même zone que votre couche Application déployée. En cas de basculement de SAP Central Service ou de l’instance SGBD, vous voulez être sûr de pouvoir basculer manuellement aussi vite que possible dans la zone avec la couche Application SAP déployée.
- Pour les équilibreurs de charge des clusters de basculement de SAP Central Services et la couche SGBD, vous devez utiliser le service [Azure Load Balancer de la référence SKU standard](../../../load-balancer/load-balancer-standard-availability-zones.md). L’équilibreur de charge de base ne fonctionne pas entre des zones.
- Le réseau virtuel Azure que vous avez déployé pour héberger le système SAP, ainsi que ses sous-réseaux, est réparti sur plusieurs zones. Vous n’avez pas besoin de réseaux virtuels distincts pour chaque zone.
- Pour toutes les machines virtuelles que vous déployez, vous devez utiliser [Azure Disques managés](https://azure.microsoft.com/services/managed-disks/). Les disques non managés ne sont pas pris en charge pour les déploiements sur des zones.
- Le stockage Premium Azure et le [stockage SSD Ultra](../../disks-types.md#ultra-disk) ne prennent en charge aucun type de réplication de stockage entre des zones. L’application (SGBD ou SAP Central Services) doit répliquer des données importantes.
- Il en va de même pour le répertoire partagé sapmnt, qui est un disque partagé (Windows), un partage CIFS (Windows) ou un partage NFS (Linux). Vous devez utiliser une technologie qui réplique ces disques partagés ou ces partages entre les zones. Ces technologies sont prises en charge :
    - Pour Windows, une solution de cluster qui utilise SIOS Datakeeper, comme décrit dans [Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster dans Azure](./sap-high-availability-guide-wsfc-shared-disk.md).
    - Pour SUSE Linux, un partage NFS créé comme décrit dans [Haute disponibilité pour NFS sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md).
    
  Actuellement, la solution qui utilise le serveur de fichiers avec montée en puissance parallèle Microsoft, comme décrit dans [Préparation d’infrastructure Azure pour la haute disponibilité SAP à l’aide de cluster de basculement Windows et de partage de fichiers pour une instance SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md), n’est pas prise en charge entre plusieurs zones.
- La troisième zone est utilisée pour héberger le périphérique SBD au cas où vous créez un [cluster Pacemaker SUSE Linux](./high-availability-guide-suse-pacemaker.md#create-azure-fence-agent-stonith-device) ou d’autres instances d’application.
- Vous devez déployer des machines virtuelles dormantes dans la zone passive (du point de vue du SGBD) pour être en mesure de démarrer les ressources de l’application en cas de défaillance d’une zone.
    - [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/) est actuellement incapable de répliquer des machines virtuelles actives sur des machines virtuelles dormantes entre des zones. 
- Il est conseillé d’investir dans l’automatisation qui vous permet, en cas de défaillance d’une zone, de démarrer automatiquement la couche Application SAP dans la deuxième zone.

## <a name="combined-high-availability-and-disaster-recovery-configuration"></a>Configuration combinée de la haute disponibilité et de la reprise d’activité
Microsoft ne partage aucune information concernant la distance géographique qui existe entre les sites qui hébergent différentes zones de disponibilité Azure dans une région Azure. Pourtant, certains clients utilisent des zones pour obtenir une configuration de haute disponibilité et de reprise d’activité qui promet un objectif de point de récupération (RPO) de zéro. Cela signifie que vous ne devez perdre aucune transaction de base de données validée même en cas de reprise d’activité après sinistre. 

> [!NOTE]
> Nous vous recommandons d’utiliser une configuration comme celle-ci uniquement dans certaines circonstances. Par exemple, vous pouvez l’utiliser quand les données ne peuvent pas quitter la région Azure pour des raisons de sécurité ou de conformité. 

Voici un exemple de la façon dont une telle configuration peut se présenter :

![Reprise d’activité et haute disponibilité combinées dans les zones](./media/sap-ha-availability-zones/combined_ha_dr_in_zones.png)

Les considérations suivantes s’appliquent pour cette configuration :

- Vous supposez qu’il existe une distance importante entre les sites qui hébergent une zone de disponibilité ou que vous êtes obligé de rester au sein d’une certaine région Azure. Vous ne pouvez pas déployer des groupes à haute disponibilité dans des zones de disponibilité Azure. Pour compenser cela, vous pouvez utiliser des groupes de placement de proximité Azure, tel que décrit dans l’article [Groupes de placements de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md).
- Quand vous utilisez cette architecture, vous devez superviser l’état attentivement et essayer de maintenir actives les instances SGDB et SAP Central Services dans la même zone que votre couche Application déployée. En cas de basculement de SAP Central Service ou de l’instance SGBD, vous voulez être sûr de pouvoir basculer manuellement aussi vite que possible dans la zone avec la couche Application SAP déployée.
- Vous devez avoir des instances d’application de production préinstallées dans les machines virtuelles qui exécutent les instances d’application d’assurance qualité actives.
- En cas de défaillance d’une zone, arrêtez les instances d’application d’assurance qualité et démarrez à la place les instances de production. Notez que vous devez utiliser les noms virtuels des instances d’application pour que cela fonctionne.
- Pour les équilibreurs de charge des clusters de basculement de SAP Central Services et la couche SGBD, vous devez utiliser le service [Azure Load Balancer de la référence SKU standard](../../../load-balancer/load-balancer-standard-availability-zones.md). L’équilibreur de charge de base ne fonctionne pas entre des zones.
- Le réseau virtuel Azure que vous avez déployé pour héberger le système SAP, ainsi que ses sous-réseaux, est réparti sur plusieurs zones. Vous n’avez pas besoin de réseaux virtuels distincts pour chaque zone.
- Pour toutes les machines virtuelles que vous déployez, vous devez utiliser [Azure Disques managés](https://azure.microsoft.com/services/managed-disks/). Les disques non managés ne sont pas pris en charge pour les déploiements sur des zones.
- Le stockage Premium Azure et le [stockage SSD Ultra](../../disks-types.md#ultra-disk) ne prennent en charge aucun type de réplication de stockage entre des zones. L’application (SGBD ou SAP Central Services) doit répliquer des données importantes.
- Il en va de même pour le répertoire partagé sapmnt, qui est un disque partagé (Windows), un partage CIFS (Windows) ou un partage NFS (Linux). Vous devez utiliser une technologie qui réplique ces disques partagés ou ces partages entre les zones. Ces technologies sont prises en charge :
    - Pour Windows, une solution de cluster qui utilise SIOS Datakeeper, comme décrit dans [Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster dans Azure](./sap-high-availability-guide-wsfc-shared-disk.md).
    - Pour SUSE Linux, un partage NFS créé comme décrit dans [Haute disponibilité pour NFS sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md).

  Actuellement, la solution qui utilise le serveur de fichiers avec montée en puissance parallèle Microsoft, comme décrit dans [Préparation d’infrastructure Azure pour la haute disponibilité SAP à l’aide de cluster de basculement Windows et de partage de fichiers pour une instance SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md), n’est pas prise en charge entre plusieurs zones.
- La troisième zone est utilisée pour héberger le périphérique SBD au cas où vous créez un [cluster Pacemaker SUSE Linux](./high-availability-guide-suse-pacemaker.md#create-azure-fence-agent-stonith-device) ou d’autres instances d’application.





## <a name="next-steps"></a>Étapes suivantes
Voici certaines des étapes suivantes du déploiement sur des zones de disponibilité Azure :

- [Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster dans Azure](./sap-high-availability-guide-wsfc-shared-disk.md)
- [Préparer une infrastructure Azure pour la haute disponibilité SAP à l’aide d’un cluster de basculement Windows et un partage de fichiers pour une instance SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md)