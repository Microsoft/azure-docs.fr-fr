---
title: Créer des clusters sur Windows Server et Linux
description: Les clusters Service Fabric s'exécutent sous Windows Server et Linux. Vous pouvez déployer et héberger des applications Service Fabric partout où vous pouvez exécuter Windows Server ou Linux.
services: service-fabric
documentationcenter: .net
ms.topic: conceptual
ms.date: 02/01/2019
ms.openlocfilehash: 25e6854491f35dd0aa46b5de218d312f57854760
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018917"
---
# <a name="overview-of-service-fabric-clusters-on-azure"></a>Vue d’ensemble des clusters Service Fabric sur Azure
Un cluster Service Fabric est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés. Une machine ou machine virtuelle faisant partie d’un cluster est appelée un nœud de cluster. Les clusters peuvent être mis à l’échelle pour des milliers de nœuds. Si vous ajoutez des nœuds au cluster, Service Fabric rééquilibre les réplicas de partition du service et les instances sur le nombre de nœuds augmenté. Les performances globales de l’application s’améliorent tandis que le conflit d’accès à la mémoire diminue. Si les nœuds du cluster ne sont pas utilisés efficacement, vous pouvez diminuer le nombre de nœuds dans le cluster. Service Fabric rééquilibre à nouveau les réplicas de partition et les instances sur le nombre réduit de nœuds afin de mieux utiliser le matériel sur chaque nœud.

Un type de nœud définit la taille, le nombre et les propriétés d’un ensemble de nœuds (machines virtuelles) du cluster. Chaque type de nœud peut ensuite faire l’objet d’une montée ou descente en puissance de manière indépendante, avoir différents jeux de ports ouverts et présenter différentes métriques de capacité. Les types de nœuds permettent de définir les rôles d’un groupe de nœuds de cluster, par exemple « frontal » ou « back end ». Votre cluster peut avoir plusieurs types de nœuds, mais le type de nœud principal doit avoir au moins cinq machines virtuelles pour les clusters de production (ou au moins trois machines virtuelles pour les clusters de test). Les [services système de Service Fabric](service-fabric-technical-overview.md#system-services) sont placés sur les nœuds du type de nœud principal. 

## <a name="cluster-components-and-resources"></a>Composants et ressources de cluster
Un cluster Service Fabric sur Azure est une ressource Azure qui utilise d’autres ressources Azure et interagit avec celles-ci :
* machines virtuelles et cartes réseaux virtuelles
* Jeux de mise à l’échelle de machine virtuelle
* réseaux virtuels
* équilibreurs de charge
* comptes de stockage
* adresses IP publiques

![Cluster Service Fabric][Image]

### <a name="virtual-machine"></a>Machine virtuelle
Une [machine virtuelle](../virtual-machines/index.yml) qui fait partie d’un cluster est appelée « nœud » même si, techniquement, un nœud de cluster est un processus de runtime Service Fabric. Un nom (chaîne) est affecté à chaque nœud. Les nœuds présentent certaines caractéristiques comme des [propriétés de sélection élective](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints). Chaque machine ou machine virtuelle est dotée d’un service à démarrage automatique, *FabricHost.exe*, qui commence à s’exécuter au moment du démarrage, puis démarre deux exécutables, *Fabric.exe* et *FabricGateway.exe*, qui constituent le nœud. Un déploiement de production comprend un nœud par machine physique ou virtuelle. Pour les scénarios de test, vous pouvez héberger plusieurs nœuds sur une seule et même machine ou machine virtuelle en exécutant plusieurs instances de *Fabric.exe* et de *FabricGateway.exe*.

Chaque machine virtuelle est associée à une carte d’interface réseau (NIC) virtuelle et une adresse IP privée est affectée à chaque carte réseau.  Une machine virtuelle est affectée à un réseau virtuel et à un équilibreur local via la carte réseau.

Toutes les machines virtuelles d’un cluster sont placées dans un réseau virtuel.  Tous les nœuds du même type de nœud/groupe identique sont placés sur le même sous-réseau sur le réseau virtuel.  Ces nœuds ont uniquement des adresses IP privées et ne sont pas directement adressables à l’extérieur du réseau virtuel.  Les clients peuvent accéder aux services sur les nœuds par le biais de l’équilibreur de charge Azure.

### <a name="scale-setnode-type"></a>Groupe identique/type de nœud
Quand vous créez un cluster, vous définissez un ou plusieurs types de nœuds.  Les nœuds, ou machines virtuelles, d’un type de nœud ont la même taille et les mêmes caractéristiques, comme le nombre de processeurs, la mémoire, le nombre de disques et les E/S de disque.  Par exemple, un type de nœud peut être pour les machines virtuelles front-end de petite taille avec des ports ouverts sur Internet tandis qu’un autre type de nœud peut être pour les machines virtuelles back-end de grande taille qui traitent des données. Dans les clusters Azure, chaque type de nœud est mappé à un [groupe de machines virtuelles identiques](../virtual-machine-scale-sets/index.yml).

Vous pouvez utiliser des groupes identiques pour déployer et gérer une collection de machines virtuelles comme un groupe. Chaque type de nœud que vous définissez dans un cluster Azure Service Fabric configure un groupe identique distinct. Le runtime Service Fabric est amorcé sur chaque machine virtuelle du groupe identique à l’aide d’extensions de machine virtuelle Azure. Vous pouvez faire monter ou descendre en puissance chaque type de nœud de manière indépendante, modifier la référence SKU du système d’exploitation s’exécutant sur chaque nœud de cluster, avoir différents ensembles de ports ouverts et utiliser différentes métriques de capacité. Un groupe identique possède cinq [domaines de mise à niveau](service-fabric-cluster-resource-manager-cluster-description.md#upgrade-domains) et cinq [domaines d’erreur](service-fabric-cluster-resource-manager-cluster-description.md#fault-domains), et peut comporter jusqu’à 100 machines virtuelles.  Vous créez des clusters de plus de 100 nœuds en créant plusieurs groupes uniques/types de nœuds.

> [!IMPORTANT]
> Choisir le nombre de types de nœuds pour votre cluster et les propriétés de chaque type de nœud (taille, principal, accessible sur Internet, nombre de machines virtuelles, etc.) est une tâche importante.  Pour plus d’informations, lisez les [Considérations relatives à la planification de la capacité des clusters](service-fabric-cluster-capacity.md).

Pour plus d’informations, consultez [Types de nœuds Service Fabric et groupes de machines virtuelles identiques](service-fabric-cluster-nodetypes.md).

### <a name="azure-load-balancer"></a>Azure Load Balancer
Les instances de machine virtuelle sont jointes derrière un [équilibreur de charge Azure](../load-balancer/load-balancer-overview.md), qui est associé à une [adresse IP publique](../virtual-network/public-ip-addresses.md) et à des étiquette DNS.  Quand vous configurez un cluster avec *&lt;nom_cluster&gt;* , le nom DNS, *&lt;nom_cluster&gt;.&lt;emplacement&gt;.cloudapp.azure.com* est l’étiquette DNS associée à l’équilibreur de charge placé devant le groupe identique.

Les machines virtuelles d’un cluster ont seulement des [adresses IP privées](../virtual-network/private-ip-addresses.md).  Le trafic de gestion et le trafic de service sont routés par le biais de l’équilibreur de charge public.  Le trafic réseau est routé vers ces machines à l’aide de règles NAT (les clients se connectent à des instances/nœuds spécifiques) ou de règles d’équilibrage de charge (le trafic est dirigé vers des machines virtuelles par tourniquet (Round Robin)).  Un équilibreur de charge a une adresse IP publique associée avec un nom DNS au format suivant : *&lt;nom_cluster&gt;.&lt;emplacement&gt;.cloudapp.azure.com*.  Une adresse IP publique est une autre ressource Azure dans le groupe de ressources.  Si vous définissez plusieurs types de nœuds dans un cluster, un équilibreur de charge est créé pour chaque type de nœud/groupe identique. Vous pouvez aussi configurer un seul équilibreur de charge pour plusieurs types de nœuds.  Le type de nœud principal a l’étiquette DNS *&lt;nom_cluster&gt;.&lt;emplacement&gt;.cloudapp.azure.com*. Les autres types de nœuds ont l’étiquette DNS *&lt;nom_cluster&gt;-&lt;type_nœud&gt;.&lt;emplacement&gt;.cloudapp.azure.com*.

### <a name="storage-accounts"></a>Comptes de stockage
Chaque type de nœud de cluster est pris en charge par un [compte de stockage Azure](../storage/common/storage-introduction.md) et des disques managés.

## <a name="cluster-security"></a>Sécurité des clusters
Un cluster Service Fabric est une ressource que vous possédez.  Il vous incombe la responsabilité de sécuriser vos clusters pour empêcher les utilisateurs non autorisés de s’y connecter. La sécurisation des clusters est particulièrement importante lorsque vous exécutez des charges de travail de production sur le cluster. 

### <a name="node-to-node-security"></a>Sécurité nœud à nœud
La sécurité nœud à nœud sécurise la communication entre les machines virtuelles ou les ordinateurs d’un cluster. Ce scénario de sécurité garantit que seuls les ordinateurs qui sont autorisés à rejoindre le cluster peuvent participer à l’hébergement des applications et des services du cluster. Service Fabric utilise des certificats X.509 pour sécuriser un cluster et fournir des fonctionnalités de sécurité d’applications.  Un certificat de cluster est nécessaire pour sécuriser le trafic de cluster et fournir une authentification de cluster et de serveur.  Les certificats auto-signés peuvent être utilisés pour les clusters de test, mais un certificat émis par une autorité de certification approuvée doit être utilisé pour sécuriser les clusters de production.

Pour plus d’informations, consultez [Sécurité nœud à nœud](service-fabric-cluster-security.md#node-to-node-security)

### <a name="client-to-node-security"></a>Sécurité client à nœud
La sécurité client à nœud authentifie les clients et sécurise la communication entre un client et les nœuds du cluster. Ce type de sécurité aide à garantir que seuls les utilisateurs autorisés peuvent accéder au cluster et aux applications déployées sur le cluster. Les clients sont identifiés de manière unique grâce à leurs informations d’identification de sécurité de certificat X.509. Il est possible d’utiliser un nombre quelconque de certificats clients facultatifs pour authentifier les clients d’administration ou d’utilisateur auprès du cluster.

En plus des certificats clients, Azure Active Directory peut également être configuré pour authentifier des clients auprès du cluster.

Pour plus d’informations, consultez [Sécurité client à nœud](service-fabric-cluster-security.md#client-to-node-security)

### <a name="role-based-access-control"></a>Contrôle d’accès en fonction du rôle
Le contrôle d’accès en fonction du rôle Azure (RBAC Azure) vous permet d’affecter des contrôles d’accès affinés sur des ressources Azure.  Vous pouvez affecter des règles d’accès différentes à des abonnements, des groupes de ressources et des ressources.  Les règles de RBAC Azure sont héritées le long de la hiérarchie des ressources, sauf si elles sont remplacées à un niveau inférieur.  Vous pouvez affecter n’importe quel utilisateur ou groupe d’utilisateurs sur votre instance AAD avec des règles de RBAC Azure afin que les utilisateurs et les groupes désignés puissent modifier votre cluster.  Pour plus d’informations, lisez la [vue d’ensemble des rôles RBAC Azure](../role-based-access-control/overview.md).

Service Fabric prend également en charge le contrôle d’accès pour limiter l’accès à certaines opérations de cluster pour différents groupes d’utilisateurs. Ainsi, vous rendez le cluster plus sécurisé. Deux types de contrôle d’accès sont pris en charge pour les clients qui se connectent à un cluster : le rôle Administrateur et le rôle Utilisateur.  

Pour plus d’informations, consultez [Contrôle d’accès en fonction du rôle Service Fabric](service-fabric-cluster-security.md#service-fabric-role-based-access-control).

### <a name="network-security-groups"></a>Groupes de sécurité réseau 
Les groupes de sécurité réseau (NSG) contrôlent le trafic entrant et sortant d’un sous-réseau, d’une machine virtuelle ou d’une carte réseau spécifique.  Par défaut, quand plusieurs machines virtuelles sont placées sur le même réseau virtuel, elles peuvent communiquer entre elles via n’importe quel port.  Si vous voulez limiter les communications entre les machines, vous pouvez définir les groupes de sécurité réseau pour segmenter le réseau ou isoler les machines virtuelles les unes des autres.  Si vous avez plusieurs types de nœuds dans un cluster, vous pouvez appliquer des groupes de sécurité réseau à des sous-réseaux pour empêcher les machines appartenant à des types de nœuds différents de communiquer entre elles.  

Pour plus d’informations, consultez [Groupes de sécurité](../virtual-network/network-security-groups-overview.md).

## <a name="scaling"></a>Mise à l'échelle

La demande des applications change au fil du temps. Il vous faut éventuellement augmenter les ressources de cluster pour prendre en charge l’augmentation de la charge applicative ou du trafic réseau ou diminuer les ressources du cluster en cas de baisse de la demande. Une fois que vous avez créé un cluster Service Fabric, vous pouvez le mettre à l’échelle horizontalement (modifier le nombre de nœuds) ou verticalement (modifier les ressources des nœuds). Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster. Lorsque vous mettez vos nœuds à l’échelle, vos applications sont automatiquement mises à l’échelle.

Pour plus d’informations, consultez [Mise à l’échelle des clusters Azure](service-fabric-cluster-scaling.md).

## <a name="upgrading"></a>Mise à niveau
Un cluster Azure Service Fabric est une ressource que vous possédez mais qui est en partie gérée par Microsoft. Microsoft est responsable de la mise à jour corrective du système d’exploitation sous-jacent et des mises à niveau du runtime Service Fabric sur votre cluster. Vous pouvez définir votre cluster de façon à recevoir des mises à niveau automatiques du runtime quand Microsoft publie une nouvelle version, ou choisir de sélectionner la version de runtime prise en charge de votre choix. En plus des mises à niveau du runtime, vous pouvez également mettre à jour la configuration des clusters comme les certificats ou les ports d’application.

Pour plus d’informations, consultez [Mise à niveau des clusters](service-fabric-cluster-upgrade.md).

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge
Vous pouvez créer des clusters sur des machines virtuelles qui exécutent ces systèmes d’exploitation :

| Système d’exploitation | Version prise en charge de Service Fabric la plus ancienne |
| --- | --- |
| Windows Server 2012 R2 | Toutes les versions |
| Windows Server 2016 | Toutes les versions |
| Windows Server 1709 | 6.0 |
| Windows Server 1803 | 6.4 |
| Windows Server 1809 | 6.4.654.9590 |
| Windows Server 2019 | 6.4.654.9590 |
| Linux Ubuntu 16.04 | 6.0 |
| Linux Ubuntu 18.04 | 7.1 |

Pour plus d’informations, consultez [Versions de cluster prises en charge dans Azure](./service-fabric-versions.md#supported-operating-systems)

> [!NOTE]
> Si vous décidez de déployer Service Fabric sur Windows Server 1709, notez que (1) ce n’est pas une branche destinée à des services à long terme, ce qui vous obligera probablement à changer de versions à l’avenir, et (2) si vous déployez des conteneurs, les conteneurs reposant sur Windows Server 2016 ne fonctionnent pas sur Windows Server 1709, et vice versa (vous devez les régénérer pour les déployer).
>


## <a name="next-steps"></a>Étapes suivantes
Apprenez-en plus sur la [sécurisation](service-fabric-cluster-security.md), la [mise à l’échelle](service-fabric-cluster-scaling.md) et la [mise à niveau](service-fabric-cluster-upgrade.md) des clusters Azure.

Découvrez les [options de support de Service Fabric](service-fabric-support.md).

[Image]: media/service-fabric-azure-clusters-overview/Cluster.PNG