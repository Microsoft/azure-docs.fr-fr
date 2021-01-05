---
title: Convertir un Peering Exchange hérité en ressource Azure à l’aide du Portail Azure
titleSuffix: Azure
description: Convertir un Peering Exchange hérité en ressource Azure à l’aide du Portail Azure
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: how-to
ms.date: 5/21/2020
ms.author: derekol
ms.openlocfilehash: 6249d68486af754f2d2980aaed9d5fd8287dcb5e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84700159"
---
# <a name="convert-a-legacy-exchange-peering-to-an-azure-resource-by-using-the-azure-portal"></a>Convertir un Peering Exchange hérité en ressource Azure à l’aide du Portail Azure

Cet article explique comment convertir un Peering Exchange hérité en ressource Azure à l’aide du Portail Azure.

Si vous préférez, vous pouvez suivre ce guide en utilisant [PowerShell](howto-legacy-exchange-powershell.md).

## <a name="before-you-begin"></a>Avant de commencer
* Passez en revue les [prérequis](prerequisites.md) et la [procédure pas à pas du Peering Exchange](walkthrough-exchange-all.md) avant de commencer la configuration.

## <a name="convert-a-legacy-exchange-peering-to-an-azure-resource"></a>Convertir un Peering Exchange hérité en ressource Azure

En tant que fournisseur Internet Exchange, vous pouvez créer une requête Peering Exchange en [créant un Peering]( https://go.microsoft.com/fwlink/?linkid=2129593).

1. Dans la page **Créer un Peering**, sur l’onglet **De base**, remplissez les cases comme illustré ici :

   ![Inscrire Peering Service](./media/setup-basics-tab.png)

* Sélectionnez votre abonnement Azure.

* Pour le groupe de ressources, sélectionnez un groupe de ressources existant dans la liste déroulante ou créez un nouveau groupe en sélectionnant Créer nouveau. Dans cet exemple, ,nous allons créer un nouveau groupe de ressources.

* Le nom correspond au nom de la ressource et vous pouvez le choisir librement.

* La région est sélectionnée automatiquement si vous avez choisi un groupe de ressources existant. Si vous avez choisi de créer un nouveau groupe de ressources, vous devez également choisir la région Azure dans laquelle vous souhaitez que la ressource réside.

  >[!NOTE]
  >La région dans laquelle un groupe de ressources réside est indépendante de l’emplacement où vous souhaitez créer le Peering avec Microsoft. Toutefois, il est recommandé d’organiser vos ressources de Peering dans des groupes de ressources qui résident dans les régions Azure les plus proches. Par exemple : pour les Peerings à Ashburn, vous pouvez créer un groupe de ressources dans les régions USA Est ou USA Est2.

* Sélectionnez votre ASN dans la boîte **PeerASN**.

  >[!IMPORTANT]  
  >Vous pouvez uniquement choisir un ASN dont le paramètre ValidationState est « approuvé » avant de soumettre une demande de Peering. Si vous venez d’envoyer votre demande PeerAsn, patientez environ 12 heures pour que l’association ASN soit « approuvée ». Si l’ASN que vous sélectionnez est en attente de validation, un message d’erreur s’affiche. Si vous ne voyez pas l’ASN que vous devez choisir, vérifiez que vous avez sélectionné le bon abonnement. Si c’est le cas, vérifiez si vous avez déjà créé PeerAsn à l’aide de **[l’association du numéro ASN de l’homologue à un abonnement Azure](https://go.microsoft.com/fwlink/?linkid=2129592)** .

* Sélectionnez **Suivant : Configuration** pour continuer.


#### <a name="configure-connections-and-submit"></a>Configurer les connexions et les soumettre
[!INCLUDE [exchange-peering-configuration](./includes/exchange-portal-configuration-legacy.md)]

### <a name="verify-exchange-peering"></a><a name=get></a>Vérifier le peering Exchange
[!INCLUDE [peering-exchange-get-portal](./includes/exchange-portal-get.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations, consultez les [FAQ sur le Peering Internet](faqs.md).

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un Peering Exchange à l’aide du portail](howto-exchange-portal.md)
