---
title: 'Démarrage rapide : Créer un serveur – Portail Azure – Azure Database pour MySQL'
description: Cet article vous guide dans le portail Azure pour créer un exemple de serveur Azure Database pour MySQL en environ cinq minutes.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 11/04/2020
ms.openlocfilehash: f71bcc1fd3b92a32a3e6d9fa056bae7131a663bd
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492605"
---
# <a name="quickstart-create-an-azure-database-for-mysql-server-by-using-the-azure-portal"></a>Démarrage rapide : Création d’un serveur Azure Database pour MySQL à l’aide du portail Azure

Azure Database pour MySQL est un service géré qui vous permet d’exécuter, de gérer et de mettre à l’échelle des bases de données MySQL hautement disponibles dans le cloud. Ce guide de démarrage rapide vous montre comment utiliser le portail Azure pour créer un serveur unique Azure Database pour MySQL. Il vous indique également comment se connecter au serveur.

## <a name="prerequisites"></a>Prérequis
Un abonnement Azure est requis. Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-an-azure-database-for-mysql-single-server"></a>Créer un serveur unique Azure Database pour MySQL
1. Accédez au [portail Azure](https://portal.azure.com/) pour créer une base de données de serveur unique MySQL. Recherchez et sélectionnez **Azure Database pour MySQL** :

   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/find-azure-mysql-in-portal.png" alt-text="Trouver Azure Database pour MySQL":::

1. Sélectionnez **Ajouter**.

2. Dans la page **Sélectionner l’option de déploiement Azure Database pour MySQL**, sélectionnez **Serveur unique** :
   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/choose-singleserver.png" alt-text="Capture d’écran montrant l’option de serveur unique.":::

3. Entrez les paramètres de base d’un nouveau serveur unique :

   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/4-create-form.png" alt-text="Capture d’écran montrant la page Créer un serveur MySQL.":::

   **Paramètre** | **Valeur suggérée** | **Description**
   ---|---|---
   Abonnement | Votre abonnement | Sélectionnez l’abonnement Azure souhaité.
   Resource group | **myresourcegroup** | Entrez un groupe de ressources nouveau ou existant dans votre abonnement.
   Nom du serveur | **mydemoserver** | Entrez un nom unique. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit contenir entre 3 et 63 caractères.
   Source de données |**Aucun** | Sélectionnez **Aucun** pour créer un serveur en partant de zéro. Sélectionnez **Sauvegarde** uniquement si vous effectuez une restauration à partir d’une géosauvegarde d’un serveur existant.
   Emplacement |L’emplacement souhaité | Sélectionnez un emplacement dans la liste.
   Version | La version principale la plus récente| Utilisez la version principale la plus récente. Consultez [toutes les versions prises en charge](../postgresql/concepts-supported-versions.md).
   Calcul + stockage | Utiliser les valeurs par défaut| Le niveau tarifaire par défaut est **Usage général** avec **4 vCores** et **100 Go** de stockage. La conservation des sauvegardes est définie sur **7 jours** avec l’option de sauvegarde **Géographiquement redondant**.<br/>Passez en revue la page des [tarifs](https://azure.microsoft.com/pricing/details/mysql/) et mettez à jour les valeurs par défaut si nécessaire.
   Nom d’utilisateur administrateur | **mydemoadmin** | Entrez le nom d’utilisateur de l’administrateur de serveur. Vous ne pouvez pas utiliser **azure_superuser**, **admin**, **administrator** (administrateur), **root** (racine), **guest** (invité) ou **public** comme nom d’utilisateur administrateur.
   Mot de passe | Un mot de passe | Un nouveau mot de passe pour l’utilisateur administrateur de serveur. Le mot de passe doit comprendre entre 8 et 128 caractères, et contenir une combinaison de lettres majuscules ou minuscules, de chiffres et de caractères non alphanumériques ( !, $, #,%, etc.).
  

   > [!NOTE]
   > Choisissez le niveau tarifaire De base si votre charge de travail n’a pas besoin d’une grande capacité de calcul et d’E/S. Notez que les serveurs créés avec le niveau tarifaire De base ne peuvent plus ensuite être mis à l’échelle vers le niveau Usage général ou Mémoire optimisée.

4. Sélectionnez **Vérifier + créer** pour provisionner le serveur.

5. Attendez que la page du portail affiche **Votre déploiement est terminé**. Sélectionnez **Accéder à la ressource** pour afficher la page du serveur nouvellement créé :

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/deployment-complete.png" alt-text="Capture d’écran montrant le message Votre déploiement a été effectué.":::

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/mysql-doc-feedback)

## <a name="configure-a-server-level-firewall-rule"></a>Configurer une règle de pare-feu au niveau du serveur

Par défaut, le nouveau serveur est protégé par un pare-feu. Pour vous connecter, vous devez fournir l’accès à votre IP en effectuant les étapes suivantes :

1. Accédez à **Sécurité de la connexion** dans le volet gauche pour votre ressource de serveur. Si vous ne savez pas comment trouver votre ressource, consultez le [Guide pratique pour ouvrir une ressource](../azure-resource-manager/management/manage-resources-portal.md#open-resources).

   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/add-current-ip-firewall.png" alt-text="Capture d’écran montrant la page Sécurité de la connexion > Règles de pare-feu.":::

2. Sélectionnez **Ajouter l’adresse IP actuelle du client**, puis sélectionnez **Enregistrer**.

   > [!NOTE]
   > Afin d’éviter des problèmes de connectivité, vérifiez que votre réseau autorise le trafic sortant sur le port 3306 qu’utilise Azure Database pour MySQL. 

Vous pouvez ajouter d’autres IP ou fournir une plage d’IP pour la connexion à votre serveur à partir de celles-ci. Pour plus d’informations, consultez le [Guide pratique pour gérer les règles de pare-feu sur un serveur Azure Database pour MySQL](./concepts-firewall-rules.md).


[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/mysql-doc-feedback)

## <a name="connect-to-the-server-by-using-mysqlexe"></a>Se connecter au serveur à l’aide de mysql.exe
Vous pouvez utiliser [mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) ou [MySQL Workbench](./connect-workbench.md) pour vous connecter au serveur à partir de votre environnement local. Dans ce guide de démarrage rapide, nous allons exécuter mysql.exe dans [Azure Cloud Shell](../cloud-shell/overview.md) pour se connecter au serveur.


1. Ouvrez Azure Cloud Shell dans le portail en sélectionnant le premier bouton de la barre d’outils, comme illustré dans la capture d’écran suivante. Notez le nom du serveur, le nom de l’administrateur du serveur et l’abonnement de votre nouveau serveur dans la section **Vue d’ensemble**, comme indiqué dans la capture d’écran.

    > [!NOTE]
    > Si vous ouvrez Cloud Shell pour la première fois, vous êtes invité à créer un groupe de ressources et un compte de stockage. Il s’agit d’une étape unique. Elle sera automatiquement associée à toutes les sessions.

   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/use-in-cloud-shell.png" alt-text="Capture d’écran montrant Cloud Shell dans le portail Azure.":::
2. Exécutez la commande suivante dans le terminal Azure Cloud Shell. Remplacez les valeurs affichées ici par le nom réel du serveur et de l’utilisateur administrateur. En ce qui concerne Azure Database pour MySQL, vous devez ajouter `@\<servername>` au nom d’utilisateur administrateur, comme illustré ici : 

      ```azurecli-interactive
      mysql --host=mydemoserver.mysql.database.azure.com --user=myadmin@mydemoserver -p
      ```

      Voici le résultat obtenu dans le terminal Cloud Shell :

      ```
      Requesting a Cloud Shell.Succeeded.
      Connecting terminal...

      Welcome to Azure Cloud Shell

      Type "az" to use Azure CLI
      Type "help" to learn about Cloud Shell

      user@Azure:~$mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
      Enter password:
      Welcome to the MySQL monitor.  Commands end with ; or \g.
      Your MySQL connection id is 64796
      Server version: 5.6.42.0 Source distribution

      Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.
      Oracle is a registered trademark of Oracle Corporation and/or its
      affiliates. Other names may be trademarks of their respective
      owners.

      Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
      mysql>
      ```
3. Dans le même terminal Azure Cloud Shell, créez une base de données nommée `guest` :
      ```
      mysql> CREATE DATABASE guest;
      Query OK, 1 row affected (0.27 sec)
      ```
4. Basculez vers la base de données `guest` :
      ```
      mysql> USE guest;
      Database changed
      ```
5. Entrez `quit`, puis sélectionnez **Entrée** pour quitter mysql.

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/mysql-doc-feedback)

## <a name="clean-up-resources"></a>Nettoyer les ressources
Vous venez de créer un serveur Azure Database pour MySQL dans un groupe de ressources.  Si vous ne pensez pas avoir besoin de ces ressources à l’avenir, vous pouvez les supprimer en supprimant le groupe de ressources ou en supprimant simplement le serveur MySQL. Pour supprimer le groupe de ressources, effectuez ces étapes :
1. Depuis le portail Azure, recherchez et sélectionnez **Groupes de ressources**.
2. Dans la liste des groupes de ressources, sélectionnez le nom de votre groupe de ressources.
3. Dans la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
4. Dans la boîte de dialogue de confirmation, entrez le nom de votre groupe de ressources, puis sélectionnez **Supprimer**.

Pour supprimer le serveur, vous pouvez sélectionner **Supprimer** dans la page **Vue d’ensemble** de votre serveur, comme indiqué ici :
> [!div class="mx-imgBorder"]
> :::image type="content" source="media/quickstart-create-mysql-server-database-using-azure-portal/delete-server.png" alt-text="Capture d’écran montrant le bouton Supprimer dans la page Vue d’ensemble du serveur.":::

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
>[Créer une application PHP sur Windows avec MySQL](../app-service/tutorial-php-mysql-app.md) <br/>

> [!div class="nextstepaction"]
>[Créer une application PHP sur Linux avec MySQL](../app-service/tutorial-php-mysql-app.md?pivots=platform-linux%3fpivots%3dplatform-linux)<br/><br/>

[Vous ne trouvez pas ce que vous cherchez ? Faites-le nous savoir.](https://aka.ms/mysql-doc-feedback)