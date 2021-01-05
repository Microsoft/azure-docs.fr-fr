---
title: 'Passerelle VPN Azure : Publier des routes personnalisées pour des clients VPN point à site'
description: Étapes à suivre pour publier des routes personnalisées pour vos clients de point à site
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: a02bd5519b776a063646c11be2a34366fe429f99
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89392389"
---
# <a name="advertise-custom-routes-for-p2s-vpn-clients"></a>Publier des routes personnalisées pour des clients VPN point à site

Vous pouvez être amené à publier des routes personnalisées pour tous vos clients VPN point à site. Ce peut être le cas si vous avez activé des points de terminaison de stockage dans votre réseau virtuel et que vous souhaitez que les utilisateurs distants puissent accéder à ces comptes de stockage par le biais de la connexion VPN. Vous pouvez publier l’adresse IP du point de terminaison de stockage pour tous vos utilisateurs distants afin que le trafic à destination du compte de stockage passe par le tunnel VPN, et non par l’Internet public.

![Exemple de connexion multisites de passerelle VPN Azure](./media/vpn-gateway-p2s-advertise-custom-routes/custom-routes.png)

## <a name="to-advertise-custom-routes"></a>Pour publier des routes personnalisées

Pour publier des routes personnalisées, utilisez `Set-AzVirtualNetworkGateway cmdlet`. L’exemple suivant montre comment publier l’adresse IP pour les [tables de compte de stockage Contoso](https://contoso.table.core.windows.net).

1. Effectuez un test ping sur *contoso.table.core.windows.net* et notez l’adresse IP. Par exemple :

    ```cmd
    C:\>ping contoso.table.core.windows.net
    Pinging table.by4prdstr05a.store.core.windows.net [13.88.144.250] with 32 bytes of data:
    ```

2. Lancer les commandes PowerShell suivantes :

    ```azurepowershell-interactive
    $gw = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroupName <name of resource group>
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -CustomRoute 13.88.144.250/32
    ```

3. Pour ajouter plusieurs routes personnalisées, utilisez une virgule et des espaces afin de séparer les adresses. Par exemple :

    ```azurepowershell-interactive
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -CustomRoute x.x.x.x/xx , y.y.y.y/yy
    ```
## <a name="to-view-custom-routes"></a>Pour voir les routes personnalisées

Utilisez l’exemple suivant pour afficher les routes personnalisées :

  ```azurepowershell-interactive
  $gw = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroupName <name of resource group>
  $gw.CustomRoutes | Format-List
  ```
## <a name="to-delete-custom-routes"></a>Pour supprimer des itinéraires personnalisés

Utilisez l’exemple suivant pour supprimer des itinéraires personnalisés :

  ```azurepowershell-interactive
  $gw = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroupName <name of resource group>
  Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -CustomRoute @0
  ```
## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le routage de point à site, consultez [À propos du routage de point à site](vpn-gateway-about-point-to-site-routing.md).
