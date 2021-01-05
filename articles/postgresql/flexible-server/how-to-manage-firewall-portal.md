---
title: Gérer des règles de pare-feu - Portail Azure - Azure Database pour PostgreSQL - Serveur flexible
description: Créer et gérer des règles de pare-feu pour Azure Database pour PostgreSQL - Serveur flexible à l’aide du portail Azure
author: ambhatna
ms.author: ambhatna
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 8c107b88f964d8e657d6833fc3a2e8425d9053ce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929791"
---
# <a name="create-and-manage-firewall-rules-for-azure-database-for-postgresql---flexible-server-using-the-azure-portal"></a>Créer et gérer des règles de pare-feu pour Azure Database pour PostgreSQL - Serveur flexible à l’aide du portail Azure

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en actuellement en préversion publique.

Azure Database pour PostgreSQL - Serveur flexible prend en charge deux types de méthodes de connectivité réseau mutuellement exclusives pour se connecter à votre serveur flexible. Les deux options sont les suivantes :

* Accès public (adresses IP autorisées)
* Accès privé (intégration au réseau virtuel)

Cet article est axé sur la création d’un serveur PostgreSQL avec **accès public (adresses IP autorisées)** à l’aide du portail Azure, et fournit une vue d’ensemble de la gestion des règles de pare-feu après la création d’un serveur flexible. Avec un *accès public (adresses IP autorisées)* , les connexions au serveur PostgreSQL sont limitées aux adresses IP autorisées. Les adresses IP des clients doivent être autorisées dans les règles de pare-feu. Pour en savoir plus à ce sujet, consultez [Accès public (adresses IP autorisées)](./concepts-networking.md#public-access-allowed-ip-addresses). Vous pouvez définir les règles de pare-feu au moment de la création du serveur (recommandé), mais aussi les ajouter ultérieurement. Cet article fournit une vue d’ensemble de la façon de créer et de gérer des règles de pare-feu à l’aide d’un accès public (adresses IP autorisées).

## <a name="create-a-firewall-rule-when-creating-a-server"></a>Créer une règle de pare-feu lors de la création d’un serveur

1. Sélectionnez **Créer une ressource** (+) dans l’angle supérieur gauche du portail.
2. Sélectionnez **Bases de données** > **Azure Database pour PostgreSQL**. Vous pouvez également entrer **PostgreSQL** dans la zone de recherche pour trouver le service.
3. Sélectionnez **Serveur flexible** comme option de déploiement.
4. Remplissez le formulaire **Informations de base**.
5. Accédez à l’onglet **Réseau** pour configurer la façon dont vous souhaitez vous connecter à votre serveur.
6. Dans **Méthode de connectivité**, sélectionnez *Accès public (adresses IP autorisées)* . Pour créer les **Règles de pare-feu**, spécifiez le nom de la règle de pare-feu ainsi qu’une adresse IP unique, ou une plage d’adresses. Si vous souhaitez limiter la règle à une seule adresse IP, tapez la même adresse dans les champs d’adresse IP de début et d’adresse IP de fin. L'ouverture du pare-feu permet aux administrateurs, utilisateurs et applications d'accéder à toute base de données du serveur PostgreSQL pour laquelle ils disposent d'informations d'identification valides.
   > [!Note]
   > Azure Database pour PostgreSQL - Serveur flexible crée un pare-feu au niveau du serveur. Il empêche les applications et les outils externes de se connecter au serveur et à toute base de données sur le serveur, sauf si vous créez une règle de pare-feu pour ouvrir le pare-feu à des adresses IP spécifiques.
7. Sélectionnez **Vérifier + créer** pour passer en revue la configuration de votre serveur flexible.
8.  Sélectionnez **Créer** pour approvisionner le serveur. Le provisionnement peut prendre quelques minutes.

## <a name="create-a-firewall-rule-after-server-is-created"></a>Créer une règle de pare-feu une fois le serveur créé

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez le serveur flexible Azure Database pour PostgreSQL sur lequel vous voulez ajouter des règles de pare-feu.
2. Dans la page du serveur flexible, sous l’en-tête **Paramètres**, cliquez sur **Réseau** pour ouvrir la page Réseau du serveur flexible.

   <!--![Azure portal - click Connection Security](./media/howto-manage-firewall-portal/1-connection-security.png)-->

3. Cliquez sur **Ajouter l’adresse IP actuelle du client** dans les règles de pare-feu. Cette opération crée automatiquement une règle de pare-feu avec l’adresse IP publique de votre ordinateur, telle que détectée par le système Azure.

   <!--![Azure portal - click Add My IP](./media/howto-manage-firewall-portal/2-add-my-ip.png)-->

4. Vérifiez votre adresse IP avant d’enregistrer la configuration. Dans certains cas, l’adresse IP identifiée par le portail Azure diffère de l’adresse IP utilisée lors de l’accès à Internet et aux serveurs Azure. Par conséquent, vous devrez peut-être modifier l’adresse IP de début et l’adresse IP de fin pour que la règle fonctionne comme prévu.

   Vous pouvez utiliser un moteur de recherche ou tout autre outil en ligne pour vérifier votre adresse IP. (par exemple, lancez une recherche sur « quelle est mon adresse IP »).

   <!--![Bing search for What is my IP](./media/howto-manage-firewall-portal/3-what-is-my-ip.png)-->

5. Ajoutez des plages d’adresses supplémentaires. Dans les règles de pare-feu d’Azure Database pour PostgreSQL - Serveur flexible, vous pouvez spécifier une adresse IP unique ou une plage d’adresses. Si vous souhaitez limiter la règle à une seule adresse IP, tapez la même adresse dans les champs d’adresse IP de début et d’adresse IP de fin. L'ouverture du pare-feu permet aux administrateurs, utilisateurs et applications d'accéder à toute base de données du serveur PostgreSQL pour laquelle ils disposent d'informations d'identification valides.

   <!--![Azure portal - firewall rules](./media/howto-manage-firewall-portal/4-specify-addresses.png)-->

6. Cliquez sur **Enregistrer** dans la barre d’outils pour enregistrer cette règle de pare-feu. Attendez la confirmation que la mise à jour des règles de pare-feu a réussi.

   <!--![Azure portal - click Save](./media/howto-manage-firewall-portal/5-save-firewall-rule.png)-->

## <a name="connecting-from-azure"></a>Connexion à partir d’Azure

Vous pouvez autoriser des ressources ou des applications déployées dans Azure à se connecter à votre serveur flexible. Cela comprend les applications web hébergées dans Azure App Service, exécutées sur une machine virtuelle Azure, une passerelle de gestion des données Azure Data Factory et bien plus encore. 

Quand une application dans Azure tente de se connecter à votre serveur, le pare-feu vérifie que les connexions Azure sont autorisées. Vous pouvez activer ce paramètre en sélectionnant l’option **Autoriser l’accès public à partir des ressources et services Azure dans Azure sur ce serveur** dans le portail à partir de l’onglet **Réseau**. Ensuite, cliquez sur **Enregistrer**.

Les ressources ne doivent pas obligatoirement se trouver sur le même réseau virtuel (VNET) ou dans le même groupe de ressources pour que la règle de pare-feu autorise ces connexions. Si la tentative de connexion n’est pas autorisée, la demande n’atteint pas le serveur flexible Azure Database pour PostgreSQL.

> [!IMPORTANT]
> Cette option configure le pare-feu pour autoriser toutes les connexions à partir d’Azure, notamment les connexions issues des abonnements d’autres clients. Lorsque vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.
>
> Nous vous recommandons de choisir **Accès privé (intégration au réseau virtuel)** pour accéder de manière sécurisée au serveur flexible.
>
## <a name="manage-existing-firewall-rules-through-the-azure-portal"></a>Gérer les règles de pare-feu existantes par le biais du portail Azure

Répétez les étapes suivantes pour gérer les règles de pare-feu.

- Pour ajouter l’ordinateur actuel, cliquez sur + **Ajouter l’adresse IP actuelle du client** dans les règles de pare-feu. Cliquez sur **Enregistrer** pour enregistrer les modifications.
- Pour ajouter des adresses IP supplémentaires, remplissez les champs Nom de la règle, Adresse IP de début et Adresse IP de fin. Cliquez sur **Enregistrer** pour enregistrer les modifications.
- Pour modifier une règle existante, cliquez sur les champs dans la règle et modifiez-les. Cliquez sur **Enregistrer** pour enregistrer les modifications.
- Pour supprimer une règle existante, cliquez sur les points de suspension [...], puis sur **Supprimer**. Cliquez sur **Enregistrer** pour enregistrer les modifications.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [réseaux dans Azure Database pour PostgreSQL - Serveur flexible](./concepts-networking.md)
- En savoir plus sur les [règles de pare-feu pour Azure Database pour PostgreSQL - Serveur flexible](./concepts-networking.md#public-access-allowed-ip-addresses)
- [Créer et gérer des règles de pare-feu d’Azure Database pour PostgreSQL à l’aide d’Azure CLI](./how-to-manage-firewall-cli.md)