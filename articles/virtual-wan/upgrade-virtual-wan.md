---
title: Mettre à niveau un réseau Azure Virtual WAN de base vers le type Standard - Portail Azure | Microsoft Docs
description: Vous pouvez mettre à niveau le type de votre réseau étendu virtuel pour bénéficier de fonctionnalités supplémentaires.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 11/04/2019
ms.author: cherylmc
ms.openlocfilehash: 6290d89ed0ee539b4df4c2c8bc9070097da98c81
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91447359"
---
# <a name="upgrade-a-virtual-wan-from-basic-to-standard"></a>Mettre à niveau un réseau étendu virtuel de base vers le type Standard

Cet article a pour but de vous aider à mettre à niveau un réseau étendu de base vers le type Standard. Un type de réseau étendu « De base » applique à tous les hubs qu'il héberge la référence SKU De base. Dans un hub De base, vous êtes limité à la fonctionnalité VPN site à site. Un type de réseau étendu « Standard » applique à tous les hubs qu'il héberge la référence SKU Standard. Lorsque vous utilisez des hubs Standard, vous pouvez activer ExpressRoute, un VPN utilisateur (point à site), un hub à maillage complet et le transit de réseau virtuel à réseau virtuel via les hubs Azure.

Le tableau suivant présente les configurations disponibles pour chaque type de réseau étendu :

[!INCLUDE [Basic and Standard SKUs](../../includes/virtual-wan-standard-basic-include.md)]

## <a name="to-change-the-virtual-wan-type"></a>Pour modifier le type de réseau étendu virtuel

1. Sur la page de votre réseau étendu virtuel, sélectionnez **Configuration** pour ouvrir la page Configuration.

   ![Capture d'écran représentant la page « Configuration » avec, en bas, une zone de texte en surbrillance contenant des informations sur la mise à niveau vers un réseau étendu virtuel de type standard.](./media/upgrade-virtual-wan/1.png)
2. Pour le type de réseau étendu virtuel, sélectionnez **Standard** dans la liste déroulante.

   ![Capture d'écran représentant le menu déroulant « Type de réseau étendu virtuel ».](./media/upgrade-virtual-wan/2.png)
3. Sachez que si vous procédez à une mise à niveau vers un réseau étendu virtuel Standard, vous ne pourrez pas revenir à un réseau étendu virtuel De base. Sélectionnez **Confirmer** si vous souhaitez procéder à la mise à niveau.

   ![Capture d'écran représentant la boîte de dialogue « Confirmation de la mise à jour ».](./media/upgrade-virtual-wan/4.png)
4. Une fois la modification enregistrée, la page de votre réseau étendu virtuel est semblable à l'exemple suivant.

   ![Diagramme WAN virtuel](./media/upgrade-virtual-wan/5.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le WAN virtuel, consultez la page [Vue d'ensemble de WAN virtuel](virtual-wan-about.md).