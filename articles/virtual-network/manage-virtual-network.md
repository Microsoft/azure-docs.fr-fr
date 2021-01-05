---
title: Créer, modifier ou supprimer un réseau virtuel Azure
titlesuffix: Azure Virtual Network
description: Créez et supprimez un réseau virtuel et modifiez des paramètres, comme les serveurs DNS et les espaces d’adressage IP, pour un réseau virtuel existant.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/10/2019
ms.author: kumud
ms.openlocfilehash: 5581a4c43f0b78dc8c14c44bfb1ded371a925fd0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88706028"
---
# <a name="create-change-or-delete-a-virtual-network"></a>Créer, modifier ou supprimer un réseau virtuel

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Découvrez comment créer et supprimer un réseau virtuel, ainsi que modifier des paramètres tels que les serveurs DNS et les espaces d’adressage IP, pour un réseau virtuel existant. Si vous découvrez les réseaux virtuels, vous trouverez plus d’informations à leur sujet dans l’article [Vue d’ensemble des réseaux virtuels](virtual-networks-overview.md) ou en suivant un [tutoriel](quick-create-portal.md). Un réseau virtuel contient des sous-réseaux. Pour savoir comment créer, modifier et supprimer des sous-réseaux, voir [Gérer des sous-réseaux](virtual-network-manage-subnet.md).

## <a name="before-you-begin"></a>Avant de commencer

Avant de suivre les étapes décrites dans les sections de cet article, accomplissez les tâches suivantes :

- Si vous n’avez pas encore de compte, inscrivez-vous pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/free).
- Si vous utilisez le portail, ouvrez https://portal.azure.com, puis connectez-vous avec votre compte Azure.
- Si vous utilisez des commandes PowerShell pour accomplir les tâches décrites dans cet article, exécutez-les dans l’[Azure Cloud Shell](https://shell.azure.com/powershell), ou en exécutant PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Ce didacticiel requiert le module Azure PowerShell version 1.0.0 ou version ultérieure. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.
- Si vous utilisez des commandes de l’interface de ligne de commande (CLI) Azure pour accomplir les tâches décrites dans cet article, exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/bash) ou en exécutant Azure CLI sur votre ordinateur. Ce tutoriel exige la version 2.0.31 ou une version ultérieure d’Azure CLI. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli). Si vous exécutez Azure CLI localement, vous devez également exécuter `az login` pour créer une connexion avec Azure.
- Le compte auquel vous vous connectez, ou avec lequel vous vous connectez à Azure, doit avoir le rôle [contributeur réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) disposant des autorisations appropriées, listées dans [Autorisations](#permissions).

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

1. Sélectionnez **Créer une ressource** > **Mise en réseau** > **Réseau virtuel**.
2. Entrez ou sélectionnez des valeurs pour les paramètres suivants, puis cliquez sur **Créer** :
   - **Name** : le nom doit être unique dans le [groupe de ressources](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group) dans lequel vous choisissez de créer le réseau virtuel. Vous ne pouvez pas modifier le nom une fois le réseau virtuel créé. Vous pouvez créer plusieurs réseaux virtuels au fil du temps. Pour des suggestions de dénomination, voir [Conventions d’affectation de noms](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#naming-and-tagging-resources). Le respect d’une convention d’affectation de noms peut simplifier la gestion de plusieurs réseaux virtuels.
   - **Espace d’adressage** : l’espace d’adressage d’un réseau virtuel est composé d’une ou de plusieurs plages d’adresses ne se chevauchant pas et spécifiées dans la notation CIDR. La plage d’adresses que vous définissez peut être publique ou privée (RFC 1918). Que vous définissiez la plage d’adresses comme publique ou privée, celle-ci est uniquement accessible à partir du réseau virtuel, à partir de réseaux virtuels interconnectés et à partir de tout réseau local que vous avez connecté au réseau virtuel. Vous ne pouvez pas ajouter les plages d’adresses suivantes :
     - 224.0.0.0/4 (multidiffusion)
     - 255.255.255.255/32 (diffusion)
     - 127.0.0.0/8 (bouclage)
     - 169.254.0.0/16 (lien-local)
     - 168.63.129.16/32 ([sonde d’intégrité](../load-balancer/load-balancer-custom-probe-overview.md#probesource) Azure Load Balancer, DHCP et DNS interne)

     Si vous ne pouvez définir qu’une seule plage d’adresses durant la création du réseau virtuel dans le portail, vous pouvez en ajouter d’autres à l’espace d’adressage une fois le réseau virtuel créé. Pour savoir comment ajouter une plage d’adresses à un réseau virtuel existant, consultez [Ajouter ou supprimer une plage d’adresses](#add-or-remove-an-address-range).

     > [!WARNING]
     > Si un réseau virtuel comporte des plages d’adresses qui chevauchent celles d’un autre réseau virtuel ou local, il est impossible de connecter les deux réseaux. Avant de définir une plage d’adresses, demandez-vous si vous souhaiteriez peut-être connecter le réseau virtuel à d’autres réseaux virtuels ou locaux par la suite. Microsoft recommande de configurer des plages d’adresses de réseau virtuel avec un espace d’adressage privé ou public appartenant à votre organisation.
     >

     - **Nom du sous-réseau** : Le nom du sous-réseau doit être unique au sein du réseau virtuel. Vous ne pouvez pas modifier le nom du sous-réseau une fois celui-ci créé. Lorsque vous créez un réseau virtuel, le portail exige que vous définissiez un sous-réseau, même si un réseau virtuel ne doit pas obligatoirement comprendre des sous-réseaux. Dans le portail, vous ne pouvez définir qu’un seul sous-réseau lorsque vous créez un réseau virtuel. Vous pouvez ajouter des sous-réseaux au réseau virtuel une fois celui-ci créé. Pour ajouter un sous-réseau à un réseau virtuel, consultez [Gérer des sous-réseaux](virtual-network-manage-subnet.md). Vous pouvez créer un réseau virtuel comprenant plusieurs sous-réseaux en utilisant PowerShell ou Azure CLI.

       >[!TIP]
       >Les administrateurs créent parfois des sous-réseaux distincts pour filtrer ou contrôler le routage du trafic entre ceux-ci. Avant de définir des sous-réseaux, réfléchissez à la manière dont vous pourriez vouloir filtrer et router le trafic entre ceux-ci. Pour en savoir plus sur le filtrage du trafic entre des sous-réseaux, voir [Filtrer le trafic réseau avec les groupes de sécurité réseau](security-overview.md). Azure route automatiquement le trafic entre sous-réseaux, mais vous pouvez remplacer les itinéraires par défaut d’Azure. Pour en savoir plus sur le routage du trafic de sous-réseau par défaut Azure, consultez [Vue d’ensemble du routage](virtual-networks-udr-overview.md).
       >

     - **Plage d’adresses de sous-réseau** : la plage d’adresses doit s’inscrire dans l’espace d’adressage que vous avez entré pour le réseau virtuel. La plus petite plage que vous puissiez spécifier est /29. Celle-ci fournit huit adresses IP pour le sous-réseau. Azure réserve la première et la dernière adresses dans chaque sous-réseau pour la conformité du protocole. Trois adresses supplémentaires sont réservées à l’usage du service Azure. Par conséquent, un réseau virtuel dont la plage d’adresses de sous-réseau est /29 ne comprend que trois adresses IP utilisables. Si vous envisagez de connecter un réseau virtuel à une passerelle VPN, vous devez créer un sous-réseau de passerelle. Pour en savoir plus, voir [Sous-réseau de passerelle](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub). Vous pouvez modifier la plage d’adresses après la création du sous-réseau sous certaines conditions. Pour savoir comment modifier une plage d’adresses de sous-réseau, consultez [Gérer des sous-réseaux](virtual-network-manage-subnet.md).
     - **Abonnement**: sélectionnez un [abonnement](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription). Vous ne pouvez pas utiliser un même réseau virtuel dans plus d’un abonnement Azure. En revanche, vous pouvez connecter un réseau virtuel figurant dans un abonnement à des réseaux virtuels figurant dans d’autres abonnements via un [peering de réseau virtuel](virtual-network-peering-overview.md). Toute ressource Azure que vous connectez au réseau virtuel doit figurer dans le même abonnement que le réseau virtuel.
     - **Groupe de ressources** : sélectionnez un [groupe de ressources](../azure-resource-manager/management/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-groups) existant ou créez-en un. Une ressource Azure que vous connectez au réseau virtuel peut figurer dans le même groupe de ressources que le réseau virtuel ou dans un autre groupe de ressources.
     - **Emplacement** : sélectionnez un [emplacement](https://azure.microsoft.com/regions/) Azure (également appelé région). Un réseau virtuel peut figurer que dans un seul emplacement d’Azure. En revanche, vous pouvez connecter un réseau virtuel figurant dans un emplacement à un réseau virtuel figurant dans un autre emplacement en utilisant une passerelle VPN. Toute ressource Azure que vous connectez au réseau virtuel doit figurer dans le même emplacement que le réseau virtuel.

**Commandes**

- Azure CLI : [az network vnet create](/cli/azure/network/vnet)
- PowerShell : [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork)

## <a name="view-virtual-networks-and-settings"></a>Afficher des réseaux virtuels et des paramètres

1. Dans la zone de recherche située en haut du portail, entrez *réseaux virtuels*. Quand la mention **Réseaux virtuels** apparaît dans les résultats de recherche, sélectionnez-la.
2. Dans la liste des réseaux virtuels, sélectionnez le réseau dont vous souhaitez afficher les paramètres.
3. Les paramètres répertoriés pour le réseau virtuel que vous avez sélectionné sont les suivants :
   - **Vue d’ensemble** : fournit des informations sur le réseau virtuel, dont l’espace d’adressage et les serveurs DNS. La capture d’écran suivante présente les paramètres de vue d’ensemble pour un réseau virtuel nommé **MyVNet** :

     ![Vue d’ensemble de l’interface réseau](./media/manage-virtual-network/vnet-overview.png)

     Vous pouvez déplacer un réseau virtuel vers un autre abonnement ou un autre groupe de ressources en sélectionnant l’option **Modifier** en regard de **Groupe de ressources** ou **Nom de l’abonnement**. Pour savoir comment déplacer un réseau virtuel, voir [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Cet article répertorie les conditions préalables et explique comment déplacer des ressources à l’aide du portail Azure, de PowerShell ou d’Azure CLI. Toutes les ressources connectées au réseau virtuel doivent être déplacées avec celui-ci.
   - **Espace d’adressage** : les espaces d’adressage affectés au réseau virtuel sont listés. Pour savoir comment ajouter et supprimer une plage d’adresses, procédez comme indiqué dans la section [Ajouter ou supprimer une plage d’adresses](#add-or-remove-an-address-range).
   - **Appareils connectés** : toutes les ressources connectées au réseau virtuel sont listées. Dans la capture d’écran précédente, trois interfaces réseau et un équilibreur de charge sont connectés au réseau virtuel. Toutes les ressources que vous créez et connectez au réseau virtuel sont répertoriées. Si vous supprimez une ressource connectée au réseau virtuel, elle n’apparaît plus dans la liste.
   - **Sous-réseaux** : la liste des sous-réseaux présents dans le réseau virtuel est affichée. Pour savoir comment ajouter et supprimer un sous-réseau, consultez [Gérer des sous-réseaux](virtual-network-manage-subnet.md).
   - **Serveurs DNS** : vous pouvez spécifier le serveur chargé d’assurer la résolution de noms pour les appareils connectés au réseau virtuel : le serveur DNS interne Azure ou le serveur DNS personnalisé. Lorsque vous créez un réseau virtuel via le portail Azure, par défaut, les serveurs DNS d’Azure sont utilisés pour la résolution de noms au sein du réseau virtuel. Pour modifier les serveurs DNS, procédez comme indiqué dans la section [Modifier les serveurs DNS](#change-dns-servers) de cet article.
   - **Peerings** : s’il existe des peerings dans l’abonnement, ceux-ci sont listés ici. Vous pouvez afficher les paramètres des peerings existants, ou créer, modifier ou supprimer des peerings. Pour en savoir plus sur les peerings, voir [Peering de réseaux virtuels](virtual-network-peering-overview.md).
   - **Propriétés** : affiche les paramètres du réseau virtuel, dont l’ID de ressource et l’abonnement Azure dans lequel il figure.
   - **Diagramme** : le diagramme fournit une représentation visuelle de tous les appareils connectés au réseau virtuel. Il comporte des informations clés sur les appareils. Pour gérer un appareil affiché dans le diagramme, sélectionnez-le.
   - **Paramètres Azure courants** : Pour en savoir plus sur les paramètres Azure communs, consultez les informations suivantes :
     - [Journal d’activité](../azure-monitor/platform/platform-logs-overview.md)
     - [Contrôle d’accès (IAM)](../role-based-access-control/overview.md)
     - [Balises](../azure-resource-manager/management/tag-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
     - [Verrous](../azure-resource-manager/management/lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
     - [Script Automation](../azure-resource-manager/management/manage-resource-groups-portal.md#export-resource-groups-to-templates)

**Commandes**

- Azure CLI : [az network vnet show](/cli/azure/network/vnet)
- PowerShell : [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork)

## <a name="add-or-remove-an-address-range"></a>Ajouter ou supprimer une plage d’adresses

Vous pouvez ajouter et supprimer des plages d’adresses pour un réseau virtuel. Une plage d’adresses doit être spécifiée dans une notation CIDR. Elle ne peut pas chevaucher d’autres plages d’adresses au sein du même réseau virtuel. Les plages d’adresses que vous définissez peuvent être publiques ou privées (RFC 1918). Que vous définissiez la plage d’adresses comme publique ou privée, celle-ci est uniquement accessible à partir du réseau virtuel, à partir de réseaux virtuels interconnectés et à partir de tout réseau local que vous avez connecté au réseau virtuel. 

Vous pouvez réduire la plage d’adresses d’un réseau virtuel tant qu’il comprend toujours les plages de tous les sous-réseaux associés. En outre, vous pouvez étendre la plage d’adresses, par exemple en remplaçant /16 par /8. 

<!-- the above statement has been edited to reflect the most recent comments on the reopened issue: https://github.com/MicrosoftDocs/azure-docs/issues/20572 -->

Vous ne pouvez pas ajouter les plages d’adresses suivantes :

- 224.0.0.0/4 (multidiffusion)
- 255.255.255.255/32 (diffusion)
- 127.0.0.0/8 (bouclage)
- 169.254.0.0/16 (lien-local)
- 168.63.129.16/32 ([sonde d’intégrité](../load-balancer/load-balancer-custom-probe-overview.md#probesource) Azure Load Balancer, DHCP et DNS interne)

Pour ajouter ou supprimer une plage d’adresses :

1. Dans la zone de recherche située en haut du portail, entrez *réseaux virtuels*. Quand la mention **Réseaux virtuels** apparaît dans les résultats de recherche, sélectionnez-la.
2. Dans la liste des réseaux virtuels, sélectionnez le réseau pour lequel vous souhaitez ajouter ou supprimer une plage d’adresses.
3. Sous **PARAMÈTRES**, sélectionnez **Espace d'adressage**.
4. Choisissez l'une des options suivantes :
    - **Ajouter une plage d’adresses** : entrez la nouvelle plage d’adresses. La plage d’adresses ne peut pas chevaucher une plage d’adresses existante définie pour le réseau virtuel.
    - **Supprimer une plage d’adresses** : à droite de la plage d’adresses à supprimer, sélectionnez **...** , puis **Supprimer**. S’il existe déjà un sous-réseau dans la plage d’adresses, vous ne pouvez pas supprimer la plage. Avant de supprimer une plage d’adresses, vous devez supprimer tous les sous-réseaux (et toutes les ressources des sous-réseaux) existant dans celle-ci.
5. Sélectionnez **Enregistrer**.

**Commandes**

- Azure CLI : [az network vnet update](/cli/azure/network/vnet)
- PowerShell : [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork)

## <a name="change-dns-servers"></a>Modifier les serveurs DNS

Toutes les machines virtuelles connectées au réseau virtuel s’inscrivent auprès des serveurs DNS que vous spécifiez pour le réseau virtuel. Elles utilisent également le serveur DNS spécifié pour la résolution de noms. Chaque carte réseau (NIC) sur un ordinateur virtuel peut avoir ses propres paramètres de serveur DNS. Si une carte réseau a ses propres paramètres de serveur DNS, ceux-ci remplacent les paramètres du serveur DNS pour le réseau virtuel. Pour en savoir plus sur les paramètres DNS de carte réseau, voir [Interfaces réseau (cartes d’interface réseau)](virtual-network-network-interface.md#change-dns-servers). Pour en savoir plus sur la résolution de noms pour des machines virtuelles et des instances de rôle dans Azure Cloud Services, voir [Résolution de noms pour les machines virtuelles et les instances de rôle](virtual-networks-name-resolution-for-vms-and-role-instances.md). Pour ajouter, modifier ou supprimer un serveur DNS :

1. Dans la zone de recherche située en haut du portail, entrez *réseaux virtuels*. Quand la mention **Réseaux virtuels** apparaît dans les résultats de recherche, sélectionnez-la.
2. Dans la liste des réseaux virtuels, sélectionnez le réseau dont vous souhaitez modifier les serveurs DNS.
3. Sous**PARAMÈTRES**, sélectionnez **Serveurs DNS**.
4. Sélectionnez l’une des options suivantes :
   - **Par défaut (fournie par Azure)**  : les noms de ressources et adresses IP privées sont tous inscrits automatiquement sur les serveurs Azure DNS. Vous pouvez résoudre les noms parmi toutes les ressources connectées à un même réseau virtuel. Vous ne pouvez pas utiliser cette option pour résoudre les noms dans plusieurs réseaux virtuels. Pour résoudre les noms dans plusieurs réseaux virtuels, vous devez utiliser un serveur DNS personnalisé.
   - **Personnalisé** : vous pouvez ajouter un ou plusieurs serveurs, jusqu’à la limite autorisée par Azure pour un réseau virtuel. Pour en savoir plus sur les limites du serveur DNS, voir [Limites d’Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#virtual-networking-limits-classic). Vous disposez des options suivantes :
   - **Ajouter une adresse** : ajoute le serveur à la liste des serveurs DNS de votre réseau virtuel. Cette option inscrit également le serveur DNS auprès d’Azure. Si vous avez déjà inscrit un serveur DNS auprès d’Azure, vous pouvez sélectionner ce serveur dans la liste.
   - **Supprimer une adresse** : sélectionnez **...** en regard du serveur à supprimer, puis **Supprimer**. La suppression du serveur ne supprime que le serveur de cette liste de réseaux virtuels. Le serveur DNS reste enregistré dans Azure pour être utilisé par vos autres réseaux virtuels.
   - **Réorganiser les adresses des serveurs DNS** : il est important de vérifier que les serveurs DNS sont listés dans l’ordre approprié pour votre environnement. Ils sont utilisés dans l’ordre où ils sont spécifiés. Ils ne fonctionnent pas comme un tourniquet (round robin). Si le premier serveur DNS de la liste est accessible, le client utilise celui-ci, qu’il fonctionne correctement ou non. Supprimez tous les serveurs DNS répertoriés, puis rajoutez-les dans l’ordre souhaité.
   - **Changer une adresse** : mettez en surbrillance le serveur DNS dans la liste, puis entrez la nouvelle adresse.
5. Sélectionnez **Enregistrer**.
6. Redémarrez les machines virtuelles connectées au réseau virtuel, afin que les nouveaux paramètres de serveur DNS leur soient assignés. Les machines virtuelles continuent d’utiliser leurs paramètres DNS actifs jusqu’à ce qu’elles soient redémarrés.

**Commandes**

- Azure CLI : [az network vnet update](/cli/azure/network/vnet)
- PowerShell : [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork)

## <a name="delete-a-virtual-network"></a>Supprimer un réseau virtuel

Vous pouvez supprimer un réseau virtuel uniquement si aucune ressource n’est connectée à celui-ci. Si des ressources sont connectées à un sous-réseau du réseau virtuel, vous devez commencer par supprimer les ressources connectées à tous les sous-réseaux du réseau virtuel. La procédure à suivre pour supprimer une ressource varient en fonction de celle-ci. Pour savoir comment supprimer des ressources connectées à des sous-réseaux, lisez la documentation relative à chaque type de ressource que vous souhaitez supprimer. Pour supprimer un réseau virtuel :

1. Dans la zone de recherche située en haut du portail, entrez *réseaux virtuels*. Quand la mention **Réseaux virtuels** apparaît dans les résultats de recherche, sélectionnez-la.
2. Dans la liste des réseaux virtuels, sélectionnez le réseau à supprimer.
3. Vérifiez qu’aucun appareil n’est connecté au réseau virtuel en sélectionnant **Appareils connectés** sous **PARAMÈTRES**. Si des appareils sont connectés, vous devez les supprimer avant de supprimer le réseau virtuel. Si aucun appareil n’est connecté, sélectionnez **Vue d’ensemble**.
4. Sélectionnez **Supprimer**.
5. Pour confirmer la suppression du réseau virtuel, cliquez sur **Oui**.

**Commandes**

- Azure CLI : [azure network vnet delete](/cli/azure/network/vnet)
- PowerShell : [Remove-AzVirtualNetwork](/powershell/module/az.network/remove-azvirtualnetwork)

## <a name="permissions"></a>Autorisations

Pour effectuer des tâches sur des réseaux virtuels, votre compte doit posséder le rôle de [contributeur de réseaux](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou un rôle [personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) disposant des actions appropriées qui sont répertoriées dans le tableau suivant :

| Action                                  |   Nom                                |
|---------------------------------------- |   --------------------------------    |
|Microsoft.Network/virtualNetworks/read   |   Lire un réseau virtuel              |
|Microsoft.Network/virtualNetworks/write  |   Création ou mise à jour d’un réseau virtuel  |
|Microsoft.Network/virtualNetworks/delete |   Supprimer un réseau virtuel            |

## <a name="next-steps"></a>Étapes suivantes

- Créez un réseau virtuel avec les exemples de scripts [PowerShell](powershell-samples.md) ou [Azure CLI](cli-samples.md), ou à l’aide des[modèles Azure Resource Manager](template-samples.md)
- Créez et attribuez des [définitions Azure Policy](policy-samples.md) pour les réseaux virtuels
