---
title: Configurer une adresse IP sortante publique pour les environnements ISE
description: Découvrez comment configurer une adresse IP sortante publique unique pour les environnements de service d’intégration (ISE) dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 05/06/2020
ms.openlocfilehash: 9ee804f7bed01ca0c7f365a04e6108afd9598157
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87066394"
---
# <a name="set-up-a-single-ip-address-for-one-or-more-integration-service-environments-in-azure-logic-apps"></a>Configurer une adresse IP unique pour un ou plusieurs environnements de service d’intégration dans Azure Logic Apps

Quand vous utilisez Azure Logic Apps, vous pouvez configurer un [*environnement de service d’intégration* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) pour héberger des applications logiques qui ont besoin d’accéder à des ressources se trouvant sur un [réseau virtuel Azure](../virtual-network/virtual-networks-overview.md). Lorsque vous avez plusieurs instances d’environnement ISE qui doivent accéder à d’autres points de terminaison dotés de restrictions d’adresses IP, déployez un [Pare-feu Azure](../firewall/overview.md) ou une [appliance virtuelle réseau](../virtual-network/virtual-networks-overview.md#filter-network-traffic) dans votre réseau virtuel, et routez le trafic sortant via ce pare-feu ou cette appliance virtuelle réseau. Vous pouvez alors faire en sorte que toutes les instances d’environnement ISE de votre réseau virtuel utilisent une seule adresse IP publique, statique et prédictible pour communiquer avec les systèmes de destination souhaités. De cette façon, vous n’avez pas besoin de configurer des ouvertures de pare-feu supplémentaires sur vos systèmes de destination pour chaque ISE.

Cette rubrique montre comment router le trafic sortant via un Pare-feu Azure, mais vous pouvez appliquer des concepts similaires à une appliance virtuelle de réseau, comme un pare-feu tiers de la Place de marché Azure. Cette rubrique porte sur la configuration pour plusieurs instances d’environnement ISE, mais vous pouvez également utiliser cette approche pour un environnement ISE unique lorsque votre scénario nécessite de limiter le nombre d’adresses IP qui ont besoin d’un accès. Déterminez si les coûts supplémentaires pour le pare-feu ou l’appliance de réseau virtuel ont un sens pour votre scénario. Découvrez plus en détail la [tarification du Pare-feu Azure](https://azure.microsoft.com/pricing/details/azure-firewall/).

## <a name="prerequisites"></a>Prérequis

* Un pare-feu Azure qui s’exécute dans le même réseau virtuel que votre environnement ISE. Si vous ne disposez pas d’un pare-feu, commencez par [ajouter un sous-réseau](../virtual-network/virtual-network-manage-subnet.md#add-a-subnet) nommé `AzureFirewallSubnet` à votre réseau virtuel. Vous pouvez ensuite [créer et déployer un pare-feu](../firewall/tutorial-firewall-deploy-portal.md#deploy-the-firewall) dans votre réseau virtuel.

* Une [table de route](../virtual-network/manage-route-table.md) Azure. Commencez par [créer une table de routage](../virtual-network/manage-route-table.md#create-a-route-table) si vous n’en avez pas. Pour plus d’informations sur le routage, consultez [Routage du trafic de réseau virtuel](../virtual-network/virtual-networks-udr-overview.md).

## <a name="set-up-route-table"></a>Configurer une table de routage

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez la table de routage, par exemple :

   ![Sélectionner une table de routage avec une règle de redirection du trafic sortant](./media/connect-virtual-network-vnet-set-up-single-ip-address/select-route-table-for-virtual-network.png)

1. Pour [ajouter une nouvelle route](../virtual-network/manage-route-table.md#create-a-route), dans le menu Table de routage, sélectionnez **Routes** > **Ajouter**.

   ![Ajouter une route pour diriger le trafic sortant](./media/connect-virtual-network-vnet-set-up-single-ip-address/add-route-to-route-table.png)

1. Dans le volet **Ajouter une route**, [configurez la nouvelle route](../virtual-network/manage-route-table.md#create-a-route) avec une règle qui spécifie que tout le trafic sortant vers le système de destination adopte ce comportement :

   * Utilisez l’[**Appliance virtuelle**](../virtual-network/virtual-networks-udr-overview.md#user-defined) comme type de tronçon suivant.

   * Accédez à l’adresse IP privée de l’instance de pare-feu comme adresse de tronçon suivant.

     Pour trouver cette adresse IP, dans le menu de votre pare-feu, sélectionnez **Vue d’ensemble**, puis recherchez l’adresse sous **Adresse IP privée** ; par exemple :

     ![Rechercher l’adresse IP privée du pare-feu](./media/connect-virtual-network-vnet-set-up-single-ip-address/find-firewall-private-ip-address.png)

   Voici un exemple qui montre comment une telle règle peut se présenter :

   ![Configurer une règle de redirection du trafic sortant](./media/connect-virtual-network-vnet-set-up-single-ip-address/add-rule-to-route-table.png)

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **Nom de l’itinéraire** | <*nom_route_unique*> | Nom unique pour la route dans la table de routage |
   | **Préfixe de l’adresse** | <*adresse_de_destination*> | Préfixe d’adresse de votre système de destination où vous voulez que le trafic sortant soit acheminé. Veillez à utiliser la [notation de routage interdomaine sans classe (CIDR, Classless Inter-Domain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) pour cette adresse. Dans cet exemple, ce préfixe d’adresse concerne un serveur SFTP, qui est décrit dans la section [Configurer la règle de réseau](#set-up-network-rule). |
   | **Type de tronçon suivant** | **Appliance virtuelle** | Le [type de tronçon](../virtual-network/virtual-networks-udr-overview.md#next-hop-types-across-azure-tools) utilisé par le trafic sortant |
   | **Adresse du tronçon suivant** | <*adresse_IP_privée_du_pare-feu*> | Adresse IP privée de votre pare-feu |
   |||

<a name="set-up-network-rule"></a>

## <a name="set-up-network-rule"></a>Configurer la règle de réseau

1. Dans le portail Azure, recherchez et sélectionnez votre pare-feu. Dans le menu Pare-feu, sous **Paramètres**, sélectionnez **Règles**. Dans le volet Règles, sélectionnez **Collection de règles de réseau** > **Ajouter une collection de règles de réseau**.

   ![Ajouter une collection de règles de réseau au pare-feu](./media/connect-virtual-network-vnet-set-up-single-ip-address/add-network-rule-collection.png)

1. Dans la collection, ajoutez une règle qui autorise le trafic vers le système de destination.

   Supposons, par exemple, que vous disposiez d’une application logique qui s’exécute dans un environnement ISE et qui doit communiquer avec un serveur SFTP. Vous créez une collection de règles de réseau nommée `LogicApp_ISE_SFTP_Outbound` qui contient une règle de réseau appelée `ISE_SFTP_Outbound`. Cette règle autorise le trafic en provenance de l’adresse IP de tout sous-réseau où votre environnement ISE s’exécute dans votre réseau virtuel vers le serveur SFTP de destination à l’aide de l’adresse IP privée de votre pare-feu.

   ![Configurer une règle de réseau pour le pare-feu](./media/connect-virtual-network-vnet-set-up-single-ip-address/set-up-network-rule-for-firewall.png)

   **Propriétés de la collection de règles de réseau**

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **Nom** | <*nom_collection_règles_réseau*> | Nom de votre collection de règles de réseau |
   | **Priorité** | <*niveau_de_priorité*> | Ordre de priorité à utiliser pour l’exécution de la collection de règles. Pour plus d’informations, consultez [Quels sont les concepts de pare-feu Azure ?](../firewall/firewall-faq.md#what-are-some-azure-firewall-concepts) |
   | **Action** | **Autoriser** | Type d’action à effectuer pour cette règle |
   |||

   **Propriétés de règle de réseau**

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **Nom** | <*nom_règle_réseau*> | Nom de votre règle de réseau |
   | **Protocole** | <*protocoles_de_connexions*> | Protocoles de connexion à utiliser. Par exemple, si vous utilisez des règles de groupe de sécurité réseau (NSG), sélectionnez à la fois **TCP** et **UDP**, et pas seulement **TCP**. |
   | **Adresses sources** | <*adresses_de_sous-réseau_ISE*> | Adresses IP de sous-réseau où votre environnement ISE s’exécute et d’où émane le trafic en provenance de votre application logique |
   | **Adresses de destination** | <*adresse_IP_de_destination*> | Adresse IP de votre système de destination où vous voulez que le trafic sortant soit acheminé. Dans cet exemple, il s’agit de l’adresse IP du serveur SFTP. |
   | **Ports de destination** | <*ports_de_destination*> | Tous les ports que votre système de destination utilise pour la communication entrante |
   |||

   Pour plus d’informations sur les règles de réseau, consultez les articles suivants :

   * [Configurer une règle de réseau](../firewall/tutorial-firewall-deploy-portal.md#configure-a-network-rule)
   * [Logique de traitement des règles de pare-feu Azure](../firewall/rule-processing.md#network-rules-and-applications-rules)
   * [Questions fréquentes (FAQ) sur le pare-feu Azure](../firewall/firewall-faq.md)
   * [Azure PowerShell : New-AzFirewallNetworkRule](/powershell/module/az.network/new-azfirewallnetworkrule)
   * [Azure CLI : az network firewall network-rule](/cli/azure/ext/azure-firewall/network/firewall/network-rule?view=azure-cli-latest#ext-azure-firewall-az-network-firewall-network-rule-create)

## <a name="next-steps"></a>Étapes suivantes

* [Se connecter à des réseaux virtuels Azure à partir d’Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)
