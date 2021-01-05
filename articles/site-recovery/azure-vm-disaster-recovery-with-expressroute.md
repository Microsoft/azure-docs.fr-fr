---
title: Intégrer la récupération d'urgence de machines virtuelles Azure ExpressRoute avec Azure Site Recovery
description: Décrit comment configurer la reprise d’activité pour des machines virtuelles Azure avec Azure Site Recovery et Azure ExpressRoute
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: mayg
ms.openlocfilehash: 0e1f670f2ba5ad31f29d56b2de40acd6e2bf18a9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88654376"
---
# <a name="integrate-expressroute-with-disaster-recovery-for-azure-vms"></a>Intégrer ExpressRoute avec la récupération d'urgence pour les machines virtuelles Azure


Cet article décrit comment intégrer Azure ExpressRoute à [Azure Site Recovery](site-recovery-overview.md) lors de la configuration de la reprise d’activité pour des machines virtuelles Azure dans une région Azure secondaire.

Site Recovery permet la reprise d’activité de machines virtuelles Azure en répliquant les données des machines virtuelles Azure sur Azure.

- Si les machines virtuelles Azure utilisent des [disques managés Azure](../virtual-machines/managed-disks-overview.md), les données des machines virtuelles sont répliquées sur un disque managé répliqué dans la région secondaire.
- Si les machines virtuelles Azure n’utilisent pas de disques managés, les données des machines virtuelles sont répliquées vers un compte de stockage Azure.
- Les points de terminaison de réplication sont publics, mais le trafic de réplication des machines virtuelles Azure ne passe pas par Internet.

ExpressRoute vous permet d’étendre vos réseaux locaux dans le cloud Microsoft Azure via une connexion privée assurée par un fournisseur de connectivité. Si vous avez configuré ExpressRoute, il s’intègre à Site Recovery comme suit :

- **Durant la réplication entre des régions Azure** : le trafic de réplication pour la reprise d’activité de machines virtuelles Azure se fait uniquement au sein d’Azure, et ExpressRoute n’est pas nécessaire ni utilisé pour la réplication. Cependant, si vous vous connectez à partir d’un site local à des machines virtuelles Azure dans le site Azure principal, plusieurs problèmes sont à prendre en compte quand vous configurez la reprise d’activité pour ces machines virtuelles Azure.
- **Basculement entre des régions Azure** : quand des pannes se produisent, vous basculez les machines virtuelles Azure de la région primaire vers une région Azure secondaire. Après avoir basculé vers une région secondaire, vous devez effectuer un certain nombre d’étapes pour accéder avec ExpressRoute aux machines virtuelles Azure dans la région secondaire.


## <a name="before-you-begin"></a>Avant de commencer

Avant de commencer, vérifiez que vous comprenez bien les concepts suivants :

- [Circuits](../expressroute/expressroute-circuit-peerings.md) ExpressRoute
- [Domaines de routage](../expressroute/expressroute-circuit-peerings.md#routingdomains) ExpressRoute
- Emplacements [ExpressRoute](../expressroute/expressroute-locations.md)
- [Architecture de réplication](azure-to-azure-architecture.md) des machines virtuelles Azure
- Comment [configurer la réplication](azure-to-azure-tutorial-enable-replication.md) pour les machines virtuelles Azure.
- Comment [basculer](azure-to-azure-tutorial-failover-failback.md) des machines virtuelles Azure.


## <a name="general-recommendations"></a>Recommandations générales

Pour respecter les bonnes pratiques et garantir des objectifs de temps de récupération efficaces pour la reprise d’activité, nous vous recommandons de procéder comme suit quand vous configurez Site Recovery pour l’intégrer à ExpressRoute :

- Provisionnez les composants réseau avant le basculement vers une région secondaire :
    - Quand vous activez la réplication de machines virtuelles Azure, Site Recovery peut déployer automatiquement des ressources réseau, comme des réseaux, des sous-réseaux et des passerelles, dans la région Azure cible, en fonction des paramètres du réseau source.
    - Site Recovery ne peut pas configurer automatiquement des ressources réseau comme des passerelles de réseau virtuel.
    - Nous vous recommandons de provisionner ces ressources réseau supplémentaires avant un basculement. Un petit temps d’arrêt est associé à ce déploiement, et il peut avoir un impact sur le temps de récupération global si vous n’en avez pas tenu compte lors de la planification du déploiement.
- Effectuez des simulations régulières de reprise d’activité :
    - Une simulation valide votre stratégie de réplication sans perte de données ou temps d’arrêt et n’affecte pas votre environnement de production. Ceci permet d’éviter des problèmes de configuration de dernière minute qui peuvent avoir un impact négatif sur l’objectif de temps de récupération.
    - Quand vous effectuez un test de basculement pour la simulation, nous vous recommandons d’utiliser un réseau distinct de machines virtuelles Azure au lieu du réseau par défaut qui est configuré quand vous activez la réplication.
- Utilisez des espaces d’adressage IP différents si vous avez un seul circuit ExpressRoute.
    - Nous vous recommandons d’utiliser un espace d’adressage IP différent pour le réseau virtuel cible. Ceci évite des problèmes lors de l’établissement de connexions en cas de pannes régionales.
    - Si vous ne pouvez pas utiliser un espace d’adressage distinct, veillez à effectuer le test de basculement de la simulation de reprise d’activité sur un réseau de test distinct avec des adresses IP différentes. Vous ne pouvez pas connecter deux réseaux virtuels avec un chevauchement d’espace d’adressage IP au même circuit ExpressRoute.

## <a name="replicate-azure-vms-when-using-expressroute"></a>Répliquer des machines virtuelles Azure lors de l’utilisation d’ExpressRoute


Si vous voulez configurer la réplication pour des machines virtuelles Azure dans un site principal et que vous vous connectez à ces machines virtuelles à partir de votre site local via ExpressRoute, voici ce que vous devez faire :

1. [Activez la réplication](azure-to-azure-tutorial-enable-replication.md) pour chaque machine virtuelle Azure.
2. Vous pouvez laisser Site Recovery configurer la mise en réseau :
    - Quand vous configurez et que vous activez la réplication, Site Recovery définit des réseaux, des sous-réseaux et les sous-réseaux de passerelle dans la région Azure cible, de façon à les faire correspondre à ceux de la région source. Site Recovery effectue aussi un mappage entre les réseaux virtuels sources et cibles.
    - Si vous ne voulez pas que Site Recovery effectue ceci automatiquement, créez les ressources réseau du côté cible avant d’activer la réplication.
3. Créez les autres éléments du réseau :
    - Site Recovery ne crée pas les tables de routage, les passerelles de réseau virtuel, les connexions de passerelle de réseau virtuel, le peering des réseaux virtuels, ni d’autres ressources et connexions réseau dans la région secondaire.
    - Vous devez créer ces éléments de mise en réseau supplémentaires dans la région secondaire, n’importe quand mais avant d’effectuer un basculement depuis la région primaire.
    - Vous pouvez utiliser des [plans de récupération](site-recovery-create-recovery-plans.md) et des scripts d’automatisation pour configurer et connecter ces ressources réseau.
1. Si une appliance virtuelle réseau est déployée pour contrôler le flux de trafic réseau, notez que :
    - La route système par défaut d’Azure pour la réplication de machines virtuelles Azure est 0.0.0.0/0.
    - En règle générale, les déploiements d’appliance virtuelle réseau définissent également un itinéraire par défaut (0.0.0.0/0) qui force le trafic Internet sortant à circuler via l’appliance virtuelle réseau. La route par défaut est utilisée quand aucune autre configuration de routage spécifique ne peut être trouvée.
    - Si c’est le cas, l’appliance virtuelle réseau peut se trouver surchargée si tout le trafic de réplication passe par cette appliance.
    - La même limitation s’applique également lors de l’utilisation de routes par défaut pour le routage de tout le trafic des machines virtuelles Azure vers des déploiements locaux.
    - Dans ce scénario, nous vous recommandons de [créer un point de terminaison de service réseau](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage) dans votre réseau virtuel pour le service Microsoft.Storage, afin que le trafic de réplication ne quitte pas les limites d’Azure.

## <a name="replication-example"></a>Exemple de réplication

Les déploiements d’entreprise classiques ont des charges de travail réparties sur plusieurs réseaux virtuels Azure, avec un hub de connectivité central pour la connectivité externe à Internet et aux sites locaux. Une topologie hub-and-spoke est généralement utilisée avec ExpressRoute.

![Site local vers Azure avec ExpressRoute avant le basculement](./media/azure-vm-disaster-recovery-with-expressroute/site-recovery-with-expressroute-before-failover.png)

- **Région**. Les applications sont déployées dans la région Azure Asie Est.
- **Réseaux virtuels spoke**. Les applications sont déployées dans deux réseaux virtuels spoke :
    - **Réseau virtuel source 1** : 10.1.0.0/24.
    - **Réseau virtuel source 2** : 10.2.0.0/24.
    - Chaque réseau virtuel spoke est connecté au **réseau virtuel hub**.
- **Réseau virtuel hub**. Il existe un réseau virtuel hub **Réseau virtuel hub source** : 10.10.10.0/24.
  - Ce réseau virtuel hub agit comme opérateur de contrôle.
  - Toutes les communications entre les sous-réseaux passent par ce hub.
    - **Sous-réseaux du réseau virtuel hub**. Le réseau virtuel hub a deux sous-réseaux :
    - **Sous-réseau de l’appliance virtuelle réseau** : 10.10.10.0/25. Ce sous-réseau contient une appliance virtuelle réseau (10.10.10.10).
    - **Sous-réseau de passerelle** : 10.10.10.128/25. Ce sous-réseau contient une passerelle ExpressRoute connectée à une connexion ExpressRoute qui route le trafic vers le site local via un domaine de routage de peering privé.
- Le centre de données local a une connexion de circuit ExpressRoute via un réseau de périphérie partenaire à Hong Kong (R.A.S.).
- Tout le routage est contrôlé via des tables de routage Azure.
- Tout le trafic sortant entre les réseaux virtuels ou en direction du centre de données local est routé via l’appliance virtuelle réseau.

### <a name="hub-and-spoke-peering-settings"></a>Paramètres du peering hub-and-spoke

#### <a name="spoke-to-hub"></a>Spoke à hub

**Sens** | **Paramètre** | **State**
--- | --- | ---
Spoke à hub | Autoriser l’adresse du réseau virtuel | activé
Spoke à hub | Autoriser le trafic transféré | activé
Spoke à hub | Autoriser le transit par passerelle | Désactivé
Spoke à hub | Utiliser des passerelles à distance | activé

 ![Configuration du peering spoke vers hub](./media/azure-vm-disaster-recovery-with-expressroute/spoke-to-hub-peering-configuration.png)

#### <a name="hub-to-spoke"></a>Hub à spoke

**Sens** | **Paramètre** | **State**
--- | --- | ---
Hub à spoke | Autoriser l’adresse du réseau virtuel | activé
Hub à spoke | Autoriser le trafic transféré | activé
Hub à spoke | Autoriser le transit par passerelle | activé
Hub à spoke | Utiliser des passerelles à distance | Désactivé

 ![Configuration du peering hub vers spoke](./media/azure-vm-disaster-recovery-with-expressroute/hub-to-spoke-peering-configuration.png)

### <a name="example-steps"></a>Étapes de l’exemple

Dans notre exemple, voici ce qui doit se produire lors de l’activation de la réplication pour des machines virtuelles Azure dans le réseau source :

1. Vous [activez la réplication](azure-to-azure-tutorial-enable-replication.md) pour une machine virtuelle.
2. Site Recovery crée les réseaux virtuels, les sous-réseaux et les sous-réseaux de passerelle du réplica dans la région cible.
3. Site Recovery crée des mappages entre les réseaux sources et les réseaux de cible du réplica qu’il crée.
4. Vous créez manuellement les passerelles de réseau virtuel, les connexions de passerelle de réseau virtuel, le peering de réseaux virtuels ou les autres ressources et connexions réseau.


## <a name="fail-over-azure-vms-when-using-expressroute"></a>Basculer des machines virtuelles Azure lors de l’utilisation d’ExpressRoute

Après avoir basculé des machines virtuelles Azure vers la région Azure cible avec Site Recovery, vous pouvez y accéder en utilisant le [peering privé](../expressroute/expressroute-circuit-peerings.md#privatepeering) ExpressRoute.

- Vous devez connecter ExpressRoute au réseau virtuel cible avec une nouvelle connexion. La connexion ExpressRoute existante n’est pas transférée automatiquement.
- La façon dont vous configurez votre connexion ExpressRoute au réseau virtuel cible dépend de votre topologie ExpressRoute.


### <a name="access-with-two-circuits"></a>Accès avec deux circuits

#### <a name="two-circuits-with-two-peering-locations"></a>Deux circuits avec deux emplacements de peering

Cette configuration protège les circuits ExpressRoute contre les sinistres régionaux. Si votre emplacement de peering principal tombe en panne, les connexions peuvent être maintenues à partir de l'autre emplacement.

- Le circuit connecté à l’environnement de production est généralement le circuit principal. Le circuit secondaire a généralement une bande passante inférieure, qui peut être augmentée si un sinistre se produit.
- Après le basculement, vous pouvez établir des connexions du circuit ExpressRoute secondaire vers le réseau virtuel cible. Vous pouvez aussi avoir des connexions configurées et prêtes en cas de sinistre, de façon à réduire le temps de récupération global.
- Avec des connexions simultanées aux réseaux virtuels primaires et cibles, vérifiez que votre routage local utilise le circuit et la connexion secondaires seulement après le basculement.
- Les réseaux virtuels sources et cibles peuvent recevoir de nouvelles adresses IP ou conserver les mêmes après le basculement. Dans les deux cas, les connexions secondaires peuvent être établies avant le basculement.


#### <a name="two-circuits-with-single-peering-location"></a>Deux circuits avec un seul emplacement de peering

Cette configuration permet de se protéger contre les défaillances du circuit ExpressRoute principal, mais pas si le seul emplacement de peering ExpressRoute tombe en panne, en impactant les deux circuits.

- Vous pouvez avoir des connexions simultanées du centre de données local vers le réseau virtuel source avec le circuit principal, et vers le réseau virtuel cible avec le circuit secondaire.
- Avec des connexions simultanées vers les réseaux virtuels primaire et cible, vérifiez que le routage local utilise le circuit et la connexion secondaires seulement après le basculement.
-   Vous ne pouvez pas connecter les deux circuits au même réseau virtuel quand les circuits sont créés au même emplacement de peering.

### <a name="access-with-a-single-circuit"></a>Accès avec un seul circuit

Dans cette configuration, il n’y a qu’un seul circuit ExpressRoute. Bien que le circuit ait une connexion redondante au cas où une des deux soit défaillante, un circuit avec une seule route n’offre pas de résilience si votre région de peering connaît une défaillance. Notez les points suivants :

- Vous pouvez répliquer des machines virtuelles Azure vers n’importe quelle région Azure au [même emplacement géographique](azure-to-azure-support-matrix.md#region-support). Si la région Azure cible n’est pas au même emplacement que la source, vous devez activer ExpressRoute Premium si vous utilisez un seul circuit ExpressRoute. Découvrez plus d’informations sur les [Emplacements ExpressRoute](../expressroute/expressroute-locations.md) et la [Tarification ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/).
- Vous ne pouvez pas connecter simultanément les réseaux virtuels sources et cibles au circuit si le même espace d’adressage IP est utilisé sur la région cible. Dans ce scénario :    
    -  Déconnectez la connexion du côté source, puis établissez la connexion du côté cible. Cette modification de la connexion peut faire l’objet d’un script dans le cadre d’un plan de récupération Site Recovery. Notez les points suivants :
        - En cas de défaillance régionale, si la région primaire n’est pas accessible, l’opération de déconnexion peut échouer. Ceci peut affecter la création d’une connexion vers la région cible.
        - Si vous avez créé la connexion dans la région cible et que la région primaire récupère ultérieurement, vous pouvez être confronté à des pertes de paquets quand deux connexions simultanées essaient de se connecter au même espace d’adressage.
        - Pour éviter cela, mettez fin immédiatement à la connexion principale.
        - Après la restauration automatique des machines virtuelles sur la région primaire, la connexion principale peut être à nouveau établie après l’interruption de la connexion secondaire.
-   Si un espace d’adressage différent est utilisé sur le réseau virtuel cible, vous pouvez vous connecter simultanément aux réseaux virtuels sources et cibles du même circuit ExpressRoute.


## <a name="failover-example"></a>Exemple de basculement

Dans notre exemple, nous utilisons la topologie suivante :

- Deux circuits ExpressRoute différents dans deux emplacements de peering ExpressRoute différents.
- Les adresses IP privées pour les machines virtuelles Azure sont conservées après le basculement.
- La région de récupération cible est Azure Asie Sud-Est.
- Une connexion de circuit ExpressRoute secondaire est établie via un réseau de périphérie partenaire à Singapour.

Pour une topologie simple qui utilise un seul circuit ExpressRoute avec la même adresse IP après le basculement, [lisez cet article](site-recovery-retain-ip-azure-vm-failover.md#hybrid-resources-full-failover).

### <a name="example-steps"></a>Étapes de l’exemple
Pour automatiser la récupération dans cet exemple, voici ce que vous devez faire :

1. Suivez les étapes pour configurer la réplication.
2. [Basculez les machines virtuelles Azure](azure-to-azure-tutorial-failover-failback.md) avec ces étapes supplémentaires, pendant ou après le basculement.

    a. Créez la passerelle Azure ExpressRoute dans le réseau virtuel hub de la région cible. Ceci est nécessaire pour connecter le réseau virtuel hub cible au circuit ExpressRoute.

    b. Créez la connexion depuis le réseau virtuel hub cible vers le circuit ExpressRoute cible.

    c. Configurez les peerings de réseau virtuel entre les réseaux virtuels hub and spoke de la région cible. Les propriétés de peering sur la région cible seront identiques à celles de la région source.

    d. Configurez les UDR dans le réseau virtuel hub et les deux réseaux virtuels spoke.

    - Les propriétés des UDR côté cible sont les mêmes que celles côté source lorsque vous utilisez des adresses IP identiques.
    - Avec des adresses IP cibles différentes, les UDR doivent être modifiés en conséquence.


Les étapes ci-dessus peuvent faire l’objet d’un script dans le cadre d’un [plan de récupération](site-recovery-create-recovery-plans.md). En fonction des exigences en matière de temps de récupération et de connectivité de l’application, les étapes ci-dessus peuvent également être réalisées avant le début du basculement.

#### <a name="after-recovery"></a>Après la récupération

Après la récupération des machines virtuelles et l’établissement de la connectivité, l’environnement de récupération se présente comme suit.

![Site local vers Azure avec ExpressRoute après le basculement](./media/azure-vm-disaster-recovery-with-expressroute/site-recovery-with-expressroute-after-failover.png)



## <a name="next-steps"></a>Étapes suivantes

Découvrez plus d’informations sur les [plans de récupération](site-recovery-create-recovery-plans.md) pour automatiser le basculement d’application.
