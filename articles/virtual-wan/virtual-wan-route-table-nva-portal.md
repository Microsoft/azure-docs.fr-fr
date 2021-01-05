---
title: 'Virtual WAN : Créer une table de routes de hub virtuel sur des appliances réseau virtuelles : Portail Azure'
description: Table de routage de hub virtuel Azure Virtual WAN pour diriger le trafic vers une appliance réseau virtuelle à l’aide du portail.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to create a route table using the portal.
ms.openlocfilehash: 8de7ad2808e5319819410b3125472e28496647b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91267140"
---
# <a name="create-a-virtual-wan-hub-route-table-for-nvas-azure-portal"></a>Créer une table de routage de hub Virtual WAN pour des appliances réseau virtuelles : Portail Azure

Cet article explique comment diriger le trafic entre une branche (site local) connectée au hub Virtual WAN et un réseau virtuel spoke via une appliance virtuelle réseau.

![Diagramme WAN virtuel](./media/virtual-wan-route-table-nva/vwanroute.png)

## <a name="before-you-begin"></a>Avant de commencer

Vérifiez que vous respectez les critères suivants :

*  Vous avez une appliance virtuelle réseau (NVA). Une appliance réseau virtuelle est un logiciel tiers de votre choix qui est généralement provisionné à partir de la Place de marché Azure dans un réseau virtuel.

    * Une adresse IP privée doit être attribuée à l’interface réseau de l’appliance réseau virtuelle.

    * L’appliance réseau virtuelle n’est pas déployée dans le hub virtuel. Elle doit être déployée dans un réseau virtuel distinct.

    *  Un ou plusieurs réseaux virtuels peuvent être connectés au réseau virtuel de l'appliance réseau virtuelle. Dans cet article, nous désignons le réseau virtuel de l'appliance réseau virtuelle par le terme « réseau virtuel spoke indirect ». Ces réseaux virtuels peuvent être connectés au réseau virtuel de l'appliance réseau virtuelle à l'aide de VNet Peering. Les liens VNet Peering sont représentés par des flèches noires dans la figure ci-dessus, entre le réseau virtuel 1, le réseau virtuel 2 et le réseau virtuel d'appliance virtuelle réseau.
*  Vous avez créé deux réseaux virtuels. Ils seront utilisés en tant que réseaux virtuels spoke.

    * Les espaces d'adressage des réseaux virtuels spoke sont les suivants : VNet1 : 10.0.2.0/24 et VNet2 : 10.0.3.0/24. Pour plus d'informations sur la création d'un réseau virtuel, consultez [Créer un réseau virtuel](../virtual-network/quick-create-portal.md).

    * Assurez-vous qu’aucune passerelle de réseau virtuel n’existe dans les réseaux virtuels.

    * Les réseaux virtuels n'ont pas besoin de sous-réseau de passerelle.

## <a name="1-sign-in"></a><a name="signin"></a>1. Se connecter

Dans un navigateur, accédez au [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

## <a name="2-create-a-virtual-wan"></a><a name="vwan"></a>2. Créer un WAN virtuel

Créez un WAN virtuel. Utilisez les exemples de valeurs suivants :

* **Nom du WAN virtuel :** monWANvirtuel
* **Groupe de ressources :** testRG
* **Emplacement :** USA Ouest

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-vwan-include.md)]

## <a name="3-create-a-hub"></a><a name="hub"></a>3. Créer un hub

Créez le hub. Utilisez les exemples de valeurs suivants :

* **Emplacement :** USA Ouest
* **Nom :** westushub
* **Espace d’adressage privé du hub :** 10.0.1.0/24

[!INCLUDE [Create a hub](../../includes/virtual-wan-tutorial-hub-include.md)]

## <a name="4-create-and-apply-a-hub-route-table"></a><a name="route"></a>4. Créer et implémenter une table de routage de hub

Mettez à jour le hub avec une table de routage de hub. Utilisez les exemples de valeurs suivants :

* **Espaces d'adressage du réseau virtuel Spoke :** (VNet1 et VNet2) 10.0.2.0/24 et 10.0.3.0/24
* **Adresses IP privées pour l’interface réseau de l’appliance réseau virtuelle DMZ :** 10.0.4.5

1. Accédez à votre WAN virtuel.
2. Cliquez sur le hub pour lequel vous souhaitez créer une table de routage.
3. Cliquez sur **...** , puis cliquez sur **Modifier le hub virtuel**.
4. Dans la page **Modifier le hub virtuel**, faites défiler le contenu vers le bas et cochez la case **Utiliser la table pour le routage**.
5. Dans la colonne **Si le préfixe de destination est**, ajoutez les espaces d’adressage. Dans la colonne **Envoyer à l'adresse du tronçon suivant**, ajoutez l’adresse IP privée pour l’interface réseau de l’appliance réseau virtuelle DMZ.

   > [!NOTE]
   > Le réseau de l’appliance réseau virtuelle DMZ est applicable au hub local.
   
6. Cliquez sur **Confirmer** pour mettre à jour la ressource hub avec les paramètres de la table de routage.

## <a name="5-create-the-vnet-connections"></a><a name="connections"></a>5. Créer les connexions de réseau virtuel

Créez une connexion de réseau virtuel au hub à partir de chaque réseau virtuel spoke indirect (VNet1 et VNet2). Ces connexions de réseau virtuel sont représentées par les flèches bleues dans la figure ci-dessus. Créez ensuite une connexion de réseau virtuel au hub à partir du réseau virtuel d'appliance virtuelle réseau (flèche noire dans la figure).

 Pour cette étape, vous pouvez utiliser les valeurs suivantes :

| Nom du réseau virtuel| Nom de la connexion|
| --- | --- |
| VNet1 | testconnexion1 |
| VNet2 | testconnexion2 |
| NVAVNet | testconnexion3 |

Répétez la procédure suivante pour chaque réseau virtuel que vous souhaitez connecter.

1. Sur la page de votre WAN virtuel, cliquez sur **Connexion de réseau virtuel**.
2. Dans la page de connexion de réseau virtuel, cliquez sur **+ Ajouter une connexion**.
3. Dans la page **Ajouter une connexion**, renseignez les champs suivants :

    * **Nom de connexion** : nommez votre connexion.
    * **Hubs** : sélectionnez le hub que vous souhaitez associer à cette connexion.
    * **Abonnement** : vérifiez l’abonnement.
    * **Réseau virtuel** : sélectionnez le réseau virtuel que vous souhaitez connecter à ce hub. Le réseau virtuel ne peut pas avoir une passerelle de réseau virtuel déjà existante.
4. Cliquez sur **OK** pour créer la connexion.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Virtual WAN, consultez la page [Vue d’ensemble de Virtual WAN](virtual-wan-about.md).
