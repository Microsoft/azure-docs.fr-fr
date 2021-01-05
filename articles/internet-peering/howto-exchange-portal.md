---
title: Créer ou modifier un Peering Exchange à l’aide du Portail Azure
titleSuffix: Azure
description: Créer ou modifier un Peering Exchange à l’aide du Portail Azure
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: how-to
ms.date: 5/2/2020
ms.author: derekol
ms.openlocfilehash: 69201c97882846fb929b3b6f9a90be6647603bcc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84700478"
---
# <a name="create-or-modify-an-exchange-peering-by-using-the-azure-portal"></a>Créer ou modifier un Peering Exchange à l’aide du Portail Azure

Cet article explique comment créer un Peering Microsoft Exchange à l’aide du Portail Azure. Il montre également comment vérifier l’état de la ressource, la mettre à jour, la supprimer et la déprovisionner.

Si vous préférez, vous pouvez suivre ce guide en utilisant [PowerShell](howto-exchange-powershell.md).

## <a name="before-you-begin"></a>Avant de commencer
* Passez en revue les [prérequis](prerequisites.md) et la [procédure pas à pas du Peering Exchange](walkthrough-exchange-all.md) avant de commencer la configuration.
* Si vous avez déjà des Peerings Exchange avec Microsoft qui ne sont pas convertis en ressources Azure, consultez [Convertir un Peering Exchange hérité en ressource Azure à l’aide du portail](howto-legacy-exchange-portal.md).

## <a name="create-and-provision-an-exchange-peering"></a>Créer et provisionner un Peering Exchange

### <a name="sign-in-to-the-portal-and-select-your-subscription"></a>Se connecter au portail et sélectionner votre abonnement
[!INCLUDE [Account](./includes/account-portal.md)]

### <a name="create-an-exchange-peering"></a><a name=create></a>Créer un Peering Exchange


En tant que fournisseur Internet Exchange, vous pouvez créer une requête Peering Exchange en [créant un Peering]( https://go.microsoft.com/fwlink/?linkid=2129593).

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

* Sélectionnez **Suivant : Configuration** pour continuer.

#### <a name="configure-connections-and-submit"></a>Configurer les connexions et les soumettre
[!INCLUDE [exchange-peering-configuration](./includes/exchange-portal-configuration.md)]

### <a name="verify-an-exchange-peering"></a><a name=get></a>Vérifier un Peering Exchange
[!INCLUDE [peering-exchange-get-portal](./includes/exchange-portal-get.md)]

## <a name="modify-an-exchange-peering"></a><a name="modify"></a>Modifier un Peering Exchange
[!INCLUDE [peering-exchange-modify-portal](./includes/exchange-portal-modify.md)]

## <a name="deprovision-an-exchange-peering"></a><a name="delete"></a>Déprovisionner un Peering Exchange
[!INCLUDE [peering-exchange-delete-portal](./includes/delete.md)]

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un Peering direct à l’aide du portail](howto-direct-portal.md)
* [Convertir un Peering direct hérité en ressource Azure à l’aide du portail](howto-legacy-direct-portal.md)

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations, consultez les [FAQ sur le Peering Internet](faqs.md).
