---
title: 'Démarrage rapide : Se connecter - MySQL Workbench - Azure Database pour MySQL - Serveur flexible'
description: Ce guide de démarrage rapide indique les étapes à suivre pour utiliser MySQL Workbench pour vous connecter et interroger des données depuis Azure Database pour MySQL - Serveur flexible.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 9/21/2020
ms.openlocfilehash: d604ca8c8979ec98b990f8002ce29d0df92ac2af
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90945067"
---
# <a name="quickstart-use-mysql-workbench-to-connect-and-query-data-in-azure-database-for-mysql---flexible-server-preview"></a>Démarrage rapide : Utiliser MySQL Workbench pour se connecter et interroger des données dans Azure Database pour MySQL - Serveur flexible (préversion)


> [!IMPORTANT] 
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Ce guide de démarrage rapide explique comment se connecter à un serveur flexible Azure Database pour MySQL à l’aide de l’application MySQL Workbench.

## <a name="prerequisites"></a>Prérequis

Ce guide de démarrage rapide s’appuie sur les ressources créées dans l’un de ces guides :

- [Créer un serveur flexible Azure Database pour MySQL à l'aide du portail Azure](./quickstart-create-server-portal.md)
- [Créer un serveur flexible Azure Database pour MySQL à l'aide d'Azure CLI](./quickstart-create-server-cli.md)

## <a name="preparing-your-client-workstation"></a>Préparation de votre station de travail cliente
- Si vous avez créé votre serveur flexible avec l'option *Accès privé (intégration au réseau virtuel)* , vous devez vous connecter à votre serveur à partir d'une ressource qui se trouve au sein du même réseau virtuel que votre serveur. Vous pouvez créer une machine virtuelle et l'ajouter au réseau virtuel créé avec votre serveur flexible. Reportez-vous à [Créer et gérer un réseau virtuel Azure Database pour MySQL - Serveur flexible à l'aide d'Azure CLI](./how-to-manage-virtual-network-cli.md).
- Si vous avez créé votre serveur flexible avec l'option *Accès public (adresses IP autorisées)* , vous pouvez ajouter votre adresse IP locale à la liste des règles de pare-feu sur votre serveur. Reportez-vous à [Créer et gérer des règles de pare-feu Azure Database pour MySQL - Serveur flexible à l'aide d'Azure CLI](./how-to-manage-firewall-cli.md).

- Téléchargez et installez MySQL Workbench sur votre ordinateur depuis le [site web MySQL](https://dev.mysql.com/downloads/workbench/).

## <a name="get-connection-information"></a>Obtenir des informations de connexion

Obtenez les informations de connexion requises pour la connexion au serveur flexible. Vous devez disposer du nom du serveur complet et des informations d’identification.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans le menu de gauche du portail Azure, sélectionnez **Toutes les ressources**, puis recherchez le serveur que vous venez de créer, par exemple **mydemoserver**.
3. Sélectionnez le nom du serveur.
4. Dans le panneau **Vue d’ensemble** du serveur, notez le **nom du serveur** et le **nom de connexion de l’administrateur du serveur**. Si vous oubliez votre mot de passe, vous pouvez également le réinitialiser dans ce panneau.
<!--- :::image type="content" source="./media/connect-php/1_server-overview-name-login.png" alt-text="Azure Database for MySQL Flexible Server name":::--->

## <a name="connect-to-the-server-using-mysql-workbench"></a>Se connecter au serveur à l’aide de MySQL Workbench

Pour se connecter au serveur flexible Azure Database pour MySQL à l’aide de MySQL Workbench :

1. Lancez l’application MySQL Workbench sur votre ordinateur.

2. Dans la boîte de dialogue **Configurer une nouvelle connexion**, entrez les informations suivantes dans l’onglet **Paramètres** :

    :::image type="content" source="./media/connect-workbench/2-setup-new-connection.png" alt-text="Azure Database for MySQL Flexible Server name":::

    | **Paramètres** | **Valeur suggérée** | **Description du champ** |
    |---|---|---|
    |    Nom de connexion | Connexion démo | Spécifiez une étiquette pour cette connexion. |
    | Méthode de connexion | Standard (TCP/IP) | Standard (TCP/IP) est suffisant. |
    | HostName | *nom du serveur* | Spécifiez la valeur de nom de serveur utilisée lorsque vous avez créé la base de données Azure pour MySQL. Notre exemple de serveur affiché est mydemoserver.mysql.database.azure.com. Utilisez le nom de domaine complet (\*.mysql.database.azure.com), comme indiqué dans l’exemple. Si vous ne vous souvenez pas du nom de votre serveur, suivez les instructions de la section précédente pour obtenir les informations de connexion.  |
    | Port | 3306 | Utilisez toujours le port 3306 lorsque vous vous connectez au service Base de données Azure pour MySQL. |
    | Nom d’utilisateur |  *nom de connexion d’administrateur du serveur* | Tapez le nom d’utilisateur de connexion d’administrateur du serveur fourni lorsque vous avez créé la base de données Azure pour MySQL. Le nom d’utilisateur de notre exemple est myadmin. Si vous ne vous souvenez pas du nom d’utilisateur, suivez les instructions de la section précédente pour obtenir les informations de connexion.
    | Mot de passe | votre mot de passe | Cliquez sur le bouton **Stocker dans le coffre-fort…** pour enregistrer le mot de passe. |

3. Cliquez sur **Tester la connexion** pour tester si tous les paramètres sont correctement configurés.

4. Cliquez sur **OK** pour enregistrer la connexion.

5. Dans la liste de **connexions MySQL**, cliquez sur le nom correspondant à votre serveur puis patientez jusqu’à ce que la connexion soit établie.

    Un nouvel onglet SQL s’ouvre avec un éditeur vide où vous pouvez saisir vos requêtes.

> [!NOTE]
> Une connexion chiffrée à l’aide de TLS 1.2 est requise et appliquée sur votre serveur flexible Azure Database pour MySQL. Habituellement, bien qu’aucune configuration supplémentaire avec certificats TLS/SSL ne soit requise pour MySQL Workbench afin de vous connecter à votre serveur, nous recommandons de lier la certification d’autorité de certification TLS/SSL à MySQL Workbench. Pour plus d’informations, consultez [Se connecter à l’aide de TLS/SSL](./how-to-connect-tls-ssl.md).

## <a name="create-a-table-insert-data-read-data-update-data-delete-data"></a>Créer une table, insérer des données, lire les données, mettre à jour des données et supprimer des données

1. Copiez et collez l’exemple de code SQL dans un onglet SQL vide pour illustrer des exemples de données.

    Ce code crée une base de données vide nommée quickstartdb, puis crée un exemple de table nommée inventory. Il insère des lignes, puis lit les lignes. Il modifie les données avec une instruction de mise à jour et lit de nouveau les lignes. Enfin, il supprime une ligne puis lit de nouveau les lignes.

    ```sql
    -- Create a database
    -- DROP DATABASE IF EXISTS quickstartdb;
    CREATE DATABASE quickstartdb;
    USE quickstartdb;

    -- Create a table and insert rows
    DROP TABLE IF EXISTS inventory;
    CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);
    INSERT INTO inventory (name, quantity) VALUES ('banana', 150);
    INSERT INTO inventory (name, quantity) VALUES ('orange', 154);
    INSERT INTO inventory (name, quantity) VALUES ('apple', 100);

    -- Read
    SELECT * FROM inventory;

    -- Update
    UPDATE inventory SET quantity = 200 WHERE id = 1;
    SELECT * FROM inventory;

    -- Delete
    DELETE FROM inventory WHERE id = 2;
    SELECT * FROM inventory;
    ```

    La capture d’écran montre un exemple de code SQL dans SQL Workbench et la sortie obtenue après son exécution.

    :::image type="content" source="./media/connect-workbench/3-workbench-sql-tab.png" alt-text="Azure Database for MySQL Flexible Server name":::

2. Pour exécuter l’exemple de Code SQL, cliquez sur l’icône d’éclair dans la barre d’outils de l’onglet**Fichier SQL**.
3. Vous observerez trois onglets de résultats dans la section **Grille de résultats** au milieu de la page.
4. La liste **Sortie** apparaît en bas de la page. L’état de chaque commande s’affiche.

Vous êtes à présent connecté au serveur flexible Azure Database pour MySQL à l’aide de MySQL Workbench et avez interrogé des données à l’aide du langage SQL.

## <a name="next-steps"></a>Étapes suivantes
- [Connectivité chiffrée à l'aide du protocole TLS (Transport Layer Security) 1.2 dans Azure Database pour MySQL - Serveur flexible](./how-to-connect-tls-ssl.md)
- En savoir plus sur la [Mise en réseau dans Azure Database pour MySQL - Serveur flexible](./concepts-networking.md)
- [Créer et gérer des règles de pare-feu Azure Database pour MySQL - Serveur flexible à l'aide du portail Azure](./how-to-manage-firewall-portal.md)
- [Créer et gérer un réseau virtuel Azure Database pour MySQL - Serveur flexible à l'aide du portail Azure](./how-to-manage-virtual-network-portal.md)
