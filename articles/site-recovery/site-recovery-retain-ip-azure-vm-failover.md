---
title: Conserver des adresses IP après un basculement des machines virtuelles Azure avec Azure Site Recovery
description: Décrit comment conserver les adresses IP en cas de défaillance des machines virtuelles Azure pour la reprise après sinistre dans une région secondaire avec Azure Site Recovery.
ms.service: site-recovery
ms.date: 4/9/2019
author: mayurigupta13
ms.topic: conceptual
ms.author: mayg
ms.openlocfilehash: 650fb7f0877a98ef53ed3868550f9c084ecb5885
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023548"
---
# <a name="retain-ip-addresses-during-failover"></a>Conserver les adresses IP pendant le basculement

[Azure Site Recovery](site-recovery-overview.md) permet la reprise d’activité pour les machines virtuelles Azure grâce à la réplication des machines virtuelles vers une autre région Azure, le basculement en cas de panne et la restauration automatique de la région primaire quand les choses sont revenues à la normale.

Pendant le basculement, vous souhaiterez peut-être conserver le même adressage IP dans la région cible que dans la région source :

- Par défaut, quand vous activez la reprise d’activité pour les machines virtuelles Azure, Site Recovery crée des ressources cibles en fonction des paramètres de ressources sources. Pour les machines virtuelles Azure configurées avec des adresses IP statiques, Site Recovery tente de provisionner la même adresse IP pour la machine virtuelle cible, si elle n’est pas en cours d’utilisation. Pour obtenir une explication complète de la façon dont Site Recovery gère l’adressage, [consultez cet article](azure-to-azure-network-mapping.md#set-up-ip-addressing-for-target-vms).
- Pour les applications simples, la configuration par défaut suffit. Pour les applications plus complexes, vous devrez peut-être provisionner des ressources supplémentaires pour être sûr que la connectivité fonctionne comme prévu après le basculement.


Cet article fournit des exemples permettant de conserver les adresses IP dans les exemples de scénarios plus complexes. Il s’agit des scénarios suivants :

- Basculement pour une société dont toutes les ressources s’exécutent dans Azure
- Basculement pour une société avec un déploiement hybride et des ressources exécutées à la fois localement et dans Azure

## <a name="resources-in-azure-full-failover"></a>Ressources dans Azure : basculement complet

Toutes les applications de la société A s’exécutent dans Azure.

### <a name="before-failover"></a>Avant le basculement

Voici l’architecture avant le basculement.

- La société A a des réseaux et des sous-réseaux identiques dans les régions Azure source et cible.
- Pour réduire l’objectif de délai de récupération (RTO), la société utilise des nœuds de réplica pour SQL Server Always On, les contrôleurs de domaine, et ainsi de suite. Ces nœuds de réplica sont sur un autre réseau virtuel dans la région cible, afin de pouvoir établir la connectivité de site à site VPN entre les régions source et cible. Cela n’est pas possible si le même espace d’adressage IP est utilisé dans la source et la cible.  
- Avant le basculement, l’architecture réseau est la suivante :
    - La région primaire Azure est Asie Est
        - Asie Est a un réseau virtuel (**Source VNet**) avec l’espace d’adressage 10.1.0.0/16
        - Asie Est a des charges de travail réparties entre trois sous-réseaux sur le réseau virtuel :
            - **Sous-réseau 1** : 10.1.1.0/24
            - **Sous-réseau 2** : 10.1.2.0/24
            - **Sous-réseau 3** : 10.1.3.0/24
    - La région Azure secondaire (cible) est Asie Sud-Est
        - Asie Sud-Est a un réseau virtuel de récupération (**Recovery VNet**) identique à **Source VNet**.
        - Asie Sud-Est a un réseau virtuel supplémentaire (**Azure VNet**) avec l’espace d’adressage 10.2.0.0/16
        - **Azure VNet** contient un sous-réseau (**Subnet 4**) avec l’espace d’adressage 10.2.4.0/24
        - Les nœuds de réplica pour SQL Server Always On, le contrôleur de domaine, et ainsi de suite, sont situés sur **Subnet 4**.
    - **Source VNet** et **Azure VNet** sont connectés par le biais d’une connexion VPN de site à site
    - **Recovery VNet** n’est connecté à aucun autre réseau virtuel.
    - La **société A** affecte/vérifie les adresses IP cibles pour les éléments répliqués. L’adresse IP cible est identique à l’adresse IP source pour chaque machine virtuelle.

![Ressources dans Azure avant le basculement complet](./media/site-recovery-retain-ip-azure-vm-failover/azure-to-azure-connectivity-before-failover2.png)

### <a name="after-failover"></a>Après le basculement

Si une panne se produit dans la région source, la société A peut basculer toutes ses ressources vers la région cible.

- Avec des adresses IP cibles déjà en place avant le basculement, la société A peut orchestrer le basculement et établir automatiquement des connexions après le basculement entre **Recovery VNet** et **Azure VNet**. C’est ce qu’illustre le diagramme suivant.
- En fonction des exigences des applications, les connexions entre les deux réseaux virtuels (**Recovery VNet** et **Azure VNet**) dans la région cible peuvent être établies avant, pendant (en tant qu’étape intermédiaire) ou après le basculement.
  - La société peut utiliser des [plans de récupération](site-recovery-create-recovery-plans.md) pour spécifier quand les connexions seront établies
  - Elle peut établir une connexion entre les réseaux virtuels à l’aide de VNET Peering ou d’un VPN de site à site
      - VNet Peering n’utilise pas de passerelle VPN et a d’autres contraintes
      - La [tarification](https://azure.microsoft.com/pricing/details/virtual-network) de VNet Peering est calculée différemment de la [tarification](https://azure.microsoft.com/pricing/details/vpn-gateway) de la passerelle VPN de réseau virtuel à réseau virtuel. Pour les basculements, nous conseillons généralement d’utiliser la même méthode de connectivité que les réseaux source, y compris le type de connexion, afin de limiter les incidents réseau imprévisibles

    ![Ressources dans Azure : basculement complet](./media/site-recovery-retain-ip-azure-vm-failover/azure-to-azure-connectivity-full-region-failover2.png)



## <a name="resources-in-azure-isolated-app-failover"></a>Ressources dans Azure : basculement d’application isolée

Vous devrez peut-être effectuer un basculement au niveau de l’application, par exemple pour effectuer le basculement d’une application ou d’une couche application spécifique situé sur un sous-réseau dédié.

- Dans ce scénario, bien que vous puissiez conserver l’adressage IP, c’est généralement déconseillé car cela augmente le risque d’incohérences de connectivité. Vous perdrez également la connectivité de sous-réseau aux autres sous-réseaux du même réseau virtuel Azure.
- Une meilleure façon d’effectuer le basculement d’application au niveau du sous-réseau consiste à utiliser différentes adresses IP cibles pour le basculement (si vous avez besoin d’une connectivité à d’autres sous-réseaux sur le réseau virtuel source) ou à isoler chaque application sur son propre réseau virtuel dédié dans la région source. Avec cette dernière approche, vous pouvez établir une connectivité entre les réseaux de la région source et émuler le même comportement quand vous basculez vers la région cible.  

Dans cet exemple, la société A place des applications dans la région source sur des réseaux virtuels dédié, et établit une connectivité entre ces réseaux virtuels. Grâce à cette conception, elle peut effectuer un basculement d’application isolée et conserver les adresses IP privées sources sur le réseau cible.

### <a name="before-failover"></a>Avant le basculement

Avant le basculement, l’architecture est la suivante :

- Les machines virtuelles d’application sont hébergées dans la région primaire Azure Asie Est :
    - Les machines virtuelles d’**App1** sont situées sur le réseau virtuel **Source VNet 1** : 10.1.0.0/16.
    - Les machines virtuelles d’**App2** sont situées sur le réseau virtuel **Source VNet 2** : 10.2.0.0/16.
    - **Source VNet 1** a deux sous-réseaux.
    - **Source VNet 2** a deux sous-réseaux.
- La région secondaire (cible) Azure est Asie Sud-Est. Asie Sud-Est a des réseaux virtuels de récupération (**Recovery VNet 1** et **Recovery VNet 2**) qui sont identiques à **Source VNet 1** et **Source VNet 2**
        - **Recovery VNet 1** et **Recovery VNet 2** ont chacun deux sous-réseaux qui correspondent aux sous-réseaux **Source VNet 1** and **Source VNet 2** - Asie Sud-Est a un réseau virtuel supplémentaire (**Azure VNet**) avec l’espace d’adressage 10.3.0.0/16
        - **Azure VNet** contient un sous-réseau (**Subnet 4**) avec l’espace d’adressage 10.3.4.0/24
        - Les nœuds de réplica pour SQL Server Always On, le contrôleur de domaine, et ainsi de suite, sont situés sur **Subnet 4**.
- Il existe plusieurs connexions VPN de site à site : 
    - **Source VNet 1** et **Azure VNet**
    - **Source VNet 2** et **Azure VNet**
    - **Source VNet 1** et **Source VNet 2** sont connectés avec un VPN de site à site
- **Recovery VNet 1** and **Recovery VNet 2** ne sont connectés à aucun autre réseau virtuel
- La **société A** configure des passerelles VPN sur **Recovery VNet 1** et **Recovery VNet 2** afin de réduire le RTO  
- **Recovery VNet1** et **Recovery VNet2** ne sont connectés à aucun autre réseau virtuel.
- Pour réduire l’objectif de délai de récupération (RTO), les passerelles VPN sont configurées sur **Recovery VNet1** et **Recovery VNet2** avant le basculement.

    ![Ressources dans Azure avant le basculement d’application](./media/site-recovery-retain-ip-azure-vm-failover/azure-to-azure-connectivity-isolated-application-before-failover2.png)

### <a name="after-failover"></a>Après le basculement

En cas de panne ou de problème qui affecte une seule application (dans **Source VNet 2 dans notre exemple), la société A peut récupérer l’application affectée comme suit :


- Désactivation des connexions VPN entre **Source VNet1** et **Source VNet2**, et entre **Source VNet2** et **Azure VNet**.
- Établissement de connexions VPN entre **Source VNet1** et **Recovery VNet2**, et entre **Recovery VNet2** et **Azure VNet**.
- Basculement des machines virtuelles sur **Source VNet2** vers **Recovery VNet2**.

![Ressources dans un basculement d’application Azure](./media/site-recovery-retain-ip-azure-vm-failover/azure-to-azure-connectivity-isolated-application-after-failover2.png)


- Cet exemple peut être étendu pour inclure d’autres applications et connexions réseau. Dans la mesure du possible, nous vous recommandons de suivre un modèle de connexion de type égal à égal pour le basculement de la source vers la cible.
- Les passerelles VPN utilisent des adresses IP publiques et des tronçons de passerelle pour établir les connexions. Si vous ne souhaitez pas utiliser d’adresses IP publiques, ou si vous préférez éviter tout tronçon supplémentaire, vous pouvez utiliser [Azure VNet Peering](../virtual-network/virtual-network-peering-overview.md) pour homologuer des réseaux virtuels dans les [régions Azure prises en charge](../virtual-network/virtual-network-manage-peering.md#cross-region).

## <a name="hybrid-resources-full-failover"></a>Ressources hybrides : basculement complet

Dans ce scénario, la **société B** dispose d’une infrastructure hybride, une partie de son infrastructure d’applications s’exécutant sur Azure et le reste localement. 

### <a name="before-failover"></a>Avant le basculement

Voici à quoi ressemble l’architecture réseau avant le basculement :

- Les machines virtuelles des applications sont hébergées dans la région Azure Asie Est.
- Asie Est a un réseau virtuel (**Source VNet**) avec l’espace d’adressage 10.1.0.0/16
  - Asie Est a des charges de travail réparties entre trois sous-réseaux sur **Source VNet** :
    - **Sous-réseau 1** : 10.1.1.0/24
    - **Sous-réseau 2** : 10.1.2.0/24
    - **Sous-réseau 3** : 10.1.3.0/24, utilisant un réseau virtuel Azure avec l’espace d’adressage 10.1.0.0/16. Ce réseau virtuel se nomme **Source VNet**
      - La région Azure secondaire (cible) est Asie Sud-Est :
  - Asie Sud-Est a un réseau virtuel de récupération (**Recovery VNet**) identique à **Source VNet**.
- Les machines virtuelles dans Asie Est sont connectées à un centre de données local avec Azure ExpressRoute ou un réseau privé virtuel de site à site.
- Pour réduire le RTO, la société B provisionne des passerelles sur Recovery VNet dans la région Azure Asie Sud-Est avant le basculement
- La société B affecte/vérifie les adresses IP cibles pour les machines virtuelles répliquées. L’adresse IP cible est identique à l’adresse IP source pour chaque machine virtuelle


![Connectivité locale vers Azure avant le basculement](./media/site-recovery-retain-ip-azure-vm-failover/on-premises-to-azure-connectivity-before-failover2.png)

### <a name="after-failover"></a>Après le basculement


Si une panne se produit dans la région source, la société B peut basculer toutes ses ressources vers la région cible.

- Avec des adresses IP cibles déjà en place avant le basculement, la société B peut orchestrer le basculement et établir automatiquement des connexions après le basculement entre **Recovery VNet** et **Azure VNet**
- En fonction des exigences des applications, les connexions entre les deux réseaux virtuels (**Recovery VNet** et **Azure VNet**) dans la région cible peuvent être établies avant, pendant (en tant qu’étape intermédiaire) ou après le basculement. La société peut utiliser des [plans de récupération](site-recovery-create-recovery-plans.md) pour spécifier quand les connexions seront établies
- La connexion d’origine entre Azure Asie Est et le centre de données local doit être déconnectée avant d’établir la connexion entre Azure Asie Sud-Est et le centre de données local.
- Le routage local est reconfiguré pour pointer vers la région cible et les passerelles après le basculement

![Connectivité locale vers Azure après le basculement](./media/site-recovery-retain-ip-azure-vm-failover/on-premises-to-azure-connectivity-after-failover2.png)

## <a name="hybrid-resources-isolated-app-failover"></a>Ressources hybrides : basculement d’application isolée

La société B ne peut pas faire basculer des applications isolées au niveau du sous-réseau, car l’espace d’adressage sur les réseaux virtuels source et de récupération est le même, et la connexion « source à local » d’origine est active.

 - Pour assurer la résilience d’application, la société B devra placer chaque application sur son propre réseau virtuel Azure dédié.
 - Avec chaque application sur un réseau virtuel distinct, la société B peut faire basculer les applications isolées et router les connexions sources vers la région cible.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [plans de récupération](site-recovery-create-recovery-plans.md)
