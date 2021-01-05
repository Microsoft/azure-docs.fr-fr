---
title: 'Tutoriel : Concevoir une base de données Azure Database for MariaDB - Portail Azure'
description: Ce didacticiel explique comment créer et gérer une base de données et un serveur Azure Database for MariaDB à l’aide du Portail Azure.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: tutorial
ms.date: 3/18/2020
ms.custom: mvc
ms.openlocfilehash: fe503d0e579e30b0c355d3be3e8d02b2bd7a44a8
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542437"
---
# <a name="tutorial-design-an-azure-database-for-mariadb-database-by-using-the-azure-portal"></a>Tutoriel : Créer une base de données Azure Database for MariaDB à l’aide du portail Azure

Azure Database for MariaDB est un service géré que vous pouvez utiliser pour exécuter, gérer et mettre à l’échelle des bases de données MySQL hautement disponibles dans le cloud. À l’aide du Portail Azure, vous pouvez facilement gérer votre serveur et créer une base de données.

Ce didacticiel vous montre comment utiliser le portail Azure pour :

> [!div class="checklist"]
> * Créer une base de données Azure Database for MariaDB
> * Configurer le pare-feu du serveur
> * Utiliser l’outil en ligne de commande mysql pour créer une base de données
> * Charger un exemple de données
> * Interroger des données
> * Mettre à jour des données
> * Restaurer des données

Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans votre navigateur, accédez au [Portail Azure](https://portal.azure.com/). Entrez vos informations d’identification pour vous connecter au portail. Il s’ouvre par défaut sur le tableau de bord des services.

## <a name="create-an-azure-database-for-mariadb-server"></a>Créer un serveur Azure Database for MariaDB

Vous créez un serveur Azure Database for MariaDB avec un ensemble défini de [ressources de calcul et de stockage](concepts-pricing-tiers.md). Le serveur est créé dans un [groupe de ressources Azure](../azure-resource-manager/management/overview.md).

1. Cliquez sur le bouton **Créer une ressource** (+) dans le coin supérieur gauche du portail.

2. Sélectionnez **Bases de données** > **Azure Database for MariaDB**. Vous pouvez également taper **MariaDB** dans la zone de recherche pour trouver le service.

   ![Accéder à MySQL](./media/tutorial-design-database-using-portal/1-Navigate-to-mariadb.png)

3. Sélectionnez la vignette **Azure Database for MariaDB**. Entrez ou sélectionnez les informations nécessaires.

   ![Créer un formulaire](./media/tutorial-design-database-using-portal/2-create-form.png)

    Paramètre | Valeur suggérée | Description du champ
    ---|---|---
    Nom du serveur | *un nom de serveur unique* | Choisissez un nom unique qui identifie votre serveur Azure Database for MariaDB. Par exemple, **mydemoserver**. Le nom de domaine *.mariadb.database.azure.com* est ajouté au nom du serveur que vous entrez. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit contenir entre 3 et 63 caractères.
    Abonnement | *votre abonnement* | Sélectionnez l’abonnement Azure que vous souhaitez utiliser pour votre serveur. Si vous avez plusieurs abonnements, choisissez celui dans lequel la ressource est facturée.
    Resource group | **myresourcegroup** | Entrez le nom d’un nouveau groupe de ressources, ou sélectionnez un groupe de ressources existant.
    Sélectionner une source | **Vide** | Sélectionnez **Vide** pour créer un serveur. (Si vous créez un serveur à partir d’une sauvegarde géographique d’un serveur Azure Database for MariaDB existant, sélectionnez **Sauvegarde** ).
    Connexion d’administrateur serveur | **myadmin** | Un compte de connexion à utiliser lorsque vous vous connectez au serveur. Le nom de connexion d’administrateur ne peut pas être **azure_superuser** , **admin** , **administrator** , **root** , **guest** ou **public**.
    Mot de passe | *votre choix* | Entrez un nouveau mot de passe pour le compte d’administrateur du serveur. Il doit contenir entre 8 et 128 caractères. Votre mot de passe doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres (0-9) et caractères non alphanumériques (comme !, $, #, %, etc.).
    Confirmer le mot de passe | *votre choix*| Confirmez le mot de passe du compte d’administrateur.
    Emplacement | *La région la plus proche de vos utilisateurs*| Sélectionnez l’emplacement le plus proche de vos utilisateurs ou de vos autres applications Azure.
    Version | *La version la plus récente*| La version la plus récente (sauf si vous avez des exigences spécifiques impliquant d’utiliser une autre version).
    Niveau tarifaire | Consultez la description. | Les configurations de calcul, de stockage et de sauvegarde pour votre nouveau serveur. Sélectionnez **Niveau tarifaire** > **Usage général**. Conservez les valeurs par défaut des paramètres suivants :<br><ul><li>**Génération de calcul** (Gen 5)</li><li>**vCore** (4 vCores)</li><li>**Stockage** (100 Go)</li><li>**Période de rétention de sauvegarde** (7 jours)</li></ul><br>Pour activer les sauvegardes de votre serveur dans le stockage géoredondant, sélectionnez **Géographiquement redondant** pour **Options de redondance de sauvegarde**. <br><br>Pour enregistrer cette sélection du niveau tarifaire, sélectionnez **OK**. La capture d’écran suivante capture ces sélections.
    
   ![Niveau tarifaire](./media/tutorial-design-database-using-portal/3-pricing-tier.png)

   > [!TIP]
   > Lorsque la **croissance automatique** est activée, votre serveur augmente le stockage quand vous vous approchez de la limite allouée, sans impacter votre charge de travail.

4. Cliquez sur **Vérifier + créer**. Dans la barre d’outils, cliquez sur le bouton **Notifications** pour superviser le processus de déploiement. Le déploiement peut prendre jusqu’à 20 minutes.

## <a name="configure-the-firewall"></a>Configurer le pare-feu

Une base de données Azure Database for MariaDB est protégée par un pare-feu. Par défaut, toutes les connexions au serveur et aux bases de données du serveur sont rejetées. Avant de vous connecter à Azure Database for MariaDB pour la première fois, configurez le pare-feu afin d’ajouter l’adresse IP (ou la plage d’adresses IP) du réseau public de l’ordinateur client.

1. Sélectionnez le serveur qui vient d’être créé, puis **Sécurité de la connexion**.

   ![Sécurité de la connexion](./media/tutorial-design-database-using-portal/1-Connection-security.png)
2. Vous pouvez sélectionner **Ajouter mon adresse IP** ou configurer les règles de pare-feu ici. Pensez à sélectionner **Enregistrer** après avoir créé les règles.

Vous pouvez maintenant vous connecter au serveur en utilisant l’outil en ligne de commande mysql ou MySQL Workbench.

> [!TIP]
> Le serveur Azure Database for MariaDB communique via le port 3306. Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 3306 peut être bloqué par le pare-feu de votre réseau. Dans ce cas, pour vous connecter au serveur Azure Database for MariaDB, votre service informatique doit ouvrir le port 3306.

## <a name="get-connection-information"></a>Obtenir des informations de connexion

Récupérez les valeurs des champs **Nom du serveur** (complet) et **Nom de connexion d’administrateur du serveur** pour votre serveur Azure Database for MariaDB à partir du Portail Azure. Vous utilisez le nom du serveur complet pour vous connecter à votre serveur avec l’outil en ligne de commande mysql.

1. Dans le menu de gauche du [Portail Azure](https://portal.azure.com/), sélectionnez **Toutes les ressources**. Entrez le nom du serveur et recherchez votre serveur Azure Database for MariaDB. Sélectionnez le nom du serveur pour afficher les détails de celui-ci.

2. Sur la page **Vue d’ensemble** , notez les valeurs des champs **Nom du serveur** et **Nom de connexion de l’administrateur du serveur**. Vous pouvez également cliquer sur le bouton de **copie** en regard de chaque champ pour copier la valeur dans le Presse-papiers.

   ![Propriétés du serveur](./media/tutorial-design-database-using-portal/2-server-properties.png)

Dans notre exemple, le nom du serveur est **mydemoserver.mariadb.database.azure.com** et le nom de connexion d’administrateur du serveur est **myadmin\@mydemoserver**.

## <a name="connect-to-the-server-by-using-mysql"></a>Se connecter au serveur à l’aide de mysql

Utilisez [l’outil en ligne de commande mysql](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) pour établir une connexion à votre serveur Azure Database for MariaDB. Vous pouvez exécuter l’outil en ligne de commande mysql depuis Azure Cloud Shell dans le navigateur ou depuis votre propre ordinateur à l’aide des outils mysql installés localement. Pour ouvrir Azure Cloud Shell, cliquez sur le bouton **Try It** (Essayer) dans un bloc de code de cet article, ou accédez au Portail Azure et cliquez sur l’icône **>_** dans la barre d’outils en haut à droite.

Saisissez la commande pour vous connecter :

```bash
mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
```

## <a name="create-a-blank-database"></a>Créer une base de données vide

Une fois que vous êtes connecté au serveur, créez une base de données vide sur laquelle travailler :

```sql
CREATE DATABASE mysampledb;
```

À l’invite, exécutez la commande suivante pour basculer la connexion sur votre base de données nouvellement créée :

```sql
USE mysampledb;
```

## <a name="create-tables-in-the-database"></a>Créer des tables dans la base de données

Une fois que vous savez comment vous connecter à la base de données Azure Database for MariaDB, vous pouvez effectuer certaines tâches de base.

Tout d’abord, créez une table et chargez-y des données. Nous allons créer une table qui stocke des données d’inventaire :

```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

## <a name="load-data-in-the-tables"></a>Charger des données dans les tables

Maintenant que vous disposez d’une table, insérez-y des données. Dans la fenêtre d’invite de commandes ouverte, exécutez la requête suivante pour insérer des lignes de données :

```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

## <a name="query-and-update-the-data-in-the-tables"></a>Interroger et mettre à jour les données des tables

Exécutez la requête suivante pour récupérer des informations à partir de la table de base de données :

```sql
SELECT * FROM inventory;
```

Vous pouvez également mettre à jour les données des tables :

```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

La ligne est mise à jour lorsque vous récupérez des données :

```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>Restaurer une version antérieure d’une base de données

Imaginez que vous avez accidentellement supprimé une table de base de données importante et que vous ne pouvez pas récupérer les données facilement. Dans Azure Database for MariaDB, vous pouvez restaurer le serveur à un point dans le temps en créant une copie des bases de données sur votre nouveau serveur. Vous pouvez alors utiliser ce nouveau serveur pour récupérer les données supprimées. Les étapes suivantes restaurent l’exemple de serveur à un point dans le temps antérieur au moment où la table a été ajoutée :

1. Dans le Portail Azure, recherchez votre base de données Azure Database for MariaDB. Sur la page **Vue d’ensemble** , sélectionnez **Restaurer**.

   ![Restaurer une base de données](./media/tutorial-design-database-using-portal/1-restore-a-db.png)

2. Sur la page **Restaurer** , entrez ou sélectionnez les informations suivantes :

   ![Formulaire de restauration](./media/tutorial-design-database-using-portal/2-restore-form.png)
   
   - **Point de restauration** : sélectionnez un point dans le temps vers lequel vous souhaitez effectuer une restauration, dans la période listée. Veillez à convertir votre fuseau horaire local en fuseau horaire UTC.
   - **Restaurer sur un nouveau serveur**  : entrez le nom du nouveau serveur sur lequel effectuer la restauration.
   - **Emplacement** : la région est identique à celle du serveur source et ne peut pas être changée.
   - **Niveau tarifaire**  : le niveau tarifaire est identique à celui du serveur source et ne peut pas être changé.
   
3. Sélectionnez **OK** pour [restaurer le serveur à un point dans le temps](./howto-restore-server-portal.md) antérieur au moment de la suppression de la table. La restauration d’un serveur crée une copie du serveur à partir du point dans le temps que vous avez sélectionné. 

## <a name="next-steps"></a>Étapes suivantes
Ce didacticiel vous montre comment utiliser le portail Azure pour :

> [!div class="checklist"]
> * Créer une base de données Azure Database for MariaDB
> * Configurer le pare-feu du serveur
> * Utiliser l’outil en ligne de commande mysql pour créer une base de données
> * Charger un exemple de données
> * Interroger des données
> * Mettre à jour des données
> * Restaurer des données

> [!div class="nextstepaction"]
> [Guide pratique pour connecter des applications à Azure Database for MariaDB](howto-connection-string.md)
