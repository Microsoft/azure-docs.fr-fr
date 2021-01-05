---
title: Connecter des réseaux virtuels inter-locataires à un hub:PowerShell
titleSuffix: Azure Virtual WAN
description: Cet article vous aide à connecter des réseaux virtuels inter-locataires à un hub virtuel à l’aide de PowerShell.
services: virtual-wan
author: wtnlee
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/28/2020
ms.author: wellee
ms.openlocfilehash: c49a85c71c9b877be7e143f5caf27dc307fe0c12
ms.sourcegitcommit: 8a1ba1ebc76635b643b6634cc64e137f74a1e4da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94381267"
---
# <a name="connect-cross-tenant-vnets-to-a-virtual-wan-hub"></a>Connecter des réseaux virtuels inter-locataires à un hub Virtual WAN

Cet article vous aide à utiliser Virtual WAN pour connecter un réseau virtuel à un hub virtuel situé dans un autre locataire. Cette architecture est utile si vous avez des charges de travail de client qui doivent être connectées au même réseau mais se trouvent dans des locataire différents. Par exemple, comme l’illustre le diagramme suivant, vous pouvez connecter un réseau virtuel autre que Contoso (locataire distant) à un hub virtuel Contoso (locataire parent).

:::image type="content" source="./media/cross-tenant-vnet/connectivity.png" alt-text="Définir la configuration du routage" :::

Dans cet article, vous apprendrez comment :

* Ajouter un autre locataire comme contributeur sur votre abonnement Azure.
* Connecter un réseau virtuel inter-locataires à un hub virtuel.

La procédure suivie pour cette configuration fait appel au portail Azure et à PowerShell. Cependant, la fonctionnalité elle-même n’est disponible que dans PowerShell et l’interface CLI.

## <a name="before-you-begin"></a>Avant de commencer

### <a name="prerequisites"></a>Prérequis

Pour utiliser la procédure décrite dans cet article, la configuration suivante doit déjà être définie dans votre environnement :

* Un WAN virtuel et un hub virtuel dans votre abonnement parent.
* Un réseau virtuel configuré dans un abonnement situé dans un autre locataire (distant).
* Veillez à ce que l’espace d’adressage du réseau virtuel dans le locataire distant ne se superpose pas à d’autres espaces d’adressage dans d’autres réseaux virtuels connectés au hub virtuel parent.

### <a name="working-with-azure-powershell"></a>Utilisation d’Azure PowerShell

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell.md)]

## <a name="assign-permissions"></a><a name="rights"></a>Affecter des autorisations

Pour que l’abonnement parent doté du hub virtuel puisse accéder aux réseaux virtuels du locataire distant et les modifier, vous devez lui attribuer des autorisations de **Contributeur** à partir de l’abonnement du locataire distant.

1. Ajoutez l’attribution de rôle **Contributeur** au compte parent (celui doté du hub WAN virtuel). Pour attribuer ce rôle, vous pouvez utiliser PowerShell ou le portail Azure. Pour connaître les procédures, consultez les articles **Ajouter ou supprimer des attributions de rôle** suivants :

   * [PowerShell](../role-based-access-control/role-assignments-powershell.md)
   * [Portail](../role-based-access-control/role-assignments-portal.md)

1. Ajoutez ensuite l’abonnement du locataire distant et l’abonnement du locataire parent à la session active de PowerShell. Exécutez la commande suivante : Si vous êtes connecté à l’abonnement parent, il vous suffit d’exécuter la commande pour le locataire distant.

   ```azurepowershell-interactive
   Add-AzAccount -SubscriptionId "xxxxx-b34a-4df9-9451-4402dcaecc5b"
   ```

1. Vérifiez que l’attribution de rôle a réussi en vous connectant à Azure PowerShell à l’aide des informations d’identification du parent, et exécutez la commande suivante :

   ```azurepowershell-interactive
   Get-AzSubscription
   ```

1. Si les autorisations ont été correctement diffusées au parent et ajoutées à la session, l’abonnement détenu par le locataire distant apparaît dans la sortie de la commande.

## <a name="connect-vnet-to-hub"></a><a name="connect"></a>Connecter un réseau virtuel à un hub

Dans la procédure suivante, vous passez d’un contexte d’abonnement à un autre pour établir la liaison entre le réseau virtuel et le hub virtuel. Remplacez les exemples de valeurs pour les adapter à votre propre environnement.

1. Vérifiez que vous êtes dans le contexte de votre compte distant en exécutant la commande suivante :

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionId "[remote ID]"
   ```

1. Créez une variable locale pour stocker les métadonnées du réseau virtuel que vous voulez connecter au hub.

   ```azurepowershell-interactive
   $remote = Get-AzVirtualNetwork -Name "[vnet name]" -ResourceGroupName "[resource group name]"
   ```

1. Revenez au compte parent.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionId "[parent ID]"
   ```

1. Connectez le réseau virtuel au hub.

   ```azurepowershell-interactive
   New-AzVirtualHubVnetConnection -ResourceGroupName "[parent resource group name]" -VirtualHubName "[virtual hub name]" -Name "[name of connection]" -RemoteVirtualNetwork $[local variable name]
   ```

1. La nouvelle connexion apparaît dans PowerShell et dans le portail Azure.

   * **PowerShell :** Les métadonnées de la connexion fraîchement formée apparaissent dans la console PowerShell si la connexion a été correctement formée.
   * **Portail Azure :** Accédez au hub virtuel, **Connectivité -> Connexions de réseau virtuel**. Vous pouvez afficher le pointeur vers la connexion. Pour afficher la ressource actuelle, vous devez disposer des autorisations appropriées.
   
## <a name="troubleshooting"></a><a name="troubleshoot"></a>Résolution des problèmes

* Vérifiez que les métadonnées dans $remote (de la [section](#connect) précédente) correspondent aux informations du portail Azure.
* Vous pouvez vérifier les autorisations à l’aide des paramètres IAM du groupe de ressources du locataire distant ou à l’aide des commandes Azure PowerShell (Get-AzSubscription).
* Veillez à mettre les noms des groupes de ressources ou de toute autre variable d’environnement entre guillemets (par ex. "VirtualHub1" ou "VirtualNetwork1").

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Virtual WAN, consultez :

* [FAQ](virtual-wan-faq.md) sur Virtual WAN
