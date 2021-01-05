---
title: Configurer des adresses IP pour une interface réseau Azure | Microsoft Docs
description: Découvrez comment ajouter, modifier et supprimer des adresses IP publiques et privées pour une interface réseau.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/22/2020
ms.author: allensu
ms.openlocfilehash: 265ed0f4cb58a321bde78714f36123bf197d42f6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84710998"
---
# <a name="add-change-or-remove-ip-addresses-for-an-azure-network-interface"></a>Ajouter, modifier ou supprimer des adresses IP pour une interface réseau Azure

Découvrez comment ajouter, modifier et supprimer des adresses IP publiques et privées pour une interface réseau. Les adresses IP privées assignées à une interface réseau permettent à une machine virtuelle de communiquer avec d’autres ressources dans un réseau virtuel Azure et des réseaux connectés. Une adresse IP privée permet également une communication sortante vers Internet à l’aide d’une adresse IP non prédictible. Une [adresse IP publique](virtual-network-public-ip-address.md) assignée à une interface réseau permet d’établir une communication entrante vers une machine virtuelle à partir d’Internet. L’adresse permet également d’établir une communication sortante de la machine virtuelle vers Internet à l’aide d’une adresse IP prédictible. Pour en savoir plus, consultez [Comprendre les connexions sortantes dans Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Si vous avez besoin de créer, modifier ou supprimer une interface réseau, consultez l’article [Ajouter, modifier ou supprimer des adresses IP pour des cartes réseau Azure](virtual-network-network-interface.md). Si vous devez ajouter ou supprimer des interfaces réseau sur une machine virtuelle, lisez l’article [Ajouter ou supprimer des interfaces réseau](virtual-network-network-interface-vm.md).

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Avant de suivre les étapes décrites dans les sections de cet article, accomplissez les tâches suivantes :

- Si vous n’avez pas encore de compte, inscrivez-vous pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/free).
- Si vous utilisez le portail, ouvrez https://portal.azure.com, puis connectez-vous avec votre compte Azure.
- Si vous utilisez des commandes PowerShell pour accomplir les tâches décrites dans cet article, exécutez-les dans l’[Azure Cloud Shell](https://shell.azure.com/powershell), ou en exécutant PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Ce didacticiel requiert le module Azure PowerShell version 1.0.0 ou version ultérieure. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.
- Si vous utilisez des commandes de l’interface de ligne de commande (CLI) Azure pour accomplir les tâches décrites dans cet article, exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/bash) ou en exécutant Azure CLI sur votre ordinateur. Ce tutoriel exige la version 2.0.31 ou une version ultérieure d’Azure CLI. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli). Si vous exécutez Azure CLI localement, vous devez également exécuter `az login` pour créer une connexion avec Azure.

Le compte auquel vous vous connectez, ou avec lequel vous vous connectez à Azure, doit avoir le rôle [contributeur réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) disposant des autorisations appropriées, listées dans [Autorisations relatives à l’interface réseau](virtual-network-network-interface.md#permissions).

## <a name="add-ip-addresses"></a>Ajouter des adresses IP

Vous pouvez ajouter autant d’adresses [privées](#private) et [publiques](#public) [IPv4](#ipv4) que nécessaire à une interface réseau, dans les limites listées dans l’article [Limites d’Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits). Vous pouvez ajouter une adresse IPv6 privée à une [configuration IP secondaire](#secondary) (à condition qu’il n’existe pas déjà de configuration IP secondaire) pour une interface réseau existante. Chaque interface réseau peut avoir une adresse IPv6 privée maximum. Vous pouvez si vous le souhaitez ajouter une adresse IPv6 publique à une configuration d’interface réseau IPv6. Consultez [IPv6](#ipv6) pour plus d’informations sur l’utilisation des adresses IPv6.

1. Dans la zone qui contient le texte *Rechercher des ressources* en haut du portail Azure, saisissez *interfaces réseau*. Lorsque la mention **interfaces réseau** apparaît dans les résultats de recherche, sélectionnez-la.
2. Sélectionnez dans la liste l’interface réseau pour laquelle vous souhaitez ajouter une adresse IPv4.
3. Sous **PARAMÈTRES**, sélectionnez **Configurations IP**.
4. Sous **Configurations IP**, sélectionnez **+ Ajouter**.
5. Spécifiez les éléments suivants, puis sélectionnez **OK** :

   |Paramètre|Requis ?|Détails|
   |---|---|---|
   |Nom|Oui|Doit être unique pour l’interface réseau|
   |Type|Oui|Étant donné que vous ajoutez une configuration IP à une interface réseau existante et que chaque interface réseau doit disposer d’une configuration IP [principale](#primary), la seule option possible est **Secondaire**.|
   |Méthode d’affectation d’adresses IP privées|Oui|[**Dynamique**](#dynamic) : Azure attribue la prochaine adresse disponible pour la plage d’adresses de sous-réseau dans laquelle l’interface réseau est déployée. [**Statique**](#static) : vous attribuez une adresse inutilisée pour la plage d’adresses de sous-réseau dans laquelle l’interface réseau est déployée.|
   |Adresse IP publique|Non|**Désactivé :** aucune ressource d’adresse IP publique n’est associée à la configuration IP. **Activée :** sélectionnez une adresse IP publique IPv4 existante, ou créez-en une. Pour savoir comment créer une adresse IP publique, consultez l’article [Adresses IP publiques](virtual-network-public-ip-address.md#create-a-public-ip-address).|
6. Ajoutez manuellement des adresses IP privées secondaires au système d’exploitation de la machine virtuelle en suivant les instructions de l’article [Ajouter des adresses IP à un système d’exploitation de machine virtuelle](virtual-network-multiple-ip-addresses-portal.md#os-config). Consultez les adresses IP [privées](#private) pour connaître les considérations spécifiques avant d’ajouter manuellement des adresses IP à un système d’exploitation de machine virtuelle. N’ajoutez pas d’adresse IP publique au système d’exploitation de la machine virtuelle.

**Commandes**

|Outil|Commande|
|---|---|
|Interface de ligne de commande|[az network nic ip-config create](/cli/azure/network/nic/ip-config)|
|PowerShell|[Add-AzNetworkInterfaceIpConfig](/powershell/module/az.network/add-aznetworkinterfaceipconfig)|

## <a name="change-ip-address-settings"></a>Modifier les paramètres d’adresse IP

Vous pouvez modifier la méthode d’affectation d’une adresse IPv4, modifier l’adresse IPv4 statique ou modifier l’adresse IP publique assignées à une interface réseau. Si vous modifiez l’adresse IPv4 privée d’une configuration IP secondaire associée à une interface réseau secondaire dans une machine virtuelle (en savoir plus sur les [interfaces réseau principales et secondaires](virtual-network-network-interface-vm.md)), mettez la machine virtuelle dans l’état Arrêté (libéré) avant d’effectuer les opérations suivantes :

1. Dans la zone qui contient le texte *Rechercher des ressources* en haut du portail Azure, saisissez *interfaces réseau*. Lorsque la mention **interfaces réseau** apparaît dans les résultats de recherche, sélectionnez-la.
2. Sélectionnez dans la liste l’interface réseau pour laquelle vous souhaitez consulter ou modifiez les paramètres d’adresse IP.
3. Sous **PARAMÈTRES**, sélectionnez **Configurations IP**.
4. Sélectionnez dans la liste la configuration IP que vous souhaitez modifier.
5. Modifiez les paramètres comme vous le souhaitez en utilisant les informations fournies à ce sujet à l’étape 5 de la section [Ajouter une configuration IP](#add-ip-addresses).
6. Sélectionnez **Enregistrer**.

>[!NOTE]
>Si l’interface réseau principale dispose de plusieurs configurations IP et si vous modifiez l’adresse IP privée de la configuration IP principale, vous devez réassigner manuellement les adresses IP principales et secondaires à l’interface réseau dans Windows (non requis pour Linux). Pour attribuer manuellement des adresses IP à une interface réseau au sein d’un système d’exploitation, voir [Attribuer plusieurs adresses IP à des machines virtuelles](virtual-network-multiple-ip-addresses-portal.md#os-config). Pour connaître les considérations spécifiques avant d’ajouter manuellement des adresses IP à un système d’exploitation de machine virtuelle, voir les adresses IP [privées](#private). N’ajoutez pas d’adresse IP publique au système d’exploitation de la machine virtuelle.

**Commandes**

|Outil|Commande|
|---|---|
|Interface de ligne de commande|[az network nic ip-config update](/cli/azure/network/nic/ip-config)|
|PowerShell|[Set-AzNetworkInterfaceIpConfig](/powershell/module/az.network/set-aznetworkinterfaceipconfig)|

## <a name="remove-ip-addresses"></a>Supprimer des adresses IP

Vous pouvez supprimer des adresses IP [privées](#private) et [publiques](#public) à partir d’une interface réseau, mais une interface réseau doit toujours avoir au moins une adresse IPv4 privée.

1. Dans la zone qui contient le texte *Rechercher des ressources* en haut du portail Azure, saisissez *interfaces réseau*. Lorsque la mention **interfaces réseau** apparaît dans les résultats de recherche, sélectionnez-la.
2. Sélectionnez l’interface réseau pour laquelle vous souhaitez supprimer des adresses IP dans la liste.
3. Sous **PARAMÈTRES**, sélectionnez **Configurations IP**.
4. Sélectionnez avec le bouton droit une configuration IP [secondaire](#secondary) (vous ne pouvez pas supprimer la configuration [principale](#primary)) que vous souhaitez supprimer. Sélectionnez **Supprimer**, puis **Oui** pour confirmer la suppression. Si une ressource d’adresse IP publique est associée à la configuration, la ressource est dissociée de la configuration IP, mais la ressource n’est pas supprimée.

**Commandes**

|Outil|Commande|
|---|---|
|Interface de ligne de commande|[az network nic ip-config delete](/cli/azure/network/nic/ip-config)|
|PowerShell|[Remove-AzNetworkInterfaceIpConfig](/powershell/module/az.network/remove-aznetworkinterfaceipconfig)|

## <a name="ip-configurations"></a>Configurations IP

Les adresses IP [privées](#private) et (éventuellement) [publiques](#public) sont assignées à une ou plusieurs configurations IP assignées à une interface réseau. Il existe deux types de configuration IP :

### <a name="primary"></a>Principal

Une configuration IP principale est assignée à chaque interface réseau. Une configuration IP principale :

- Dispose d’une adresse [privée](#private)[IPv4](#ipv4) qui lui est affectée. Vous ne pouvez pas assigner d’adresse [IPv6](#ipv6) privée à une configuration IP principale.
- Une adresse IPv4 [publique](#public) peut également lui être assignée. On ne peut pas assigner une adresse IPv6 publique à une configuration IP (IPv4) principale. 

### <a name="secondary"></a>Secondary

En plus d’une configuration IP principale, une interface réseau peut avoir zéro, une ou plusieurs configurations IP secondaires qui lui sont assignées. Une configuration IP secondaire :

- Doit avoir une adresse IPv4 ou IPv6 privée qui lui est assignée. Si l’adresse est de type IPv6, l’interface réseau peut uniquement avoir une seule configuration IP secondaire. Si l’adresse est de type IPv4, plusieurs configurations IP secondaires peuvent être assignées à l’interface réseau. Pour plus d’informations sur le nombre d’adresses IPv4 privées et publiques qui peuvent être assignées à une interface réseau, consultez l’article [Limites d’Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).
- Une adresse IPv4 ou IPv6 publique peut également lui être assignée. Il est utile d’assigner plusieurs adresses IPv4 à une interface réseau dans différents scénarios :
  - Hébergement de plusieurs sites web ou services avec des adresses IP différentes et des certificats TLS/SSL sur un serveur unique.
  - Une machine virtuelle joue le rôle d’appliance virtuelle réseau, comme un pare-feu ou un équilibreur de charge.
  - Possibilité d’ajouter l’une des adresses IPv4 privées pour toutes les interfaces réseau à un pool principal Azure Load Balancer. Auparavant, seule l’adresse IPv4 principale de l’interface réseau principale pouvait être ajoutée à un pool principal. Pour plus d’informations sur la façon d’équilibrer la charge entre plusieurs configurations IPv4, consultez l’article [Équilibrage de la charge entre plusieurs configurations IP](../load-balancer/load-balancer-multiple-ip.md?toc=%2fazure%2fvirtual-network%2ftoc.json). 
  - Possibilité d’équilibrer la charge d’une adresse IPv6 assignée à une interface réseau. Pour en savoir plus sur l’équilibrage de charge vers une adresse IPv6 privée, consultez l’article [Vue d’ensemble du protocole IPv6 pour Azure Load Balancer](../load-balancer/load-balancer-ipv6-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="address-types"></a>Types d’adresse

Vous pouvez assigner les types d’adresse IP suivants à une [IP de configuration](#ip-configurations) :

### <a name="private"></a>Privé

Les adresses [IPv4](#ipv4) et IPv6 privées permettent à une machine virtuelle de communiquer avec d’autres ressources dans un réseau virtuel ou d’autres réseaux connectés. 

Par défaut, les serveurs DHCP Azure assignent l’adresse IPv4 privée pour la [configuration IP principale](#primary) de l’interface réseau Azure à l’interface réseau située au sein du système d’exploitation de la machine virtuelle. Sauf obligation, vous ne devez jamais définir manuellement l’adresse IP d’une interface réseau au sein du système d’exploitation de la machine virtuelle.

> [!WARNING]
> Si l’adresse IPv4 définie en tant qu’adresse IP principale d’une interface réseau au sein du système d’exploitation d’une machine virtuelle diffère de l’adresse IPv4 privée assignée à la configuration IP principale de l’interface réseau principale attachée à une machine virtuelle dans Azure, la connectivité à la machine virtuelle est interrompue.

Il est parfois nécessaire de définir manuellement l’adresse IP d’une interface réseau au sein du système d’exploitation de la machine virtuelle. Par exemple, vous devez définir manuellement les adresses IP principales et secondaires d’un système d’exploitation Windows lors de l’ajout de plusieurs adresses IP à une machine virtuelle Azure. Pour une machine virtuelle Linux, vous devrez peut-être uniquement définir manuellement les adresses IP secondaires. Consultez [Ajouter des adresses IP à un système d’exploitation de machine virtuelle](virtual-network-multiple-ip-addresses-portal.md#os-config) pour en savoir plus. Si vous devez modifier l’adresse assignée à une configuration IP, il est recommandé de :

1. Assurez-vous que la machine virtuelle reçoit une adresse à partir des serveurs DHCP Azure. Ensuite, redéfinissez l’attribution de l’adresse IP sur la valeur DHCP au sein du système d’exploitation, puis redémarrez la machine virtuelle.
2. Arrêter (libérer) la machine virtuelle.
3. Modifier l’adresse IP pour la configuration IP dans Azure.
4. Démarrez la machine virtuelle.
5. [Configurer manuellement](virtual-network-multiple-ip-addresses-portal.md#os-config) les adresses IP secondaires dans le système d’exploitation (ainsi que l’adresse IP principale dans Windows) pour obtenir ce que vous avez défini dans Azure.

Si vous suivez les étapes précédentes, l’adresse IP privée assignée à l’interface réseau dans Azure reste identique à celle assignée dans le système d’exploitation de la machine virtuelle. Pour garder une trace des machines virtuelles de votre abonnement au sein desquelles vous avez défini manuellement les adresses IP dans un système d’exploitation, pensez à ajouter une [balise](../azure-resource-manager/management/tag-resources.md) Azure aux machines virtuelles. Vous pouvez utiliser « attribution d’adresses IP : statique », par exemple. De cette manière, vous pouvez facilement trouver les machines virtuelles de votre abonnement pour lesquelles vous avez défini manuellement l’adresse IP dans le système d’exploitation.

En plus de permettre à une machine virtuelle de communiquer avec d’autres ressources dans les mêmes réseaux virtuels ou des réseaux virtuels connectés, une adresse IP privée permet à une machine virtuelle de communiquer en sortie vers Internet. Les connexions sortantes correspondent à une adresse de réseau source convertie par Azure en adresse IP publique non prédictible. Pour en savoir plus sur la connectivité Internet sortante d’Azure, voir [À propos de la connectivité Internet sortante d’Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Vous ne pouvez communiquer en entrée vers l’adresse IP privée d’une machine virtuelle à partir d’Internet. Si vos connexions sortantes requièrent une adresse IP publique prédictible, associez une ressource d’adresse IP publique à une interface réseau.

### <a name="public"></a>Public

Les adresses IP publiques assignées via une ressource d’adresse IP publique permettent une connectivité entrante vers une machine virtuelle à partir d’Internet. Les connexions sortantes vers Internet utilisent une adresse IP prédictible. Pour en savoir plus, consultez [Comprendre les connexions sortantes dans Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Vous pouvez affecter une adresse IP publique à une configuration IP, mais ce n’est pas obligatoire. Si vous n’assignez pas d’adresse IP publique à une machine virtuelle par l’association d’une ressource d’adresse IP publique, cette dernière peut encore communiquer en sortie vers Internet. Dans ce cas l’adresse IP privée correspond à une adresse de réseau source convertie par Azure en adresse IP publique non prédictible. Pour en savoir plus sur les ressources d’adresses IP publiques, voir [Ressource d’adresse IP publique](virtual-network-public-ip-address.md).

Le nombre d’adresses IP publiques et privées que vous pouvez assigner à une interface réseau est limité. Pour en savoir plus, consultez l’article [Limites d’Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

> [!NOTE]
> Azure convertit l’adresse IP privée d’une machine virtuelle en adresse IP publique. Par conséquent, le système d’exploitation de la machine virtuelle n’a pas connaissance d’adresses IP publiques qui lui sont assignées, et il est inutile d’assigner manuellement une adresse IP publique au sein du système d’exploitation.

## <a name="assignment-methods"></a>Méthodes d’affectation

Les adresses IP privées et publiques sont assignées à l’aide d’une des méthodes d’affectation suivantes :

### <a name="dynamic"></a>Dynamique

Les adresses IPv4 et (éventuellement) IPv6 privées dynamiques sont assignées par défaut.

- **Publique uniquement** : Azure attribue l’adresse à partir d’une plage propre à chaque région Azure. Pour savoir quelles plages sont affectées à chaque région, consultez [Microsoft Azure Datacenter IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653) (Plages d’adresses IP du centre de données Azure Microsoft). L’adresse peut changer lorsqu’une machine virtuelle est arrêtée (désallouée), puis redémarrée. Vous ne pouvez pas assigner d’adresse IPv6 publique à une configuration IP, quelle que soit la méthode employée.
- **Privée uniquement** : Azure réserve les quatre premières adresses dans chaque plage d’adresses de sous-réseau, sans les attribuer. Azure assigne la prochaine adresse disponible à une ressource de la plage d’adresses du sous-réseau. Par exemple, si la plage d’adresses du sous-réseau est 10.0.0.0/16, et que les adresses 10.0.0.4 à 10.0.0.14 sont déjà attribuées (celles de .0 à .3 sont réservées), Azure attribue l’adresse 10.0.0.15 à la ressource. La méthode d’allocation par défaut est dynamique. Une fois assignées, les adresses IP dynamiques ne sont libérées que si l’interface réseau est supprimée, assignée à un sous-réseau différent au sein du même réseau virtuel ou bien si la méthode d’allocation devient statique et qu’une adresse IP différente est spécifiée. Par défaut, Azure définit l’adresse statique sur l’adresse dynamique attribuée précédemment quand vous sélectionnez la méthode d’allocation statique à la place de la méthode dynamique. 

### <a name="static"></a>statique

Vous pouvez (si vous le souhaitez) assigner une adresse IPv4 ou IPv6 statique publique ou privée à une configuration IP. Pour en savoir plus sur la façon dont Azure attribue les adresses IPv4 publiques statiques, consultez [Adresses IP publiques](virtual-network-public-ip-address.md).

- **Publique uniquement** : Azure attribue l’adresse à partir d’une plage propre à chaque région Azure. Vous pouvez télécharger la liste des plages (préfixes) pour les clouds Azure [Public](https://www.microsoft.com/download/details.aspx?id=56519), [Gouvernement américain](https://www.microsoft.com/download/details.aspx?id=57063), [Chine](https://www.microsoft.com/download/details.aspx?id=57062), et [Allemagne](https://www.microsoft.com/download/details.aspx?id=57064). L’adresse ne change pas tant que la ressource d’adresse IP publique à laquelle elle est assignée n’est pas supprimée, ou que la méthode d’affectation dynamique n’est pas modifiée. Si la ressource d’adresse IP publique est associée à une configuration IP, elle doit être dissociée de la configuration IP avant de modifier sa méthode d’affectation.
- **Privée uniquement** : vous sélectionnez et attribuez une adresse à partir de la plage d’adresses du sous-réseau. L’adresse que vous assignez peut correspondre à n’importe quelle adresse qui se trouve au sein de la plage d’adresses de sous-réseau n’étant pas une de ses quatre premières adresses, et qui n’est pas déjà assignée à une autre ressource du sous-réseau. Les adresses statiques ne sont libérées que si l’interface réseau est supprimée. Si vous sélectionnez la méthode d’allocation statique à la place de la méthode dynamique, Azure définit l’adresse statique sur l’adresse IP dynamique précédemment attribuée, même si celle-ci n’est pas la première adresse disponible de la plage d’adresses du sous-réseau. L’adresse change aussi si l’interface réseau est assignée à un autre sous-réseau dans le même réseau virtuel. Pour l’assigner à un autre sous-réseau, vous devez d’abord modifier la méthode d’allocation statique en dynamique. Une fois que l’interface réseau est assignée à un autre sous-réseau, vous pouvez redéfinir la méthode d’allocation en statique, et assigner une adresse IP de la nouvelle plage d’adresses du sous-réseau.

## <a name="ip-address-versions"></a>Versions d’adresse IP

Vous pouvez spécifier les versions suivantes lors de l’attribution d’adresses :

### <a name="ipv4"></a>IPv4

Chaque interface réseau doit avoir une configuration IP [principale](#primary) avec une adresse [privée](#private) [IPv4](#ipv4) qui lui est affectée. Vous pouvez ajouter une ou plusieurs configurations IP [secondaires](#secondary) disposant chacune d’une adresse IPv4 privée et (éventuellement ) d’une adresse IPv4 [publique](#public).

### <a name="ipv6"></a>IPv6

Vous pouvez assigner zéro ou une adresse [IPv6](#ipv6) privée à une configuration IP secondaire d’une interface réseau. L’interface réseau ne peut pas déjà avoir de configuration IP secondaire. Chaque interface réseau peut avoir une adresse IPv6 privée maximum. Vous pouvez si vous le souhaitez ajouter une adresse IPv6 publique à une configuration d’interface réseau IPv6.

> [!NOTE]
> Bien que vous puissiez créer une interface réseau avec une adresse IPv6 à l’aide du portail, il vous est impossible de l’utiliser pour ajouter une interface réseau existante à un ordinateur virtuel nouveau ou existant. Utilisez PowerShell ou Azure CLI pour créer une interface réseau avec une adresse IPv6 privée, puis attachez l’interface réseau pendant la création d’une machine virtuelle. Vous ne pouvez pas attacher d’interface réseau, avec une adresse IPv6 privée qui lui est assignée, à une machine virtuelle existante. Vous ne pouvez pas ajouter d’adresse IPv6 privée à la configuration IP d’une interface réseau attachée à une machine virtuelle, quel que soit l’outil utilisé (portail, CLI ou PowerShell).

Vous ne pouvez pas assigner d’adresse IPv6 publique à une configuration IP principale ou secondaire.

## <a name="skus"></a>Références (SKU)

Une adresse IP publique est créée avec la référence SKU de base ou standard. Pour plus d’informations sur les différences entre les références SKU, voir [Gérer les adresses IP publiques](virtual-network-public-ip-address.md).

> [!NOTE]
> Quand vous assignez une adresse IP publique de référence SKU Standard à l’interface réseau d’une machine virtuelle, vous devez explicitement autoriser le trafic prévu avec un [groupe de sécurité réseau](security-overview.md#network-security-groups). La communication avec la ressource est possible uniquement si vous créez et associez un groupe de sécurité réseau et que vous autorisez explicitement le trafic prévu.

## <a name="next-steps"></a>Étapes suivantes
Pour créer une machine virtuelle avec différentes configurations IP, consultez les articles suivants :

|Tâche|Outil|
|---|---|
|Créer une machine virtuelle avec plusieurs interfaces réseau|[CLI](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [PowerShell](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|
|Créer une machine virtuelle à carte réseau unique avec plusieurs adresses IPv4|[CLI](virtual-network-multiple-ip-addresses-cli.md), [PowerShell](virtual-network-multiple-ip-addresses-powershell.md)|
|Créer une machine virtuelle à carte réseau unique avec une adresse IPv6 privée (derrière Azure Load Balancer)|[CLI](../load-balancer/load-balancer-ipv6-internet-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [PowerShell](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [modèle Azure Resource Manager](../load-balancer/load-balancer-ipv6-internet-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|
