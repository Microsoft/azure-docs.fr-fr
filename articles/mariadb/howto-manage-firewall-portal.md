---
title: Gérer les règles de pare-feu - Portail Azure - Azure Database for MariaDB
description: Découvrez comment créer et gérer des règles de pare-feu Azure Database for MariaDB à l’aide du Portail Azure.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: 895c8e51e16cb0b82ebf3edcf954dabe62b08ff4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021773"
---
# <a name="create-and-manage-azure-database-for-mariadb-firewall-rules-by-using-the-azure-portal"></a>Créer et gérer des règles de pare-feu Azure Database for MariaDB à l’aide du Portail Azure
Les règles de pare-feu au niveau du serveur sont utiles pour gérer l’accès à un serveur Azure Database for MariaDB à partir d’une adresse IP spécifique ou d’une plage d’adresses IP.

Des règles de réseau virtuel (VNet) peuvent également être utilisées pour sécuriser l'accès à votre serveur. Découvrez comment [créer et gérer des règles et des points de terminaison de service de réseau virtuel à l'aide du portail Azure](howto-manage-vnet-portal.md).

## <a name="create-a-server-level-firewall-rule-in-the-azure-portal"></a>Créer une règle de pare-feu au niveau du serveur dans le portail Azure

1. Dans la page du serveur MariaDB, sous le titre Paramètres, cliquez sur **Sécurité des connexions** pour ouvrir la page Sécurité des connexions pour Azure Database for MariaDB.

   ![Portail Azure - cliquez sur Sécurité des connexions](./media/howto-manage-firewall-portal/1-connection-security.png)

2. Cliquez sur **Ajouter mon adresse IP** dans la barre d’outils. Cette opération crée automatiquement une règle de pare-feu avec l’adresse IP publique de votre ordinateur, telle que détectée par le système Azure.

   ![Portail Azure - cliquez sur Ajouter mon adresse IP](./media/howto-manage-firewall-portal/2-add-my-ip.png)

3. Vérifiez votre adresse IP avant d’enregistrer la configuration. Dans certains cas, l’adresse IP identifiée par le portail Azure diffère de l’adresse IP utilisée lors de l’accès à Internet et aux serveurs Azure. Par conséquent, vous devrez peut-être modifier l’adresse IP de début et l’adresse IP de fin pour que la règle fonctionne comme prévu.

   Utilisez un moteur de recherche ou tout autre outil en ligne pour vérifier votre adresse IP (par exemple, lancez une recherche « quelle est mon adresse IP »).

4. Ajoutez des plages d’adresses supplémentaires. Dans les règles de pare-feu Azure Database for MariaDB, vous pouvez spécifier une seule adresse IP ou une plage d’adresses. Si vous souhaitez limiter la règle à une seule adresse IP, tapez la même adresse dans les champs d’adresse IP de début et d’adresse IP de fin. En ouvrant le pare-feu, les administrateurs, les utilisateurs et l’application peuvent accéder à toute base de données sur le serveur MariaDB pour laquelle ils disposent d’informations d’identification valides.

   ![Portail Azure - règles de pare-feu](./media/howto-manage-firewall-portal/4-specify-addresses.png)

5. Cliquez sur **Enregistrer** sur la barre d’outils pour enregistrer cette règle de pare-feu de niveau serveur. Attendez la confirmation de la mise à jour des règles de pare-feu.

   ![Portail Azure - cliquez sur Enregistrer](./media/howto-manage-firewall-portal/5-save-firewall-rule.png)

## <a name="connecting-from-azure"></a>Connexion à partir d’Azure
Pour autoriser des applications provenant d’Azure à se connecter à votre serveur Azure Database for MariaDB, les connexions Azure doivent être activées. Par exemple, pour héberger une application Azure Web Apps ou une application qui s’exécute sur une machine virtuelle Azure, ou pour vous connecter à partir d’une passerelle de gestion des données Azure Data Factory. Les ressources ne doivent pas obligatoirement se trouver sur le même réseau virtuel (VNET) ou dans le même groupe de ressources pour que la règle de pare-feu autorise ces connexions. Quand une application à partir d’Azure tente de se connecter à votre serveur de base de données, le pare-feu vérifie que les connexions Azure sont autorisées. Il existe deux méthodes pour activer ces types de connexions. Un paramètre de pare-feu avec des adresses de début et de fin égales à 0.0.0.0 indique que ces connexions sont autorisées. Dans le portail, vous pouvez également affecter la valeur **ON** à l’option **Autoriser l’accès aux services Azure** à partir du volet **Sécurité de la connexion**, puis cliquer sur **Enregistrer**. Si la tentative de connexion n’est pas autorisée, la demande n’atteint pas le serveur Azure Database for MariaDB.

> [!IMPORTANT]
> Cette option configure le pare-feu pour autoriser toutes les connexions à partir d’Azure, notamment les connexions issues des abonnements d’autres clients. Lorsque vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.
> 

## <a name="manage-existing-firewall-rules-in-the-azure-portal"></a>Gérer les règles de pare-feu existantes dans le Portail Azure
Répétez les étapes pour gérer les règles de pare-feu.
* Pour ajouter l’ordinateur actuel, cliquez sur **Ajouter mon adresse IP**. Cliquez sur **Enregistrer** pour enregistrer les modifications.
* Pour ajouter des adresses IP supplémentaires, remplissez les champs **Nom de la règle**, **Adresse IP de début** et **Adresse IP de fin**. Cliquez sur **Enregistrer** pour enregistrer les modifications.
* Pour modifier une règle existante, cliquez sur l’un de ses champs, puis modifiez-le. Cliquez sur **Enregistrer** pour enregistrer les modifications.
* Pour supprimer une règle existante, cliquez sur les points de suspension [...], puis sur **Supprimer**. Cliquez sur **Enregistrer** pour enregistrer les modifications.

## <a name="next-steps"></a>Étapes suivantes
 - De même, vous pouvez utiliser un script pour [créer et gérer des règles de pare-feu Azure Database for MariaDB à l’aide d’Azure CLI](howto-manage-firewall-cli.md).
 - Pour sécuriser davantage l'accès à votre serveur, [créez et gérez des règles et des points de terminaison de service de réseau virtuel à l'aide du portail Azure](howto-manage-vnet-portal.md).