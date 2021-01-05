---
title: Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle | Microsoft Docs
description: Découvrez comment diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle en consultant les règles de sécurité effectives pour une machine virtuelle.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: a54feccf-0123-4e49-a743-eb8d0bdd1ebc
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2018
ms.author: kumud
ms.openlocfilehash: ec70f7820994898cb8e552dab66547cc9c9f10ec
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93042875"
---
# <a name="diagnose-a-virtual-machine-network-traffic-filter-problem"></a>Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle

Dans cet article, vous allez apprendre à diagnostiquer un problème de filtre de trafic réseau en consultant les règles de sécurité du groupe de sécurité réseau (NSG) qui entrent en vigueur pour une machine virtuelle.

Les groupes de sécurité réseau vous permettent de contrôler les types de trafic transitant sur une machine virtuelle. Vous pouvez associer un groupe de sécurité réseau à un sous-réseau dans un réseau virtuel Azure, une interface réseau attachée à une machine virtuelle, ou les deux. Les règles de sécurité effectives appliquées à une interface réseau sont une agrégation des règles qui existent dans un NSG associé à l’interface réseau, et le sous-réseau sur lequel celle-ci se trouve. Les règles dans différents groupe de sécurité réseau sont parfois en conflit entre elles et peuvent avoir une incidence sur la connectivité réseau d’une machine virtuelle. Vous pouvez afficher toutes les règles de sécurité effectives de vos groupes de sécurité réseau, telles qu’appliquées aux interfaces réseau de votre machine virtuelle. Si vous n’êtes pas familiarisé avec les concepts de groupe de sécurité réseau, d’interface réseau ou de réseau virtuel, consultez [Vue d’ensemble du réseau virtuel](virtual-networks-overview.md), [Interface réseau](virtual-network-network-interface.md), et [Vue d’ensemble des groupes de sécurité réseau](security-overview.md).

## <a name="scenario"></a>Scénario

Vous essayez de vous connecter à une machine virtuelle via le port 80 à partir d’internet, mais la connexion échoue. Pour déterminer la raison pour laquelle le port 80 n’est pas accessible à partir d’internet, vous pouvez afficher les règles de sécurité effectives pour une interface réseau à l’aide du [portail](#diagnose-using-azure-portal) Azure, de [PowerShell](#diagnose-using-powershell), ou de [Azure CLI](#diagnose-using-azure-cli).

Les étapes qui suivent supposent que vous avez une machine virtuelle existante dont les règles de sécurité efficace peuvent être affichées. Si vous ne possédez pas une telle machine, commencez par déployer une machine virtuelle [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour pouvoir accomplir les tâches de cet article. Les exemples contenus dans cet article sont prévus pour une machine virtuelle nommée *myVM* , et une interface réseau appelée *myVMVMNic*. La machine virtuelle et l’interface réseau se trouvent dans un groupe de ressources nommé *myResourceGroup* , et se situent dans la région *USA Est*. Modifiez les valeurs dans les étapes, selon le cas, pour la machine virtuelle dont vous analysez le problème.

## <a name="diagnose-using-azure-portal"></a>Diagnostiquer à l’aide du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte Azure disposant des [autorisations nécessaires](virtual-network-network-interface.md#permissions).
2. En haut du portail Azure, entrez le nom de la machine virtuelle dans la zone de recherche. Quand le nom de cette machine virtuelle apparaît dans les résultats de la recherche, sélectionnez-le.
3. Sous **PARAMÈTRES** , sélectionnez **Mise en réseau** , comme indiqué dans l’image suivante :

   ![Capture d’écran représentant les paramètres de mise en réseau de ma carte réseau V M V M dans le Portail Azure.](./media/diagnose-network-traffic-filter-problem/view-security-rules.png)

   Les règles répertoriées dans l’image précédente concernent une interface réseau nommée **myVMVMNic**. Vous voyez qu’il existe des **RÈGLES DE PORT ENTRANT** pour l’interface réseau en provenance de deux groupes de sécurité réseau différents :
   
   - **mySubnetNSG**  : associé au sous-réseau dans lequel se trouve l’interface réseau.
   - **myVMNSG**  : associé à l’interface réseau de la machine virtuelle nommée **myVMVMNic**.

   La règle nommée **DenyAllInBound** est celle qui empêche les communications entrantes vers la machine virtuelle via le port 80 à partir d’internet, comme décrit dans le [scénario](#scenario). La règle liste *0.0.0.0/0* pour **SOURCE** , ce qui inclut l’internet. Aucune autre règle avec une priorité plus élevée (numéro inférieur) ne permet de trafic entrant par le port 80. Pour autoriser le trafic entrant par le port 80 vers la machine virtuelle à partir d’internet, consultez [Résoudre un problème](#resolve-a-problem). Pour plus d’informations sur les règles de sécurité et de quelle façon Azure les applique, consultez [Groupes de sécurité réseau](security-overview.md).

   Au bas de l’image, vous voyez également les **RÈGLES DE PORT SORTANT**. Les règles de port sortant pour l’interface réseau se trouvent en-dessous. Bien que l’image montre uniquement quatre règles de trafic entrant pour chaque groupe de sécurité réseau, vos groupes de sécurité réseau peuvent avoir beaucoup plus de quatre règles. Dans l’image, vous voyez **VirtualNetwork** sous **SOURCE** et **DESTINATION** et **AzureLoadBalancer** sous  **SOURCE**. **VirtualNetwork** et **AzureLoadBalancer** sont des [balises de service](security-overview.md#service-tags). Les balises de service représentent un groupe de préfixes d’adresses IP qui permet de simplifier la création de règles de sécurité.

4. Assurez-vous que la machine virtuelle est en cours d’exécution, et sélectionnez **Règles de sécurité effectives** , comme illustré dans l’image précédente, pour voir les règles de sécurité effectives illustrées dans l’image suivante :

   ![Capture d’écran représentant le volet Règles de sécurité effectives avec l’option Télécharger sélectionnée et la règle de trafic entrant AllowAzureLoadBalancerInbound sélectionnée.](./media/diagnose-network-traffic-filter-problem/view-effective-security-rules.png)

   Les règles répertoriées sont identiques à celles de l’étape 3, bien qu’il existe différents onglets pour le groupe de sécurité réseau associé à l’interface réseau et au sous-réseau. Comme vous pouvez le voir dans l’image, seules les 50 premières règles sont affichées. Pour télécharger un fichier CSV contenant toutes les règles, sélectionnez **Télécharger**.

   Pour voir les préfixes représentés par chaque balise de service, sélectionnez une règle, telle que la règle nommée **AllowAzureLoadBalancerInbound**. L’illustration suivante montre les préfixes pour la balise de service **AzureLoadBalancer** :

   ![Capture d’écran affichant les préfixes d’adresse pour AllowAzureLoadBalancerInbound saisi.](./media/diagnose-network-traffic-filter-problem/address-prefixes.png)

   Bien que la balise de service **AzureLoadBalancer** représente uniquement un préfixe, d’autres balises de service en représentent plusieurs.

5. Les étapes précédentes ont montré les règles de sécurité pour une interface réseau nommée **myVMVMNic** , mais vous avez également vu une interface réseau nommée **myVMVMNic2** dans certaines des images précédentes. La machine virtuelle dans cet exemple a deux interfaces réseau attachées. Les règles de sécurité effectives peuvent être différentes pour chaque interface réseau.

   Pour voir les règles pour l’interface réseau **myVMVMNic2** , sélectionnez-le. Comme indiqué dans l’illustration qui suit, l’interface réseau a les mêmes règles associées à son sous-réseau que l’interface réseau **myVMVMNic** , car les deux interfaces réseau sont dans le même sous-réseau. Lorsque vous associez un groupe de sécurité réseau à un sous-réseau, ses règles sont appliquées à toutes les interfaces réseau dans le sous-réseau.

   ![Capture d’écran représentant les paramètres de mise en réseau de ma carte réseau 2 V M V M dans le Portail Azure.](./media/diagnose-network-traffic-filter-problem/view-security-rules2.png)

   Contrairement à l’interface réseau **myVMVMNic** , l’interface réseau **myVMVMNic2** ne dispose pas d’un groupe de sécurité réseau associé. Chaque interface réseau et sous-réseau peuvent avoir aucun ou un groupe de sécurité réseau associé. Le groupe de sécurité réseau associé à chaque interface réseau ou sous-réseau peut être le même ou ils peuvent être différents. Vous pouvez associer le même groupe de sécurité réseau à toutes les interfaces réseau individuelles et à tous les sous-réseaux que vous souhaitez.

Bien que les règles de sécurité effectives aient été affichées au moyen de la machine virtuelle, vous pouvez aussi les consulter avec les éléments individuels suivants :
- **Interface réseau**  : découvrez comment [afficher une interface réseau](virtual-network-network-interface.md#view-network-interface-settings).
- **Groupe de sécurité réseau**  : découvrez comment [afficher un groupe de sécurité réseau](manage-network-security-group.md#view-details-of-a-network-security-group).

## <a name="diagnose-using-powershell"></a>Diagnostiquer à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez exécuter les commandes qui suivent dans [Azure Cloud Shell](https://shell.azure.com/powershell), ou en exécutant PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif gratuit. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Si vous exécutez PowerShell sur votre ordinateur, vous devez utiliser le module Azure PowerShell version 1.0.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` sur votre ordinateur pour trouver la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell localement, vous devez aussi exécuter `Connect-AzAccount` pour vous connecter à Azure avec un compte disposant des [autorisations nécessaires](virtual-network-network-interface.md#permissions).

Obtenez les règles de sécurité effectives pour une interface réseau avec [Get-AzEffectiveNetworkSecurityGroup](/powershell/module/az.network/get-azeffectivenetworksecuritygroup). L’exemple suivant obtient les règles de sécurité effectifs d’une interface réseau nommée *myVMVMNic* , qui se trouve dans un groupe de ressources appelé *myResourceGroup* :

```azurepowershell-interactive
Get-AzEffectiveNetworkSecurityGroup `
  -NetworkInterfaceName myVMVMNic `
  -ResourceGroupName myResourceGroup
```

La sortie est retournée au format json. Pour comprendre la sortie, consultez [Interpréter la sortie de commande](#interpret-command-output).
La sortie est renvoyée uniquement si un groupe de sécurité réseau est associé à l’interface réseau, le sous-réseau auquel l’interface réseau appartient, ou les deux. La machine virtuelle doit être en cours d’exécution. Une machine virtuelle peut avoir plusieurs interfaces réseau à laquelle différents groupes de sécurité réseau sont appliqués. Lors de la résolution du problème, exécutez la commande pour chaque interface réseau.

Si vous rencontrez toujours un problème de connectivité, consultez [Diagnostic supplémentaire](#additional-diagnosis) et [Considérations](#considerations).

Si vous ignorez le nom d’une interface réseau, mais que vous connaissez le nom de la machine virtuelle à laquelle l’interface réseau est attachée, les commandes suivantes retournent les ID de toutes les interfaces réseau attachées à une machine virtuelle :

```azurepowershell-interactive
$VM = Get-AzVM -Name myVM -ResourceGroupName myResourceGroup
$VM.NetworkProfile
```

Le résultat ressemble à ce qui suit :

```output
NetworkInterfaces
-----------------
{/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic
```

Dans la précédente sortie, le nom d’interface réseau est *myVMVMNic*.

## <a name="diagnose-using-azure-cli"></a>Diagnostiquer à l’aide d’Azure CLI

Si vous utilisez des commandes de l’interface de ligne de commande (CLI) Azure pour accomplir les tâches décrites dans cet article, exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/bash) ou en exécutant Azure CLI sur votre ordinateur. Azure CLI version 2.0.32 ou ultérieure est nécessaire pour cet article. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli). Si vous exécutez Azure CLI localement, vous devez aussi exécuter `az login` et vous connecter à Azure avec un compte disposant des [autorisations nécessaires](virtual-network-network-interface.md#permissions).

Obtenez les règles de sécurité effectives d’une interface réseau avec [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg). L’exemple suivant obtient les règles de sécurité effectives d’une interface réseau nommée *myVMVMNic* , qui se trouve dans un groupe de ressources appelé *myResourceGroup* :

```azurecli-interactive
az network nic list-effective-nsg \
  --name myVMVMNic \
  --resource-group myResourceGroup
```

La sortie est retournée au format json. Pour comprendre la sortie, consultez [Interpréter la sortie de commande](#interpret-command-output).
La sortie est renvoyée uniquement si un groupe de sécurité réseau est associé à l’interface réseau, le sous-réseau auquel l’interface réseau appartient, ou les deux. La machine virtuelle doit être en cours d’exécution. Une machine virtuelle peut avoir plusieurs interfaces réseau à laquelle différents groupes de sécurité réseau sont appliqués. Lors de la résolution du problème, exécutez la commande pour chaque interface réseau.

Si vous rencontrez toujours un problème de connectivité, consultez [Diagnostic supplémentaire](#additional-diagnosis) et [Considérations](#considerations).

Si vous ignorez le nom d’une interface réseau, mais que vous connaissez le nom de la machine virtuelle à laquelle l’interface réseau est attachée, les commandes suivantes retournent les ID de toutes les interfaces réseau attachées à une machine virtuelle :

```azurecli-interactive
az vm show \
  --name myVM \
  --resource-group myResourceGroup
```

Dans les résultats retournés, vous obtenez des informations similaires à l’exemple suivant :

```output
"networkProfile": {
    "additionalProperties": {},
    "networkInterfaces": [
      {
        "additionalProperties": {},
        "id": "/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
        "primary": true,
        "resourceGroup": "myResourceGroup"
      },
```

Dans la sortie précédente, le nom de l’interface réseau est *interface myVMVMNic*.

## <a name="interpret-command-output"></a>Interpréter la sortie de commande

Indépendamment du fait d’avoir utilisé [PowerShell](#diagnose-using-powershell) ou [Azure CLI](#diagnose-using-azure-cli) pour diagnostiquer le problème, vous recevez une sortie contenant les informations suivantes :

- **NetworkSecurityGroup** : l’ID du groupe de sécurité réseau.
- **Association** : si le groupe de sécurité réseau est associé à une *interface réseau* ou à un *sous-réseau*. Si un groupe de sécurité réseau est associé aux deux, la sortie est retournée avec le **NetworkSecurityGroup** , **Association** , et les **EffectiveSecurityRules** , pour chaque groupe de sécurité réseau. Si le groupe de sécurité réseau est associé ou dissocié immédiatement avant l’exécution de cette commande pour afficher les règles de sécurité effectives, il se peut que vous deviez attendre quelques secondes pour que la modification apparaisse dans la sortie de la commande.
- **EffectiveSecurityRules** : une explication de chaque propriété est détaillée dans [Créer une règle de sécurité](manage-network-security-group.md#create-a-security-rule). Les noms de règles commençant par *defaultSecurityRules/* sont des règles de sécurité par défaut qui existent dans chaque groupe de sécurité réseau. Les noms commençant par la règle *securityRules/* sont des règles que vous avez créées. Les règles qui spécifient une [balise de service](security-overview.md#service-tags), comme **Internet** , **VirtualNetwork** , et **AzureLoadBalancer** pour les propriétés  **destinationAddressPrefix** ou **sourceAddressPrefix** , ont également des valeurs pour la propriété **expandedDestinationAddressPrefix**. La propriété **expandedDestinationAddressPrefix** répertorie tous les préfixes d’adresse représentés par la balise de service.

Si vous voyez des règles en double répertoriées dans la sortie, cela est dû au fait qu’un groupe de sécurité réseau est associé à la fois à l’interface réseau et au sous-réseau. Les deux groupes de sécurité réseau ont les mêmes règles par défaut et peuvent avoir des règles supplémentaires en double, si vous avez créé vos propres règles identiques pour les deux groupes de sécurité réseau.

La règle nommée **defaultSecurityRules/DenyAllInBound** est celle qui empêche les communications entrantes vers la machine virtuelle via le port 80, à partir d’internet, comme décrit dans le [scénario](#scenario). Aucune autre règle avec une priorité plus élevée (numéro inférieur) ne permet de trafic entrant par le port 80 à partir d’internet.

## <a name="resolve-a-problem"></a>Résoudre un problème

Si vous utilisez le [portail](#diagnose-using-azure-portal) Azure, [PowerShell](#diagnose-using-powershell), ou [Azure CLI](#diagnose-using-azure-cli) pour diagnostiquer le problème présenté dans le [scénario](#scenario) de cet article, la solution consiste à créer une règle de sécurité réseau avec les propriétés suivantes :

| Propriété                | Valeur                                                                              |
|---------                |---------                                                                           |
| Source                  | Quelconque                                                                                |
| Source port ranges      | Quelconque                                                                                |
| Destination             | L’adresse IP de la machine virtuelle, une plage d’adresses IP ou toutes les adresses dans le sous-réseau. |
| Plages de ports de destination | 80                                                                                 |
| Protocol                | TCP                                                                                |
| Action                  | Allow                                                                              |
| Priority                | 100                                                                                |
| Name                    | Allow-HTTP-All                                                                     |

Après avoir créé la règle, le port 80 autorise le trafic entrant à partir d’internet, étant donné que la priorité de la règle est supérieure à la règle de sécurité par défaut nommée *DenyAllInBound* , qui interdit le trafic. Apprenez à [créer une règle de sécurité](manage-network-security-group.md#create-a-security-rule). Si différents groupes de sécurité réseau sont associés à l’interface réseau et au sous-réseau, vous devez créer la même règle dans chaque groupe de sécurité réseau.

Lorsque Azure traite le trafic entrant, il traite les règles dans le groupe de sécurité réseau associé au sous-réseau (s’il existe un groupe de sécurité réseau associé) et il traite ensuite les règles dans le groupe de sécurité réseau associé à l’interface réseau. S’il existe un groupe de sécurité réseau associé à l’interface réseau et au sous-réseau, le port doit être ouvert dans chaque groupe de sécurité réseau, pour que le trafic atteigne la machine virtuelle. Pour faciliter les problèmes d’administration et de communication, nous vous recommandons d’associer un groupe de sécurité réseau à un sous-réseau, plutôt que des interfaces réseau individuelles. Si des machines virtuelles au sein d’un sous-réseau ont besoin de règles de sécurité différentes, vous pouvez rendre les interfaces réseau membres d’un groupe de sécurité d’application (ASG) et spécifier un ASG comme source et destination d’une règle de sécurité. Pour en savoir plus sur les [groupes de sécurité d’application](security-overview.md#application-security-groups).

Si des problèmes de communication subsistent, consultez [Considérations](#considerations) et Diagnostic supplémentaire.

## <a name="considerations"></a>Considérations

Lors de la résolution de problèmes de connectivité, considérez les points suivants :

* Des règles de sécurité par défaut bloquent l’accès entrant à partir d’internet et n’autorisent que le trafic entrant à partir du réseau virtuel. Pour autoriser le trafic entrant à partir d’internet, ajoutez des règles de sécurité avec une priorité plus élevée que les règles par défaut. En savoir plus sur les [règles de sécurité par défaut](security-overview.md#default-security-rules), ou comment [ajouter une règle de sécurité](manage-network-security-group.md#create-a-security-rule).
* Si vous avez des réseaux virtuels homologues, par défaut, la balise de service **VIRTUAL_NETWORK** s’étend automatiquement pour inclure les préfixes des réseaux virtuels homologues. Pour résoudre des problèmes liés au peering de réseau virtuel, vous pouvez consulter les préfixes dans la liste **ExpandedAddressPrefix**. En savoir plus sur le [peering de réseaux virtuels](virtual-network-peering-overview.md) et les [balises de service](security-overview.md#service-tags).
* Les règles de sécurité effectives sont uniquement affichées pour une interface réseau s’il existe un groupe de sécurité réseau associé à l’interface réseau de la machine virtuelle et/ ou au sous-réseau, et si la machine virtuelle est en cours d’exécution.
* Si aucun groupe de sécurité réseau n’est associé à la carte réseau ou au sous-réseau, et si une [adresse IP publique](virtual-network-public-ip-address.md) est affectée à une machine virtuelle, tous les ports sont ouverts pour les accès entrants et sortants pour toutes les destinations. Si la machine virtuelle possède une adresse IP publique, nous vous recommandons d’appliquer un groupe de sécurité réseau au sous-réseau de l’interface réseau.

## <a name="additional-diagnosis"></a>Diagnostic supplémentaire

* Pour exécuter un test rapide pour déterminer si le trafic est autorisé vers ou à partir d’une machine virtuelle, utilisez la capacité [IP flow verify](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) de Azure Network Watcher. IP Flow Verify vous indique si le trafic est autorisé ou refusé. S’il est refusé, IP Flow Verify vous indique la règle de sécurité refusant le trafic.
* Si aucune règle de sécurité n’est à l’origine d’un échec de connectivité réseau d’une machine virtuelle, les causes du problème peuvent être les suivantes :
  * Logiciel de pare-feu s’exécutant à l’intérieur du système d’exploitation de la machine virtuelle
  * Itinéraires configurés pour des appliances virtuelles ou un trafic local. Le trafic Internet peut être redirigé localement vers votre réseau local au moyen d’un [tunneling forcé](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si vous forcez le trafic internet de tunnel vers une appliance virtuelle ou sur site, vous n’êtes peut-être pas en mesure de vous connecter à la machine virtuelle à partir d’internet. Pour savoir comment diagnostiquer les problèmes de routage pouvant entraver le flux du trafic sortant de la machine virtuelle, consultez [Diagnostiquer un problème de routage de trafic réseau de la machine virtuelle](diagnose-network-routing-problem.md).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur toutes les tâches, les propriétés et les paramètres pour un [groupe de sécurité réseau](manage-network-security-group.md#work-with-network-security-groups) et les [règles de sécurité](manage-network-security-group.md#work-with-security-rules).
- En savoir plus sur mes [règles de sécurité par défaut](security-overview.md#default-security-rules), les [balises de service](security-overview.md#service-tags), et [comment Azure traite les règles de sécurité pour le trafic entrant et sortant](security-overview.md#network-security-groups) pour une machine virtuelle.
