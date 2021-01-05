---
title: 'Tutoriel : Partager en dehors de votre organisation - Azure Data Share'
description: Tutoriel - Partager des données avec des clients et des partenaires via Azure Data Share
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: tutorial
ms.date: 11/12/2020
ms.openlocfilehash: 89c2a725b853b5a2a7578dccc1fd503917e12962
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659622"
---
# <a name="tutorial-share-data-using-azure-data-share"></a>Tutoriel : Partagez des données avec Azure Data Share  

Dans ce tutoriel, vous allez découvrir comment créer un partage Azure Data Share et commencer à partager vos données avec des clients et des partenaires externes à votre organisation Azure. 

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un partage Data Share
> * Ajouter des jeux de données à votre partage Data Share
> * Activer une planification d’instantanés pour votre partage Data Share. 
> * Ajouter des destinataires à votre partage Data Share 

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* L’adresse e-mail de connexion Azure de vos destinataires (l’utilisation de leur alias de courrier ne fonctionnera pas).
* Si le magasin de données Azure source se trouve dans un autre abonnement Azure que celui que vous utiliserez pour créer une ressource de partage de données, inscrivez le [fournisseur de ressources Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) dans l’abonnement où se trouve le magasin de données Azure. 

### <a name="share-from-a-storage-account"></a>Partager à partir d’un compte de stockage

* Compte Stockage Azure : Si vous n’en avez pas déjà, vous pouvez créer un [compte Stockage Azure](../storage/common/storage-account-create.md)
* Autorisation d’écrire dans le compte de stockage, qui est présent dans *Microsoft.Storage/storageAccounts/write*. Cette autorisation existe dans le rôle **Contributeur**.
* Autorisation d’ajouter l’attribution de rôle au compte de stockage, qui est présente dans *Microsoft.Authorization/role assignments/write*. Cette autorisation existe dans le rôle **Propriétaire**. 


### <a name="share-from-a-sql-based-source"></a>Partager à partir d’une source SQL
Vous trouverez ci-dessous la liste des prérequis pour le partage de données à partir d’une source SQL. 

#### <a name="prerequisites-for-sharing-from-azure-sql-database-or-azure-synapse-analytics-formerly-azure-sql-dw"></a>Prérequis pour le partage depuis Azure SQL Database ou Azure Synapse Analytics (anciennement Azure SQL DW)
Vous pouvez suivre la [démonstration pas à pas](https://youtu.be/hIE-TjJD8Dc) pour configurer les prérequis.

* Azure SQL Database ou Azure Synapse Analytics (anciennement Azure SQL DW) avec des tables et des vues que vous voulez partager.
* Autorisation d’écrire dans les bases de données sur SQL Server, qui est présente dans *Microsoft.Sql/servers/databases/write*. Cette autorisation existe dans le rôle **Contributeur**.
* Autorisation permettant à l’identité managée de la ressource Data Share d’accéder à la base de données. Pour ce faire, procédez comme suit : 
    1. Dans le portail Azure, accédez au serveur SQL et attribuez-vous le rôle **Administrateur Azure Active Directory**.
    1. Connectez-vous à Azure SQL Database ou à Data Warehouse à l’aide de l’[éditeur de requête](../azure-sql/database/connect-query-portal.md#connect-using-azure-active-directory), ou à SQL Server Management Studio avec l’authentification Azure Active Directory. 
    1. Exécutez le script suivant pour ajouter l’identité managée de la ressource Data Share en tant que db_datareader. Vous devez vous connecter avec Active Directory et non avec l’authentification SQL Server. 
    
        ```sql
        create user "<share_acct_name>" from external provider;     
        exec sp_addrolemember db_datareader, "<share_acct_name>"; 
        ```                   
       Notez que *<share_acc_name>* est le nom de votre ressource Data Share. Si vous n’avez pas encore créé de ressource Data Share, vous pourrez le faire plus tard.  

* Un utilisateur Azure SQL Database avec un accès **« db_datareader »** pour parcourir et sélectionner les tables et/ou les vues que vous voulez partager. 

* Accès au pare-feu SQL Server. Pour ce faire, procédez comme suit : 
    1. Dans le portail Azure, accédez à SQL Server. Sélectionnez *Pare-feu et réseaux virtuels* dans le volet de navigation gauche.
    1. Cliquez sur **Oui** pour l’option *Autoriser les services et les ressources Azure à accéder à ce serveur*.
    1. Cliquez sur **+ Ajouter une adresse IP cliente**. Cette adresse IP est susceptible d’être modifiée. Il peut être nécessaire de répéter ce processus la prochaine fois que vous partagerez des données SQL à partir du portail Azure. Vous pouvez également ajouter une plage d’adresses IP.
    1. Cliquez sur **Enregistrer**. 

#### <a name="prerequisites-for-sharing-from-azure-synapse-analytics-workspace-sql-pool"></a>Prérequis pour le partage depuis un pool SQL Azure Synapse Analytics (espace de travail)

* * Pool SQL dédié Azure Synapse Analytics (espace de travail) avec les tables que vous souhaitez partager. Le partage de vue n’est actuellement pas pris en charge. Le partage depuis un pool SQL serverless n’est actuellement pas pris en charge.
* Autorisation d’écrire dans le pool SQL de l’espace de travail Synapse, qui est présent dans *Microsoft.Synapse/workspaces/sqlPools/write*. Cette autorisation existe dans le rôle **Contributeur**.
* Autorisation permettant à l’identité managée de la ressource Data Share d’accéder au pool SQL de l’espace de travail Synapse. Pour ce faire, procédez comme suit : 
    1. Dans le portail Azure, accédez à l’espace de travail Synapse. Sélectionnez Administrateur SQL Active Directory dans le volet de navigation gauche et attribuez-vous le rôle **Administrateur Azure Active Directory**.
    1. Ouvrez Synapse Studio et sélectionnez *Gérer* dans le volet de navigation gauche. Sélectionnez *Contrôle d’accès* sous Sécurité. Attribuez-vous le rôle **Administrateur SQL** ou **Administrateur d’espace de travail**.
    1. Dans Synapse Studio, sélectionnez *Développer* dans le volet de navigation gauche. Exécutez le script suivant dans le pool SQL pour ajouter l’identité managée de la ressource Data Share en tant que db_datareader. 
    
        ```sql
        create user "<share_acct_name>" from external provider;     
        exec sp_addrolemember db_datareader, "<share_acct_name>"; 
        ```                   
       Notez que *<share_acc_name>* est le nom de votre ressource Data Share. Si vous n’avez pas encore créé de ressource Data Share, vous pourrez le faire plus tard.  

* Accès au pare-feu de l’espace de travail Synapse. Pour ce faire, procédez comme suit : 
    1. Dans le portail Azure, accédez à l’espace de travail Synapse. Sélectionnez *Pare-feu* dans le volet de navigation gauche.
    1. Cliquez sur **ON** pour *Autoriser les services et ressources Azure à accéder à cet espace de travail*.
    1. Cliquez sur **+ Ajouter une adresse IP cliente**. Cette adresse IP est susceptible d’être modifiée. Il peut être nécessaire de répéter ce processus la prochaine fois que vous partagerez des données SQL à partir du portail Azure. Vous pouvez également ajouter une plage d’adresses IP.
    1. Cliquez sur **Enregistrer**. 


### <a name="share-from-azure-data-explorer"></a>Partager depuis Azure Data Explorer
* Un cluster Azure Data Explorer avec des bases de données que vous voulez partager.
* Autorisation d’écrire sur le cluster Azure Data Explorer, qui est présente dans *Microsoft.Kusto/clusters/write*. Cette autorisation existe dans le rôle **Contributeur**.
* Autorisation d’ajouter l’attribution de rôle au cluster Azure Data Explorer, qui est présente dans *Microsoft.Authorization/role assignments/write*. Cette autorisation existe dans le rôle **Propriétaire**.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="create-a-data-share-account"></a>Créer un compte Data Share

### <a name="portal"></a>[Portail](#tab/azure-portal)

Créez une ressource Azure Data Share dans un groupe de ressources Azure.

1. Sélectionnez le bouton du menu en haut à gauche du portail, puis **Créer une ressource** (+).

1. Recherchez *Data Share*.

1. Sélectionnez Data Share, puis sélectionnez **Créer**.

1. Indiquez les détails de base de votre ressource Azure Data Share à partir des informations suivantes. 

     **Paramètre** | **Valeur suggérée** | **Description du champ**
    |---|---|---|
    | Abonnement | Votre abonnement | Sélectionnez l’abonnement Azure à utiliser pour votre compte de partage de données.|
    | Groupe de ressources | *testresourcegroup* | Utilisez un groupe de ressources existant ou créez-en un. |
    | Emplacement | *USA Est 2* | Sélectionnez une région pour votre compte de partage de données.
    | Nom | *datashareaccount* | Spécifiez un nom pour votre compte de partage de données. |
    | | |

1. Sélectionnez **Vérifier + créer** puis **Créer** pour provisionner votre compte de partage de données. En règle générale, le provisionnement d’un nouveau compte de partage de données prend environ 2 minutes au maximum. 

1. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Créez une ressource Azure Data Share dans un groupe de ressources Azure.

Commencez par préparer votre environnement pour Azure CLI :

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Utilisez les commandes suivantes pour créer la ressource :

1. Utilisez la commande [az account set](/cli/azure/account#az_account_set) pour définir votre abonnement en tant qu’abonnement par défaut actuel :

   ```azurecli
   az account set --subscription 00000000-0000-0000-0000-000000000000
   ```

1. Exécutez la commande [az provider register](/cli/azure/provider#az_provider_register) pour inscrire le fournisseur de ressources :

   ```azurecli
   az provider register --name "Microsoft.DataShare"
   ```

1. Exécutez la commande [az group create](/cli/azure/group#az_group_create) pour créer un groupe de ressources ou utiliser un groupe de ressources existant :

   ```azurecli
   az group create --name testresourcegroup --location "East US 2"
   ```

1. Exécutez la commande [az datashare account create](/cli/azure/ext/datashare/datashare/account#ext_datashare_az_datashare_account_create) pour créer un compte Data Share :

   ```azurecli
   az datashare account create --resource-group testresourcegroup --name datashareaccount --location "East US 2" 
   ```

   Exécutez la commande [az datashare account list](/cli/azure/ext/datashare/datashare/account#ext_datashare_az_datashare_account_list) pour afficher vos comptes Data Share :

   ```azurecli
   az datashare account list --resource-group testresourcegroup
   ```

---

## <a name="create-a-share"></a>Créer un partage

### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Accédez à la page de présentation de votre partage Data Share.

    ![Partager vos données](./media/share-receive-data.png "Partager vos données") 

1. Sélectionnez **Start sharing your data** (Commencer à partager vos données).

1. Sélectionnez **Create** (Créer).   

1. Renseignez les détails de votre partage. Spécifiez un nom, un type de partage, une description du contenu du partage et des conditions d’utilisation (facultatif). 

    ![EnterShareDetails](./media/enter-share-details.png "Entrer les détails de partage") 

1. Sélectionnez **Continuer**.

1. Pour ajouter des jeux de données à votre partage, sélectionnez **Add Datasets** (Ajouter des jeux de données). 

    ![Ajouter des jeux de données à votre partage](./media/datasets.png "Groupes de données")

1. Sélectionnez le type de jeu de données à ajouter. Vous verrez une liste différente de types de jeux de données en fonction du type de partage (instantané ou sur place) que vous avez sélectionné à l’étape précédente. Si vous partagez à partir d’Azure SQL Database ou Azure Synapse Analytics (anciennement Azure SQL DW), vous êtes invité à entrer des informations d’identification SQL pour lister les tables.

    ![AddDatasets](./media/add-datasets.png "Ajouter des jeux de données")    

1. Accédez à l’objet à partager, puis sélectionnez Add Datasets (Ajouter des jeux de données). 

    ![SelectDatasets](./media/select-datasets.png "Sélectionner des jeux de données")    

1. Sous l’onglet Destinataires, entrez les adresses e-mail de votre consommateur de données en sélectionnant + Add Recipient (Ajouter un destinataire). 

    ![AddRecipients](./media/add-recipient.png "Ajouter des destinataires") 

1. Sélectionnez **Continuer**.

1. Si vous avez sélectionné le type de partage d’instantané, vous pouvez configurer la planification d’instantanés pour fournir des mises à jour de vos données à votre consommateur de données. 

    ![EnableSnapshots](./media/enable-snapshots.png "Activer les captures instantanées") 

1. Sélectionnez une heure de début et un intervalle de périodicité. 

1. Sélectionnez **Continuer**.

1. Sous l’onglet Review + Create (Revoir + Créer), passez en revue le contenu du package, les paramètres, les destinataires ainsi que les paramètres de synchronisation. Sélectionnez **Create** (Créer).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Exécutez la commande [az storage account create](/cli/azure/storage/account#az_storage_account_create) pour créer un partage Data Share :

   ```azurecli
   az storage account create --resource-group testresourcegroup --name ContosoMarketplaceAccount
   ```

1. Utilisez la commande [az storage container create](/cli/azure/storage/container#az_storage_container_create) pour créer un conteneur pour le partage dans la commande précédente :

   ```azurecli
   az storage container create --name ContosoMarketplaceContainer --account-name ContosoMarketplaceAccount
   ```

1. Exécutez la commande [az datashare create](/cli/azure/ext/datashare/datashare#ext_datashare_az_datashare_create) pour créer votre partage Data Share :

   ```azurecli
   az datashare create --resource-group testresourcegroup \
     --name ContosoMarketplaceDataShare --account-name ContosoMarketplaceAccount \
     --description "Data Share" --share-kind "CopyBased" --terms "Confidential"
   ```

1. Utilisez la commande [az datashare invitation create](/cli/azure/ext/datashare/datashare/invitation#ext_datashare_az_datashare_invitation_create) pour créer l’invitation pour l’adresse spécifiée :

   ```azurecli
   az datashare invitation create --resource-group testresourcegroup \
     --name DataShareInvite --share-name ContosoMarketplaceDataShare \
     --account-name ContosoMarketplaceAccount --target-email "jacob@fabrikam"
   ```

---

Votre partage Azure Data Share est désormais créé. Le destinataire de votre partage Data Share est maintenant prêt à accepter votre invitation.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque la ressource n’est plus nécessaire, accédez à la page **Vue d’ensemble du partage de données**, puis sélectionnez **Supprimer** pour la supprimer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à créer un partage Azure Data Share et à inviter des destinataires. Pour découvrir comment un consommateur de données peut accepter et recevoir un partage de données, passez au tutoriel Accepter et recevoir des données.

> [!div class="nextstepaction"]
> [Tutoriel : Accepter et recevoir des données avec Azure Data Share](subscribe-to-data-share.md)
