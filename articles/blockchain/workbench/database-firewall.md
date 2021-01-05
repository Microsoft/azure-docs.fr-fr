---
title: Configurer le pare-feu de la base de données Azure Blockchain Workbench
description: Découvrez comment configurer le pare-feu de la base de données Azure Blockchain Workbench (préversion) pour permettre aux clients et applications externes de se connecter.
ms.date: 09/09/2019
ms.topic: how-to
ms.reviewer: mmercuri
ms.openlocfilehash: dc7a1dc7e4a083ef8f5f7650f29ed1430b32e88b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009447"
---
# <a name="configure-the-azure-blockchain-workbench-database-firewall"></a>Configurer le pare-feu de la base de données SQL Azure Blockchain Workbench

Cet article explique comment configurer une règle de pare-feu à l’aide du portail Azure. Les règles du pare-feu permettent aux clients externes ou aux applications de se connecter à votre base de données Azure Blockchain Workbench.

## <a name="connect-to-the-blockchain-workbench-database"></a>Se connecter à la base de données Blockchain Workbench

Pour vous connecter à la base de données dans laquelle vous souhaitez configurer une règle :

1. Connectez-vous au portail Azure avec un compte détenant des autorisations de **propriétaire** pour les ressources Azure Blockchain Workbench.
2. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.
3. Choisissez le nom du groupe de ressources correspondant à votre déploiement Blockchain Workbench.
4. Sélectionnez **Type** pour trier la liste des ressources, puis votre **serveur SQL**.
5. La liste de ressources présentée dans la capture d’écran suivante montre les deux bases de données : *master* et *lsgn-sdk*. Vous configurez la règle de pare-feu sur *lsgn-sdk*.

![Liste des ressources Blockchain Workbench](./media/database-firewall/list-database-resources.png)

## <a name="create-a-database-firewall-rule"></a>Créer une règle de pare-feu de base de données

Pour créer une règle de pare-feu :

1. Choisissez le lien vers la base de données « lsgn sdk ».
2. Dans la barre de menus, sélectionnez **Définir le pare-feu du serveur**.

   ![Définir le pare-feu du serveur](./media/database-firewall/configure-server-firewall.png)

3. Pour créer une règle pour votre organisation :

   * Entrez un nom dans **NOM DE LA RÈGLE**
   * Dans la plage d’adresses, entrez une adresse IP pour **ADRESSE IP DE DÉBUT**.
   * Dans la plage d’adresses, entrez une adresse IP pour **ADRESSE IP DE FIN**.

   ![Créer une règle de pare-feu](./media/database-firewall/create-firewall-rule.png)

    > [!NOTE]
    > Si vous souhaitez uniquement ajouter l’adresse IP de votre ordinateur, choisissez **+ Ajouter une adresse IP de client**.
        
1. Pour enregistrer votre configuration de pare-feu, sélectionnez **Enregistrer**.
2. Testez la plage d’adresses IP que vous avez configurée pour la base de données en vous connectant à partir d’une application ou d’un outil. Utilisez par exemple SQL Server Management Studio.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Vues de la base de données dans Azure Blockchain Workbench](database-views.md)