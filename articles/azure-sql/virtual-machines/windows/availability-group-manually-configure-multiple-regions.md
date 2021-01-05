---
title: Configurer un groupe de disponibilité SQL Server Always On dans différentes régions
description: Cet article explique comment configurer un groupe de disponibilité SQL Server Always On sur des machines virtuelles avec un réplica dans une autre région.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.assetid: 388c464e-a16e-4c9d-a0d5-bb7cf5974689
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/02/2017
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: 60bb5ac652a80b5ae52c91f91fa0c80440e9cc82
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359079"
---
# <a name="configure-a-sql-server-always-on-availability-group-across-different-azure-regions"></a>Configurer un groupe de disponibilité SQL Server Always On dans différentes régions Azure

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment configurer un réplica de groupe de disponibilité SQL Server AlwaysOn sur des machines virtuelles Azure dans un emplacement Azure distant. Utilisez cette configuration pour prendre en charge la récupération d’urgence.

Cet article s’applique aux machines virtuelles Azure s’exécutant en mode Resource Manager.

L’illustration suivante montre un déploiement classique d’un groupe de disponibilité sur des machines virtuelles Azure :

   ![Diagramme qui montrant l’équilibreur de charge Azure et le groupe à haute disponibilité avec un « cluster de basculement Windows Server » et un « groupe de disponibilité Always On ».](./media/availability-group-manually-configure-multiple-regions/00-availability-group-basic.png)

Dans ce déploiement, toutes les machines virtuelles sont dans une région Azure. Les réplicas de groupe de disponibilité peuvent avoir une validation synchrone avec basculement automatique sur SQL-1 et SQL-2. Pour générer cette architecture, consultez [Modèle de groupe de disponibilité ou didacticiel](availability-group-overview.md).

Cette architecture est vulnérable aux interruptions de service si la région Azure devient inaccessible. Pour résoudre cette vulnérabilité, ajoutez un réplica dans une autre région Azure. Le schéma suivant illustre l’aspect de la nouvelle architecture :

   ![Récupération d’urgence du groupe de disponibilité](./media/availability-group-manually-configure-multiple-regions/00-availability-group-basic-dr.png)

Le diagramme précédent illustre une nouvelle machine virtuelle appelée SQL-3. SQL-3 se trouve dans une autre région Azure. SQL-3 est ajoutée au cluster de basculement Windows Server. SQL-3 peut héberger un réplica de groupe de disponibilité. Notez enfin que la région Azure de SQL-3 a un nouvel équilibrage de charge Azure.

>[!NOTE]
> Un groupe à haute disponibilité Azure est requis lorsque plusieurs machines virtuelles se trouvent dans la même région. Si une seule machine virtuelle se trouve dans la région, le groupe à haute disponibilité n’est pas nécessaire. Vous ne pouvez placer une machine virtuelle dans un groupe à haute disponibilité que lors de la création. Si la machine virtuelle se trouve déjà dans un groupe à haute disponibilité, vous pouvez ajouter une machine virtuelle à un réplica supplémentaire ultérieurement.

Dans cette architecture, le réplica dans la région distante est normalement configuré avec le mode de disponibilité à validation asynchrone et le mode de basculement manuel.

Lorsque les réplicas du groupe de disponibilité se trouvent sur des machines virtuelles Azure dans différentes régions Azure, chaque région requiert :

* Une passerelle de réseau virtuel
* Une connexion à la passerelle de réseau virtuel

Le schéma suivant montre comment les réseaux communiquent entre les centres de données.

   ![Diagramme montrant les deux réseaux virtuels dans différentes régions Azure communiquant à l’aide de passerelles VPN.](./media/availability-group-manually-configure-multiple-regions/01-vpngateway-example.png)

>[!IMPORTANT]
>Cette architecture facture les données sortantes des données répliquées entre des régions Azure. Consultez [Détails de la tarification de la bande passante](https://azure.microsoft.com/pricing/details/bandwidth/).  

## <a name="create-remote-replica"></a>Créer un réplica distant

Pour créer un réplica dans un centre de données distant, procédez comme suit :

1. [Créez un réseau virtuel dans la nouvelle région](../../../virtual-network/manage-virtual-network.md#create-a-virtual-network).

1. [Configurez une connexion de réseau virtuel à réseau virtuel à l’aide du portail Azure](../../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md).

   >[!NOTE]
   >Parfois, vous serez amené à utiliser PowerShell pour créer la connexion de réseau virtuel à réseau virtuel. Par exemple, si vous utilisez différents comptes Azure, vous ne pouvez pas configurer la connexion dans le portail. Dans ce cas, [configurez une connexion de réseau virtuel à réseau virtuel à l’aide du portail Azure](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md).

1. [Créez un contrôleur de domaine dans la nouvelle région](/windows-server/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100).

   Ce contrôleur de domaine assure l’authentification si le contrôleur de domaine dans le site principal n’est pas disponible.

1. [Créez une machine virtuelle SQL Server dans la nouvelle région](create-sql-vm-portal.md).

1. [Créez un équilibrage de charge Azure dans le réseau sur la nouvelle région](availability-group-manually-configure-tutorial.md#configure-internal-load-balancer).

   Cet équilibrage de charge doit :

   - Être dans les mêmes réseau et sous-réseau que la nouvelle machine virtuelle.
   - Avoir une adresse IP statique pour l’écouteur du groupe de disponibilité.
   - Inclure un pool principal comprenant uniquement des machines virtuelles situées dans la même région que l’équilibrage de charge.
   - Utiliser une sonde de port TCP propre à l’adresse IP.
   - Avoir une règle d’équilibrage de charge propre à SQL Server dans la même région.  
   - Être un équilibreur de charge Standard si les machines virtuelles dans le pool de back-ends ne font pas partie d’un groupe à haute disponibilité ni d’un groupe de machines virtuelles identiques. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Load Balancer Standard](../../../load-balancer/load-balancer-overview.md).

1. [Ajoutez la fonction de Clustering avec basculement au nouveau serveur SQL Server](availability-group-manually-configure-prerequisites-tutorial.md#add-failover-clustering-features-to-both-sql-server-vms).

1. [Joignez le nouveau SQL Server au domaine](availability-group-manually-configure-prerequisites-tutorial.md#joinDomain).

1. [Configurez le nouveau compte de service SQL Server pour qu’il utilise un compte de domaine](availability-group-manually-configure-prerequisites-tutorial.md#setServiceAccount).

1. [Ajoutez le nouveau SQL Server au cluster de basculement Windows Server](availability-group-manually-configure-tutorial.md#addNode).

1. Ajoutez une ressource d’adresse IP au cluster.

   Vous pouvez créer la ressource à adresse IP dans le Gestionnaire du cluster de basculement. Sélectionnez le nom du cluster, cliquez dessus avec le bouton droit sous **Principales ressources du cluster**, puis sélectionnez **Propriétés** : 

   ![Capture d’écran montrant le « Gestionnaire du cluster de basculement » avec un nom de cluster, « Nom du serveur » et « Propriétés » sélectionnés.](./media/availability-group-manually-configure-multiple-regions/cluster-name-properties.png)

   Dans la boîte de dialogue **Propriétés**, sous **Adresse IP**, sélectionnez **Ajouter**, puis ajoutez l’adresse IP correspondant au nom du cluster à partir de la région du réseau distant. Sélectionnez **OK** dans la boîte de dialogue **Adresse IP**, puis de nouveau **OK** dans la boîte de dialogue **Propriétés du cluster** pour enregistrer la nouvelle adresse IP. 

   ![Ajouter l’adresse IP du cluster](./media/availability-group-manually-configure-multiple-regions/add-cluster-ip-address.png)


1. Ajoutez l’adresse IP en tant que dépendance pour le nom du cluster principal.

   Rouvrez les propriétés du cluster, puis sélectionnez l’onglet **Dépendances**. Configurez une dépendance OU pour les deux adresses IP : 

   ![Propriétés du cluster](./media/availability-group-manually-configure-multiple-regions/cluster-ip-dependencies.png)

1. Ajoutez une ressource d’adresse IP au rôle du groupe de disponibilité dans le cluster. 

   Dans Gestionnaire du cluster de basculement, cliquez avec le bouton droit sur le rôle du groupe de disponibilité, choisissez **Ajouter une ressource**, **Autres ressources**, puis **Adresse IP**.

   ![Création d’une adresse IP](./media/availability-group-manually-configure-multiple-regions/20-add-ip-resource.png)

   Configurez cette adresse IP comme suit :

   - Utilisez le réseau à partir du centre de données distant.
   - Attribuez l’adresse IP à partir du nouvel équilibrage de charge Azure. 

1. Ajoutez la ressource à adresse IP comme une dépendance du cluster (nom de réseau) du point d’accès au client écouteur.

   La capture d’écran suivante illustre une ressource de cluster à adresse IP configurée correctement :

   ![Groupe de disponibilité](./media/availability-group-manually-configure-multiple-regions/50-configure-dependency-multiple-ip.png)

   >[!IMPORTANT]
   >Le groupe de ressources de cluster inclut les deux adresses IP. Ces deux adresses IP sont des dépendances du point d’accès au client écouteur. Utilisez l’opérateur **OR** dans la configuration de dépendance de cluster.

1. [Définissez les paramètres de cluster dans PowerShell](availability-group-manually-configure-tutorial.md#setparam).

   Exécutez le script PowerShell avec le nom réseau du cluster, une adresse IP et un port de sonde que vous avez configurés sur l’équilibrage de charge dans la nouvelle région.

   ```powershell
   $ClusterNetworkName = "<MyClusterNetworkName>" # The cluster name for the network in the new region (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name).
   $IPResourceName = "<IPResourceName>" # The cluster name for the new IP Address resource.
   $ILBIP = "<n.n.n.n>" # The IP Address of the Internal Load Balancer (ILB) in the new region. This is the static IP address for the load balancer you configured in the Azure portal.
   [int]$ProbePort = <nnnnn> # The probe port you set on the ILB.

   Import-Module FailoverClusters

   Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   ```

1. Sur le nouveau SQL Server dans le Gestionnaire de configuration SQL Server, [activez les groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/enable-and-disable-always-on-availability-groups-sql-server).

1. [Ouvrez les ports de pare-feu sur le nouveau SQL Server](availability-group-manually-configure-prerequisites-tutorial.md#endpoint-firewall).

   Les numéros de port que vous devez ouvrir varient selon votre environnement. Ouvrez les ports du point de terminaison de la mise en miroir et de la sonde d’intégrité d’Azure Load Balancer.


1. [Ajoutez un réplica au groupe de disponibilité sur le nouveau SQL Server](/sql/database-engine/availability-groups/windows/use-the-add-replica-to-availability-group-wizard-sql-server-management-studio).

   Pour un réplica dans une région Azure distante, configurez-le pour une réplication asynchrone avec basculement manuel.  

## <a name="set-connection-for-multiple-subnets"></a>Configurer la connexion à plusieurs sous-réseaux

Le réplica dans le centre de données distant fait partie du groupe de disponibilité, mais il se trouve dans un autre sous-réseau. Si ce réplica devient le réplica principal, la connexion d’application peut être perturbée. Ce comportement est identique à un groupe de disponibilité local dans un déploiement de plusieurs sous-réseaux. Pour autoriser les connexions des applications clientes, mettez à jour la connexion cliente ou configurez la mise en cache de la résolution du nom sur la ressource de nom de réseau de cluster.

Idéalement, mettez à jour les chaînes de connexion au client pour qu’elles indiquent `MultiSubnetFailover=Yes`. Consultez [Connexion à MultiSubnetFailover](/sql/relational-databases/native-client/features/sql-server-native-client-support-for-high-availability-disaster-recovery#Anchor_0).

Si vous ne pouvez pas modifier les chaînes de connexion, vous pouvez configurer la mise en cache de la résolution des noms. Consultez [Erreur de délai d’attente et vous ne pouvez pas vous connecter à un écouteur de groupe de disponibilité AlwaysOn SQL Server 2012 dans un environnement à plusieurs sous-réseaux](https://support.microsoft.com/help/2792139/time-out-error-and-you-cannot-connect-to-a-sql-server-2012-alwayson-av).

## <a name="fail-over-to-remote-region"></a>Basculer vers la région distante

Pour tester la connectivité de l’écouteur à la région distante, vous pouvez basculer le réplica vers la région distante. Si le réplica est asynchrone, le basculement est vulnérable à la perte potentielle de données. Pour effectuer un basculement sans perte de données, modifiez le mode de disponibilité en synchrone et définissez le mode de basculement automatique. Utiliser les étapes suivantes :

1. Dans l’**Explorateur d’objets**, connectez-vous à l’instance de SQL Server qui héberge le réplica principal.
1. Sous **Groupes de disponibilité AlwaysOn**, **Groupes de disponibilité**, cliquez avec le bouton droit sur votre groupe de disponibilité, puis sélectionnez **Propriétés**.
1. Dans la page **Général**, sous **Réplicas de disponibilité**, configurez le réplica secondaire dans le site de récupération d’urgence pour qu’il utilise le mode de disponibilité **Validation synchrone** et le mode de basculement **Automatique**.
1. Si vous avez un réplica secondaire dans le même site que le réplica principal pour la haute disponibilité, configurez ce réplica en **Validation asynchrone** et **Manuel**.
1. Sélectionnez OK.
1. Dans l’**Explorateur d’objets**, cliquez avec le bouton droit sur le groupe de disponibilité, puis sélectionnez **Afficher le tableau de bord**.
1. Dans le tableau de bord, vérifiez que le réplica sur le site de récupération d’urgence est synchronisé.
1. Dans l’**Explorateur d’objets**, cliquez avec le bouton droit sur le groupe de disponibilité, puis sélectionnez **Basculer...** . SQL Server Management Studio ouvre un assistant pour effectuer le basculement vers SQL Server.  
1. Sélectionnez **Suivant** et sélectionnez l’instance de SQL Server sur le site de reprise d’activité. Sélectionnez **Suivant** de nouveau.
1. Connectez-vous à l’instance de SQL Server sur le site de reprise d’activité et sélectionnez **Suivant**.
1. Dans la page **Synthèse**, vérifiez les paramètres et sélectionnez **Terminer**.

Après avoir testé la connectivité, replacez le réplica principal dans votre centre de données principal et rétablissez les paramètres de fonctionnement normaux du mode de disponibilité. Le tableau suivant présente les paramètres de fonctionnement normaux de l’architecture décrite dans ce document :

| Emplacement | Instance de serveur | Role | Mode de disponibilité | Mode de basculement
| ----- | ----- | ----- | ----- | -----
| Centre de données principal | SQL-1 | Principal | Synchrone | Automatique
| Centre de données principal | SQL-2 | Secondary | Synchrone | Automatique
| Centre de données secondaire ou distant | SQL-3 | Secondary | Asynchrone | Manuel


### <a name="more-information-about-planned-and-forced-manual-failover"></a>Plus d’informations sur le basculement manuel forcé et planifié

Pour plus d'informations, voir les rubriques suivantes :

- [Effectuer un basculement manuel planifié d'un groupe de disponibilité (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-planned-manual-failover-of-an-availability-group-sql-server)
- [Effectuer un basculement manuel forcé d'un groupe de disponibilité (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server)

## <a name="next-steps"></a>Étapes suivantes

* [Groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server)
* [Machines virtuelles Azure](../../../virtual-machines/windows/index.yml)
* [Équilibrages de charge Azure](availability-group-manually-configure-tutorial.md#configure-internal-load-balancer)
* [Groupes à haute disponibilité Azure](../../../virtual-machines/manage-availability.md)