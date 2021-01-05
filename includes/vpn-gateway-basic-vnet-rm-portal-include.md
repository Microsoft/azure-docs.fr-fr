---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/20/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: a66596ecb926c1bf3c6b61cc99e1eb1b56e99158
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "92328428"
---
1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans **Rechercher dans les ressources, services et documents (G+/)** , tapez *réseau virtuel*.

   ![Localiser la page de ressource Réseau virtuel](./media/vpn-gateway-basic-vnet-rm-portal-include/marketplace.png "Localiser la page de ressource de réseau virtuel")
1. Sélectionnez **Réseau virtuel** dans les résultats sous **Place de marché**.

   ![Sélectionner un réseau virtuel](./media/vpn-gateway-basic-vnet-rm-portal-include/marketplace-results.png "Localiser la page de ressource de réseau virtuel")
1. Dans la page **Réseau virtuel**, sélectionnez **Créer**.

   ![Page Réseau virtuel](./media/vpn-gateway-basic-vnet-rm-portal-include/vnet-click-create.png "Sélectionner Créer")
1. Quand vous sélectionnez **Créer**, la page **Créer un réseau virtuel** s’ouvre.
1. Sous l’onglet **Informations de base**, configurez les paramètres de réseau virtuel **Détails du projet** et **Détails de l’instance**.

   ![Onglet Informations de base](./media/vpn-gateway-basic-vnet-rm-portal-include/basics.png "Onglet Informations de base") Quand vous renseignez les champs, une coche verte indique que les caractères entrés sont validés. Certaines valeurs sont renseignées automatiquement, et vous pouvez les remplacer par vos propres valeurs :

   - **Abonnement**: vérifiez que l’abonnement listé est approprié. Vous pouvez modifier des abonnements à l’aide de la liste déroulante.
   - **Groupe de ressources** : sélectionnez un groupe de ressources existant ou cliquez sur **Créer** pour en créer un. Pour plus d’informations sur les groupes de ressources, consultez [Vue d’ensemble d’Azure Resource Manager](../articles/azure-resource-manager/management/overview.md#resource-groups).
   - **Name** : entrez le nom du réseau virtuel.
   - **Région** : sélectionnez l’emplacement du réseau virtuel. L’emplacement détermine où se trouveront les ressources que vous déployez sur ce réseau virtuel.

1. Configurez les valeurs de l’onglet **Adresses IP**. Les valeurs indiquées dans les exemples ci-dessous sont présentées à des fins de démonstration. Définissez ces valeurs selon les paramètres dont vous avez besoin.

   ![Onglet Adresses IP](./media/vpn-gateway-basic-vnet-rm-portal-include/addresses.png "Onglet Adresses IP")  
   - **Espace d’adressage IPv4** : Un espace d’adressage est créé automatiquement par défaut. Vous pouvez cliquer sur l’espace d’adressage pour le définir selon vos propres valeurs. Vous pouvez également ajouter des espaces d’adressage.
   - **Sous-réseau** : Si vous utilisez l’espace d’adressage par défaut, un sous-réseau par défaut est créé automatiquement. Si vous modifiez l’espace d’adressage, vous devez ajouter un sous-réseau. Sélectionnez **+ Ajouter un sous-réseau** pour ouvrir la fenêtre **Ajouter un sous-réseau**. Configurez les paramètres suivants, puis sélectionnez **Ajouter** pour ajouter les valeurs :
      - **Nom du sous-réseau** : dans cet exemple, nous avons nommé le sous-réseau « FrontEnd ».
      - **Plage d’adresses de sous-réseau** : plage d’adresses de ce sous-réseau.

1. Dans l’onglet **Sécurité**, conservez les valeurs par défaut pour le moment :

   - **Protection DDoS** : De base
   - **Pare-feu** : Désactivé
1. Sélectionnez **Vérifier + créer** pour vérifier les paramètres de réseau virtuel.
1. Une fois les paramètres vérifiés, sélectionnez **Créer**.
