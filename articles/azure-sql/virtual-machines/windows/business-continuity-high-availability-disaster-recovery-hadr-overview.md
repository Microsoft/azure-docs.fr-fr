---
title: Haute disponibilité, reprise d’activité après sinistre, continuité d’activité
description: Découvrez les options de haute disponibilité, de reprise d’activité après sinistre (HADR) et de continuité d’activité disponibles pour SQL Server sur des machines virtuelles Azure, comme les groupes de disponibilité Always On, l’instance de cluster de basculement, la mise en miroir de bases de données, la copie des journaux de sauvegarde, et la sauvegarde et restauration vers Stockage Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: 53981f7e-8370-4979-b26a-93a5988d905f
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/27/2020
ms.author: mathoma
ms.openlocfilehash: 1a0d1018991be9d78623b0826aeab3d13958e996
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504132"
---
# <a name="business-continuity-and-hadr-for-sql-server-on-azure-virtual-machines"></a>Continuité d’activité et HADR pour SQL Server sur Machines virtuelles Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

La continuité d’activité consiste à poursuivre votre activité en cas de sinistre, à planifier la récupération et à garantir la haute disponibilité de vos données. SQL Server sur des machines virtuelles Azure peut aider à réduire le coût d’une solution de base de données à haute disponibilité et reprise d’activité (HADR). 

La plupart des solutions HADR SQL Server sont prises en charge sur les machines virtuelles, en tant que solutions Azure uniquement et solutions hybrides. Dans une solution Azure uniquement, le système HADR s’exécute dans Azure. Dans une configuration hybride, une partie de la solution est exécutée dans Azure, tandis que l’autre est exécutée localement dans votre organisation. La flexibilité de l’environnement Azure vous permet de migrer partiellement ou totalement vers Azure afin de répondre aux exigences HADR et en termes de budget de vos systèmes de base de données SQL Server.

Cet article compare les solutions de continuité d’activité disponibles pour SQL Server sur machines virtuelles Azure. 

## <a name="overview"></a>Vue d’ensemble

Il vous incombe de garantir que votre système de base de données dispose des fonctions HADR requises par le contrat de niveau de service. Le fait qu’Azure fournisse des mécanismes haute disponibilité, comme le service de réparation pour les services cloud et la détection de la récupération après défaillance pour les machines virtuelles, ne garantit pas que vous puissiez respecter le contrat de niveau de service. Bien que ces mécanismes contribuent à protéger la haute disponibilité de la machine virtuelle, ils ne protègent pas la disponibilité de SQL Server exécuté sur la machine virtuelle. 

Il est possible que l’instance de SQL Server échoue pendant que la machine virtuelle est en ligne et saine. Même les mécanismes haute disponibilité fournis par Azure tiennent compte des temps morts des machines virtuelles en raison d’événements tels que la récupération après une défaillance matérielle ou logicielle et des mises à niveau du système d’exploitation.

Le stockage géoredondant (GRS) dans Azure est implémenté avec une fonctionnalité appelée géoréplication. GRS peut ne pas être une solution de reprise d’activité adéquate pour vos bases de données. Comme la géoréplication envoie les données de manière asynchrone, il est possible que les mises à jour récentes soient perdues en cas de sinistre. Vous trouverez plus d’informations sur les limitations de la géoréplication dans la section [Prise en charge de la géoréplication](#geo-replication-support).

## <a name="deployment-architectures"></a>Architectures de déploiement
Azure prend en charge les technologies SQL Server suivantes pour la continuité d’activité :

* [Groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server)
* [Instances de cluster de basculement AlwaysOn](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
* [Copie des journaux de transaction](/sql/database-engine/log-shipping/about-log-shipping-sql-server)
* [Sauvegarde et restauration SQL Server avec Stockage Blob Azure](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service)
* [Mise en miroir de base de données](/sql/database-engine/database-mirroring/database-mirroring-sql-server) - Dépréciée dans SQL Server 2016

Vous pouvez combiner les technologies pour implémenter une solution SQL Server qui offre des fonctions de haute disponibilité et de reprise d’activité. Selon la technologie que vous utilisez, un déploiement hybride peut nécessiter un tunnel VPN avec le réseau virtuel Azure. Les sections suivantes présentent des exemples d’architectures de déploiement.

## <a name="azure-only-high-availability-solutions"></a>Azure uniquement : Solutions de haute disponibilité

Vous pouvez avoir une solution de haute disponibilité pour SQL Server au niveau de la base de données avec des groupes de disponibilité AlwaysOn. Vous pouvez également créer une solution de haute disponibilité au niveau de l’instance avec des instances de cluster de basculement AlwaysOn. Pour une protection accrue, vous pouvez créer une redondance aux deux niveaux en créant des groupes de disponibilité sur des instances de cluster de basculement. 

| Technology | Exemples d’architecture |
| --- | --- |
| **Groupes de disponibilité** |Les réplicas de disponibilité exécutés sur les machines virtuelles Azure dans la même région offrent une haute disponibilité. Vous devez configurer une machine virtuelle de contrôleur de domaine, car le clustering de basculement Windows nécessite un domaine Active Directory.<br/><br/> Pour une redondance et une disponibilité plus élevées, les machines virtuelles Azure peuvent être déployées dans différentes [zones de disponibilité](../../../availability-zones/az-overview.md), comme indiqué dans la [vue d’ensemble des groupes de disponibilité](availability-group-overview.md). Si les machines virtuelles SQL Server d’un groupe de disponibilité sont déployées dans des zones de disponibilité, utilisez [Azure Standard Load Balancer](../../../load-balancer/load-balancer-overview.md) pour l’écouteur, comme indiqué dans les articles [Utiliser Azure CLI pour configurer un groupe de disponibilité Always On pour SQL Server sur une machine virtuelle Azure](./availability-group-az-commandline-configure.md) et [Utiliser des modèles de démarrage rapide Azure pour configurer un groupe de disponibilité Always On pour SQL Server sur une machine virtuelle Azure](availability-group-quickstart-template-configure.md).<br/> ![Diagramme montrant le « Contrôleur de domaine » au-dessus du « Cluster WSFC » constitué du « Réplica principal », du « Réplica secondaire » et du « Témoin de partage de fichiers ».](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-ha-always-on.png)<br/>Pour plus d’informations, consultez la section [Configurer des groupes de disponibilité dans Azure (GUI)](./availability-group-quickstart-template-configure.md). |
| **Instances de cluster de basculement** |Les instances de cluster de basculement sont prises en charge sur les machines virtuelles SQL Server. Étant donné que la fonctionnalité d’instance de cluster de basculement nécessite un stockage partagé, cinq solutions fonctionnent avec SQL Server sur machines virtuelles Azure : <br/><br/> - Utilisation de [disques partagés Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) pour Windows Server 2019. Les disques managés partagés sont un produit Azure qui permet d’attacher un disque managé à plusieurs machines virtuelles simultanément. Les machines virtuelles du cluster peuvent lire ou écrire sur le disque que vous avez attaché en fonction de la réservation choisie par l’application en cluster par le biais des réservations persistantes SCSI (SCSI PR). SCSI PR est une solution de stockage de norme industrielle dont tirent parti les applications qui s’exécutent sur un réseau de zone de stockage (SAN) local. L’activation des réservations persistantes SCSI sur un disque managé vous permet de migrer ces applications vers Azure en l’état. <br/><br/>- Utilisation d’[espaces de stockage direct \(S2D\)](failover-cluster-instance-storage-spaces-direct-manually-configure.md) pour fournir un réseau SAN virtuel basé sur logiciel pour Windows Server 2016 et versions ultérieures.<br/><br/>- Utilisation d’un [partage de fichiers Premium](failover-cluster-instance-premium-file-share-manually-configure.md) pour Windows Server 2012 et versions ultérieures. Les partages de fichiers Premium offrent une faible latence, s’appuient sur des disques SSD, et sont entièrement pris en charge pour une utilisation avec une instance de cluster de basculement.<br/><br/>- Utilisation de stockage pris en charge par une solution de partenaire pour le clustering. Pour obtenir un exemple spécifique qui utilise SIOS DataKeeper, consultez l’entrée de blog sur le [clustering de basculement et SIOS DataKeeper](https://azure.microsoft.com/blog/high-availability-for-a-file-share-using-wsfc-ilb-and-3rd-party-software-sios-datakeeper/).<br/><br/>- Utilisation de stockage de bloc partagé pour une cible iSCSI distante par le biais d’Azure ExpressRoute. Par exemple, NPS (NetApp Private Storage) expose une cible iSCSI via ExpressRoute avec Equinix dans les machines virtuelles Azure.<br/><br/>Pour les solutions de stockage partagé et de réplication de données proposées par des partenaires Microsoft, contactez le fournisseur pour tout problème lié à l’accès aux données lors du basculement.<br/><br/>||

## <a name="azure-only-disaster-recovery-solutions"></a>Azure uniquement : Solutions de reprise d’activité après sinistre
Vous pouvez disposer d’une solution de reprise d’activité pour vos bases de données SQL Server dans Azure en utilisant des groupes de disponibilité, la mise en miroir de bases de données ou la sauvegarde et la restauration à l’aide d’objets blob de stockage.

| Technology | Exemples d’architecture |
| --- | --- |
| **Groupes de disponibilité** |Réplicas de disponibilité exécutés dans plusieurs centres de données sur les machines virtuelles Azure pour la récupération d’urgence. Cette solution inter-régions vous aide à vous protéger contre l’indisponibilité totale du site. <br/> ![Diagramme montrant deux régions avec un « Réplica principal » et un « Réplica secondaire » connectés par une « Validation asynchrone ».](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-dr-alwayson.png)<br/>Au sein d’une région, tous les réplicas doivent se trouver dans le même service cloud et sur le même réseau virtuel. Étant donné que chaque région aura un réseau virtuel distinct, ces solutions nécessitent une connectivité réseau-à-réseau. Pour plus d’informations, consultez [Configurer une connexion réseau-à-réseau à l’aide du portail Azure](../../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md). Pour obtenir des instructions détaillées, consultez [Configurer un groupe de disponibilité AlwaysOn SQL Server dans différentes régions Azure](availability-group-manually-configure-multiple-regions.md).|
| **Mise en miroir de bases de données** |Serveurs principal et miroir s’exécutant dans des centres de données différents pour la récupération d’urgence. Vous devez les déployer à l’aide de certificats de serveur. La mise en miroir de base de données SQL Server n’est pas prise en charge pour SQL Server 2008 ou SQL Server 2008 R2 sur une machine virtuelle Azure. <br/>![Diagramme affichant dans une région le « Principal », qui est connecté au « Miroir » dans une autre région, avec des « Hautes performances ».](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-dr-dbmirroring.png) |
| **Sauvegarde et restauration avec Stockage Blob Azure** |Bases de données de production sauvegardées directement dans le stockage d’objets blob dans un autre centre de données pour la reprise d’activité.<br/>![Diagramme montrant dans une région une « Base de données » sauvegardant le « Stockage Blob » dans une autre région.](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-dr-backup-restore.png)<br/>Pour plus d’informations, consultez [Sauvegarde et restauration pour SQL Server sur des machines virtuelles Azure](../../../azure-sql/virtual-machines/windows/backup-restore.md). |
| **Réplication et basculement de SQL Server vers Azure avec Azure Site Recovery** |Instance SQL Server de production dans un centre de données Azure répliquée directement dans Stockage Azure dans un autre centre de données Azure pour la reprise d’activité.<br/>![Diagramme qui illustre une « Base de données » dans un centre de données Azure utilisant la « Réplication ASR » pour la reprise d’activité dans un autre centre de données. ](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/azure-only-dr-standalone-sqlserver-asr.png)<br/>Pour plus d’informations, consultez l’article [Protéger SQL Server à l’aide de la récupération d’urgence SQL Server et d’Azure Site Recovery](../../../site-recovery/site-recovery-sql.md). |


## <a name="hybrid-it-disaster-recovery-solutions"></a>Informatique hybride : Solutions de reprise d’activité après sinistre
Vous pouvez disposer d’une solution de reprise d’activité pour vos bases de données SQL Server dans un environnement informatique hybride utilisant des groupes de disponibilité, la mise en miroir de bases de données, la copie des journaux de transaction et la sauvegarde et la restauration avec Stockage Blob Azure.

| Technology | Exemples d’architecture |
| --- | --- |
| **Groupes de disponibilité** |Certains réplicas de disponibilité s’exécutant dans les machines virtuelles Azure et d’autres réplicas s’exécutant sur site pour la récupération d’urgence entre sites. Le site de production peut être local ou situé dans un centre de données Azure.<br/>![Groupes de disponibilité](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-alwayson.png)<br/>Étant donné que tous les réplicas de disponibilité doivent être dans le même cluster de basculement, ce dernier doit couvrir les deux réseaux (un cluster de basculement de plusieurs sous-réseaux). Cette configuration nécessite une connexion VPN entre Azure et le réseau local.<br/><br/>Pour une récupération d’urgence réussie de vos bases de données, vous devez également installer un contrôleur de domaine de réplica sur le site de récupération d’urgence.|
| **Mise en miroir de bases de données** |Un serveur partenaire exécuté sur une machine virtuelle Azure et l’autre exécuté sur site pour la reprise d’activité entre sites utilisant des certificats de serveur. Les serveurs partenaires n’ont pas besoin d’être dans le même domaine Active Directory, et aucune connexion VPN n’est requise.<br/>![Mise en miroir de bases de données](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-dbmirroring.png)<br/>Un autre scénario de mise en miroir de bases de données implique un serveur partenaire exécuté sur une machine virtuelle Azure et l’autre exécuté localement dans le même domaine Active Directory pour la récupération d’urgence entre sites. Une [connexion VPN entre le réseau virtuel Azure et le réseau local](../../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md) est requise.<br/><br/>Pour une récupération d’urgence réussie de vos bases de données, vous devez également installer un contrôleur de domaine de réplica sur le site de récupération d’urgence. La mise en miroir de base de données SQL Server n’est pas prise en charge pour SQL Server 2008 ou SQL Server 2008 R2 sur une machine virtuelle Azure. |
| **Copie des journaux de transaction** |Un serveur exécuté sur une machine virtuelle Azure et l’autre exécuté localement pour la récupération d’urgence entre sites. La copie des journaux de transaction dépendant du partage de fichiers Windows, une connexion VPN entre le réseau virtuel Azure et le réseau local est requise.<br/>![Copie des journaux de transaction](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-log-shipping.png)<br/>Pour une récupération d’urgence réussie de vos bases de données, vous devez également installer un contrôleur de domaine de réplica sur le site de récupération d’urgence. |
| **Sauvegarde et restauration avec Stockage Blob Azure** |Bases de données de production locales sauvegardées directement dans Stockage Blob Azure pour la reprise d’activité.<br/>![Sauvegarde et restauration](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-backup-restore.png)<br/>Pour plus d’informations, consultez [Sauvegarde et restauration pour SQL Server sur des machines virtuelles Azure](../../../azure-sql/virtual-machines/windows/backup-restore.md). |
| **Réplication et basculement de SQL Server vers Azure avec Azure Site Recovery** |Instance SQL Server de production locale répliquée directement dans Stockage Azure pour la reprise d’activité.<br/>![Réplication à l’aide d’Azure Site Recovery](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-dr-standalone-sqlserver-asr.png)<br/>Pour plus d’informations, consultez l’article [Protéger SQL Server à l’aide de la récupération d’urgence SQL Server et d’Azure Site Recovery](../../../site-recovery/site-recovery-sql.md). |


## <a name="free-dr-replica-in-azure"></a>Réplica de récupération d’urgence gratuit dans Azure

Si vous avez [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?rtc=1&activetab=software-assurance-default-pivot:primaryr3), vous pouvez implémenter des plans de reprise d’activité hybride avec SQL Server sans entraîner des coûts de licence supplémentaires pour l’instance de reprise d’activité passive.

Par exemple, vous pouvez avoir deux réplicas secondaires passifs gratuits lorsque les trois réplicas sont hébergés dans Azure : 

![Deux réplicas passifs gratuits quand tout est dans Azure](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/failover-with-primary-in-azure.png)

Ou vous pouvez configurer un environnement de basculement hybride avec un réplica local principal sous licence, un réplica passif gratuit pour la haute disponibilité, un réplica passif gratuit pour la récupération d’urgence locale et un réplica passif gratuit pour la récupération d’urgence dans Azure :

![Trois réplicas passifs gratuits lorsque l’environnement est hybride avec un réplica local principal](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/hybrid-with-primary-on-prem.png)

Pour plus d’informations, consultez les [conditions de licence du produit](https://www.microsoft.com/licensing/product-licensing/products). 

Pour activer cet avantage, accédez à votre [ressource de machine virtuelle SQL Server](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource). Sélectionnez **Configurer** sous **Paramètres**, puis choisissez l’option **Récupération d’urgence** sous **Licence SQL Server**. Cochez la case pour confirmer que cette machine virtuelle SQL Server sera utilisée comme réplica passif, puis sélectionnez **Appliquer** pour enregistrer vos paramètres. 

![Configurer un réplica de reprise d’activité dans Azure](./media/business-continuity-high-availability-disaster-recovery-hadr-overview/dr-replica-in-portal.png)


## <a name="important-considerations-for-sql-server-hadr-in-azure"></a>Considérations importantes pour HADR SQL Server dans Azure
Les machines virtuelles Azure, le stockage et le réseau ont des caractéristiques opérationnelles différentes par rapport à celles d’une infrastructure informatique non virtualisée sur site. Pour une implémentation réussie d’une solution HADR SQL Server dans Azure, vous devez comprendre ces différences et concevoir votre solution de façon à les gérer.

### <a name="high-availability-nodes-in-an-availability-set"></a>Nœuds haute disponibilité d’un groupe à haute disponibilité
Les groupes à haute disponibilité dans Azure vous permettent de placer les nœuds haute disponibilité dans des domaines d’erreur et des domaines de mise à niveau distincts. La plateforme Azure attribue un domaine de mise à jour et un domaine d’erreur à chaque machine virtuelle de votre groupe à haute disponibilité. Cette configuration au sein d’un centre de données assure la disponibilité d’au moins une des machines virtuelles pendant un événement de maintenance planifié ou non, avec le niveau de 99,95 % stipulé dans le contrat de niveau de service (SLA) Azure. 

Pour une configuration à haute disponibilité, placez toutes les machines virtuelles SQL Server correspondantes dans le même groupe à haute disponibilité afin d’éviter la perte d’applications ou de données lors d’un événement de maintenance. Seuls les nœuds du même service cloud peuvent faire partie du même groupe à haute disponibilité. Pour plus d’informations, consultez [Gestion de la disponibilité des machines virtuelles](../../../virtual-machines/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

### <a name="high-availability-nodes-in-an-availability-zone"></a>Nœuds haute disponibilité d’une zone de disponibilité
Les Zones de disponibilité sont des emplacements physiques uniques au sein d’une région Azure. Chaque zone est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. La séparation physique des zones de disponibilité dans une région contribue à protéger les applications et les données des défaillances dans le centre de données en veillant à ce qu’au moins une machine soit disponible et réponde au contrat de niveau de service Azure de 99,99 %. 

Pour une configuration à haute disponibilité, placez les machines virtuelles SQL Server correspondantes réparties sur les zones de disponibilité dans la région. Des frais supplémentaires sont facturés pour les transferts réseau-à-réseau entre les zones de disponibilité. Pour plus d’informations, consultez [Zones de disponibilité](../../../availability-zones/az-overview.md). 


### <a name="failover-cluster-behavior-in-azure-networking"></a>Comportement d’un cluster de basculement sur le réseau Azure
Le service DHCP non compatible RFC dans Azure peut provoquer l’échec de la création de certaines configurations de cluster de basculement. Cet échec se produit car une adresse IP dupliquée est affectée au nom réseau du cluster, par exemple la même adresse IP que l’un des nœuds du cluster. Cela constitue un problème quand vous utilisez des groupes de disponibilité, qui dépendent de la fonctionnalité Cluster de basculement Windows.

Examinez le scénario où un cluster à deux nœuds est créé et mis en ligne :

1. Le cluster est en ligne, puis NODE1 demande une adresse IP assignée dynamiquement pour le nom du réseau de clusters.
2. Le service DHCP ne fournit aucune adresse IP autre que celle de NODE1, car il reconnaît que la demande provient de NODE1.
3. Windows détecte qu’une adresse en double est affectée à NODE1 et au nom réseau de cluster de basculement, et le groupe de cluster par défaut n’est pas mis en ligne.
4. Le groupe dr cluster par défaut est déplacé vers NODE2. NODE2 traite l’adresse IP de NODE1 comme adresse IP du cluster et met le groupe de cluster par défaut en ligne.
5. Quand NODE2 tente d’établir la connexion avec NODE1, les paquets dirigés vers NODE1 ne quittent jamais NODE2, car il résout l’adresse IP de NODE1 en lui-même. NODE2 ne peut pas établir la connexion avec NODE1, puis perd le quorum et arrête le cluster.
6. NODE1 peut envoyer des paquets à NODE2, mais NODE2 ne peut pas répondre. NODE1 perd le quorum et arrête le cluster.

Vous pouvez éviter ce scénario en affectant une adresse IP statique inutilisée au nom réseau du cluster afin de mettre le nom réseau de cluster en ligne. Par exemple, vous pouvez utiliser une adresse IP Link Local comme 169.254.1.1. Pour simplifier ce processus, consultez la section [Configuration d’un cluster de basculement Windows dans Azure pour les groupes de disponibilité](https://social.technet.microsoft.com/wiki/contents/articles/14776.configuring-windows-failover-cluster-in-windows-azure-for-alwayson-availability-groups.aspx).

Pour plus d’informations, consultez la section [Configurer des groupes de disponibilité dans Azure (GUI)](./availability-group-quickstart-template-configure.md).

### <a name="support-for-availability-group-listeners"></a>Prise en charge des écouteurs de groupe de disponibilité
Les écouteurs de groupe de disponibilité sont pris en charge sur les machines virtuelles Azure exécutant Windows Server 2012 et versions ultérieures. Cette prise en charge est rendue possible par l’utilisation de points de terminaison à charge équilibrée activés sur les machines virtuelles Azure qui sont des nœuds de groupe de disponibilité. Vous devez suivre des étapes de configuration spéciales de façon à ce que les écouteurs fonctionnent avec les applications clientes exécutées dans Azure et avec celles qui s’exécutent localement.

Il existe deux options principales de configuration de votre écouteur : externe (public) ou interne. L’écouteur externe (public) utilise un équilibrage de charge accessible sur Internet et est associé à une adresse IP virtuelle publique accessible via Internet. Un écouteur interne utilise un équilibrage de charge interne et prend uniquement en charge les clients qui se trouvent sur le même réseau virtuel. Quel que soit le type d’équilibrage de charge, vous devez activer le retour direct du serveur. 

Si le groupe de disponibilité s’étend sur plusieurs sous-réseaux Azure (comme un déploiement qui traverse des régions Azure), la chaîne de connexion du client doit inclure `MultisubnetFailover=True`. Ainsi des tentatives de connexion parallèle aux réplicas sont générées dans les différents sous-réseaux. Pour obtenir des instructions sur la configuration d’un écouteur, consultez [Configurer un écouteur ILB pour des groupes de disponibilité dans Azure](availability-group-listener-powershell-configure.md).


Vous pouvez encore vous connecter à chaque réplica de disponibilité séparément en vous connectant directement à l’instance de service. En outre, puisque les groupes de disponibilité sont à compatibilité descendante avec les clients de mise en miroir de bases de données, vous pouvez vous connecter aux réplicas de disponibilité comme les serveurs partenaires de mise en miroir de bases de données tant que les réplicas sont configurés de façon similaire à la mise en miroir de bases de données :

* Il y a un réplica principal et un réplica secondaire.
* Le réplica secondaire est configuré comme non lisible (option **Secondaire accessible en lecture** définie sur **Non**).

Voici un exemple de chaîne de connexion cliente, qui correspond à cette configuration apparentée à une mise en miroir de bases de données, à l’aide d’ADO.NET ou de SQL Server Native Client :

```console
Data Source=ReplicaServer1;Failover Partner=ReplicaServer2;Initial Catalog=AvailabilityDatabase;
```

Pour plus d’informations sur la connectivité client, consultez :

* [Utilisation de mots clés de chaîne de connexion avec SQL Server Native Client](/sql/relational-databases/native-client/applications/using-connection-string-keywords-with-sql-server-native-client)
* [Connecter des clients à une session de mise en miroir de bases de données (SQL Server)](/sql/database-engine/database-mirroring/connect-clients-to-a-database-mirroring-session-sql-server)
* [Connexion à l’écouteur du groupe de disponibilité dans un environnement hybride](/archive/blogs/sqlalwayson/connecting-to-availability-group-listener-in-hybrid-it)
* [Écouteurs de groupe de disponibilité, connectivité client et basculement d’application (SQL Server)](/sql/database-engine/availability-groups/windows/listeners-client-connectivity-application-failover)
* [Utilisation de chaînes de connexion de mise en miroir de bases de données avec des groupes de disponibilité](/sql/database-engine/availability-groups/windows/listeners-client-connectivity-application-failover)

### <a name="network-latency-in-hybrid-it"></a>Latence du réseau dans un environnement hybride
Déployez votre solution HADR en partant du principe qu’il peut y avoir des périodes de latence réseau élevée entre votre réseau local et Azure. Quand vous déployez des réplicas sur Azure, utilisez la validation asynchrone au lieu de la validation synchrone comme mode de synchronisation. Quand vous déployez des serveurs de mise en miroir de bases de données localement et dans Azure, utilisez le mode haute performance plutôt que le mode haute sécurité.

### <a name="geo-replication-support"></a>Prise en charge de la géo-réplication
La géo-réplication dans les disques Azure ne prend pas en charge le fichier de données et le fichier journal de la même base de données à stocker sur des disques distincts. GRS réplique les modifications sur chaque disque indépendamment et de manière asynchrone. Ce mécanisme garantit l’ordre d’écriture dans un seul disque sur la copie géo-répliquée, mais pas entre les copies géo-répliquées de plusieurs disques. Si vous configurez une base de données pour stocker le fichier de données et le fichier journal sur des disques distincts, les disques récupérés après un sinistre peuvent contenir une copie plus à jour du fichier de données que le fichier journal, ce qui interrompt le journal WAL (write-ahead log) dans SQL Server et les propriétés ACID (atomicité, cohérence, isolation et durabilité) des transactions. 

Si vous n’avez pas l’option de désactiver la géoréplication sur le compte de stockage, conservez tous les fichiers de données et fichiers journaux pour une base de données sur le même disque. Si vous devez utiliser plusieurs disques en raison de la taille de la base de données, déployez l’une des solutions de reprise d’activité indiquées plus haut pour assurer la redondance des données.

## <a name="next-steps"></a>Étapes suivantes

Déterminez si un [groupe de disponibilité](availability-group-overview.md) ou une [instance de cluster de basculement](failover-cluster-instance-overview.md) est la meilleure solution de continuité d’activité pour votre entreprise. Passez ensuite en revue les [bonnes pratiques](hadr-cluster-best-practices.md) en matière de configuration de votre environnement pour la haute disponibilité et la reprise d’activité.
