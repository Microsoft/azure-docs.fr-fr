---
title: Afficher la topologie de réseau virtuel Azure | Microsoft Docs
description: Découvrez comment afficher les ressources d’un réseau virtuel et les relations entre ces ressources.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2018
ms.author: damendo
ms.openlocfilehash: c04da65af27ebd5ac654bc059ae004c157a20f33
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84737527"
---
# <a name="view-the-topology-of-an-azure-virtual-network"></a>Afficher la topologie d’un réseau virtuel Azure

Dans cet article, vous allez apprendre à afficher les ressources d’un réseau virtuel Microsoft Azure et les relations entre ces ressources. Par exemple, un réseau virtuel contient des sous-réseaux. Les sous-réseaux contiennent des ressources, telles que les machines virtuelles Azure (VM). Une machine virtuelle peut avoir une ou plusieurs interfaces réseau. Chaque sous-réseau peut être associé à un groupe de sécurité réseau et une table de routage. La capacité de topologie d’Azure Network Watcher vous permet de voir toutes les ressources d’un réseau virtuel, les ressources associées aux ressources d’un réseau virtuel et les relations entre ces ressources.

Vous pouvez utiliser le [portail Azure](#azure-portal), l’[interface de ligne de commande Azure](#azure-cli) ou [PowerShell](#powershell) pour afficher une topologie.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="view-topology---azure-portal"></a><a name = "azure-portal"></a>Afficher la topologie - Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte disposant des [autorisations](required-rbac-permissions.md) nécessaires.
2. En haut à gauche du portail, sélectionnez **Tous les services**.
3. Dans la zone de filtre **Tous les services**, entrez *Network Watcher*. Quand la mention **Network Watcher** apparaît dans les résultats, sélectionnez-la.
4. Sélectionnez **Topologie**. Pour générer une topologie, vous devez disposer d’un observateur réseau dans la même région que celle dans laquelle figure le réseau virtuel pour lequel vous souhaitez générer la topologie. Si vous n’avez pas d’observateur réseau activé dans la région où se trouve le réseau virtuel pour lequel vous souhaitez générer une topologie, les observateurs réseau sont créés automatiquement dans toutes les régions. Les observateurs réseau sont créés dans un groupe de ressources nommé **NetworkWatcherRG**.
5. Sélectionnez un abonnement, le groupe de ressources d’un réseau virtuel pour lequel vous souhaitez afficher la topologie, puis le réseau virtuel. Dans l’image suivante, vous allez voir la topologie d’un réseau virtuel nommé *MyVnet*, dans le groupe de ressources nommé *MyResourceGroup* :

    ![Afficher la topologie](./media/view-network-topology/view-topology.png)

    Comme vous pouvez le voir dans l’image précédente, le réseau virtuel contient trois sous-réseaux. Un sous-réseau comprend une machine virtuelle qui a été déployée. La machine virtuelle dispose d’une interface réseau et est associée à une adresse IP publique. Les deux autres sous-réseaux sont associés à une table de routage. Chaque table de routage contient deux itinéraires. Un sous-réseau est associé à un groupe de sécurité réseau. Les informations de topologie s’affichent uniquement pour les ressources :
    
    - Dans le même groupe de ressources et dans la même la région que le réseau virtuel *myVnet*. Par exemple, un groupe de sécurité réseau qui se trouve dans un groupe de ressources autre que *MyResourceGroup* n’apparaît pas, même si le groupe de sécurité réseau est associé à un sous-réseau dans le réseau virtuel *MyVnet*.
    - Dans le réseau virtuel *myVnet* ou associées à des ressources de ce réseau virtuel. Par exemple, un groupe de sécurité réseau qui n’est pas associé à un sous-réseau ou une interface réseau dans le réseau virtuel *myVnet* n’apparaît pas, même si le groupe de sécurité réseau se trouve dans le groupe de ressources *MyResourceGroup*.

   La topologie présentée dans l’image correspond au réseau virtuel créé après le déploiement de l’**Exemple de script pour acheminer le trafic via une appliance virtuelle réseau**, que vous pouvez déployer à l’aide de l’[interface de ligne de commande Azure](../virtual-network/scripts/virtual-network-cli-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) ou [PowerShell](../virtual-network/scripts/virtual-network-powershell-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

6. Sélectionnez **Télécharger la topologie** pour télécharger l’image sous forme de fichier modifiable, au format svg.

Les ressources affichées dans le diagramme sont un sous-ensemble des composants de mise en réseau du réseau virtuel. Par exemple, lorsqu’un groupe de sécurité réseau apparaît, les règles de sécurité qu’il contient ne figurent pas dans le diagramme. Bien qu’elles ne soient pas différenciées dans le diagramme, les lignes représentent l’une des deux relations : *Relation contenant-contenu* ou *associés*. Pour afficher la liste complète des ressources du réseau virtuel ainsi que le type de relation entre les ressources, générez la topologie avec [PowerShell](#powershell) ou l’[interface de ligne de commande Azure](#azure-cli).

## <a name="view-topology---azure-cli"></a><a name = "azure-cli"></a>Afficher la topologie - Interface de ligne de commande Azure

Vous pouvez exécuter les commandes dans les étapes qui suivent :
- Dans Azure Cloud Shell, en sélectionnant **Essayer** en haut à droite d’une commande. Azure Cloud Shell est un interpréteur de commandes interactif gratuit dans lequel les outils Azure communs ont été préinstallés et configurés pour pouvoir être utilisés avec votre compte.
- En exécutant l’interface de ligne de commande depuis votre ordinateur. Si vous exécutez l’interface CLI depuis votre ordinateur, les étapes décrites dans cet article nécessitent la version 2.0.31 d’Azure CLI ou une version ultérieure. Exécutez `az --version` pour rechercher la version installée. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli). Si vous exécutez Azure CLI localement, vous devez également exécuter `az login` pour créer une connexion avec Azure.

Le compte que vous utilisez doit disposer des [autorisations](required-rbac-permissions.md) nécessaires.

1. Si vous avez déjà un observateur réseau dans la même région que le réseau virtuel pour lequel vous souhaitez créer une topologie, passez à l’étape 3. Créez un groupe de ressources qui contiendra un observateur réseau avec la commande [az group create](/cli/azure/group). L’exemple suivant permet de créer le groupe de ressources dans la région *eastus* :

    ```azurecli-interactive
    az group create --name NetworkWatcherRG --location eastus
    ```

2. Créez un observateur réseau avec la commande [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure). L’exemple suivant permet de créer un observateur réseau dans la région *eastus* :

    ```azurecli-interactive
    az network watcher configure \
      --resource-group NetworkWatcherRG \
      --location eastus \
      --enabled true
    ```

3. Affichez la topologie avec [az network watcher show-topology](/cli/azure/network/watcher#az-network-watcher-show-topology). L’exemple suivant affiche la topologie d’un groupe de ressources nommé *myResourceGroup* :

    ```azurecli-interactive
    az network watcher show-topology --resource-group MyResourceGroup
    ```

    Les informations sur la topologie ne sont renvoyées que pour les ressources qui se trouvent dans le même groupe de ressources que le groupe de ressources *MyResourceGroup* et la même région que l’observateur réseau. Par exemple, un groupe de sécurité réseau qui se trouve dans un groupe de ressources autre que *MyResourceGroup* n’apparaît pas, même si le groupe de sécurité réseau est associé à un sous-réseau dans le réseau virtuel *MyVnet*.

   En savoir plus sur les relations et les [propriétés](#properties) dans les résultats renvoyés. Si vous n’avez pas de réseau virtuel dont vous pouvez afficher la topologie, vous pouvez en créer un à l’aide de l’exemple de script [Acheminer le trafic via une appliance virtuelle réseau](../virtual-network/scripts/virtual-network-cli-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json). Pour afficher un diagramme de la topologie et le télécharger dans un fichier modifiable, utilisez le [portail](#azure-portal).

## <a name="view-topology---powershell"></a><a name = "powershell"></a>Afficher la topologie - PowerShell

Vous pouvez exécuter les commandes dans les étapes qui suivent :
- Dans Azure Cloud Shell, en sélectionnant **Essayer** en haut à droite d’une commande. Azure Cloud Shell est un interpréteur de commandes interactif gratuit dans lequel les outils Azure communs ont été préinstallés et configurés pour pouvoir être utilisés avec votre compte.
- En exécutant PowerShell à partir de votre ordinateur. Si vous exécutez PowerShell depuis votre ordinateur, cet article nécessite le module Azure PowerShell `Az`. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

Le compte que vous utilisez doit disposer des [autorisations](required-rbac-permissions.md) nécessaires.

1. Si vous avez déjà un observateur réseau dans la même région que le réseau virtuel pour lequel vous souhaitez créer une topologie, passez à l’étape 3. Créez un groupe de ressources qui contiendra un observateur réseau avec la commande [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). L’exemple suivant permet de créer le groupe de ressources dans la région *eastus* :

    ```azurepowershell-interactive
    New-AzResourceGroup -Name NetworkWatcherRG -Location EastUS
    ```

2. Créez un observateur réseau avec [New-AzNetworkWatcher](/powershell/module/az.network/new-aznetworkwatcher). L’exemple suivant permet de créer un observateur réseau dans la région eastus :

    ```azurepowershell-interactive
    New-AzNetworkWatcher `
      -Name NetworkWatcher_eastus `
      -ResourceGroupName NetworkWatcherRG
    ```

3. Récupérez une instance de Network Watcher avec [Get-AzNetworkWatcher](/powershell/module/az.network/get-aznetworkwatcher). L’exemple suivant permet de récupérer un observateur réseau dans la région USA Est :

    ```azurepowershell-interactive
    $nw = Get-AzResource `
      | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "EastUS" }
    $networkWatcher = Get-AzNetworkWatcher `
      -Name $nw.Name `
      -ResourceGroupName $nw.ResourceGroupName
    ```

4. Récupérez une topologie avec [Get-AzNetworkWatcherTopology](/powershell/module/az.network/get-aznetworkwatchertopology). L’exemple suivant permet de récupérer la topologie d’un réseau virtuel dans le groupe de ressources *MyResourceGroup* :

    ```azurepowershell-interactive
    Get-AzNetworkWatcherTopology `
      -NetworkWatcher $networkWatcher `
      -TargetResourceGroupName MyResourceGroup
    ```

   Les informations sur la topologie ne sont renvoyées que pour les ressources qui se trouvent dans le même groupe de ressources que le groupe de ressources *MyResourceGroup* et la même région que l’observateur réseau. Par exemple, un groupe de sécurité réseau qui se trouve dans un groupe de ressources autre que *MyResourceGroup* n’apparaît pas, même si le groupe de sécurité réseau est associé à un sous-réseau dans le réseau virtuel *MyVnet*.

   En savoir plus sur les relations et les [propriétés](#properties) dans les résultats renvoyés. Si vous n’avez pas de réseau virtuel dont vous pouvez afficher la topologie, vous pouvez en créer un à l’aide de l’exemple de script [Acheminer le trafic via une appliance virtuelle réseau](../virtual-network/scripts/virtual-network-powershell-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json). Pour afficher un diagramme de la topologie et le télécharger dans un fichier modifiable, utilisez le [portail](#azure-portal).

## <a name="relationships"></a>Relations

Toutes les ressources renvoyées dans une topologie présentent l’un des types de relation suivants avec une autre ressource :

| Type de relation | Exemple                                                                                                |
| ---               | ---                                                                                                    |
| Containment       | Un réseau virtuel contient un sous-réseau. Un sous-réseau contient une interface réseau.                            |
| Associé        | Une interface réseau est associée à une machine virtuelle. Une adresse IP publique est associée à une interface réseau. |

## <a name="properties"></a>Propriétés

Toutes les ressources renvoyées dans une topologie ont les propriétés suivantes :

- **Name** : nom de la ressource.
- **id** : URI de la ressource.
- **Location** : région Azure dans laquelle se trouve la ressource.
- **Associations** : liste des associations réalisées vis-à-vis de l’objet référencé. Chaque association présente les propriétés suivantes :
    - **AssociationType** : fait référence à la relation entre l’objet enfant et le parent. Les valeurs valides sont *Contains* et *Associated*.
    - **name** : nom de la ressource référencée.
    - **ResourceId** : URI de la ressource référencée dans l’association.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [diagnostiquer un problème de filtre du trafic réseau vers ou depuis une machine virtuelle](diagnose-vm-network-traffic-filtering-problem.md) à l’aide de la fonctionnalité de vérification du flux IP de Network Watcher
- Découvrez comment [diagnostiquer un problème de routage du trafic réseau depuis une machine virtuelle](diagnose-vm-network-routing-problem.md) à l’aide de la fonctionnalité de tronçon suivant de Network Watcher
