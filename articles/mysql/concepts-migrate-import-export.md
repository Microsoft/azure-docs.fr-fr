---
title: Importation et exportation - Azure Database pour MySQL
description: Cet article explique les méthodes courantes d’importation et d’exportation de bases de données dans une base de données Azure pour MySQL à l’aide d’outils tels que MySQL Workbench.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 10/30/2020
ms.openlocfilehash: af9f0f65e01a786d478fac0adde6174b8f03b2fd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019892"
---
# <a name="migrate-your-mysql-database-by-using-import-and-export"></a>Migration d’une base de données MySQL à l’aide de l’importation et de l’exportation
[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]
Cet article explique deux approches communes de l’importation et de l’exportation de données vers un serveur de base de données Azure pour MySQL à l’aide de MySQL Workbench.

Vous pouvez également consulter le [guide relatif à la migration de base de données](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide) pour obtenir des informations détaillées et des cas d’usage sur la migration de base de données vers Azure Database pour MySQL. Ce guide fournit des conseils pour réussir la planification et l’exécution d’une migration MySQL vers Azure.

## <a name="before-you-begin"></a>Avant de commencer
Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :
- Un serveur de base de données Azure pour MySQL (voir [Créer un serveur de base de données Azure pour MySQL à l’aide du Portail Azure](quickstart-create-mysql-server-database-using-azure-portal.md))
- [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) ou un autre outil MySQL tiers pour procéder à l’importation/exportation.

## <a name="create-a-database-on-the-azure-database-for-mysql-server"></a>Créer une base de données sur le serveur de base de données Azure pour MySQL
Créez une base de données vide sur le serveur Azure Database pour MySQL à l’aide de MySQL Workbench, Toad ou Navicat. La base de données peut avoir le même nom que celle qui contient les données sauvegardées, mais vous pouvez également créer une base de données avec un autre nom.

Pour vous connecter, repérez les informations de connexion dans la **Vue d’ensemble** de votre Azure Database pour MySQL.

:::image type="content" source="./media/concepts-migrate-import-export/1_server-overview-name-login.png" alt-text="Trouver les informations de connexion dans le Portail Azure":::

Ajoutez les informations de connexion à MySQL Workbench.

:::image type="content" source="./media/concepts-migrate-import-export/2_setup-new-connection.png" alt-text="Chaîne de connexion MySQL Workbench":::

## <a name="determine-when-to-use-import-and-export-techniques"></a>Déterminer quand utiliser des techniques d’importation et d’exportation

> [!TIP]
> Pour les scénarios dans lesquels vous voulez vider et restaurer la base de données dans sa totalité, vous devez plutôt utiliser la [méthode de vidage et restauration](concepts-migrate-dump-restore.md).

Utilisez les outils MySQL pour importer et exporter des bases de données dans la base de données Azure MySQL dans les scénarios suivants.

- Lorsque vous devez choisir spécifiquement plusieurs tables à importer à partir d’une base de données MySQL existante vers une base de données Azure MySQL, il est préférable d’utiliser la technique d’importation et d’exportation.  En procédant ainsi, vous pouvez omettre toutes les tables non nécessaires de la migration pour économiser du temps et des ressources. Par exemple, utilisez le commutateur `--include-tables` ou `--exclude-tables` avec [mysqlpump](https://dev.mysql.com/doc/refman/5.7/en/mysqlpump.html#option_mysqlpump_include-tables) et le commutateur `--tables` avec [mysqldump](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html#option_mysqldump_tables).
- Lorsque vous déplacez des objets de base de données autres que des tables, vous devez les créer explicitement. Incluez les contraintes (clé primaire, clé étrangère, index), les vues, les fonctions, les procédures, les déclencheurs et tout autre objet de base de données dont vous souhaitez effectuer la migration.
- Lorsque vous effectuez la migration de données à partir de sources de données externes autres qu’une base de données MySQL, créez des fichiers plats et importez-les à l’aide de [mysqlimport](https://dev.mysql.com/doc/refman/5.7/en/mysqlimport.html).

> [!Important]
> Le serveur unique et le serveur flexible prennent tous les deux en charge le **moteur de stockage InnoDB uniquement**. Assurez-vous que toutes les tables de la base de données utilisent le moteur de stockage InnoDB lors du chargement de données dans la base de données Azure pour MySQL.
> Si votre base de données source utilise un autre moteur de stockage, effectuez une conversion vers le moteur InnoDB avant de migrer la base de données. Par exemple, si votre site WordPress ou votre application web utilise le moteur MyISAM, commencez par convertir les tables en effectuant une migration des données vers des tables InnoDB. Utilisez la clause `ENGINE=INNODB` pour définir le moteur utilisé pour la création d’une table, puis transférez les données dans la table compatible avant la migration.

   ```sql
   INSERT INTO innodb_table SELECT * FROM myisam_table ORDER BY primary_key_columns
   ```

## <a name="performance-recommendations-for-import-and-export"></a>Recommandations en matière de performances pour l’importation et l’exportation
-   Créez des index cluster et des clés primaires avant le chargement des données. Chargez les données dans l’ordre des clés primaires.
-   Reportez la création des index secondaires après le chargement des données. Créez tous les index secondaires après le chargement.
-   Désactivez les contraintes de clé étrangère avant le chargement. La désactivation des vérifications de clé étrangère offre d’importants gains de performances. Activez les contraintes et vérifiez les données après le chargement pour garantir l’intégrité référentielle.
-   Chargez les données en parallèle. Évitez tout parallélisme excessif, qui vous ferait atteindre une limite de ressources, et surveillez les ressources à l’aide des mesures disponibles dans le Portail Azure.
-   Utilisez des tables partitionnées le cas échéant.

## <a name="import-and-export-by-using-mysql-workbench"></a>Importer et exporter des données à l’aide de MySQL Workbench
Il existe deux façons d’exporter et d’importer des données dans MySQL Workbench. Chacune répond à un objectif différent.

> [!NOTE]
> Si vous ajoutez une connexion à un serveur unique ou flexible MySQL (préversion) sur MySQL Workbench, vérifiez les points suivants :
> - Pour un serveur unique MySQL, le nom d’utilisateur doit être au format « username@servername ».
> - Pour le serveur flexible MySQL, vous pouvez simplement utiliser 'username'. Si vous utilisez 'username@servername'pour vous connecter, la connexion échoue.

### <a name="table-data-export-and-import-wizards-from-the-object-browsers-context-menu"></a>Assistants d’importation et d’exportation de données de table accessibles à partir du menu contextuel de l’Explorateur d’objets
:::image type="content" source="./media/concepts-migrate-import-export/p1.png" alt-text="Assistants MySQL Workbench accessibles à partir du menu contextuel de l’Explorateur d’objets":::

Les Assistants pour les données de table prennent en charge les opérations d’importation et d’exportation à l’aide de fichiers CSV et JSON. Ils offrent plusieurs options de configuration, notamment les séparateurs, la sélection de colonne et la sélection du codage. Vous pouvez exécuter chaque Assistant sur des serveurs MySQL locaux ou connectés à distance. L’action d’importation inclut le mappage de table, de colonne et de type.

Vous pouvez accéder ces Assistants à partir du menu contextuel de l’Explorateur d’objets en cliquant sur une table avec le bouton droit. Choisissez ensuite **Table Data Export Wizard** (Assistant Exportation de données de table) ou **Table Data Import Wizard** (Assistant Importation de données de table).

#### <a name="table-data-export-wizard"></a>Assistant Exportation de données de table
L’exemple suivant exporte la table dans un fichier CSV :
1. Cliquez avec le bouton droit sur la table de la base de données à exporter.
2. Sélectionnez **Assistant Exportation de données de table**. Sélectionnez les colonnes à exporter, le décalage de ligne (le cas échéant) et le nombre (le cas échéant).
3. Sur la page **Select data for export** (Sélectionner les données à exporter), cliquez sur **Next** (Suivant). Sélectionnez le chemin d’accès et le type de fichier CSV ou JSON. Sélectionnez également le séparateur de ligne, les caractères entourant les chaînes et le séparateur de champs.
4. Sur la page **Select output file location** (Sélectionner l’emplacement du fichier de sortie), cliquez sur **Next** (Suivant).
5. Sur la page **Export data** (Exporter les données), cliquez sur **Next** (Suivant).

#### <a name="table-data-import-wizard"></a>Assistant Importation de données de table
L’exemple suivant importe la table à partir d’un fichier CSV :
1. Cliquez avec le bouton droit sur la table de la base de données à importer.
2. Recherchez et sélectionnez le fichier CSV à importer, puis cliquez sur le bouton **Next** (Suivant).
3. Sélectionnez la table de destination (nouvelle ou existante), puis activez ou désactivez la case à cocher **Truncate table before import** (Tronquer la table avant l’importation). Cliquez sur **Suivant**.
4. Sélectionnez le codage et les colonnes à importer, puis cliquez sur **Next** (Suivant).
5. Sur la page **Import data** (Importer les données), cliquez sur **Next** (Suivant). L’Assistant importe les données en conséquence.

### <a name="sql-data-export-and-import-wizards-from-the-navigator-pane"></a>Assistants d’exportation et d’importation de données SQL accessibles à partir du volet Navigator (Navigateur)
Utilisez un Assistant pour exporter ou importer des données SQL générées à partir de MySQL Workbench ou avec la commande mysqldump. Accédez à ces Assistants à partir du volet **Navigator** (Navigateur) ou en sélectionnant **Server** (Serveur) dans le menu principal. Puis, sélectionnez **Data Export** (Exportation des données) ou **Data Import** (Importation de données).

#### <a name="data-export"></a>Exportation de données
:::image type="content" source="./media/concepts-migrate-import-export/p2.png" alt-text="Exportation de données MySQL Workbench à l’aide du volet Navigator (Navigateur)":::

Vous pouvez utiliser l’onglet **Data Export** (Exportation des données) pour exporter vos données MySQL.
1. Sélectionnez chaque schéma que vous voulez exporter, choisissez éventuellement des tables/objets spécifiques pour chacun et générez l’exportation. Parmi les options de configuration figurent la possibilité d’exporter dans un dossier de projet ou un fichier SQL autonome, de vider les événements et les routines stockés et d’ignorer les données de table.

   Vous pouvez également utiliser l’option **Export a Result Set** (Exporter un jeu de résultats) pour exporter un jeu de résultats spécifique de l’éditeur SQL vers un autre format, par exemple CSV, JSON, HTML ou XML.
3. Sélectionnez les objets de base de données à exporter et configurez les options associées.
4. Cliquez sur **Actualiser** pour charger les objets actifs.
5. Vous pouvez également ouvrir l’onglet **Advanced Options** (Options avancées) pour affiner l’opération d’exportation. Par exemple, ajoutez des verrous de table, utilisez des instructions replace à la place des instructions insert et placez les identificateurs entre accents graves.
6. Cliquez sur **Start Export** (Démarrer l’exportation) pour commencer le processus d’exportation.


#### <a name="data-import"></a>Importation de données
:::image type="content" source="./media/concepts-migrate-import-export/p3.png" alt-text="Importation de données MySQL Workbench à l’aide du navigateur de gestion":::

Vous pouvez utiliser l’onglet **Data Import** (Importation des données) pour importer ou restaurer les données exportées avec l’opération d’exportation de données ou la commande mysqldump.
1. Choisissez le dossier du projet ou un fichier SQL autonome, sélectionnez le schéma dans lequel les données seront importées ou cliquez sur **New** (Nouveau) pour définir un nouveau schéma.
2. Cliquez sur **Start Import** (Démarrer l’importation) pour commencer le processus d’importation.

## <a name="next-steps"></a>Étapes suivantes
- Pour découvrir une autre approche de migration, consultez [Migrer une base de données MySQL à l’aide des images mémoire et de la restauration dans la base de données Azure pour MySQL](concepts-migrate-dump-restore.md).
- Pour plus d’informations sur la migration de bases de données vers Azure Database pour MySQL, consultez le [Guide de migration des bases de données](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide).