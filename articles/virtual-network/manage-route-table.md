---
title: Créer, modifier ou supprimer une table de routage Azure
titlesuffix: Azure Virtual Network
description: Découvrez où trouver des informations sur le routage du trafic de réseau virtuel et comment créer, modifier ou supprimer une table de route.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/19/2020
ms.author: kumud
ms.openlocfilehash: 664e3851370be812cd7a0e58cf9beb1fddb5d991
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87291000"
---
# <a name="create-change-or-delete-a-route-table"></a>Créer, modifier ou supprimer une table de routage

Azure achemine automatiquement le trafic entre les sous-réseaux, les réseaux virtuels et les réseaux locaux Azure. Si vous souhaitez modifier un routage par défaut d’Azure, vous pouvez le faire en créant une table de routage. Si vous ne maîtrisez pas encore le routage des réseaux virtuels, vous trouverez plus d’informations à ce sujet dans [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md) ou en suivant un [tutoriel](tutorial-create-route-table-portal.md).

## <a name="before-you-begin"></a>Avant de commencer

Si vous n’en avez pas, configurez un compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Accomplissez ensuite l’une des tâches suivantes avant d’exécuter les étapes décrites dans les sections de cet article :

- **Utilisateurs du portail** : Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

- **Utilisateurs de PowerShell** : exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/powershell) ou exécutez PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Dans l’onglet de navigateur Azure Cloud Shell, ouvrez la liste déroulante **Sélectionner un environnement**, puis choisissez **PowerShell** si ce n’est pas déjà fait.

    Si vous exécutez PowerShell en local, utilisez le module Azure PowerShell version 1.0.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az.Network` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Exécutez également `Connect-AzAccount` pour créer une connexion avec Azure.

- **Utilisateurs de l’interface de ligne de commande Azure (CLI)**  : Exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/bash) ou exécutez l’interface CLI à partir de votre ordinateur. Si vous exécutez l’interface Azure CLI localement, utilisez Azure CLI version 2.0.31 ou ultérieure. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli). Exécutez également `az login` pour créer une connexion avec Azure.

Le compte auquel vous vous connectez ou avec lequel vous vous connectez à Azure doit être doté du [rôle Contributeur de réseaux](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou d’un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) affecté des actions appropriées listées dans [Autorisations](#permissions).

## <a name="create-a-route-table"></a>Créer une table de routage

Le nombre de tables de routes que vous pouvez créer par abonnement et par emplacement Azure est limité. Pour plus d’informations, consultez [Limites de mise en réseau – Azure Resource Manager](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. Dans le menu du [Portail Azure](https://portal.azure.com) ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

1. Dans la zone de recherche, entrez *table de route*. Quand le terme **table de route** s’affiche dans les résultats de recherche, sélectionnez-le.

1. Dans la page **table de route**, sélectionnez **Créer**.

1. Dans la boîte de dialogue **Créer une table de routage** :

    1. Entrez un **nom** pour la table de routes.
    1. Choisissez votre **abonnement**.
    1. Choisissez un **groupe de ressources** existant ou sélectionnez **Créer nouveau** pour en créer un nouveau.
    1. Choisissez un **Emplacement**.
    1. Si vous envisagez d’associer la table de routes à un sous-réseau d’un réseau virtuel connecté à votre réseau local via une passerelle VPN et que vous ne voulez pas propager vos routes locales aux interfaces réseau dans le sous-réseau, définissez **Propagation de la route de la passerelle de réseau virtuel** sur **Désactivé**.

1. Sélectionnez **Créer** pour créer votre nouvelle table de routes.

### <a name="create-route-table---commands"></a>Créer une table de routage (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create) |
| PowerShell | [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) |

## <a name="view-route-tables"></a>Afficher les tables de routage

Accédez au [portail Azure](https://portal.azure.com) pour gérer votre réseau virtuel. Recherchez et sélectionnez **Tables de routage**. Les tables de routage présentes dans votre abonnement sont répertoriées.

### <a name="view-route-table---commands"></a>Afficher une table de routage (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table list](/cli/azure/network/route-table#az-network-route-table-list) |
| PowerShell | [Get-AzRouteTable](/powershell/module/az.network/get-azroutetable) |

## <a name="view-details-of-a-route-table"></a>Afficher les détails d’une table de routage

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre réseau virtuel. Recherchez et sélectionnez **Tables de routage**.

1. Dans la liste des tables de routes, choisissez la table de routes dont vous souhaitez afficher les détails.

1. Dans la page de la table de routes, sous **Paramètres**, visualisez les **routes** dans la table de routes ou les **sous-réseaux** auxquels la table de routes est associée.

Pour en savoir plus sur les paramètres Azure communs, consultez les informations suivantes :

- [Journal d’activité](../azure-monitor/platform/platform-logs-overview.md)
- [Contrôle d’accès (IAM)](../role-based-access-control/overview.md)
- [Balises](../azure-resource-manager/management/tag-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Verrous](../azure-resource-manager/management/lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Script Automation](../azure-resource-manager/templates/export-template-portal.md)

### <a name="view-details-of-route-table---commands"></a>Afficher les détails d’une table de routage (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table show](/cli/azure/network/route-table#az-network-route-table-show) |
| PowerShell | [Get-AzRouteTable](/powershell/module/az.network/get-azroutetable) |

## <a name="change-a-route-table"></a>Modifier une table de routage

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre réseau virtuel. Recherchez et sélectionnez **Tables de routage**.

1. Dans la liste des tables de routes, choisissez la table de routes que vous souhaitez modifier.

Les modifications les plus courantes consistent à [ajouter](#create-a-route) des routes, [supprimer](#delete-a-route) des routes, [associer](#associate-a-route-table-to-a-subnet) des tables de routes à des sous-réseaux ou [dissocier](#dissociate-a-route-table-from-a-subnet) des tables de routes des sous-réseaux.

### <a name="change-a-route-table---commands"></a>Modifier une table de routage (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table update](/cli/azure/network/route-table#az-network-route-table-update) |
| PowerShell | [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) |

## <a name="associate-a-route-table-to-a-subnet"></a>Associer une table de routage à un sous-réseau

Vous pouvez éventuellement associer une table de routes à un sous-réseau. Une table de routes peut être associée à aucun, à un ou à plusieurs sous-réseaux. Comme les tables de routes ne sont pas associées aux réseaux virtuels, vous devez associer une table de routes à chaque sous-réseau auquel vous souhaitez l’associer. Azure route tout le trafic sortant du sous-réseau en fonction des routes que vous avez créées dans les tables de routes, les [routes par défaut](virtual-networks-udr-overview.md#default) et les routes propagées à partir d’un réseau local, si le réseau virtuel est connecté à une passerelle de réseau virtuel Azure (ExpressRoute ou VPN). Vous pouvez uniquement associer une table de routage à des sous-réseaux de réseaux virtuels qui se trouvent au même emplacement et dans le même abonnement Azure que la table de routage.

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre réseau virtuel. Recherchez et sélectionnez **Réseaux virtuels**.

1. Dans la liste des réseaux virtuels, choisissez le réseau virtuel qui contient le sous-réseau auquel vous souhaitez associer une table de routes.

1. Dans la barre de menus du réseau virtuel, choisissez **Sous-réseaux**.

1. Sélectionnez le sous-réseau auquel associer la table de routage.

1. Dans **Table de routage**, choisissez la table de routes que vous voulez associer au sous-réseau.

1. Sélectionnez **Enregistrer**.

Si votre réseau virtuel est connecté à une passerelle VPN Azure, n’associez pas de table de routes au [sous-réseau de passerelle](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub) qui inclut une route avec la destination *0.0.0.0/0*. Cela peut empêcher la passerelle de fonctionner correctement. Pour plus d’informations sur l’utilisation de *0.0.0.0/0* dans une route, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md#default-route).

### <a name="associate-a-route-table---commands"></a>Associer une table de routage (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) |
| PowerShell | [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) |

## <a name="dissociate-a-route-table-from-a-subnet"></a>Dissocier une table de routage d’un sous-réseau

Quand vous dissociez une table de routage d’un sous-réseau, Azure achemine le trafic en fonction de ses [itinéraires par défaut](virtual-networks-udr-overview.md#default).

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre réseau virtuel. Recherchez et sélectionnez **Réseaux virtuels**.

1. Dans la liste des réseaux virtuels, choisissez le réseau virtuel qui contient le sous-réseau duquel vous souhaitez dissocier une table de routes.

1. Dans la barre de menus du réseau virtuel, choisissez **Sous-réseaux**.

1. Sélectionnez le sous-réseau duquel dissocier la table de routage.

1. Dans **Table de routage**, choisissez **Aucun**.

1. Sélectionnez **Enregistrer**.

### <a name="dissociate-a-route-table---commands"></a>Dissocier une table de routage (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) |
| PowerShell | [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) |

## <a name="delete-a-route-table"></a>Supprimer une table de routage

Vous ne pouvez pas supprimer une table de routes qui est associée à des sous-réseaux. [Dissociez](#dissociate-a-route-table-from-a-subnet) la table de routage de tous les sous-réseaux avant de tenter de la supprimer.

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos tables de routes. Recherchez et sélectionnez **Tables de routage**.

1. Dans la liste des tables de routes, choisissez la table de routes que vous souhaitez supprimer.

1. Sélectionnez **Supprimer**, puis **Oui** dans la boîte de dialogue de confirmation.

### <a name="delete-a-route-table---commands"></a>Supprimer une table de routage (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table delete](/cli/azure/network/route-table#az-network-route-table-delete) |
| PowerShell | [Remove-AzRouteTable](/powershell/module/az.network/remove-azroutetable) |

## <a name="create-a-route"></a>Créer un itinéraire

Le nombre de routes par table de routes que vous pouvez créer par abonnement et par emplacement Azure est limité. Pour plus d’informations, consultez [Limites de mise en réseau – Azure Resource Manager](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos tables de routes. Recherchez et sélectionnez **Tables de routage**.

1. Dans la liste des tables de routes, choisissez la table de routes à laquelle vous souhaitez ajouter une route.

1. Dans la barre de menus de la table de routes, choisissez **Routes** > **Ajouter**.

1. Entrez un **nom** unique pour la route dans la table de routes.

1. Entrez le **préfixe d’adresse** dans la notation CIDR (routage interdomaine sans classe) vers lequel vous souhaitez router le trafic. Ce préfixe ne peut pas être dupliqué dans plusieurs routes de la table de routes, bien qu’il puisse se trouver dans un autre préfixe. Par exemple, si vous avez défini *10.0.0.0/16* comme préfixe pour une route, vous pouvez toujours définir une autre route avec le préfixe d’adresse *10.0.0.0/22*. Azure sélectionne un itinéraire pour le trafic en fonction de la correspondance de préfixe la plus longue. Pour en apprendre plus, consultez [Comment Azure choisit un itinéraire](virtual-networks-udr-overview.md#how-azure-selects-a-route).

1. Choisissez un **type de tronçon suivant**. Pour en savoir plus sur les types de tronçon suivants, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md).

1. Si vous avez choisi pour **Type de tronçon suivant** une **Appliance virtuelle**, entrez une adresse IP pour **Adresse du tronçon suivant**.

1. Sélectionnez **OK**.

### <a name="create-a-route---commands"></a>Créer un itinéraire (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create) |
| PowerShell | [New-AzRouteConfig](/powershell/module/az.network/new-azrouteconfig) |

## <a name="view-routes"></a>Afficher des itinéraires

Une table de routage peut contenir plusieurs itinéraires ou aucun. Pour en savoir plus sur les informations listées lors de l’affichage des routes, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md).

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos tables de routes. Recherchez et sélectionnez **Tables de routage**.

1. Dans la liste des tables de routes, choisissez la table de routes dont vous souhaitez afficher les routes.

1. Dans la barre de menus de la table de routes, choisissez **Routes** pour afficher la liste des routes.

### <a name="view-routes---commands"></a>Afficher les itinéraires (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table route list](/cli/azure/network/route-table/route#az-network-route-table-route-list) |
| PowerShell | [Get-AzRouteConfig](/powershell/module/az.network/get-azrouteconfig) |

## <a name="view-details-of-a-route"></a>Afficher les détails d’un itinéraire

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos tables de routes. Recherchez et sélectionnez **Tables de routage**.

1. Dans la liste des tables de routes, choisissez la table de routes contenant la route dont vous souhaitez afficher les détails.

1. Dans la barre de menus de la table de routes, choisissez **Routes** pour afficher la liste des routes.

1. Sélectionnez l’itinéraire dont vous souhaitez afficher les détails.

### <a name="view-details-of-a-route---commands"></a>Afficher les détails d’un itinéraire (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table route show](/cli/azure/network/route-table/route#az-network-route-table-route-show) |
| PowerShell | [Get-AzRouteConfig](/powershell/module/az.network/get-azrouteconfig) |

## <a name="change-a-route"></a>Modifier un itinéraire

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos tables de routes. Recherchez et sélectionnez **Tables de routage**.

1. Dans la liste des tables de routes, choisissez la table de routes contenant la route que vous souhaitez modifier.

1. Dans la barre de menus de la table de routes, choisissez **Routes** pour afficher la liste des routes.

1. Choisissez la route à modifier.

1. Modifiez les paramètres souhaités, puis cliquez sur **Enregistrer**.

### <a name="change-a-route---commands"></a>Modifier un itinéraire (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table route update](/cli/azure/network/route-table/route#az-network-route-table-route-update) |
| PowerShell | [Set-AzRouteConfig](/powershell/module/az.network/set-azrouteconfig) |

## <a name="delete-a-route"></a>Supprimer un itinéraire

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos tables de routes. Recherchez et sélectionnez **Tables de routage**.

1. Dans la liste des tables de routes, choisissez la table de routes contenant la route que vous souhaitez supprimer.

1. Dans la barre de menus de la table de routes, choisissez **Routes** pour afficher la liste des routes.

1. Choisissez la route à supprimer.

1. Sélectionnez **Supprimer**, puis **Oui** dans la boîte de dialogue de confirmation.

### <a name="delete-a-route---commands"></a>Supprimer un itinéraire (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network route-table route delete](/cli/azure/network/route-table/route#az-network-route-table-route-delete) |
| PowerShell | [Remove-AzRouteConfig](/powershell/module/az.network/remove-azrouteconfig) |

## <a name="view-effective-routes"></a>Afficher les itinéraires effectifs

Les routes effectives de chaque interface réseau attachée à une machine virtuelle sont une combinaison des tables de routes que vous avez créées, des routes par défaut d’Azure et de toutes les routes propagées à partir de réseaux locaux via le protocole BGP (Border Gateway Protocol) et via une passerelle de réseau virtuel Azure. Il est important de comprendre ce que représentent les itinéraires effectifs pour une interface réseau pour pouvoir résoudre les éventuels problèmes de routage. Vous pouvez afficher les routes effectives pour toute interface réseau attachée à une machine virtuelle en cours d’exécution.

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos machines virtuelles. Recherchez et sélectionnez **Machines virtuelles**.

1. Dans la liste des machines virtuelles, choisissez la machine virtuelle pour laquelle vous souhaitez afficher les routes effectives.

1. Dans la barre de menus de la machine virtuelle, choisissez **Mise en réseau**.

1. Sélectionnez le nom d’une interface réseau.

1. Dans la barre de menus de l’interface réseau, sélectionnez **Routages effectifs**.

1. Passez en revue la liste des routes effectives pour déterminer si la route correcte existe pour l’emplacement vers lequel vous souhaitez router le trafic. Pour en savoir plus sur les types de tronçon suivants qui apparaissent dans cette liste, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md).

### <a name="view-effective-routes---commands"></a>Afficher les itinéraires effectifs (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nic show-effective-route-table](/cli/azure/network/nic#az-network-nic-show-effective-route-table) |
| PowerShell | [Get-AzEffectiveRouteTable](/powershell/module/az.network/get-azeffectiveroutetable) |

## <a name="validate-routing-between-two-endpoints"></a>Valider le routage entre deux points de terminaison

Vous pouvez déterminer le type de tronçon suivant entre une machine virtuelle et l’adresse IP d’une autre ressource Azure, une ressource locale ou une ressource sur Internet. La détermination du routage d’Azure est utile pour pouvoir résoudre les éventuels problèmes de routage. Pour exécuter cette tâche, vous devez disposer d’un observateur réseau existant. Si vous n’avez pas d’observateur réseau existant, créez-en un en procédant comme indiqué dans [Créer une instance de Network Watcher](../network-watcher/network-watcher-create.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos observateurs réseau. Recherchez et sélectionnez **Network Watcher**.

1. Dans la barre de menus de Network Watcher, choisissez **Tronçon suivant**.

1. Dans la page **Network Watcher | Tronçon suivant** :

    1. Choisissez votre **abonnement** et le **groupe de ressources** de la machine virtuelle source à partir de laquelle vous souhaitez valider le routage.

    1. Choisissez la **machine virtuelle** et l’**interface réseau** attachée à la machine virtuelle.
    
    1. Entrez une **adresse IP source** affectée à l’interface réseau à partir de laquelle vous souhaitez valider le routage.

    1. Entrez une **adresse IP de destination** vers laquelle vous souhaitez valider le routage.

1. Sélectionnez **Tronçon suivant**.

Après un court délai d’attente, Azure vous indique le type de tronçon suivant et l’ID de la route qui a routé le trafic. Pour en savoir plus sur les types de tronçon suivants retournés, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md).

### <a name="validate-routing-between-two-endpoints---commands"></a>Valider le routage entre deux points de terminaison (commandes)

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network watcher show-next-hop](/cli/azure/network/watcher#az-network-watcher-show-next-hop) |
| PowerShell | [Get-AzNetworkWatcherNextHop](/powershell/module/az.network/get-aznetworkwatchernexthop) |

## <a name="permissions"></a>Autorisations

Pour que vous puissiez exécuter des tâches sur des tables de routes et des routes, votre compte doit être doté du [rôle Contributeur de réseaux](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou d’un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) affecté des actions appropriées listées dans le tableau suivant :

| Action                                                          |   Nom                                                  |
|--------------------------------------------------------------   |   -------------------------------------------           |
| Microsoft.Network/routeTables/read                              |   Lire une table de routage                                    |
| Microsoft.Network/routeTables/write                             |   Créer ou mettre à jour une table de routage                        |
| Microsoft.Network/routeTables/delete                            |   Supprimer une table de routage                                  |
| Microsoft.Network/routeTables/join/action                       |   Associer une table de routage à un sous-réseau                   |
| Microsoft.Network/routeTables/routes/read                       |   Lire un itinéraire                                          |
| Microsoft.Network/routeTables/routes/write                      |   Créer ou mettre à jour un itinéraire                              |
| Microsoft.Network/routeTables/routes/delete                     |   Supprimer un itinéraire                                        |
| Microsoft.Network/networkInterfaces/effectiveRouteTable/action  |   Obtenir la table de routage effective d’une interface réseau |
| Microsoft.Network/networkWatchers/nextHop/action                |   Obtenir le tronçon suivant à partir d’une machine virtuelle                           |

## <a name="next-steps"></a>Étapes suivantes

- Créer une table de routes avec les exemples de scripts [PowerShell](powershell-samples.md) ou [Azure CLI](cli-samples.md), ou les [modèles Azure Resource Manager](template-samples.md)
- Créer et attribuer des [définitions Azure Policy](policy-samples.md) pour les réseaux virtuels
