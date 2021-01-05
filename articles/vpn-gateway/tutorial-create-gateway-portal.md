---
title: 'Tutoriel : Créer et gérer une passerelle VPN : Portail Azure'
description: Suivez ce tutoriel pour découvrir comment créer, déployer et gérer une passerelle VPN Azure à l’aide du portail.
author: cherylmc
ms.author: cherylmc
ms.service: vpn-gateway
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: 933b71d75eacdca015a38524870f25a345e76d22
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746163"
---
# <a name="tutorial-create-and-manage-a-vpn-gateway-using-azure-portal"></a>Tutoriel : Créer et gérer une passerelle VPN à l’aide du portail Azure

Les passerelles VPN Azure fournissent une connectivité entre les locaux du client et Azure. Ce tutoriel décrit les éléments de base du déploiement d’une passerelle VPN Azure, notamment la création et la gestion d’une passerelle VPN. Vous pouvez également créer une passerelle à l’aide d’[Azure CLI](create-routebased-vpn-gateway-cli.md) ou d’[Azure PowerShell](create-routebased-vpn-gateway-powershell.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez un réseau virtuel
> * Créer une passerelle VPN
> * Afficher l’adresse IP publique de la passerelle
> * Redimensionner une passerelle VPN (redimensionner la référence SKU)
> * Réinitialiser une passerelle VPN

Le diagramme suivant illustre le réseau virtuel et la passerelle VPN créés dans le cadre de ce tutoriel.

:::image type="content" source="./media/tutorial-create-gateway-portal/gateway-diagram.png" alt-text="Diagramme de réseau virtuel et passerelle VPN":::

## <a name="prerequisites"></a>Prérequis

Compte Azure avec un abonnement actif. Si vous n’en avez pas, [créez-en une gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

## <a name="create-a-virtual-network"></a><a name="CreatVNet"></a>Créer un réseau virtuel

Créez un réseau virtuel à l’aide des valeurs suivantes :

* **Groupe de ressources :** TestRG1
* **Nom :** VNet1
* **Région :** (États-Unis) USA Est
* **Espace d’adressage IPv4 :** 10.1.0.0/16
* **Nom du sous-réseau** : FrontEnd
* **Espace d’adressage du sous-réseau :** 10.1.0.0/24

[!INCLUDE [Create a virtual network](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="create-a-vpn-gateway"></a><a name="VNetGateway"></a>Créer une passerelle VPN

Dans cette étape, vous créez la passerelle de réseau virtuel de votre réseau virtuel. La création d’une passerelle nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle sélectionnée.

Créez une passerelle de réseau virtuel à l’aide des valeurs suivantes :

* **Nom :** VNet1GW
* **Région :** USA Est
* **Type de passerelle :** VPN
* **Type de VPN :** basé sur la route
* **SKU :** VpnGw1
* **Génération :** Génération1
* **Réseau virtuel :** VNet1
* **Plage d’adresses du sous-réseau de passerelle :** 10.1.255.0/27
* **Adresse IP publique :** Création
* **Nom de l'adresse IP publique :** VNet1GWpip
* **Activer le mode actif/actif :** Désactivé
* **Configurer BGP :** Désactivé

[!INCLUDE [Create a vpn gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

[!INCLUDE [NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="view-the-public-ip-address"></a><a name="view"></a>Afficher l’adresse IP publique

Vous pouvez voir l’adresse IP publique de la passerelle dans la page **Vue d’ensemble** de votre passerelle.

:::image type="content" source="./media/tutorial-create-gateway-portal/address.png" alt-text="Page de présentation":::

Pour voir d’autres informations sur l’objet d’adresse IP publique, cliquez sur le lien de nom/adresse IP à côté de **Adresse IP publique**.

## <a name="resize-a-gateway-sku"></a><a name="resize"></a>Redimensionner une référence SKU de passerelle

Il existe des règles spécifiques relatives au redimensionnement et à la modification d’une référence SKU de passerelle. Dans cette section, nous allons redimensionner la référence SKU. Pour plus d’informations, consultez [Paramètres de la passerelle : Redimensionnement ou modification d’une référence SKU](vpn-gateway-about-vpn-gateway-settings.md#resizechange).

[!INCLUDE [resize a gateway](../../includes/vpn-gateway-resize-gw-portal-include.md)]

## <a name="reset-a-gateway"></a><a name="reset"></a>Réinitialiser une passerelle

[!INCLUDE [reset a gateway](../../includes/vpn-gateway-reset-gw-portal-include.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’envisagez pas de continuer à utiliser cette application ni de passer au tutoriel suivant, supprimez ces ressources en effectuant les étapes suivantes :

1. Entrez le nom de votre groupe de ressources dans la zone **Rechercher** en haut du portail et sélectionnez-le dans les résultats de la recherche.

1. Sélectionnez **Supprimer le groupe de ressources**.

1. Entrez votre groupe de ressources dans **TAPER LE NOM DU GROUPE DE RESSOURCES**, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous disposez d’une passerelle VPN, vous pouvez configurer des connexions. Les articles ci-dessous vous aideront à créer quelques-unes des configurations les plus courantes :

> [!div class="nextstepaction"]
> [Connexions VPN de site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md)

> [!div class="nextstepaction"]
> [Connexions VPN de point à site](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
