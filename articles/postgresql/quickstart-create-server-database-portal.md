---
title: 'Démarrage rapide : Créer un serveur – Portail Azure – Azure Database pour PostgreSQL – serveur unique'
description: Dans ce guide de démarrage rapide, vous allez créer et gérer un serveur Azure Database pour PostgreSQL en utilisant le portail Azure.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.custom: mvc
ms.topic: quickstart
ms.date: 10/18/2020
ms.openlocfilehash: 000ab3e3911c65554622a48d34abda79d60411df
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492384"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-server-by-using-the-azure-portal"></a>Démarrage rapide : Créer un serveur Azure Database pour PostgreSQL en utilisant le portail Azure

Azure Database pour PostgreSQL est un service administré que vous utilisez pour exécuter, gérer et mettre à l’échelle des bases de données PostgreSQL hautement disponibles dans le cloud. Ce guide de démarrage rapide vous montre comment créer un serveur unique Azure Database pour PostgreSQL et comment vous y connecter ensuite.

## <a name="prerequisites"></a>Prérequis
Un abonnement Azure est requis. Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-an-azure-database-for-postgresql-server"></a>Créer un serveur Azure Database pour PostgreSQL
Accédez au [portail Azure](https://portal.azure.com/) pour créer une base de données sur le serveur unique Azure Database pour PostgreSQL. Recherchez et sélectionnez *Serveurs Azure Database pour PostgreSQL*.

>[!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstart-create-database-portal/search-postgres.png" alt-text="Recherchez Azure Database pour PostgreSQL.":::

1. Sélectionnez **Ajouter**.

2. Dans la page Créer une base de données Azure Database pour PostgreSQL, sélectionnez **Serveur unique**.

    >[!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-database-portal/select-single-server.png" alt-text="Sélectionner un serveur unique":::

3. Maintenant, dans le formulaire **Informations de base**, entrez les renseignements suivants.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-database-portal/create-basics.png" alt-text="Capture d’écran représentant l’onglet Informations de base pour la création d’un serveur unique.":::

   |Paramètre|Valeur suggérée|Description|
   |:---|:---|:---|
   |Abonnement|Le nom de votre abonnement|Sélectionnez l’abonnement Azure souhaité.|
   |Resource group|*myresourcegroup*| Un groupe de ressources nouveau ou existant dans votre abonnement.|
   |Nom du serveur |*mydemoserver*|Un nom unique qui identifie votre serveur Azure Database pour PostgreSQL. Le nom de domaine *postgres.database.azure.com* est ajouté au nom de serveur que vous fournissez. Le serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit contenir entre 3 et 63 caractères.|
   |Source de données | Aucun | Sélectionnez **Aucun** pour créer un serveur en partant de zéro. Sélectionnez **Sauvegarde** uniquement si vous effectuez une restauration à partir d’une sauvegarde géographique d’un serveur existant.|
   |Nom d’utilisateur administrateur |*myadmin*| Entrez le nom d’utilisateur de l’administrateur de serveur. Ce nom ne peut pas commencer par **pg_** , et ces valeurs ne sont pas autorisées : **azure_superuser**, **azure_pg_admin**, **admin**, **administrator**, **root**, **guest** ou **public**.|
   |Mot de passe |votre mot de passe| Un nouveau mot de passe pour l’utilisateur administrateur de serveur. Il doit contenir entre 8 et 128 caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres (0 à 9) et caractères non alphanumériques (par exemple, !, $, #, %).|
   |Emplacement|L’emplacement souhaité| sélectionnez un emplacement dans la liste déroulante.|
   |Version|La version principale la plus récente| La version principale PostgreSQL la plus récente, sauf si vous avez des exigences précises.|
   |Calcul + stockage | *utiliser les valeurs par défaut*| Le niveau tarifaire par défaut est **Usage général** avec **4 vCores** et **100 Go** de stockage. La conservation des sauvegardes est définie sur **7 jours** avec l’option de sauvegarde **Géographiquement redondant**.<br/>Renseignez-vous sur les [tarifs](https://azure.microsoft.com/pricing/details/postgresql/server/) et modifiez les valeurs par défaut au besoin.|


   > [!NOTE]
   > Choisissez le niveau tarifaire De base si votre charge de travail n’a pas besoin d’une grande capacité de calcul et d’E/S. Notez que les serveurs créés avec le niveau tarifaire De base ne peuvent plus ensuite être mis à l’échelle vers le niveau Usage général ou Mémoire optimisée.

5. Sélectionnez **Vérifier + créer** pour passer en revue vos sélections. Sélectionnez **Créer** pour approvisionner le serveur. Cette opération peut prendre quelques minutes.
    > [!NOTE]
    > Une base de données vide, **postgres**, est créée. Vous trouvez également une base de données, **azure_maintenance**, qui est utilisée pour séparer les processus de services managés des actions de l’utilisateur. Vous ne pouvez pas accéder à la base de données **azure_maintenance**.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstart-create-database-portal/deployment-success.png" alt-text="Réussir le déploiement.":::

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/postgres-doc-feedback)

## <a name="configure-a-firewall-rule"></a>Configurer une règle de pare-feu
Par défaut, le serveur que vous créez n’est pas accessible publiquement. Vous devez accorder des autorisations d’accès à votre adresse IP. Accédez à votre ressource de serveur dans le portail Azure, puis sélectionnez **Sécurité de la connexion** dans le menu de gauche pour votre ressource de serveur. Si vous ne savez pas comment trouver votre ressource, consultez [Ouvrir des ressources](../azure-resource-manager/management/manage-resources-portal.md#open-resources).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstart-create-database-portal/add-current-ip-firewall.png" alt-text="Capture d’écran montrant les règles de pare-feu pour la sécurité de la connexion.":::

Sélectionnez **Ajouter l’adresse IP actuelle du client**, puis sélectionnez **Enregistrer**. Vous pouvez ajouter d’autres adresses IP ou fournir une plage d’adresses IP pour permettre la connexion à votre serveur à partir de ces adresses IP. Pour plus d’informations, consultez [Règles de pare-feu d’Azure Database pour PostgreSQL](./concepts-firewall-rules.md).

> [!NOTE]
> Afin d’éviter des problèmes de connectivité, vérifiez que votre réseau autorise le trafic sortant sur le port 5432. Azure Database pour PostgreSQL utilise ce port.

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/postgres-doc-feedback)

## <a name="connect-to-the-server-with-psql"></a>Se connecter au serveur avec psql

Vous pouvez utiliser [psql](http://postgresguide.com/utilities/psql.html) ou [pgAdmin](https://www.pgadmin.org/docs/pgadmin4/latest/connecting.html), qui sont des clients PostgreSQL populaires. Pour ce guide de démarrage rapide, nous allons nous connecter en utilisant psql dans [Azure Cloud Shell](../cloud-shell/overview.md) par le biais du portail Azure.

1. Notez le nom du serveur, le nom de connexion de l’administrateur du serveur, le mot de passe et l’ID d’abonnement du serveur venant d’être créé à la section **Vue d’ensemble** de votre serveur.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-database-portal/overview-new.png" alt-text="Obtenir des informations de connexion.":::


2. Ouvrez Azure Cloud Shell dans le portail en sélectionnant l’icône située en haut à gauche.

   > [!NOTE]
   > Si vous ouvrez Cloud Shell pour la première fois, vous voyez s’afficher une invite permettant de créer un groupe de ressources et un compte de stockage. Il s’agit d’une étape unique, et ces ressources sont automatiquement jointes pour toutes les sessions.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="media/quickstart-create-database-portal/use-in-cloud-shell.png" alt-text="Capture d’écran affichant des informations sur le serveur et l’icône d’ouverture d’Azure Cloud Shell.":::

3. Exécutez la commande suivante dans le terminal Azure Cloud Shell. Remplacez les valeurs par le nom réel du serveur et le nom réel de la connexion d’utilisateur administrateur. Utilisez la base de données vide **postgres** avec l’utilisateur administrateur au format suivant : `<admin-username>@<servername>`.

   ```azurecli-interactive
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

   Voici comment se présente l’expérience utilisateur dans le terminal Cloud Shell :

   ```bash
    Requesting a Cloud Shell.Succeeded.
    Connecting terminal...

    Welcome to Azure Cloud Shell

    Type "az" to use Azure CLI
    Type "help" to learn about Cloud Shell

    user@Azure:~$psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
    Password for user myadmin@mydemoserver.postgres.database.azure.com:
    psql (12.2 (Ubuntu 12.2-2.pgdg16.04+1), server 11.6)
    SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
    Type "help" for help.

    postgres=>
    ```
4. Dans le même terminal Azure Cloud Shell, créez une base de données appelée **guest**.

   ```bash
   postgres=> CREATE DATABASE guest;
   ```

5. Faites basculer les connexions vers la base de données **guest** qui vient d’être créée.

   ```bash
   \c guest
   ```
6. Tapez `\q`, puis appuyez sur la touche Entrée pour fermer psql.

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/postgres-doc-feedback)

## <a name="clean-up-resources"></a>Nettoyer les ressources
Vous avez réussi à créer un serveur Azure Database pour PostgreSQL dans un groupe de ressources. Si vous ne pensez pas avoir besoin de ces ressources à l’avenir, vous pouvez vous en débarrasser en supprimant le groupe de ressources ou le serveur PostgreSQL.

Pour supprimer le groupe de ressources :

1. Depuis le portail Azure, recherchez et sélectionnez **Groupes de ressources**.
2. Dans la liste des groupes de ressources, choisissez le nom de votre groupe de ressources.
3. Dans la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
4. Dans la boîte de dialogue de confirmation, entrez le nom de votre groupe de ressources, puis sélectionnez **Supprimer**.

Pour supprimer le serveur, sélectionnez le bouton **Supprimer** sur la page **Vue d’ensemble** de votre serveur :

> [!div class="mx-imgBorder"]
> :::image type="content" source="media/quickstart-create-database-portal/12-delete.png" alt-text="Capture d’écran montrant le bouton de suppression d’un serveur.":::

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/postgres-doc-feedback)

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Effectuer la migration de votre base de données par exportation et importation](./howto-migrate-using-export-and-import.md) <br/>

> [!div class="nextstepaction"]
> [Concevoir une base de données](./tutorial-design-database-using-azure-portal.md#create-tables-in-the-database)

[Vous ne trouvez pas ce que vous cherchez ? Faites-le nous savoir.](https://aka.ms/postgres-doc-feedback)