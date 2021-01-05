---
title: 'Tutoriel : Utiliser Azure Virtual WAN pour créer une connexion de point à site vers Azure'
description: Dans ce didacticiel, découvrez comment utiliser Azure Virtual WAN pour créer une connexion VPN point à site vers Azure.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 11/09/2020
ms.author: cherylmc
ms.openlocfilehash: e7e65d5d2941765df98b3bf3b7fb8ff2e89b7e9f
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94411199"
---
# <a name="tutorial-create-a-user-vpn-connection-using-azure-virtual-wan"></a>Tutoriel : Créer une connexion de VPN utilisateur à l’aide d’Azure Virtual WAN

Ce didacticiel vous montre comment utiliser Azure Virtual WAN pour vous connecter à vos ressources dans Azure via une connexion VPN IPsec/IKE (IKEv2) ou OpenVPN. Pour utiliser ce type de connexion, le client VPN doit être configuré sur l’ordinateur client. Pour plus d’informations sur Azure Virtual WAN, consultez l’article [Vue d’ensemble d’Azure Virtual WAN](virtual-wan-about.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un WAN virtuel
> * Créer une configuration P2S
> * Créer un hub virtuel
> * Spécifier les serveurs DNS
> * Générer un package de configuration pour le profil du client VPN
> * Configurer les clients VPN
> * Afficher votre WAN virtuel

![Diagramme WAN virtuel](./media/virtual-wan-about/virtualwanp2s.png)

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [Before beginning](../../includes/virtual-wan-before-include.md)]

## <a name="create-a-virtual-wan"></a><a name="wan"></a>Créer un WAN virtuel

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

## <a name="create-a-p2s-configuration"></a><a name="p2sconfig"></a>Créer une configuration P2S

Une configuration P2S définit les paramètres permettant de connecter des clients distants.Une configuration P2S (point à site) définit les paramètres permettant de connecter des clients distants.

[!INCLUDE [Create P2S configuration](../../includes/virtual-wan-p2s-configuration-include.md)]

## <a name="create-virtual-hub-and-gateway"></a><a name="hub"></a>Créer un hub virtuel et une passerelle

[!INCLUDE [Create hub](../../includes/virtual-wan-p2s-hub-include.md)]

## <a name="specify-dns-server"></a><a name="dns"></a>Spécifier un serveur DNS

Vous pouvez configurer ce paramètre lors de la création du hub, ou le modifier ultérieurement. Pour le modifier, localisez le hub virtuel. Sous **VPN utilisateur (point à site)** , sélectionnez **Configurer**, puis entrez les adresses IP de serveur DNS dans les zones de texte **Serveurs DNS personnalisés**. Vous pouvez spécifier jusqu’à 5 serveurs DNS.

   :::image type="content" source="media/virtual-wan-point-to-site-portal/custom-dns.png" alt-text="DNS personnalisé" lightbox="media/virtual-wan-point-to-site-portal/custom-dns-expand.png":::

## <a name="generate-vpn-client-profile-package"></a><a name="download"></a>Générer le package de profil du client VPN

Générez et téléchargez le package de profil du client VPN pour configurer vos clients VPN.

[!INCLUDE [Download profile](../../includes/virtual-wan-p2s-download-profile-include.md)]

## <a name="configure-vpn-clients"></a><a name="configure-client"></a>Configurer les clients VPN

Pour configurer les clients VPN avec accès à distance, utilisez le package de profil téléchargé. La procédure est différente pour chaque système d’exploitation. Suivez les instructions qui s’appliquent à votre système.
Une fois que vous avez terminé de configurer votre client, vous pouvez vous connecter.

[!INCLUDE [Configure clients](../../includes/virtual-wan-p2s-configure-clients-include.md)]

## <a name="view-your-virtual-wan"></a><a name="viewwan"></a>Afficher votre WAN virtuel

1. Accédez au WAN virtuel.
1. Sur la page **Vue d’ensemble**, chaque point sur la carte représente un hub.
1. Dans la section **Hubs et connexions**, vous pouvez voir l’état du hub, le site, la région, l’état de la connexion VPN et les octets entrés et sortis.

## <a name="clean-up-resources"></a><a name="cleanup"></a>Supprimer des ressources

Quand vous n’avez plus besoin de ces ressources, vous pouvez utiliser [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) pour supprimer le groupe de ressources et toutes les ressources qu’il contient. Remplacez « myResourceGroup » par le nom de votre groupe de ressources et exécutez la commande PowerShell suivante :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Ensuite, pour en savoir plus sur Virtual WAN, consultez :

> [!div class="nextstepaction"]
> * [FAQ sur Virtual WAN](virtual-wan-faq.md)
