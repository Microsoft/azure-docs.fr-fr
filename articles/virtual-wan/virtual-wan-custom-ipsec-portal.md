---
title: 'Configurer une stratégie IPsec personnalisée pour Azure Virtual WAN : Portail | Microsoft Docs'
description: Découvrez comment configurer une stratégie IPsec personnalisée pour Azure Virtual WAN à l’aide du portail.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 10/14/2019
ms.author: cherylmc
ms.openlocfilehash: 07eae453441ea8bff81d7cdb60f9c46c08a22829
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91851169"
---
# <a name="configure-a-custom-ipsec-policy-for-virtual-wan-using-the-portal"></a>Configurer une stratégie IPsec personnalisée pour Virtual WAN à l’aide du portail

Vous pouvez configurer une stratégie IPsec personnalisée pour une connexion VPN Virtual WAN dans le portail Azure. Les stratégies personnalisées sont utiles si vous souhaitez que les deux côtés (la passerelle locale et la passerelle VPN Azure) utilisent les mêmes paramètres pour IKE Phase 1 et IKE Phase 2.

## <a name="working-with-custom-policies"></a>Utilisation de stratégies personnalisées

[!INCLUDE [IPsec](../../includes/virtual-wan-ipsec-custom-include.md)]

## <a name="configure-a-policy"></a>Configurer une stratégie

1. **Localisez le hub virtuel**. Dans un navigateur, accédez au [portail Azure](https://aka.ms/azurevirtualwanpreviewfeatures) et connectez-vous avec votre compte Azure. Accédez à votre ressource Virtual WAN et localisez le hub virtuel auquel votre site VPN est connecté.
2. **Sélectionnez le site VPN**. Dans la page de présentation du hub, cliquez sur **VPN (site à site)** et sélectionnez le site VPN pour lequel vous souhaitez configurer une stratégie IPsec personnalisée.

   ![select](./media/virtual-wan-custom-ipsec-portal/locate.png)
3. **Modifiez la connexion VPN**. Dans le **Menu contextuel** **…** , sélectionnez **Modifier la connexion VPN**.

   ![modifier](./media/virtual-wan-custom-ipsec-portal/contextmenu.png)
4. **Configurez les paramètres**. Sur la page **Modifier la connexion VPN**, remplacez la valeur Par défaut du paramètre IPsec par Personnalisé et personnalisez la stratégie IPsec. Sélectionnez **Save** (Enregistrer) pour enregistrer vos paramètres.

   ![configurer et enregistrer](./media/virtual-wan-custom-ipsec-portal/edit.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Virtual WAN, consultez la page [Vue d’ensemble de Virtual WAN](virtual-wan-about.md).