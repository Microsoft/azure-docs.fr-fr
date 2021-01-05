---
title: Tunneling forcé du pare-feu Azure
description: Vous pouvez configurer un tunneling forcé pour router le trafic Internet vers une appliance virtuelle réseau ou un pare-feu supplémentaire en vue d’un traitement ultérieur.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 08/19/2020
ms.author: victorh
ms.openlocfilehash: da2b206bf24cb33180305e32e270b989eb64dfa3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88612591"
---
# <a name="azure-firewall-forced-tunneling"></a>Tunneling forcé du pare-feu Azure

Lorsque vous configurez un nouveau pare-feu Azure, vous pouvez acheminer tout le trafic Internet vers un tronçon suivant désigné au lieu d’accéder directement à Internet. Par exemple, vous pouvez disposer d’un pare-feu de périphérie local ou d’une autre appliance virtuelle réseau (NVA) pour traiter le trafic réseau avant qu’il ne soit dirigé vers Internet. Toutefois, vous ne pouvez pas configurer un pare-feu existant pour le tunneling forcé.

Par défaut, le tunneling forcé n’est pas autorisé sur le pare-feu Azure afin de garantir que toutes ses dépendances Azure sortantes sont satisfaites. Les configurations de routes définies par l’utilisateur (UDR) sur le *AzureFirewallSubnet* ayant une route par défaut qui n’est pas directement dirigé vers Internet sont désactivées.

## <a name="forced-tunneling-configuration"></a>Configuration de tunneling forcé

Pour prendre en charge le tunneling forcé, le trafic du management des services est séparé du trafic client. Un sous-réseau dédié supplémentaire nommé *AzureFirewallManagementSubnet* (taille minimale du sous-réseau /26) ayant sa propre IP publique associée est nécessaire. La seule route autorisée sur ce sous-réseau est une route par défaut vers Internet, et la propagation de route BGP doit être désactivée.

Si vous avez une route par défaut publiée via BGP pour forcer le trafic vers un emplacement local, vous devez créer *AzureFirewallSubnet* et *AzureFirewallManagementSubnet* avant de déployer votre pare-feu et avoir un UDR ayant une route par défaut vers Internet. De plus, l’option **Propager des routes de passerelle** doit être désactivée.

Dans cette configuration, *AzureFirewallSubnet* peut désormais inclure des routes vers n’importe quel pare-feu local ou n’importe quelle appliance virtuelle réseau pour traiter le trafic avant qu’il ne soit dirigé vers Internet. Vous pouvez également publier ces routes via le protocole BGP sur *AzureFirewallSubnet* si l’option **Propager des routes de passerelle** est activée sur ce sous-réseau.

Par exemple, vous pouvez créer une route par défaut sur *AzureFirewallSubnet* avec votre passerelle VPN comme tronçon suivant pour accéder à votre appareil local. Ou vous pouvez activer l’option **Propager des routes de passerelle** pour récupérer les routes appropriées sur le réseau local.

:::image type="content" source="media/forced-tunneling/route-propagation.png" alt-text="Propagation de la route de la passerelle de réseau virtuel":::

Si vous activez le tunneling forcé, le trafic Internet est « SNATé » vers l'une des adresses IP privées du pare-feu dans AzureFirewallSubnet, ce qui a pour effet de masquer la source à votre pare-feu local.

Si votre organisation utilise une plage d’adresses IP publiques pour les réseaux privés, Pare-feu Azure traduit l’adresse réseau source du trafic en une des adresses IP privées du pare-feu dans AzureFirewallSubnet. Toutefois, vous pouvez configurer Pare-feu Azure pour qu’il n’effectue **pas** une telle traduction. Pour plus d’informations, consultez [Plages d’adresses IP privées SNAT du Pare-feu Azure](snat-private-range.md).

Une fois que vous avez configuré le pare-feu Azure pour prendre en charge le tunneling forcé, vous ne pouvez pas annuler la configuration. Si vous supprimez toutes les autres configurations IP de votre pare-feu, la configuration IP de gestion est également supprimée et le pare-feu est libéré. L’adresse IP publique attribuée à la configuration IP de gestion ne peut pas être supprimée, mais vous pouvez attribuer une autre adresse IP publique.

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Déployer et configurer un pare-feu Azure dans un réseau hybride à l’aide du portail Azure](tutorial-hybrid-portal.md)
