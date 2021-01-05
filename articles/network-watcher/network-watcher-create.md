---
title: Créer une instance d’Azure Network Watcher | Microsoft Docs
description: Découvrez comment créer un Network Watcher Azure dans une région Azure à l’aide du portail Azure ou d’autres technologies, et comment supprimer un Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: b1314119-0b87-4f4d-b44c-2c4d0547fb76
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: c308824afdcae5f5c04a316c199bad71ad84a429
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961989"
---
# <a name="create-an-azure-network-watcher-instance"></a>Créer une instance d’Azure Network Watcher

Network Watcher est un service régional qui vous permet de surveiller et de diagnostiquer l’état au niveau d’un scénario réseau dans, vers et depuis Azure. La surveillance au niveau des scénarios vous permet de diagnostiquer les problèmes avec une vue de bout en bout du réseau. Les outils de visualisation et de diagnostic réseau disponibles avec Network Watcher vous aident à comprendre, diagnostiquer et obtenir des informations sur votre réseau dans Azure. Network Watcher est activé par la création d’une ressource Network Watcher. Cette ressource vous permet d’utiliser les fonctionnalités de Network Watcher.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="network-watcher-is-automatically-enabled"></a>Network Watcher est activé automatiquement
Lorsque vous créez ou mettez à jour un réseau virtuel dans votre abonnement, Network Watcher sera automatiquement activé dans la région de votre réseau virtuel. L’activation automatique de Network Watcher n’a aucun impact sur vos ressources ni sur les frais associés.

#### <a name="opt-out-of-network-watcher-automatic-enablement"></a>Refus d’activation automatique de Network Watcher
Si vous souhaitez désactiver l’activation automatique de Network Watcher, vous pouvez le faire en exécutant les commandes suivantes :

> [!WARNING]
> Le refus d’activation automatique de Network Watcher est définitif. Si vous changez d’avis après avoir fait ce choix, vous devrez [recontacter le support](https://azure.microsoft.com/support/options/)

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName DisableNetworkWatcherAutocreation -ProviderNamespace Microsoft.Network
Register-AzResourceProvider -ProviderNamespace Microsoft.Network
```

```azurecli-interactive
az feature register --name DisableNetworkWatcherAutocreation --namespace Microsoft.Network
az provider register -n Microsoft.Network
```



## <a name="create-a-network-watcher-in-the-portal"></a>Créer un Network Watcher dans le portail

Accédez à **Tous les services** > **Mise en réseau** > **Network Watcher**. Vous pouvez sélectionner tous les abonnements pour lesquels vous souhaitez activer Network Watcher. Cette action crée un Network Watcher dans chaque région disponible.

![créer un Network Watcher](./media/network-watcher-create/figure1.png)

Lorsque vous activez Network Watcher en utilisant le portail, le nom de l’instance de Network Watcher est automatiquement défini comme *NetworkWatcher_nom_région*, où *nom_région* correspond à la région Azure dans laquelle l’instance est activée. Par exemple, une instance de Network Watcher activée dans la région USA Centre-Ouest est nommée *NetworkWatcher_westcentralus*.

L’instance de Network Watcher est automatiquement créée dans un groupe de ressources nommé *NetworkWatcherRG*. Ce groupe de ressources est créé s’il n’existe pas déjà.

Si vous souhaitez personnaliser le nom d’une instance Network Watcher et le groupe de ressources dans lequel elle est placée, vous pouvez utiliser PowerShell, l’interface de ligne de commande Azure, l’API REST, ou les méthodes ARMClient décrites dans les sections suivantes. Dans chacune des options, le groupe de ressources doit déjà exister pour que vous puissiez y créer une instance de Network Watcher.  

## <a name="create-a-network-watcher-with-powershell"></a>Créer un Network Watcher avec PowerShell

Pour créer une instance de Network Watcher, exécutez l’exemple suivant :

```powershell
New-AzNetworkWatcher -Name "NetworkWatcher_westcentralus" -ResourceGroupName "NetworkWatcherRG" -Location "West Central US"
```

## <a name="create-a-network-watcher-with-the-azure-cli"></a>Créer un Network Watcher avec Azure CLI

Pour créer une instance de Network Watcher, exécutez l’exemple suivant :

```azurecli
az network watcher configure --resource-group NetworkWatcherRG --locations westcentralus --enabled
```

## <a name="create-a-network-watcher-with-the-rest-api"></a>Créer un Network Watcher avec l’API REST

L’outil ARMclient permet d’appeler l’API REST à l’aide de PowerShell. Cet outil est accessible sur le site chocolatey à partir de la page [ARMClient sur Chocolatey](https://chocolatey.org/packages/ARMClient).

### <a name="log-in-with-armclient"></a>Se connecter à ARMClient

```powerShell
armclient login
```

### <a name="create-the-network-watcher"></a>Créer le Network Watcher

```powershell
$subscriptionId = '<subscription id>'
$networkWatcherName = '<name of network watcher>'
$resourceGroupName = '<resource group name>'
$apiversion = "2016-09-01"
$requestBody = @"
{
'location': 'West Central US'
}
"@

armclient put "https://management.azure.com/subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}?api-version=${api-version}" $requestBody
```

## <a name="create-a-network-watcher-using-azure-quickstart-template"></a>Créer une instance Network Watcher à l’aide du modèle de démarrage rapide Azure

Pour créer une instance Network Watcher, reportez-vous à ce [modèle de démarrage rapide](https://azure.microsoft.com/resources/templates/101-networkwatcher-create/).

## <a name="delete-a-network-watcher-in-the-portal"></a>Supprimer un observateur réseau dans le portail

Accédez à **Tous les services** > **Mise en réseau** > **Network Watcher**.

Si ce n’est déjà fait, sélectionnez l’onglet de vue d’ensemble. Dans la liste déroulante, sélectionnez l’abonnement pour lequel vous souhaitez désactiver l’observateur réseau.
Développez la liste des régions pour l’abonnement choisi en cliquant sur la flèche. Pour toute région choisie, utilisez les 3 points à droite pour accéder au menu contextuel.
Cliquez sur « Désactiver l’observateur de réseau » pour démarrer la désactivation. Vous êtes invité à confirmer cette action. Cliquez sur Oui pour continuer.
Dans le portail, vous devrez effectuer cette opération individuellement pour chaque région de chaque abonnement.


## <a name="delete-a-network-watcher-with-powershell"></a>Supprimer un observateur réseau avec PowerShell

Pour supprimer une instance de Network Watcher, exécutez l’exemple suivant :

```powershell
New-AzResourceGroup -Name NetworkWatcherRG -Location westcentralus
New-AzNetworkWatcher -Name NetworkWatcher_westcentralus -ResourceGroup NetworkWatcherRG -Location westcentralus
Remove-AzNetworkWatcher -Name NetworkWatcher_westcentralus -ResourceGroup NetworkWatcherRG
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez une instance de Network Watcher, découvrez les fonctionnalités disponibles :

* [Topologie](./view-network-topology.md)
* [Capture de paquet](network-watcher-packet-capture-overview.md)
* [Vérification du flux IP](network-watcher-ip-flow-verify-overview.md)
* [Tronçon suivant](network-watcher-next-hop-overview.md)
* [Affichage des groupes de sécurité](network-watcher-security-group-view-overview.md)
* [Journalisation des flux de groupe de sécurité réseau](network-watcher-nsg-flow-logging-overview.md)
* [Résolution des problèmes de passerelle de réseau virtuel](network-watcher-troubleshoot-overview.md)