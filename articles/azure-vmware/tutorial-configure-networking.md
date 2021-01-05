---
title: Tutoriel - Configurer le réseau pour votre cloud privé VMware dans Azure
description: Découvrir comment créer et configurer le réseau nécessaire au déploiement de votre cloud privé dans Azure
ms.topic: tutorial
ms.date: 09/21/2020
ms.openlocfilehash: 6aff39284f3ea786080055552ac001ac5dd7b394
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578346"
---
# <a name="tutorial-configure-networking-for-your-vmware-private-cloud-in-azure"></a>Tutoriel : Configurer le réseau pour votre cloud privé VMware dans Azure

Un cloud privé Azure VMware Solution nécessite un réseau virtuel Azure. Comme Azure VMware Solution ne prend pas en charge votre vCenter local, des étapes supplémentaires sont requises pour l’intégration avec votre environnement local. La configuration d’un circuit ExpressRoute et d’une passerelle de réseau virtuel est également nécessaire.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez un réseau virtuel
> * Créer une passerelle de réseau virtuel
> * Connecter votre circuit ExpressRoute à la passerelle
> * Localiser les URL pour vCenter et NSX Manager

## <a name="prerequisites"></a>Prérequis 
Un réseau virtuel que vous avez créé dans un [cloud privé Azure VMware Solution](tutorial-create-private-cloud.md). 

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Accédez au groupe de ressources que vous avez créé dans le cadre du [tutoriel Créer un cloud privé](tutorial-create-private-cloud.md), puis sélectionnez **+ Ajouter** pour définir une nouvelle ressource. 

1. Dans la zone de texte **Rechercher dans la Place de marché**, tapez **Réseau virtuel**. Recherchez la ressource Réseau virtuel et sélectionnez-la.

1. Dans la page **Réseau virtuel**, sélectionnez **Créer** pour configurer votre réseau virtuel pour votre cloud privé.

1. Dans la page **Créer un réseau virtuel**, entrez les détails de votre réseau virtuel.

1. Sous l’onglet **De base**, entrez un nom pour le réseau virtuel et sélectionnez la région appropriée, puis sélectionnez **Suivant : Adresses IP**.

1. Sous l’onglet **Adresses IP**, sous **Espace d’adressage IPv4**, entrez l’espace d’adressage que vous avez créé dans le tutoriel précédent.

   > [!IMPORTANT]
   > Vous devez utiliser un espace d’adressage qui **ne chevauche pas** l’espace d’adressage que vous avez utilisé lors de la création de votre cloud privé dans le tutoriel précédent.

1. Sélectionnez **+ Ajouter un sous-réseau** puis, dans la page **Ajouter un sous-réseau**, spécifiez un nom et une plage d’adresses appropriée pour le sous-réseau. Quand vous avez terminé, sélectionnez **Ajouter**.

1. Sélectionnez **Revoir + créer**.

   :::image type="content" source="./media/tutorial-configure-networking/create-virtual-network.png" alt-text="Sélectionnez Vérifier + créer." border="true":::

1. Vérifiez les informations et sélectionnez **Créer**. Une fois le déploiement terminé, vous voyez votre réseau virtuel dans le groupe de ressources.

## <a name="create-a-virtual-network-gateway"></a>Créer une passerelle de réseau virtuel

À présent que vous avez créé un réseau virtuel, vous allez créer une passerelle de réseau virtuel.

1. Dans votre groupe de ressources, sélectionnez **+ Ajouter** pour ajouter une nouvelle ressource.

1. Dans la zone de texte **Rechercher dans la Place de marché**, tapez **Passerelle de réseau virtuel**. Recherchez la ressource Réseau virtuel et sélectionnez-la.

1. Dans la page **Passerelle de réseau virtuel**, sélectionnez **Créer**.

1. Sous l’onglet De base de la page **Créer une passerelle de réseau virtuel**, spécifiez des valeurs pour les champs, puis sélectionnez **Vérifier + créer**. 

   | Champ | Valeur |
   | --- | --- |
   | **Abonnement** | Une valeur déjà renseignée avec l’abonnement auquel appartient le groupe de ressources. |
   | **Groupe de ressources** | Une valeur déjà renseignée pour le groupe de ressources actuel. La valeur doit être le groupe de ressources que vous avez créé dans un test précédent. |
   | **Nom** | Entrez un nom unique pour la passerelle de réseau virtuel. |
   | **Région** | Sélectionnez l’emplacement géographique de la passerelle de réseau virtuel. |
   | **Type de passerelle** | Sélectionnez **ExpressRoute**. |
   | **Référence (SKU)** | Laissez la valeur par défaut : **Standard**. |
   | **Réseau virtuel** | Sélectionnez le réseau virtuel que vous avez créé précédemment. Si vous ne voyez pas le réseau virtuel, vérifiez que la région de la passerelle correspond à la région de votre réseau virtuel. |
   | **Plage d’adresses du sous-réseau de passerelle** | Cette valeur est renseignée quand vous sélectionnez le réseau virtuel. Ne modifiez pas la valeur par défaut. |
   | **Adresse IP publique** | Sélectionnez **Créer nouveau**. |

   :::image type="content" source="./media/tutorial-configure-networking/create-virtual-network-gateway.png" alt-text="Sélectionnez Vérifier + créer." border="true":::

1. Vérifiez que les détails sont corrects, puis sélectionnez **Créer** pour démarrer le déploiement de votre passerelle de réseau virtuel. 
1. Une fois le déploiement terminé, passez à la section suivante pour connecter votre connexion ExpressRoute à la passerelle de réseau virtuel contenant votre cloud privé Azure VMware Solution.

## <a name="connect-expressroute-to-the-virtual-network-gateway"></a>Connecter ExpressRoute à la passerelle de réseau virtuel

À présent que vous avez déployé une passerelle de réseau virtuel, vous allez ajouter une connexion entre celle-ci et votre cloud privé Azure VMware Solution.

[!INCLUDE [connect-expressroute-to-vnet](includes/connect-expressroute-vnet.md)]


## <a name="locate-the-urls-for-vcenter-and-nsx-manager"></a>Localiser les URL pour vCenter et NSX Manager

Pour vous connecter à vCenter et à NSX Manager, vous avez besoin des URL du client web vCenter et du site NSX-T Manager. 

Accédez à votre cloud privé Azure VMware Solution, puis, sous **Gérer**, sélectionnez **Identité** ; vous y trouverez les informations requises.

:::image type="content" source="./media/tutorial-configure-networking/locate-urls.png" alt-text="Sélectionnez Vérifier + créer." border="true":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créez un réseau virtuel
> * Créer une passerelle de réseau virtuel
> * Connecter votre circuit ExpressRoute à la passerelle
> * Localiser les URL pour vCenter et NSX Manager

Passez au didacticiel suivant pour apprendre à créer les segments de réseau NSX-T utilisés pour les machines virtuelles dans vCenter.

> [!div class="nextstepaction"]
> [Créer un segment réseau NSX-T](tutorial-nsx-t-network-segment.md)