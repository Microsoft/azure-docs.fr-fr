---
title: Gérer des réseaux virtuels - Portail Azure - Azure Database pour MySQL - Serveur flexible
description: Créer et gérer des réseaux virtuels dans le serveur flexible Azure Database pour MySQL à l’aide du portail Azure
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 28506ea5afe085e9f52e40495d9931ae0aeddf32
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929856"
---
# <a name="create-and-manage-virtual-networks-for-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Créer et gérer des réseaux virtuels dans le serveur flexible Azure Database pour MySQL à l’aide du portail Azure

> [!IMPORTANT]
> Le serveur flexible Azure Database pour MySQL est actuellement en préversion publique.

Le serveur flexible Azure Database pour MySQL prend en charge deux types de méthodes de connectivité réseau mutuellement exclusives pour se connecter à votre serveur flexible. Les deux options sont les suivantes :

- Accès public (adresses IP autorisées)
- Accès privé (intégration au réseau virtuel)

Dans cet article, nous allons nous concentrer sur la création d’un serveur MySQL avec l’option **Accès privé (intégration au réseau virtuel)** à l’aide du portail Azure. Avec l’option Accès privé (intégration au réseau virtuel), vous pouvez déployer votre serveur flexible sur votre propre [réseau virtuel Azure](../../virtual-network/virtual-networks-overview.md). Les réseaux virtuels Azure offrent des communications réseau privées et sécurisées. Dans un accès privé, les connexions au serveur MySQL sont limitées à votre réseau virtuel. Pour en savoir plus à ce sujet, reportez-vous à [Accès privé (intégration au réseau virtuel)](./concepts-networking.md#private-access-vnet-integration).

Vous pouvez déployer votre serveur flexible dans un réseau virtuel et un sous-réseau au moment de la création du serveur. Une fois le serveur flexible déployé, vous ne pouvez pas le déplacer vers un autre réseau virtuel ou sous-réseau, ni vers l’*accès public (adresses IP autorisées)* .

## <a name="prerequisites"></a>Prérequis
Pour créer un serveur flexible dans un réseau virtuel, vous avez besoin des éléments suivants :
- Un [réseau virtuel](../../virtual-network/quick-create-portal.md#create-a-virtual-network)
    > [!Note]
    > Le réseau virtuel et le sous-réseau doivent se trouver dans la même région et le même abonnement que votre serveur flexible.

-  Pour [déléguer un sous-réseau](../../virtual-network/manage-subnet-delegation.md#delegate-a-subnet-to-an-azure-service) à **Microsoft.DBforMySQL/flexibleServers**. Cette délégation spécifie que seuls les serveurs flexibles Azure Database pour MySQL peuvent utiliser ce sous-réseau. Aucun autre type de ressource Azure ne peut se trouver dans le sous-réseau délégué.

## <a name="create-azure-database-for-mysql-flexible-server-in-an-already-existing-virtual-network"></a>Créer un serveur flexible Azure Database pour MySQL dans un réseau virtuel déjà existant

1. Sélectionnez **Créer une ressource** (+) dans l’angle supérieur gauche du portail.
2. Sélectionnez **Bases de données** > **Azure Database pour MySQL**. Vous pouvez également entrer **MySQL** dans la zone de recherche pour localiser le service.
3. Sélectionnez **Serveur flexible** comme option de déploiement.
4. Remplissez le formulaire **Informations de base**.
5. Accédez à l’onglet **Mise en réseau** pour configurer la façon dont vous souhaitez vous connecter à votre serveur.
6. Dans **Méthode de connectivité**, sélectionnez **Accès privé (intégration au réseau virtuel)** . Accédez à **Réseau virtuel**, puis sélectionnez le *réseau virtuel* existant et le *sous-réseau* créé dans le cadre des conditions préalables ci-dessus.
7. Sélectionnez **Vérifier + créer** pour passer en revue la configuration de votre serveur flexible.
8. Sélectionnez **Créer** pour approvisionner le serveur. L’approvisionnement peut prendre quelques minutes.

>[!Note]
> Une fois le serveur flexible déployé sur un réseau virtuel et un sous-réseau, vous ne pouvez pas le déplacer vers un accès public (adresses IP autorisées).

## <a name="next-steps"></a>Étapes suivantes
- [Création et gestion d’un réseau virtuel de serveur flexible Azure Database pour MySQL à l’aide d’Azure CLI](./how-to-manage-virtual-network-cli.md).
- En savoir plus sur la [mise en réseau dans le serveur flexible Azure Database pour MySQL](./concepts-networking.md)
- Apprenez-en plus sur le [réseau virtuel de serveur flexible Azure Database pour MySQL](./concepts-networking.md#private-access-vnet-integration).
