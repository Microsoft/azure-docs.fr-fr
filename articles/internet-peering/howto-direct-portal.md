---
title: Créer ou modifier un Peering direct à l’aide du Portail Azure
titleSuffix: Azure
description: Créer ou modifier un Peering direct à l’aide du Portail Azure
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: how-to
ms.date: 5/19/2020
ms.author: derekol
ms.openlocfilehash: 1a89ce873c53e94036aa4f8ac2c2870365924187
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91537206"
---
# <a name="create-or-modify-a-direct-peering-by-using-the-azure-portal"></a>Créer ou modifier un Peering direct à l’aide du Portail Azure

Cet article explique comment créer un Peering Microsoft direct pour un fournisseur de services Internet ou un fournisseur Internet Exchange à l’aide du Portail Azure. Il montre également comment vérifier l’état de la ressource, la mettre à jour ou la supprimer et la déconfigurer.

Si vous préférez, vous pouvez suivre ce guide en utilisant Azure [PowerShell](howto-direct-powershell.md).

## <a name="before-you-begin"></a>Avant de commencer
* Passez en revue les [prérequis](prerequisites.md) et la [procédure pas à pas du Peering direct](walkthrough-direct-all.md) avant de commencer la configuration.
* Si vous avez déjà des connexions de Peering direct avec Microsoft qui ne sont pas converties en ressources Azure, consultez [Convertir un Peering direct hérité en ressource Azure à l’aide du portail](howto-legacy-direct-portal.md).

## <a name="create-and-provision-a-direct-peering"></a>Créer et provisionner un Peering direct

### <a name="sign-in-to-the-portal-and-select-your-subscription"></a>Se connecter au portail et sélectionner votre abonnement
[!INCLUDE [Account](./includes/account-portal.md)]

### <a name="create-a-direct-peering"></a><a name=create></a>Créer un Peering direct

En tant que fournisseur de services Internet ou fournisseur Internet Exchange, vous pouvez créer une nouvelle requête de Peering direct en [créant un Peering]( https://go.microsoft.com/fwlink/?linkid=2129593).

1. Dans la page **Créer un Peering**, sur l’onglet **De base**, remplissez les cases comme illustré ici :


    ![Capture d'écran représentant l'onglet De base de la page Créer un Peering, avec le lien Créer nouveau en évidence.](./media/setup-basics-tab.png)

2. Sélectionnez votre abonnement Azure.

3. Pour le groupe de ressources, sélectionnez un groupe de ressources existant dans la liste déroulante ou créez un nouveau groupe en sélectionnant Créer nouveau. Dans cet exemple, ,nous allons créer un nouveau groupe de ressources.

4. Le nom correspond au nom de la ressource et vous pouvez le choisir librement.

5. La région est sélectionnée automatiquement si vous avez choisi un groupe de ressources existant. Si vous avez choisi de créer un nouveau groupe de ressources, vous devez également choisir la région Azure dans laquelle vous souhaitez que la ressource réside.

    >[!NOTE]
    > La région dans laquelle un groupe de ressources réside est indépendante de l’emplacement où vous souhaitez créer le Peering avec Microsoft. Toutefois, il est recommandé d’organiser vos ressources de Peering dans des groupes de ressources qui résident dans les régions Azure les plus proches. Par exemple : pour les Peerings à Ashburn, vous pouvez créer un groupe de ressources dans les régions USA Est ou USA Est2.

6. Sélectionnez votre ASN dans la boîte **PeerASN**.

    >[!IMPORTANT]
    >Vous pouvez uniquement choisir un ASN dont le paramètre ValidationState est « approuvé » avant de soumettre une demande de Peering. Si vous venez d’envoyer votre demande PeerAsn, patientez environ 12 heures pour que l’association ASN soit « approuvée ». Si l’ASN que vous sélectionnez est en attente de validation, un message d’erreur s’affiche. Si vous ne voyez pas l’ASN que vous devez choisir, vérifiez que vous avez sélectionné le bon abonnement. Si c’est le cas, vérifiez si vous avez déjà créé PeerAsn à l’aide de **[l’association entre l’ASN pair et un abonnement Azure](https://go.microsoft.com/fwlink/?linkid=2129592)** .

7. Sélectionnez **Suivant : Configuration** pour continuer.



    ![Capture d'écran représentant l'onglet De base de la page Créer un Peering, avec toutes les valeurs entrées.](./media/setup-direct-basics-filled-tab.png)


#### <a name="configure-connections-and-submit"></a>Configurer les connexions et les soumettre
[!INCLUDE [direct-peering-configuration](./includes/direct-portal-configuration.md)]

### <a name="verify-direct-peering"></a><a name=get></a>Vérifier un Peering direct
[!INCLUDE [peering-direct-get-portal](./includes/direct-portal-get.md)]

## <a name="modify-a-direct-peering"></a><a name="modify"></a>Modifier un Peering direct
[!INCLUDE [peering-direct-modify-portal](./includes/direct-portal-modify.md)]

## <a name="deprovision-a-direct-peering"></a><a name="delete"></a>Déprovisionner un Peering direct
[!INCLUDE [peering-direct-delete-portal](./includes/delete.md)]

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un Peering Exchange à l’aide du portail](howto-exchange-portal.md)
* [Convertir un Peering Exchange hérité en ressource Azure à l’aide du portail](howto-legacy-exchange-portal.md)

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations, consultez les [FAQ sur le Peering Internet](faqs.md).
