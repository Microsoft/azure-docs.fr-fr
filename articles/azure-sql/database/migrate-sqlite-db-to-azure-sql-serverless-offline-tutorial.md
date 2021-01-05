---
title: 'Tutoriel : Guide pratique pour migrer votre base de données SQLite vers Azure SQL Database serverless'
description: Apprenez à effectuer une migration en mode hors connexion de SQLite vers Azure SQL Database serverless à l’aide d’Azure Data Factory.
services: sql-database
author: joplum
ms.author: joplum
ms.service: sql-database
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/08/2020
ms.custom: sqldbrb=1
ms.openlocfilehash: e2f240247cbba0f80254d504792df45be55c6a1b
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790404"
---
# <a name="how-to-migrate-your-sqlite-database-to-azure-sql-database-serverless"></a>Guide pratique pour migrer votre base de données SQLite vers Azure SQL Database serverless
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Pour de nombreuses personnes, SQLite constitue la première expérience des bases de données et de la programmation SQL. SQLite est inclus dans de nombreux systèmes d’exploitation et applications courants, ce qui en fait l’un des moteurs de base de données les plus largement déployés et utilisés dans le monde. Et comme il s’agit probablement du premier moteur de base de données que beaucoup de monde utilise, il finit souvent par être un élément central de projets ou d’applications. Dans les cas où un projet ou une application devient trop volumineux par rapport à l’implémentation initiale de SQLite, les développeurs peuvent avoir besoin de migrer leurs données vers un magasin de données centralisé et fiable.

Azure SQL Database serverless est un niveau de calcul pour les bases de données uniques qui met automatiquement à l’échelle les calculs en fonction de la demande en charge de travail, et facture la quantité de calcul utilisée par seconde. Le niveau de calcul serverless met aussi automatiquement en pause les bases de données pendant les périodes d’inactivité, quand seul le stockage est facturé, et reprend leur exécution automatiquement avec l’activité.

Une fois que vous aurez suivi les étapes ci-dessous, votre base de données sera migrée dans Azure SQL Database Serverless, ce qui vous permettra de mettre votre base de données à la disposition d’autres utilisateurs ou applications dans le cloud et de payer uniquement en fonction de votre utilisation, avec un minimum de modifications à apporter au code de l’application.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure
- La base de données SQLite2 ou SQLite3 que vous voulez migrer
- Un environnement Windows
  - Si vous n’avez pas d’environnement Windows local, vous pouvez utiliser une machine virtuelle Windows dans Azure pour la migration. Déplacez votre fichier de base de données SQLite et mettez-le à disposition sur la machine virtuelle à l’aide d’Azure Files et de l’Explorateur Stockage.

## <a name="steps"></a>Étapes

1. Provisionnez une nouvelle base de données Azure SQL Database dans le niveau de calcul Serverless.

    ![Capture d’écran du portail Azure montrant un exemple de provisionnement pour Azure SQL Database serverless](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/provision-serverless.png)

2. Vérifiez que votre fichier de base de données SQLite est disponible dans votre environnement Windows. Installez un pilote ODBC SQLite si vous n’en avez pas déjà un (il en existe un grand nombre en Open source, par exemple, http://www.ch-werner.de/sqliteodbc/).

3. Créez un nom de source de données système pour la base de données. Veillez à utiliser l’application Administrateur de source de données qui correspond à votre architecture système (32 bits ou 64 bits). La version que vous exécutez est indiquée dans vos paramètres système.

    - Ouvrez l’application Administrateur de source de données ODBC dans votre environnement
    - Cliquez sur l’onglet DSN système et cliquez sur Ajouter
    - Sélectionnez le connecteur ODBC SQLite que vous avez installé et donnez un nom explicite à la connexion, par exemple, sqlitemigrationsource
    - Attribuez le nom du fichier .db à la base de données
    - Enregistrez et quittez

4. Téléchargez et installez le runtime d’intégration autohébergé. Pour ce faire, la méthode la plus simple consiste à utiliser l’option d’installation rapide, comme indiqué dans la documentation. Si vous optez pour une installation manuelle, vous devez fournir à l’application une clé d’authentification, que vous pouvez trouver dans votre instance Data Factory comme suit :

    - Démarrez Azure Data Factory (Créer et Surveiller à partir du service sur le portail Azure)
    - Cliquez sur l’onglet Créer représenté par le crayon bleu à gauche
    - Cliquez sur Connexions en bas à gauche, puis sur Runtimes d’intégration
    - Ajoutez un nouveau runtime d’intégration auto-hébergé, donnez-lui un nom, sélectionnez *Option 2* .

5. Créez un service lié pour la base de données SQLite source dans votre Data Factory.

    ![Capture d’écran montrant le panneau Services liés vide dans Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/linked-services-create.png)

6. Dans **Connexions** , sous **Service lié** , cliquez sur **Nouveau** .

7. Recherchez et sélectionnez le connecteur ODBC

   ![Capture d’écran montrant le logo de connecteur ODBC dans le panneau Services liés dans Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/linked-services-odbc.png)

8. Donnez un nom explicite au service lié, par exemple, « sqlite_odbc ». Sélectionnez votre runtime d’intégration dans la liste déroulante Se connecter via le runtime d’intégration. Entrez la chaîne de connexion ci-dessous en remplaçant la variable Initial Catalog par le chemin du fichier .db et DSN par le nom de la connexion du nom de source de données système :

   ```
   Connection string: Provider=MSDASQL.1;Persist Security Info=False;Mode=ReadWrite;Initial Catalog=C:\sqlitemigrationsource.db;DSN=sqlitemigrationsource
    ```

9. Définissez le type d’authentification sur Anonyme.

10. Tester la connexion

    ![Capture d’écran montrant une connexion établie dans Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/linked-services-test-successful.png)

11. Créez un autre service lié pour votre cible SQL Serverless. Sélectionnez la base de données à l’aide de l’Assistant Service lié et fournissez les informations d’identification de l’authentification SQL.

    ![Capture d’écran montrant Azure SQL Database sélectionné dans Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/linked-services-create-target.png)

12. Extrayez les instructions CREATE TABLE de votre base de données SQLite. Pour ce faire, vous pouvez exécuter le script Python ci-dessous sur votre fichier de base de données.

    ```
    #!/usr/bin/python
    import sqlite3
    conn = sqlite3.connect("sqlitemigrationsource.db")
    c = conn.cursor()

    print("Starting extract job..")
    with open('CreateTables.sql', 'w') as f:
        for tabledetails in c.execute("SELECT * FROM sqlite_master WHERE type='table'"):
            print("Extracting CREATE statement for " + (str(tabledetails[1])))
            print(tabledetails)
            f.write(str(tabledetails[4].replace('\n','') + ';\n'))
    c.close()
    ```

13. Créez les tables de destination dans votre environnement cible SQL Serverless en copiant les instructions CREATE TABLE à partir du fichier CreateTables.sql et en exécutant les instructions SQL dans l’Éditeur de requête du portail Azure.

14. Revenez à l’écran d’accueil de Data Factory et cliquez sur Copier les données pour exécuter l’Assistant de création de tâche.

    ![Capture d’écran montrant le logo de l’Assistant Copier des données dans Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/copy-data.png)

15. Sélectionnez toutes les tables de la base de données SQLite source en cochant les cases correspondantes, puis mappez-les aux tables cibles dans SQL Azure. Une fois la tâche exécutée, la migration de vos données de SQLite vers Azure SQL est terminée !

## <a name="next-steps"></a>Étapes suivantes

- Pour démarrer, consultez [Démarrage rapide : Créez une base de données unique dans Azure SQL Database à l’aide du portail Azure](single-database-create-quickstart.md).
- Pour les limites de ressources, consultez [Limites des ressources du niveau de calcul serverless](./resource-limits-vcore-single-databases.md#general-purpose---serverless-compute---gen5)