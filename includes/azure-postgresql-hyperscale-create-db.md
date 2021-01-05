---
title: Fichier include
description: Fichier include
author: jonels-msft
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: include
ms.date: 12/16/2020
ms.author: jonels
ms.custom: include file
ms.openlocfilehash: 26289f6bdfa086708548d17125a6dfdf50b02a8f
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97665207"
---
## <a name="create-a-hyperscale-citus-server-group"></a>Créer un groupe de serveurs Hyperscale (Citus)

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

### <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com).


Pour créer un serveur de base de données Azure pour PostgreSQL, suivez les étapes ci-après :
1. Cliquez sur **Créer une ressource** en haut à gauche du portail Azure.
2. Sélectionnez **Bases de données** dans la page **Nouveau**, puis **Base de données Azure pour PostgreSQL** dans la page **Bases de données**.
3. Pour l’option de déploiement, cliquez sur le bouton **Créer** sous **Groupe de serveurs Hyperscale (Citus).**
4. Remplissez le formulaire de détails du nouveau serveur avec les informations suivantes :
   - Groupe de ressources : cliquez sur le lien **Créer** sous la zone de texte pour ce champ. Entrez un nom tel que **myresourcegroup**.
   - Nom du groupe de serveurs : entrez un nom unique pour le nouveau groupe de serveurs, qui sera également utilisé pour un sous-domaine de serveur.
   - Nom de l'utilisateur administrateur : valeur **citus** actuellement requise ; ne peut pas être modifiée.
   - Mot de passe : doit contenir au moins huit caractères appartenant à trois des catégories suivantes : lettres majuscules non accentuées, lettres minuscules non accentuées, chiffres (de 0 à 9) et caractères non alphanumériques (!, $, #, %, etc.).
   - Emplacement : utilisez l’emplacement le plus proche de vos utilisateurs pour leur donner l’accès le plus rapide possible aux données.

   > [!IMPORTANT]
   > Le mot de passe d'administration du serveur que vous spécifiez ici est requis pour se connecter au serveur et à ses bases de données. Retenez ou enregistrez ces informations pour une utilisation ultérieure.

5. Cliquez sur **Configurer le groupe de serveurs**. Laissez inchangés les paramètres de cette section et cliquez sur **Enregistrer**.
6. Cliquez sur **Suivant : Networking (Réseau) >** au bas de la page.

7. Sous l’onglet **Networking** (Réseau), cliquez sur la case d’option **Public endpoint** (Point de terminaison public).
   ![Public endpoint (Point de terminaison public) sélectionné](./media/azure-postgresql-hyperscale-create-db/network-public-endpoint.png)
8. Cliquez sur le lien **+ Add current client IP address** (+ Ajouter l’adresse IP actuelle du client).
   ![Adresse IP du client ajoutée](./media/azure-postgresql-hyperscale-create-db/network-add-client-ip.png)

   > [!NOTE]
   > Le serveur Azure PostgreSQL communique sur le port 5432. Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 5432 peut être bloqué par le pare-feu de votre réseau. Si c’est le cas, vous ne pourrez pas vous connecter à votre cluster Hyperscale (Citus), sauf si votre service informatique ouvre le port 5432.
   >

9. Cliquez sur **Vérifier + créer**, puis sur **Créer** pour provisionner le serveur. Le provisionnement prend quelques minutes.
10. La page redirige vers la supervision du déploiement. Quand l’état passe de **Votre déploiement est en cours** à **Votre déploiement a été effectué**, cliquez sur l’élément de menu **Sorties** à gauche de la page.
11. La page des sorties contient un nom d’hôte de coordinateur avec un bouton en regard de celui-ci pour copier la valeur dans le Presse-papiers. Prenez note de ces informations pour une utilisation ultérieure.

### <a name="connect-to-the-database-using-psql"></a>Se connecter à la base de données à l'aide de psql

Lorsque vous créez votre serveur Azure Database pour PostgreSQL, une base de données par défaut nommée **citus** est créée. Pour vous connecter à votre serveur de base de données, vous devez disposer d'une chaîne de connexion et du mot de passe d'administration.

1. Récupérez la chaîne de connexion. Sur la page du groupe de serveurs, cliquez sur l'élément de menu **Chaînes de connexion**. (Celui-ci se trouve sous **Paramètres**.) Recherchez la chaîne portant l’étiquette **psql**. Celle-ci se présente sous la forme suivante :

   ```
   psql "host=hostname.postgres.database.azure.com port=5432 dbname=citus user=citus password={your_password} sslmode=require"
   ```

   Copiez la chaîne. Vous devez remplacer « {votre\_mot de passe} » par le mot de passe d'administration que vous avez choisi précédemment. Comme le système ne stocke pas votre mot de passe en clair, il ne peut pas l'afficher pour vous dans la chaîne de connexion.

2. Ouvrez une fenêtre de terminal sur votre ordinateur local.

3. À l'invite, connectez-vous à votre serveur Azure Database pour PostgreSQL à l'aide de l'utilitaire [psql](https://www.postgresql.org/docs/current/app-psql.html). Passez votre chaîne de connexion entre guillemets, en vous assurant qu'elle contient bien votre mot de passe :
   ```bash
   psql "host=..."
   ```

   Par exemple, la commande suivante vous connecte au nœud coordinateur du groupe de serveurs **mydemoserver** :

   ```bash
   psql "host=mydemoserver-c.postgres.database.azure.com port=5432 dbname=citus user=citus password={your_password} sslmode=require"
   ```
