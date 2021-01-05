---
title: 'Tutoriel : Azure ExpressRoute – Ajouter une passerelle à un réseau virtuel (portail Azure)'
description: Ce didacticiel vous guide tout au long de l’ajout d’une passerelle de réseau virtuel à un réseau virtuel pour ExpressRoute à l’aide du portail Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/05/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 843d0b8cfd75e8cbdf45ac535cc9486aa42442d6
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "91761787"
---
# <a name="tutorial-configure-a-virtual-network-gateway-for-expressroute-using-the-azure-portal"></a>Tutoriel : Configurer une passerelle de réseau virtuel pour ExpressRoute à l’aide du portail Azure
> [!div class="op_single_selector"]
> * [Resource Manager - Portail Azure](expressroute-howto-add-gateway-portal-resource-manager.md)
> * [Resource Manager - PowerShell](expressroute-howto-add-gateway-resource-manager.md)
> * [Classic - PowerShell](expressroute-howto-add-gateway-classic.md)
> * [Vidéo - portail Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network)
> 

Ce tutoriel vous guide dans la procédure d’ajout de passerelle de réseau virtuel pour un réseau virtuel existant. Cet article vous montre les étapes nécessaires pour ajouter, redimensionner et supprimer une passerelle de réseau virtuel pour un réseau virtuel existant. Les étapes de cette configuration sont spécifiquement adaptées aux réseaux virtuels qui ont été créés à l’aide du modèle de déploiement Resource Manager qui sera utilisé dans une configuration ExpressRoute. Pour plus d’informations sur les passerelles de réseau virtuel et les paramètres de configuration de passerelle pour ExpressRoute, consultez [À propos des passerelles de réseau virtuel pour ExpressRoute](expressroute-about-virtual-network-gateways.md). 

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - créez un sous-réseau de passerelle ;
> - créer une passerelle de réseau virtuel.

## <a name="prerequisites"></a>Prérequis

Les étapes de cette tâche utilisent un réseau virtuel basé sur les valeurs figurant dans la liste de référence de configuration suivante. Nous utilisons cette liste dans notre exemple de procédure. Vous pouvez copier la liste et l’utiliser en tant que référence, en remplaçant les valeurs par les vôtres.

**Liste de référence de configuration**

* Nom du réseau virtuel : « TestVNet »
* Espace d’adressage du réseau virtuel : 192.168.0.0/16
* Nom du sous-réseau : « FrontEnd » 
    * Espace d’adressage du sous-réseau : « 192.168.1.0/24 »
* Groupe de ressources : « TestRG »
* Emplacement = « USA Est »
* Nom de sous-réseau de passerelle : « GatewaySubnet » Vous devez toujours nommer un sous-réseau de passerelle *GatewaySubnet*.
    * Espace d'adressage du sous-réseau de passerelle : « 192.168.200.0/26 »
* Nom de la passerelle : « ERGW »
* Nom d’adresse IP publique de passerelle = « MyERGWVIP »
* Type de passerelle : « ExpressRoute ». Ce type est requis pour une configuration ExpressRoute.

Vous pouvez afficher une [vidéo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network) de ces étapes avant de commencer votre configuration.

## <a name="create-the-gateway-subnet"></a>Créer le sous-réseau de passerelle

1. Dans le [portail](https://portal.azure.com), accédez au réseau virtuel Resource Manager pour lequel vous souhaitez créer une passerelle de réseau virtuel.
1. Dans la section **Paramètres** de votre réseau virtuel, sélectionnez **Sous-réseaux** pour développer les paramètres du Sous-réseau.
1. Dans les paramètres **Sous-réseaux**, sélectionnez **+ Sous-réseau de la passerelle** pour ajouter un sous-réseau de passerelle. 
   
    :::image type="content" source="./media/expressroute-howto-add-gateway-portal-resource-manager/add-gateway-subnet.png" alt-text="Ajouter le sous-réseau de passerelle":::

1. Le **Nom** de votre sous-réseau est automatiquement rempli avec la valeur « GatewaySubnet ». Cette valeur est nécessaire pour qu’Azure puisse reconnaître le sous-réseau en tant que sous-réseau de passerelle. Ajustez les valeurs de **Plage d’adresses** renseignées automatiquement pour qu’elles correspondent à la configuration requise. Nous vous recommandons de créer un sous-réseau de passerelle avec /27 ou plus (/26, /25, etc.). Ensuite, sélectionnez **OK** pour enregistrer les valeurs et créer le sous-réseau de passerelle.

    :::image type="content" source="./media/expressroute-howto-add-gateway-portal-resource-manager/add-subnet-gateway.png" alt-text="Ajout du sous-réseau":::

## <a name="create-the-virtual-network-gateway"></a>Créer la passerelle de réseau virtuel

1. Dans le portail, sur le côté gauche, sélectionnez **Créer une ressource**, puis entrez « Passerelle de réseau virtuel » dans la zone de recherche. Recherchez **passerelle de réseau virtuel** dans les résultats de la recherche et sélectionnez l’entrée. Dans la page **Passerelle de réseau virtuel**, sélectionnez **Créer**.
1. Dans la page **Créer une passerelle de réseau virtuel**, entrez ou sélectionnez les paramètres suivants :

    | Paramètre | Valeur |
    | --------| ----- |
    | Abonnement | Vérifiez que l’abonnement approprié est sélectionné. |
    | Groupe de ressources | Le groupe de ressources est choisi automatiquement une fois que vous avez sélectionné le réseau virtuel. | 
    | Nom | Nommez votre passerelle. Cela ne revient pas au même que de nommer un sous-réseau de passerelle. Il s’agit du nom de l’objet passerelle que vous créez.|
    | Région | Modifiez le champ **Région** de manière à ce qu’il pointe vers l’emplacement où se trouve votre réseau virtuel. Si l’emplacement ne pointe pas vers la région où se trouve votre réseau virtuel, le réseau virtuel n’apparaît pas dans la liste déroulante « Choisir un réseau virtuel ». |
    | Type de passerelle | Sélectionnez **ExpressRoute**.|
    | SKU | Sélectionnez la référence SKU de la passerelle dans la liste déroulante. |
    | Réseau virtuel | Sélectionnez *TestVNet*. |
    | Adresse IP publique | Sélectionnez **Créer nouveau**.|
    | Nom de l’adresse IP publique | Fournissez un nom pour l’adresse IP publique. |

1. Sélectionnez **Vérifier + créer**, puis **Créer** pour commencer à créer la passerelle. Les paramètres sont validés et la passerelle se déploie. La création d’une passerelle de réseau virtuel peut prendre jusqu’à 45 minutes.

    :::image type="content" source="./media/expressroute-howto-add-gateway-portal-resource-manager/gateway.png" alt-text="Champs de la page Créer une passerelle de réseau virtuel":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin de la passerelle ExpressRoute, localisez-la dans le groupe de ressources de réseau virtuel, puis sélectionnez **Supprimer**. Vérifiez que la passerelle n’est pas connectée à un circuit.

:::image type="content" source="./media/expressroute-howto-add-gateway-portal-resource-manager/delete-gateway.png" alt-text="Supprimer une passerelle de réseau virtuel":::

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez créé la passerelle de réseau virtuel, vous pouvez lier votre réseau virtuel à un circuit ExpressRoute. 

> [!div class="nextstepaction"]
> [Lier un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)
