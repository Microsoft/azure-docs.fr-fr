---
title: 'Passerelle VPN Azure : Supprimer une passerelle : portail'
description: Supprimer une passerelle de réseau virtuel à l’aide du portail Azure
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.date: 10/16/2020
ms.author: cherylmc
ms.topic: how-to
ms.openlocfilehash: 7d9ae31b5701707589d79fd5f3d7eb0802038eb9
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148197"
---
# <a name="delete-a-virtual-network-gateway-using-the-portal"></a>Supprimer une passerelle de réseau virtuel à l’aide du portail

> [!div class="op_single_selector"]
> * [Azure portal](vpn-gateway-delete-vnet-gateway-portal.md)
> * [PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [PowerShell (classique)](vpn-gateway-delete-vnet-gateway-classic-powershell.md)

Cet article explique comment supprimer une passerelle de réseau virtuel. Deux approches sont possibles afin de supprimer une passerelle pour une configuration de passerelle VPN.

* Si vous voulez tout supprimer et recommencer, comme dans le cas d’un environnement de test, vous pouvez supprimer le groupe de ressources. Supprimer un groupe de ressources supprime toutes les ressources du groupe. Cette méthode est recommandée seulement si vous ne voulez conserver aucune des ressources du groupe de ressources. Vous ne pouvez pas choisir de supprimer uniquement certaines ressources avec cette approche.

* Si vous souhaitez conserver certaines ressources de votre groupe de ressources, la suppression d’une passerelle de réseau virtuel devient légèrement plus complexe. Avant de supprimer la passerelle de réseau virtuel, vous devez commencer par supprimer toutes les ressources qui dépendent de la passerelle. Les étapes à suivre dépendent du type de connexions que vous avez créées et des ressources dépendantes de chaque connexion.

> [!IMPORTANT]
> Les étapes mentionnées dans cet article s’appliquent au modèle de déploiement Resource Manager. Pour supprimer une passerelle VPN déployée à l'aide du modèle de déploiement classique, suivez les étapes de l'article [Supprimer une passerelle : classique](vpn-gateway-delete-vnet-gateway-classic-powershell.md).

## <a name="delete-a-vpn-gateway"></a>Supprimer une passerelle VPN

Si vous souhaitez supprimer une passerelle de réseau virtuel, vous devez d’abord supprimer chaque ressource liée à la passerelle de réseau virtuel. Les ressources doivent être supprimées dans un certain ordre en raison des dépendances.

[!INCLUDE [delete gateway](../../includes/vpn-gateway-delete-vnet-gateway-portal-include.md)]

À ce stade, la passerelle de réseau virtuel est supprimée.

### <a name="to-delete-additional-resources"></a>Pour supprimer d'autres ressources

Les étapes suivantes vous permettent de supprimer toutes les ressources qui ne sont plus utilisées.

#### <a name="to-delete-the-local-network-gateway"></a>Pour supprimer la passerelle de réseau local

1. Sous **Toutes les ressources** , recherchez les passerelles de réseau local associées à chaque connexion.
1. Dans le panneau **Vue d’ensemble** de la passerelle de réseau local, cliquez sur **Supprimer** .

#### <a name="to-delete-the-public-ip-address-resource-for-the-gateway"></a>Pour supprimer la ressource de l’adresse IP publique de la passerelle

1. Sous **Toutes les ressources** , recherchez la ressource d’adresse IP publique qui a été associée à la passerelle. Si la passerelle de réseau virtuel était active-active, vous verrez deux adresses IP publiques.
1. Sur la page **Vue d’ensemble** de l’adresse IP publique, cliquez sur **Supprimer** , puis sur **Oui** pour confirmer.

#### <a name="to-delete-the-gateway-subnet"></a>Pour supprimer le sous-réseau de la passerelle

1. Sous **Toutes les ressources** , recherchez le réseau virtuel. 
1. Dans le panneau **Sous-réseaux** , cliquez sur **Sous-réseau de passerelle** , puis cliquez sur **Supprimer** . 
1. Cliquez sur **Oui** pour confirmer que vous souhaitez supprimer le sous-réseau de passerelle.

## <a name="delete-a-vpn-gateway-by-deleting-the-resource-group"></a><a name="deleterg"></a>Supprimer une passerelle VPN en supprimant le groupe de ressources

Si vous n’avez pas besoin de conserver de ressources dans le groupe de ressources et que vous voulez simplement recommencer à zéro, vous pouvez supprimer tout un groupe de ressources. Il s’agit d’un moyen rapide de tout supprimer. Les étapes suivantes s’appliquent uniquement au modèle de déploiement Resource Manager.

1. Sous **Toutes les ressources** , recherchez le groupe de ressources et cliquez pour ouvrir le panneau.
1. Cliquez sur **Supprimer** . Dans le panneau Supprimer, affichez les ressources affectées. Vérifiez que vous voulez supprimer toutes ces ressources. Si ce n'est pas le cas, utilisez les étapes disponibles sous Supprimer une passerelle VPN en haut de cet article.
1. Pour continuer, saisissez le nom du groupe de ressources que vous souhaitez supprimer, puis cliquez sur **Supprimer** .
