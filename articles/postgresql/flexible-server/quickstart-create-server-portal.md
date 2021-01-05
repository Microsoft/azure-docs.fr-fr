---
title: 'Démarrage rapide : Créer un serveur - Portail Azure - Azure Database pour PostgreSQL - Serveur flexible'
description: Démarrage rapide pour la création et la gestion d’une base de données Azure pour PostgreSQL - Serveur flexible à l’aide de l’interface utilisateur du portail Azure.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.custom: mvc
ms.topic: quickstart
ms.date: 09/22/2020
ms.openlocfilehash: 978fcb8c8cf9d2ffd929da5b76d7fd2ff109f420
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92535856"
---
# <a name="quickstart-create-an-azure-database-for-postgresql---flexible-server-in-the-azure-portal"></a>Démarrage rapide : Créer une instance Azure Database pour PostgreSQL - Serveur flexible dans le portail Azure

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en préversion

Azure Database pour PostgreSQL est un service administré que vous utilisez pour exécuter, gérer et mettre à l’échelle des bases de données PostgreSQL hautement disponibles dans le cloud. Ce démarrage rapide vous montre comment créer en quelques minutes une instance Azure Database pour PostgreSQL - Serveur flexible à l’aide du portail Azure.

Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Ouvrez votre navigateur web et accédez au [portail](https://portal.azure.com/). Entrez vos informations d’identification pour vous connecter au portail. Il s’ouvre par défaut sur le tableau de bord des services.

## <a name="create-an-azure-database-for-postgresql-server"></a>Créer un serveur Azure Database pour PostgreSQL

Un serveur Azure Database pour PostgreSQL est créé. Il contient un ensemble configuré de [ressources de calcul et de stockage](./concepts-compute-storage.md). Ce serveur est créé dans un [groupe de ressources Azure](../../azure-resource-manager/management/overview.md).

Pour créer un serveur Azure Database pour PostgreSQL, suivez ces étapes :

1. Sélectionnez **Créer une ressource** (+) dans l’angle supérieur gauche du portail.

2. Sélectionnez **Bases de données** > **Azure Database pour PostgreSQL** .

    :::image type="content" source="./media/quickstart-create-database-portal/1-create-database.png" alt-text="Azure Database pour PostgreSQL dans le menu":::

3. Sélectionnez l’option de déploiement **Serveur flexible** .

   :::image type="content" source="./media/quickstart-create-database-portal/2-select-deployment-option.png" alt-text="Azure Database pour PostgreSQL dans le menu":::

4. Remplissez le formulaire **Informations de base** avec les informations suivantes :

    :::image type="content" source="./media/quickstart-create-database-portal/3-create-basics.png" alt-text="Azure Database pour PostgreSQL dans le menu":::

    Paramètre|Valeur suggérée|Description
    ---|---|---
    Abonnement|Votre nom d’abonnement|L’abonnement Azure que vous souhaitez utiliser pour votre serveur. Si vous disposez de plusieurs abonnements, choisissez celui dans lequel vous souhaitez que la ressource soit facturée.
    Resource group|*myresourcegroup*| Un nouveau nom de groupe de ressources ou un nom de groupe existant dans votre abonnement.
    Nom du serveur |*mydemoserver*|Un nom unique qui identifie votre serveur Azure Database pour PostgreSQL. Le nom de domaine *postgres.database.azure.com* est ajouté au nom de serveur que vous fournissez. Le serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit contenir entre 3 et 63 caractères.
    Nom d’utilisateur administrateur |*myadmin*| Votre propre compte de connexion en tant que compte à utiliser lorsque vous vous connectez au serveur. Le nom de connexion de l’administrateur ne doit pas être **azure_superuser** , **azure_pg_admin** , **admin** , **administrator** , **root** , **guest** ou **public** . Il ne peut pas commencer par **pg_** .
    Mot de passe |Votre mot de passe| Un nouveau mot de passe pour le compte Administrateur du serveur. Il doit contenir entre 8 et 128 caractères. Votre mot de passe doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres (0 à 9) et caractères non alphanumériques (!, $, #, %, etc.).
    Emplacement|La région la plus proche de vos utilisateurs| L’emplacement géographique le plus proche de vos utilisateurs.
    Version|La version principale la plus récente| La version principale PostgreSQL la plus récente, sauf si vous avez des exigences précises.
    Calcul + stockage | **Usage général** , **4 vCores** , **512 Go** , **7 jours** | Les configurations de calcul, de stockage et de sauvegarde pour votre nouveau serveur. Sélectionnez **Configurer le serveur** . *Usage général* , *4 vCores* , *512 Go* et *7 jours* sont les valeurs par défaut pour **Niveau de calcul** , **vCore** , **Stockage** et **Période de rétention des sauvegardes** . Vous pouvez laisser ces curseurs en l’état ou les ajuster. Pour enregistrer cette sélection du niveau tarifaire, sélectionnez **OK** . La capture d’écran suivante capture ces sélections.

    :::image type="content" source="./media/quickstart-create-database-portal/4-pricing-tier.png" alt-text="Azure Database pour PostgreSQL dans le menu":::
    
5. Options de configuration de la mise en réseau

    Sous l'onglet Réseau, vous pouvez choisir la façon dont votre serveur est accessible. Azure Database pour PostgreSQL crée un pare-feu au niveau du serveur. Il empêche les applications et les outils externes de se connecter au serveur et à toute base de données sur le serveur, sauf si vous créez une règle de pare-feu pour ouvrir le pare-feu à des adresses IP spécifiques. Nous vous recommandons de rendre le serveur accessible publiquement :

    :::image type="content" source="./media/quickstart-create-database-portal/5-networking.png" alt-text="Azure Database pour PostgreSQL dans le menu":::

    Puis de le limiter à votre propre adresse IP du client :

    :::image type="content" source="./media/quickstart-create-database-portal/6-add-client-ip.png" alt-text="Azure Database pour PostgreSQL dans le menu":::

6. Sélectionnez **Vérifier + créer** pour passer en revue vos sélections. Sélectionnez **Créer** pour approvisionner le serveur. Cette opération peut prendre quelques minutes.

7. Dans la barre d’outils, sélectionnez l’icône **Notifications** (en forme de cloche) pour surveiller le processus de déploiement. Une fois le déploiement terminé, vous pouvez sélectionner **Épingler au tableau de bord** , ce qui crée une vignette pour ce serveur dans votre tableau de bord du portail Azure, en tant que raccourci vers la page **Vue d’ensemble** du serveur. En sélectionnant **Accéder à la ressource** , ouvrez la page **Vue d’ensemble** du serveur.

    :::image type="content" source="./media/quickstart-create-database-portal/7-notifications.png" alt-text="Azure Database pour PostgreSQL dans le menu":::

   Par défaut, la création d’une base de données **postgres** s’effectue sous votre serveur. La base de données [postgres](https://www.postgresql.org/docs/12/static/app-initdb.html) est une base de données par défaut qui est destinée aux utilisateurs, aux utilitaires et aux applications tierces. (L’autre base de données par défaut est **azure_maintenance** . Son rôle consiste à séparer les processus de service administré des actions des utilisateurs. Vous ne pouvez pas accéder à cette base de données.)

    > [!NOTE]
    > Les connexions à votre serveur de base de données Azure pour PostgreSQL communiquent via le port 5432. Quand vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 5432 peut être bloqué par le pare-feu de votre réseau. Si c’est le cas, vous ne pouvez pas vous connecter à votre serveur, sauf si votre service informatique ouvre le port 5432.
    >

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Lorsque vous créez votre serveur Azure Database pour PostgreSQL, une base de données par défaut nommée **postgres** est créée. Pour vous connecter à votre serveur de base de données, vous avez besoin de son nom de serveur complet et des informations d’identification de connexion d’administrateur. Il est possible que vous ayez noté ces valeurs précédemment, dans l’article se rapportant au guide de démarrage rapide. Si vous ne l’avez pas fait, vous pouvez facilement localiser le nom du serveur et les informations de connexion dans la page **Vue d’ensemble** du serveur sur le portail Azure.

Ouvrez la page **Vue d’ensemble** de votre serveur. Prenez note du **Nom du serveur** et du **Nom de connexion de l’administrateur du serveur** . Placez le curseur sur chaque champ, afin de faire apparaître le symbole de copie à droite du texte. Sélectionnez ce symbole pour copier les valeurs qui vous intéressent.

 :::image type="content" source="./media/quickstart-create-database-portal/8-server-name.png" alt-text="Azure Database pour PostgreSQL dans le menu":::

## <a name="connect-to-the-postgresql-database-using-psql"></a>Se connecter à la base de données PostgreSQL avec psql

Vous pouvez utiliser différentes applications pour vous connecter à votre serveur de base de données Azure pour PostgreSQL. Si votre ordinateur client a PostgreSQL installé, vous pouvez utiliser une instance locale de [psql](https://www.postgresql.org/docs/current/static/app-psql.html) pour vous connecter à un serveur Azure PostgreSQL. Nous allons maintenant utiliser l’utilitaire de ligne de commande psql pour nous connecter au serveur Azure pour PostgreSQL.

1. Exécutez la commande psql suivante pour vous connecter à un serveur Azure Database pour PostgreSQL

   ```bash
   psql --host=<servername> --port=<port> --username=<user> --dbname=<dbname>
   ```

   Par exemple, la commande suivante se connecte à la base de données par défaut appelée **postgres** sur votre serveur PostgreSQL **mydemoserver.postgres.database.azure.com** à l’aide des informations d’identification d’accès. Saisissez le `<server_admin_password>` que vous avez choisi à l’invite de mot de passe.
  
   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin --dbname=postgres
   ```

   Une fois connecté, l’utilitaire psql affiche une invite postgres dans laquelle vous pouvez taper des commandes sql. Il est possible qu’un avertissement s’affiche au niveau de la sortie de connexion initiale, car la version de l’utilitaire psql que vous utilisez peut être différente de celle du serveur Azure Database pour PostgreSQL.

   Exemple de sortie psql :

   ```bash
   psql (11.3, server 12.1)
   WARNING: psql major version 11, server major version 12.
            Some psql features might not work.
   SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
   Type "help" for help.

   postgres=>
   ```

   > [!TIP]
   > Si le pare-feu n’est pas configuré pour autoriser l’adresse IP de votre client, l’erreur suivante se produit :
   >
   > « psql: FATAL: aucune entrée pg_hba.conf pour l’hôte `<IP address>`, utilisateur « myadmin », base de données « postgres », SSL sur FATAL : Connexion SSL obligatoire. Spécifiez les options SSL puis réessayez.
   >
   > Confirmez que l'adresse IP de votre client est autorisée à l’étape des règles de pare-feu ci-dessus.

2. Créez une base de données vide nommée « mypgsqldb » à l’invite en tapant la commande suivante :

    ```bash
    CREATE DATABASE mypgsqldb;
    ```

3. À l’invite, exécutez la commande suivante pour basculer la connexion sur la base de données **mypgsqldb** nouvellement créée :

    ```bash
    \c mypgsqldb
    ```

4. Tapez `\q`, puis sélectionnez la touche Entrée pour quitter psql.

Vous vous êtes connecté au serveur Azure Database pour PostgreSQL via psql, et vous avez créé une base de données utilisateur vide.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous disposez au choix de deux moyens de supprimer les ressources que vous avez créées dans ce guide de démarrage rapide. Vous pouvez supprimer le groupe de ressources Azure qui inclut toutes les ressources du groupe de ressources. Si vous souhaitez conserver les autres ressources intactes, ne supprimez que la ressource de serveur.

> [!TIP]
> Les autres guides de démarrage rapide de cette collection reposent sur ce guide. Si vous souhaitez continuer à utiliser d’autres guides de démarrage rapide, ne supprimez pas les ressources créées au cours de ce guide. Sinon, suivez ces étapes pour les supprimer dans le portail.

Pour supprimer l’intégralité du groupe de ressources, y compris le serveur nouvellement créé :

1. Localisez votre groupe de ressources dans le portail. Dans le menu de gauche, sélectionnez **Groupes de ressources** . Sélectionnez ensuite le nom de votre groupe de ressources, **myresourcegroup** dans l’exemple.

2. Dans la page de votre groupe de ressources, sélectionnez **Supprimer** . Pour confirmer la suppression, dans la zone de texte, entrez le nom de votre groupe de ressources ( **myresourcegroup** dans l’exemple). Sélectionnez **Supprimer** .

Pour ne supprimer que le serveur nouvellement créé :

1. Localisez votre serveur dans le portail, s’il n’est pas ouvert. Dans le menu de gauche, sélectionnez **Toutes les ressources** . Recherchez ensuite le serveur que vous avez créé.

2. Dans la page **Vue d’ensemble** , sélectionnez **Supprimer** .

    :::image type="content" source="./media/quickstart-create-database-portal/9-delete.png" alt-text="Azure Database pour PostgreSQL dans le menu":::

3. Vérifiez le nom du serveur à supprimer et affichez les bases de données affectées situées sous celui-ci. Entrez votre nom de serveur dans la zone de texte, par exemple **mydemoserver** . Sélectionnez **Supprimer** .

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Déployer une application Django avec App Service et PostgreSQL](tutorial-django-app-service-postgres.md)