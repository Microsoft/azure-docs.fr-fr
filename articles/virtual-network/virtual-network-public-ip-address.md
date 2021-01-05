---
title: Gérer les adresses IP publiques | Microsoft Docs
titleSuffix: Azure Virtual Network
description: Gérer les adresses IP publiques.  Découvrez aussi en quoi une adresse IP publique est une ressource disposant de ses propres paramètres configurables.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
editor: ''
tags: azure-resource-manager
ms.assetid: bb71abaf-b2d9-4147-b607-38067a10caf6
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/06/2019
ms.author: kumud
ms.openlocfilehash: a8847768cf4cf1ea85df8646a8848aac02cbd621
ms.sourcegitcommit: 3e8058f0c075f8ce34a6da8db92ae006cc64151a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92629253"
---
# <a name="manage-public-ip-addresses"></a>Gérer des adresses IP publiques

Découvrez les adresses IP publiques et apprenez à en créer, modifier et supprimer une. Une adresse IP publique est une ressource disposant de ses propres paramètres configurables. L’attribution d’une adresse IP publique à une ressource Azure qui prend en charge les adresses IP publiques permet :
- Communication entrante d’Internet vers la ressource, par exemple les machines virtuelles Azure (VM), les passerelles Azure Application Gateway, les équilibreurs de charge Azure Load Balancer, les passerelles VPN Azure, etc. Vous pouvez toujours communiquer avec des ressources telles qu’une machine virtuelle, à partir d’Internet si aucune adresse IP publique ne lui est attribuée à condition que la machine virtuelle fasse partie d’un pool principal à équilibreur de charge et qu’une adresse IP publique soit attribuée à l’équilibreur de charge. Pour déterminer si une ressource d’un service Azure spécifique peut se voir attribuer une adresse IP publique ou si des communications peuvent être établies avec elle via l’adresse IP publique d’une autre ressource Azure, consultez la documentation du service.
- Connectivité sortante vers Internet à l’aide d’une adresse IP prédictible. Par exemple, une machine virtuelle peut communiquer en sortie vers Internet sans adresse IP publique attribuée, mais son adresse fait l’objet d’une traduction d’adresse réseau par Azure en adresse publique non prédictible par défaut. L’attribution d’une adresse IP publique à une ressource vous permet de savoir quelle adresse IP est utilisée pour la connexion sortante. Bien que prédictible, l’adresse peut changer en fonction de la méthode d’attribution choisie. Pour plus d’informations, consultez [Créer une adresse IP publique](#create-a-public-ip-address). Pour en savoir plus sur les connexions sortantes des ressources Azure, consultez [Comprendre les connexions sortantes dans Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Avant de suivre les étapes décrites dans les sections de cet article, accomplissez les tâches suivantes :

- Si vous n’avez pas encore de compte, inscrivez-vous pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/free).
- Si vous utilisez le portail, ouvrez https://portal.azure.com, puis connectez-vous avec votre compte Azure.
- Si vous utilisez des commandes PowerShell pour accomplir les tâches décrites dans cet article, exécutez-les dans l’[Azure Cloud Shell](https://shell.azure.com/powershell), ou en exécutant PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Ce didacticiel requiert le module Azure PowerShell version 1.0.0 ou version ultérieure. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.
- Si vous utilisez des commandes de l’interface de ligne de commande (CLI) Azure pour accomplir les tâches décrites dans cet article, exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/bash) ou en exécutant Azure CLI sur votre ordinateur. Ce tutoriel exige la version 2.0.31 ou une version ultérieure d’Azure CLI. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli). Si vous exécutez Azure CLI localement, vous devez également exécuter `az login` pour créer une connexion avec Azure.

Le compte auquel vous vous connectez, ou avec lequel vous vous connectez à Azure, doit avoir le rôle [contributeur réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) disposant des autorisations appropriées, listées dans [Autorisations](#permissions).

Le coût des adresses IP publiques est modique. Pour voir les prix, consultez la page [Tarification des adresses IP](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Pour obtenir des instructions sur la création d’adresses IP publiques à l’aide du portail, de PowerShell ou de l’interface CLI, consultez les pages suivantes :

 * [Créer des adresses IP publiques - portail](https://docs.microsoft.com/azure/virtual-network/create-public-ip-portal?tabs=option-create-public-ip-standard-zones)
 * [Créer des adresses IP publiques - PowerShell](https://docs.microsoft.com/azure/virtual-network/create-public-ip-powershell?tabs=option-create-public-ip-standard-zones)
 * [Créer des adresses IP publiques - Azure CLI](https://docs.microsoft.com/azure/virtual-network/create-public-ip-cli?tabs=option-create-public-ip-standard-zones)

>[!NOTE]
>Bien que le portail permette de créer deux ressources d’adresse IP publique (IPv4 et IPv6), les commandes PowerShell et CLI créent une ressource avec une adresse pour une version IP ou l’autre. Si vous souhaitez deux ressources d’adresse IP publique, une pour chaque version d’adresse IP, vous devez exécuter la commande à deux reprises, en spécifiant des versions d’adresse IP et des noms différents pour les ressources d’adresse IP publique.

Pour plus d’informations sur les attributs spécifiques d’une adresse IP publique lors de la création, consultez le tableau ci-dessous.

   |Paramètre|Requis ?|Détails|
   |---|---|---|
   |Version de l’adresse IP|Oui| Sélectionnez IPv4 ou IPv6, ou Les deux. Si vous sélectionnez Les deux, 2 adresses IP publiques sont créées : 1 adresse IPv4 et 1 adresse IPv6. Apprenez-en davantage sur [IPv6 dans les réseaux virtuels Azure](../virtual-network/ipv6-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).|
   |SKU|Oui|Toutes les adresses IP publiques créées avant l’introduction des références SKU sont des adresses IP publiques **De base**. Une fois l’adresse IP publique créée, vous ne pouvez pas modifier la référence SKU. Les machines virtuelles autonomes, les machines virtuelles comprises dans un groupe à haute disponibilité et les groupes de machines virtuelles identiques peuvent utiliser les références SKU De Base ou Standard. La combinaison de références SKU entre machines virtuelles dans des groupes à haute disponibilité, des groupes identiques ou des machines virtuelles autonomes n’est pas autorisée. Référence SKU **De base** : si vous créez une adresse IP publique dans une région qui prend en charge les zones de disponibilité, le paramètre **Zone de disponibilité** est défini sur *Aucun* par défaut. Les adresses IP publiques de base ne prennent pas en charge les zones de disponibilité. Référence **Standard**  : Une adresse IP publique de référence Standard peut être associée à une machine virtuelle ou à un équilibreur de charge front-end. Si vous créez une adresse IP publique dans une région qui prend en charge les zones de disponibilité, le paramètre **Zone de disponibilité** est défini sur *Redondant interzone* par défaut. Pour plus d’informations sur les zones de disponibilité, consultez **Zone de disponibilité**. La référence SKU Standard est nécessaire si vous associez l’adresse à un équilibreur de charge Standard. Pour plus d’informations sur les équilibreurs de charge Standard, consultez [Référence SKU d’Azure Load Balancer Standard](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Quand vous assignez une adresse IP publique de référence SKU Standard à l’interface réseau d’une machine virtuelle, vous devez explicitement autoriser le trafic prévu avec un [groupe de sécurité réseau](security-overview.md#network-security-groups). La communication avec la ressource est possible uniquement si vous créez et associez un groupe de sécurité réseau et que vous autorisez explicitement le trafic prévu.|
   |Nom|Oui|Le nom doit être unique au sein du groupe de ressources que vous avez sélectionné.|
   |Affectation d’adresses IP|Oui|**Dynamique :** les adresses dynamiques sont attribuées uniquement une fois qu’une adresse IP publique est associée à une ressource Azure et que la ressource est démarrée pour la première fois. Les adresses dynamiques peuvent changer si elles sont attribuées à une ressource, comme une machine virtuelle, et que la machine virtuelle est arrêtée (désallouée), puis redémarrée. L’adresse reste la même si une machine virtuelle est redémarrée ou arrêtée (mais pas désallouée). Les adresses dynamiques sont libérées quand une ressource d’adresse IP publique est dissociée d’une ressource à laquelle elle est associée. **Statique :** les adresses statiques sont attribuées durant la création d’une adresse IP publique. Les adresses statiques ne sont pas libérées tant qu’une ressource d’adresse IP publique n’est pas supprimée. Si l’adresse n’est pas associée à une ressource, vous pouvez changer la méthode d’attribution après la création de l’adresse. Si l’adresse est associée à une ressource, il est possible que vous ne puissiez pas changer la méthode d’attribution. Si vous sélectionnez *IPv6* comme **Version de l’adresse IP** , la méthode d’affectation doit être *Dynamique* pour la référence SKU de base.  Les adresses de référence SKU standard sont *statiques* pour IPv4 et IPv6. |
   |Délai d’inactivité (minutes)|Non|Durée (en minutes) de maintien d’une connexion TCP ou HTTP ouverte sans utiliser les clients pour envoyer des messages keep-alive. Si vous sélectionnez IPv6 pour **Version IP** , cette valeur ne peut pas être modifiée. |
   |Étiquette du nom DNS|Non|Doit être unique dans l’emplacement Azure où vous créez le nom (pour tous les abonnements et tous les clients). Azure inscrit automatiquement le nom et l’adresse IP dans son DNS pour que vous puissiez vous connecter à une ressource avec le nom. Azure ajoute un sous-réseau par défaut comme *emplacement.cloudapp.azure.com* (où emplacement est l’emplacement que vous sélectionnez) au nom que vous indiquez pour créer le nom DNS complet. Si vous choisissez de créer les deux versions d’adresse, le même nom DNS est assigné aux adresses IPv4 et IPv6. Le DNS par défaut d’Azure contient à la fois les enregistrements de nom AAAA IPv4 A et IPv6, et il renvoie les deux enregistrements lors de la recherche du nom DNS. Le client choisit l’adresse (IPv4 ou IPv6) avec laquelle communiquer. À la place ou en plus de l’étiquette de nom DNS avec le suffixe par défaut, vous pouvez utiliser le service Azure DNS pour configurer un nom DNS avec un suffixe personnalisé qui se résout en adresse IP publique. Pour plus d’informations, consultez [Utiliser Azure DNS avec une adresse IP publique Azure](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address).|
   |Nom (visible uniquement si vous sélectionnez la version d’adresse IP **Les deux** )|Oui, si vous sélectionnez la version d’adresse IP **Les deux**|Le nom doit être différent de celui que vous entrez pour le premier **Nom** dans cette liste. Si vous choisissez de créer une adresse IPv4 et une adresse IPv6, le portail crée deux ressources d’adresse IP publique distinctes : une pour chaque version d’adresse IP qui lui est assignée.|
   |Affectation d’adresses IP (visible uniquement si vous sélectionnez la version d’adresse IP **Les deux** )|Oui, si vous sélectionnez la version d’adresse IP **Les deux**|Mêmes restrictions que pour Affectation d’adresses IP ci-dessus|
   |Abonnement|Oui|Doit exister dans le même [abonnement](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription) que la ressource à laquelle vous allez associer l’adresse IP publique.|
   |Resource group|Oui|Peut exister dans un [groupe de ressources](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group) identique ou différent de celui de la ressource à laquelle vous allez associer l’adresse IP publique.|
   |Emplacement|Oui|Doit exister au même [emplacement](https://azure.microsoft.com/regions) (également appelé région) que la ressource à laquelle vous allez associer l’adresse IP publique.|
   |Zone de disponibilité| Non | Ce paramètre s’affiche uniquement si vous sélectionnez un emplacement pris en charge. Pour obtenir la liste des emplacements pris en charge, consultez [Vue d’ensemble des zones de disponibilité](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si vous avez sélectionné la référence SKU **De base** , l’option *Aucun* est automatiquement sélectionnée. Si vous préférez garantir une zone spécifique, vous pouvez en sélectionner une. Ces deux options sont sans redondance interzone. Si vous avez sélectionné la référence **Standard**  : l’option Redondant interzone est automatiquement sélectionnée et permet la résilience de votre chemin de données en cas de défaillance de la zone. Si vous préférez garantir une zone spécifique qui ne soit pas résiliente en cas de défaillance, vous pouvez en sélectionner une.

## <a name="view-modify-settings-for-or-delete-a-public-ip-address"></a>Afficher une adresse IP publique, modifier ses paramètres ou la supprimer

   - **Afficher / Liste**  : Pour passer en revue les paramètres d’une adresse IP publique, y compris la référence SKU, l’adresse, toute association applicable (par exemple, carte réseau de machine virtuelle, front-end de l’équilibreur de charge).
   - **Modify**  : Pour modifier les paramètres à l’aide des informations de l’étape 4 de [Créer une adresse IP publique](#create-a-public-ip-address), par exemple, le délai d’inactivité, l’étiquette de nom DNS ou la méthode d’affectation.  (Pour prendre connaissance du processus complet de mise à niveau d’une référence SKU d’adresse IP publique de De base à Standard, consultez [Mettre à niveau des adresses IP publiques Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address-upgrade).)
   >[!WARNING]
   >Pour modifier l’affectation d’une adresse IP publique et passer d’une adresse statique à une adresse dynamique, vous devez commencer par dissocier l’adresse de toutes les configurations IP applicables (voir la section **Supprimer** ).  Remarquez également que lorsque vous modifiez la méthode d’affectation pour passer d’une adresse statique à une adresse dynamique, vous perdez l’adresse IP qui a été affectée à l’adresse IP publique. Si les serveurs DNS publics Azure conservent un mappage entre les adresses statiques ou dynamiques et une étiquette de nom DNS (si vous en avez défini une), une adresse IP dynamique peut être modifiée au démarrage de la machine virtuelle après qu’elle a été arrêtée (libérée). Pour empêcher toute modification de l’adresse, affectez une adresse IP statique.
   
|Opération|Portail Azure|Azure PowerShell|Azure CLI|
|---|---|---|---|
|Vue | Dans la section **Vue d’ensemble** d’une adresse IP publique |[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) pour récupérer un objet d’adresse IP publique et afficher ses paramètres| [az network public-IP show](/cli/azure/network/public-ip#az-network-public-ip-show) pour afficher les paramètres|
|List | Sous la catégorie **Adresses IP publiques** |[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) pour récupérer un ou plusieurs objets d’adresse IP publique et afficher ses paramètres|[az network public-ip list](/cli/azure/network/public-ip#az-network-public-ip-list) pour répertorier les adresses IP publiques|
|Modifier | Pour une adresse IP dissociée, sélectionnez **configuration** pour modifier le délai d’inactivité, l’étiquette de nom DNS ou l’affectation de l’adresse IP de base de statique à dynamique  |[Set-AzPublicIpAddress](/powershell/module/az.network/set-azpublicipaddress) pour mettre à jour les paramètres |[az network public-ip update](/cli/azure/network/public-ip#az-network-public-ip-update) pour mettre à jour |

   - **Supprimer**  : La suppression des adresses IP publiques exige que l’objet de l’adresse IP publique ne soit pas associé à une configuration IP ou à une carte réseau de machine virtuelle. Pour plus d’informations, consultez le tableau ci-dessous.

|Ressource|Portail Azure|Azure PowerShell|Azure CLI|
|---|---|---|---|
|[Machine virtuelle](https://docs.microsoft.com/azure/virtual-network/remove-public-ip-address-vm)|Sélectionnez **Dissocier** pour dissocier l’adresse IP de la configuration de la carte réseau, puis sélectionnez **Supprimer**.|[Set-AzPublicIpAddress](/powershell/module/az.network/set-azpublicipaddress) pour dissocier l’adresse IP de la configuration de la carte réseau ; [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress) pour supprimer|[az network public-ip update --remove](/cli/azure/network/public-ip#az-network-public-ip-update) pour dissocier l’adresse IP de la configuration de la carte réseau ; [az network public-ip delete](/cli/azure/network/public-ip#az-network-public-ip-delete) pour supprimer |
|Front-end de l'équilibreur de charge | Accédez à une adresse IP publique inutilisée et sélectionnez **Associer** , puis sélectionnez l’équilibreur de charge avec la configuration IP frontale appropriée pour la remplacer (l’ancienne adresse IP peut alors être supprimée à l’aide de la même méthode que pour la machine virtuelle)  | [Set-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/set-azloadbalancerfrontendipconfig) pour associer une nouvelle configuration IP frontale à un équilibreur de charge public ; [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress) pour supprimer ; peut également utiliser [Remove-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/remove-azloadbalancerfrontendipconfig) pour supprimer la configuration de l’adresse IP frontale s’il en existe plusieurs |[az network lb frontend-ip update](/cli/azure/network/lb/frontend-ip?view=azure-cli-latest#az_network_lb_frontend_ip_update) pour associer une nouvelle configuration IP frontale à un équilibreur de charge public ; [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress) pour supprimer ; peut également utiliser [az network lb frontend-ip delete](/cli/azure/network/lb/frontend-ip?view=azure-cli-latest#az_network_lb_frontend_ip_delete) pour supprimer la configuration de l’adresse IP frontale s’il en existe plusieurs|
|Pare-feu|N/A| [Libérer()](https://docs.microsoft.com/azure/firewall/firewall-faq#how-can-i-stop-and-start-azure-firewall) pour libérer le pare-feu et supprimer toutes les configurations IP | [az network firewall ip-config delete](/cli/azure/ext/azure-firewall/network/firewall/ip-config#ext_azure_firewall_az_network_firewall_ip_config_delete) pour supprimer l’adresse IP (mais doit d’abord utiliser PowerShell pour la libérer)|

## <a name="virtual-machine-scale-sets"></a>Virtual Machine Scale Sets

Lorsque vous utilisez un groupe de machines virtuelles identiques avec des adresses IP publiques, il n’y a pas d’objets IP publics distincts associés à chaque instance de machine virtuelle. Toutefois, un objet de préfixe d’adresse IP publique [peut être utilisé pour générer les adresses IP d’instance](https://azure.microsoft.com/resources/templates/101-vmms-with-public-ip-prefix/).

Pour répertorier les adresses IP publiques sur un groupe de machines virtuelles identiques, vous pouvez utiliser PowerShell ([Get-AzPublicIpAddress -VirtualMachineScaleSetName](/powershell/module/az.network/get-azpublicipaddress)) ou l’interface CLI ([az vmss list-instance-public-ips](/cli/azure/vmss?view=azure-cli-latest#az_vmss_list_instance_public_ips)).

Pour plus d’informations, consultez [Fonctionnalités réseau pour les groupes de machines virtuelles identiques Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-networking#public-ipv4-per-virtual-machine).

## <a name="assign-a-public-ip-address"></a>Assigner une adresse IP publique

Découvrez comment attribuer une adresse IP publique aux ressources suivantes :

- À une machine virtuelle [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (au moment de la création) ou à une machine virtuelle [existante](virtual-network-network-interface-addresses.md#add-ip-addresses)
- [Équilibreur de charge public](../load-balancer/load-balancer-get-started-internet-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Application Gateway](../application-gateway/application-gateway-create-gateway-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Connexion site à site à l’aide d’une passerelle VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Groupe de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-portal-create.md?toc=%2fazure%2fvirtual-network%2ftoc.json)

## <a name="permissions"></a>Autorisations

Pour effectuer des tâches sur des adresses IP publiques, votre compte doit avoir le rôle de [contributeur de réseaux](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou avoir un rôle [personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) assigné aux actions appropriées répertoriées dans le tableau suivant :

| Action                                                             | Nom                                                           |
| ---------                                                          | -------------                                                  |
| Microsoft.Network/publicIPAddresses/read                           | Lire une adresse IP publique                                          |
| Microsoft.Network/publicIPAddresses/write                          | Créer ou mettre à jour une adresse IP publique                           |
| Microsoft.Network/publicIPAddresses/delete                         | Supprimer une adresse IP publique                                     |
| Microsoft.Network/publicIPAddresses/join/action                    | Associer une adresse IP publique à une ressource                    |

## <a name="next-steps"></a>Étapes suivantes

- Créez une adresse IP publique avec les exemples de scripts [PowerShell](powershell-samples.md) ou [Azure CLI](cli-samples.md), ou à l’aide des [modèles Azure Resource Manager](template-samples.md)
- Créer et attribuer des [définitions Azure Policy](policy-samples.md) pour les adresses IP publiques
