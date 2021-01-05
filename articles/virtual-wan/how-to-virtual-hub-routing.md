---
title: Guide pratique pour configurer le routage de hub virtuel
titleSuffix: Azure Virtual WAN
description: Cet article explique comment configurer le routage de hub virtuel.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: cae74a5f4859d208765c9a7e5cde05ff3c0c3096
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91313686"
---
# <a name="how-to-configure-virtual-hub-routing"></a>Guide pratique pour configurer le routage de hub virtuel

Un hub virtuel peut contenir plusieurs passerelles, comme une passerelle VPN site à site, une passerelle ExpressRoute, une passerelle point à site ou un pare-feu Azure. Les fonctionnalités de routage du hub virtuel sont fournies par un routeur qui gère tout le routage, notamment le routage de transit, entre les passerelles à l’aide du protocole BGP (Border Gateway Protocol). Ce routeur fournit également une connectivité de transit entre les réseaux virtuels qui se connectent à un hub virtuel, et peut prendre en charge un débit agrégé de 50 Gbits/s. Ces fonctionnalités de routage s’appliquent aux clients Virtual WAN Standard.

Pour plus d’informations, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="create-a-route-table"></a><a name="create-table"></a>Créer une table de routage

1. Dans le portail Azure, accédez au hub virtuel.
2. Sous **Connectivité**, sélectionnez **Routage**. Dans la page Routage figurent les tables de routage **Par défaut** et **Aucune**.

   :::image type="content" source="./media/how-to-virtual-hub-routing/routing.png" alt-text="Page Routage":::
3. Sélectionnez **+ Créer une table de routage** pour ouvrir la page **Créer une table de routage**.
4. Sous l’onglet **Informations de base** de la page Créer une table de routage, renseignez les champs suivants.

   :::image type="content" source="./media/how-to-virtual-hub-routing/basics.png" alt-text="Page Routage":::

   * **Nom**
   * **Itinéraires**
   * **Nom de l’itinéraire**
   * **Type de destination**
   * **Préfixe de destination** : vous pouvez agréger des préfixes. Par exemple : VNet 1 : 10.1.0.0/24 et VNet 2 : 10.1.1.0/24 peuvent être agrégés sous la forme 10.1.0.0/16. Les routes de **branche** s’appliquent à tous les sites VPN connectés, aux circuits ExpressRoute et aux connexions VPN utilisateur.
   * **Tronçon suivant** : liste des connexions de réseau virtuel, ou Pare-feu Azure.

     Si vous sélectionnez une connexion de réseau virtuel, vous verrez **Configurer des routes statiques**. Il s’agit d’un paramètre de configuration facultatif. Pour plus d’informations, consultez [Configurer des routes statiques](about-virtual-hub-routing.md#static).

      :::image type="content" source="./media/how-to-virtual-hub-routing/next-hop.png" alt-text="Page Routage":::

5. Sélectionnez l’onglet **Étiquettes** pour configurer les noms des étiquettes. Les étiquettes fournissent un mécanisme permettant de regrouper logiquement des tables de routage.

    :::image type="content" source="./media/how-to-virtual-hub-routing/labels.png" alt-text="Page Routage":::

6. Sélectionnez l’onglet **Associations** pour associer des connexions à la table de routage.
Vous verrez **Branches**, **Réseaux virtuels** et les **Paramètres actuels** des connexions.

    :::image type="content" source="./media/how-to-virtual-hub-routing/associations.png" alt-text="Page Routage":::

7. Sélectionnez l’onglet **Propagations** pour propager les routes des connexions à la table de routage.

    :::image type="content" source="./media/how-to-virtual-hub-routing/propagations.png" alt-text="Page Routage":::

8. Sélectionnez **Créer** pour créer la table de routage.

## <a name="to-edit-a-route-table"></a><a name="edit-table"></a>Pour modifier une table de routage

Dans le portail Azure, recherchez la table de routage de votre hub virtuel. Sélectionnez la table de routage pour modifier des informations.

## <a name="to-delete-a-route-table"></a><a name="delete-table"></a>Pour supprimer une table de routage

Dans le portail Azure, recherchez la table de routage de votre hub virtuel. Vous ne pouvez pas supprimer une table de routage Par défaut ou Aucune. En revanche, vous pouvez supprimer toutes les tables de routage personnalisées. Cliquez sur **« … »** , puis sélectionnez **Supprimer**.

## <a name="to-view-effective-routes"></a><a name="view-routes"></a>Pour afficher les routes effectives

Dans le portail Azure, recherchez la table de routage de votre hub virtuel. Cliquez sur **« ... »** et sélectionnez **Routages effectifs** pour voir les routes apprises par la table de routage sélectionnée. Les routes propagées de la connexion à la table de routage sont renseignées automatiquement dans la liste **Routages effectifs** de la table de routage. Pour plus d’informations, consultez [À propos des routes effectives](effective-routes-virtual-hub.md).

:::image type="content" source="./media/how-to-virtual-hub-routing/effective.png" alt-text="Page Routage" lightbox="./media/how-to-virtual-hub-routing/effective-expand.png":::

## <a name="to-set-up-routing-configuration-for-a-virtual-network-connection"></a><a name="routing-configuration"></a>Pour définir la configuration du routage pour une connexion de réseau virtuel

1. Dans le portail Azure, accédez à votre réseau étendu virtuel et, sous **Connectivité**, sélectionnez **Connexions de réseau virtuel**.
1. Sélectionnez **+Ajouter une connexion**.
1. Sélectionnez le réseau virtuel dans la liste déroulante.
1. Définissez la configuration du routage à associer à une table de routage. Pour **Associer une table de routage**, sélectionnez la table de routage dans la liste déroulante.
1. Définissez la configuration du routage pour qu’elle se propage à une ou plusieurs tables de routage. Pour **Propager à la table de routage**, effectuez une sélection dans la liste déroulante.
1. Pour **Itinéraires statiques**, configurez des itinéraires statiques pour l’appliance virtuelle réseau (le cas échéant). Le réseau étendu virtuel prend en charge une adresse IP de tronçon suivant unique pour l’itinéraire statique dans une connexion de réseau virtuel. Par exemple, si vous avez une appliance virtuelle distincte pour les flux de trafic entrants et sortants, il serait préférable que les appliances virtuelles se trouvent sur des réseaux virtuels distincts et que ceux-ci soient attachés au hub virtuel.


:::image type="content" source="./media/how-to-virtual-hub-routing/routing-configuration.png" alt-text="Page Routage" lightbox="./media/how-to-virtual-hub-routing/routing-configuration-expand.png":::

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).
Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).
