---
title: Convertir un Peering direct hérité en ressource Azure à l’aide du portail Azure
titleSuffix: Azure
description: Convertir un Peering direct hérité en ressource Azure à l’aide du portail Azure
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 496774f034c861131624b552b1acefca9f399ea8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84700261"
---
# <a name="convert-a-legacy-direct-peering-to-an-azure-resource-by-using-the-azure-portal"></a>Convertir un Peering direct hérité en ressource Azure à l’aide du portail Azure

Cet article explique comment convertir un Peering direct hérité en ressource Azure à l’aide du Portail Azure.

Si vous préférez, vous pouvez suivre ce guide en utilisant [PowerShell](howto-legacy-direct-powershell.md).

## <a name="before-you-begin"></a>Avant de commencer
* Passez en revue les [prérequis](prerequisites.md) et la [procédure pas à pas du Peering direct](walkthrough-direct-all.md) avant de commencer la configuration.


## <a name="convert-a-legacy-direct-peering-to-an-azure-resource"></a>Convertir un Peering direct hérité en ressource Azure

### <a name="sign-in-to-the-portal-and-select-your-subscription"></a>Se connecter au portail et sélectionner votre abonnement
[!INCLUDE [Account](./includes/account-portal.md)]

### <a name="convert-a-legacy-direct-peering"></a><a name=create></a>Convertir un Peering direct hérité

En tant que fournisseur de services Internet, vous pouvez convertir des connexions Peering direct héritées à l’aide de l’option [Créer un Peering]( https://go.microsoft.com/fwlink/?linkid=2129593).

1. Dans la page **Créer un Peering**, sur l’onglet **Informations de base**, remplissez les cases comme illustré ici :

    > [!div class="mx-imgBorder"] 
    > ![Inscrire Peering Service](./media/setup-basics-tab.png)

*    Sélectionnez votre abonnement Azure.

* Pour le groupe de ressources, sélectionnez un groupe de ressources existant dans la liste déroulante ou créez un nouveau groupe en sélectionnant Créer nouveau. Dans cet exemple, ,nous allons créer un nouveau groupe de ressources.

* Nom correspond au nom de la ressource, et vous pouvez le choisir librement.

* Région est sélectionné automatiquement si vous avez choisi un groupe de ressources existant. Si vous avez choisi de créer un nouveau groupe de ressources, vous devez également choisir la région Azure dans laquelle vous souhaitez que la ressource réside.

>[!NOTE]
>La région dans laquelle un groupe de ressources réside est indépendante de l’emplacement où vous souhaitez créer le Peering avec Microsoft. Toutefois, il est recommandé d’organiser vos ressources de Peering dans des groupes de ressources qui résident dans les régions Azure les plus proches. Par exemple : pour les Peerings à Ashburn, vous pouvez créer un groupe de ressources dans les régions USA Est ou USA Est2.

* Sélectionnez votre ASN dans la case **PeerASN**.

>[!IMPORTANT] 
>Vous pouvez uniquement choisir un ASN dont le paramètre ValidationState est « approuvé » avant de soumettre une demande de Peering. Si vous venez d’envoyer votre demande PeerAsn, patientez environ 12 heures pour que l’association ASN soit « approuvée ». Si l’ASN que vous sélectionnez est en attente de validation, un message d’erreur s’affiche. Si vous ne voyez pas l’ASN que vous devez choisir, vérifiez que vous avez sélectionné le bon abonnement. Si c’est le cas, vérifiez si vous avez déjà créé PeerAsn à l’aide de **[l’association du numéro ASN de l’homologue à un abonnement Azure](https://go.microsoft.com/fwlink/?linkid=2129592)** .

#### <a name="launch-the-resource-and-configure-basic-settings"></a>Lancer la ressource et configurer les paramètres de base
[!INCLUDE [direct-peering-basic](./includes/direct-portal-basic.md)]

#### <a name="configure-connections-and-submit"></a>Configurer les connexions et les soumettre
[!INCLUDE [direct-peering-configuration](./includes/direct-portal-configuration-legacy.md)]

### <a name="verify-direct-peering"></a><a name=get></a>Vérifier un Peering Direct
[!INCLUDE [peering-direct-get-portal](./includes/direct-portal-get.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations, consultez les [FAQ sur le Peering Internet](faqs.md).

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un Peering direct à l’aide du portail](howto-direct-portal.md)
