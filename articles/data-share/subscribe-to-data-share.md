---
title: 'Tutoriel : Accepter et recevoir des données - Azure Data Share'
description: Didacticiel - Accepter et recevoir des données avec Azure Data Share
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: tutorial
ms.date: 11/12/2020
ms.openlocfilehash: a225989f0670e9b62b00a35bac719c9357c8a130
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017047"
---
# <a name="tutorial-accept-and-receive-data-using-azure-data-share"></a>Tutoriel : Accepter et recevoir des données avec Azure Data Share  

Dans ce didacticiel, vous allez apprendre à accepter une invitation de partage de données à l’aide d’Azure Data Share. Vous allez apprendre à recevoir les données qui ont été partagées avec vous et à activer un intervalle d’actualisation régulier pour avoir la certitude de toujours disposer de la capture instantanée la plus récente des données partagées. 

> [!div class="checklist"]
> * Guide pratique pour accepter une invitation Azure Data Share
> * Créer un compte Azure Data Share
> * Spécifier une destination pour vos données
> * Créer un abonnement à votre partage de données pour l’actualisation planifiée

## <a name="prerequisites"></a>Prérequis
Avant de pouvoir accepter une invitation de partage de données, vous devez provisionner un certain nombre de ressources Azure listées ci-dessous. 

Vérifiez que tous les prérequis sont remplis avant d’accepter une invitation de partage de données. 

* Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Invitation Data Share : Invitation de Microsoft Azure dont l’objet est « Invitation Azure Data Share de **<yourdataprovider@domain.com>**  ».
* Inscrivez le [fournisseur de ressources Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) dans l’abonnement Azure où vous allez créer une ressource de partage de données et dans l’abonnement Azure où se trouvent vos magasins de données Azure cibles.

### <a name="receive-data-into-a-storage-account"></a>Recevoir des données dans un compte de stockage

* Compte Stockage Azure : Si vous n’en avez pas déjà, vous pouvez créer un [compte Stockage Azure](../storage/common/storage-account-create.md). 
* Autorisation d’écrire dans le compte de stockage, qui est présent dans *Microsoft.Storage/storageAccounts/write*. Cette autorisation existe dans le rôle Contributeur. 
* Autorisation d’ajouter l’attribution de rôle au compte de stockage, qui est présente dans *Microsoft.Authorization/role assignments/write*. Cette autorisation existe dans le rôle Propriétaire.  

### <a name="receive-data-into-a-sql-based-target"></a>Recevoir des données dans une cible basée sur SQL
Si vous choisissez de recevoir des données dans Azure SQL Database, Azure Synapse Analytics, vous trouverez ci-dessous la liste des conditions préalables. 

#### <a name="prerequisites-for-receiving-data-into-azure-sql-database-or-azure-synapse-analytics-formerly-azure-sql-dw"></a>Prérequis pour la réception de données dans Azure SQL Database ou Azure Synapse Analytics (anciennement Azure SQL DW)
Vous pouvez suivre la [démonstration pas à pas](https://youtu.be/aeGISgK1xro) pour configurer les prérequis.

* Azure SQL Database ou Azure Synapse Analytics (anciennement Azure SQL DW).
* Autorisation d’écrire dans les bases de données sur le serveur SQL Server, qui est présente dans *Microsoft.Sql/servers/databases/write*. Cette autorisation existe dans le rôle **Contributeur**. 
* Autorisation permettant à l’identité managée de la ressource Data Share d’accéder à Azure SQL Database ou à Azure Synapse Analytics. Pour ce faire, procédez comme suit : 
    1. Dans le portail Azure, accédez au serveur SQL et attribuez-vous le rôle **Administrateur Azure Active Directory**.
    1. Connectez-vous à Azure SQL Database ou à Data Warehouse à l’aide de l’[éditeur de requête](../azure-sql/database/connect-query-portal.md#connect-using-azure-active-directory), ou à SQL Server Management Studio avec l’authentification Azure Active Directory. 
    1. Exécutez le script suivant pour ajouter l’identité managée Data Share en tant que « db_datareader, db_datawriter, db_ddladmin ». Vous devez vous connecter avec Active Directory et non avec l’authentification SQL Server. 

        ```sql
        create user "<share_acc_name>" from external provider; 
        exec sp_addrolemember db_datareader, "<share_acc_name>"; 
        exec sp_addrolemember db_datawriter, "<share_acc_name>"; 
        exec sp_addrolemember db_ddladmin, "<share_acc_name>";
        ```      
        Notez que *<share_acc_name>* est le nom de votre ressource Data Share. Si vous n’avez pas encore créé de ressource Data Share, vous pourrez le faire plus tard.         

* Accès au pare-feu SQL Server. Pour ce faire, procédez comme suit : 
    1. Dans SQL Server, dans le portail Azure, accédez à *Pare-feux et réseaux virtuels*
    1. Cliquez sur **Oui** pour l’option *Autoriser les services et les ressources Azure à accéder à ce serveur*.
    1. Cliquez sur **+ Ajouter une adresse IP cliente**. Cette adresse IP est susceptible d’être modifiée. Il peut être nécessaire de répéter ce processus la prochaine fois que vous partagerez des données SQL à partir du portail Azure. Vous pouvez également ajouter une plage d’adresses IP.
    1. Cliquez sur **Enregistrer**. 
 
#### <a name="prerequisites-for-receiving-data-into-azure-synapse-analytics-workspace-sql-pool"></a>Prérequis pour la réception de données dans un pool SQL Azure Synapse Analytics (espace de travail)

* Pool SQL dédié Azure Synapse Analytics (espace de travail). La réception de données dans un pool SQL serverless n’est actuellement pas prise en charge.
* Autorisation d’écrire dans le pool SQL de l’espace de travail Synapse, qui est présent dans *Microsoft.Synapse/workspaces/sqlPools/write*. Cette autorisation existe dans le rôle **Contributeur**.
* Autorisation permettant à l’identité managée de la ressource Data Share d’accéder au pool SQL de l’espace de travail Synapse. Pour ce faire, procédez comme suit : 
    1. Dans le portail Azure, accédez à l’espace de travail Synapse. Sélectionnez Administrateur SQL Active Directory dans le volet de navigation gauche et attribuez-vous le rôle **Administrateur Azure Active Directory**.
    1. Ouvrez Synapse Studio et sélectionnez *Gérer* dans le volet de navigation gauche. Sélectionnez *Contrôle d’accès* sous Sécurité. Attribuez-vous le rôle **Administrateur SQL** ou **Administrateur d’espace de travail**.
    1. Dans Synapse Studio, sélectionnez *Développer* dans le volet de navigation gauche. Exécutez le script suivant dans le pool SQL pour ajouter l’identité managée de la ressource Data Share en tant que « db_datareader, db_datawriter, db_ddladmin ». 
    
        ```sql
        create user "<share_acc_name>" from external provider; 
        exec sp_addrolemember db_datareader, "<share_acc_name>"; 
        exec sp_addrolemember db_datawriter, "<share_acc_name>"; 
        exec sp_addrolemember db_ddladmin, "<share_acc_name>";
        ```                   
       Notez que *<share_acc_name>* est le nom de votre ressource Data Share. Si vous n’avez pas encore créé de ressource Data Share, vous pourrez le faire plus tard.  

* Accès au pare-feu de l’espace de travail Synapse. Pour ce faire, procédez comme suit : 
    1. Dans le portail Azure, accédez à l’espace de travail Synapse. Sélectionnez *Pare-feu* dans le volet de navigation gauche.
    1. Cliquez sur **ON** pour *Autoriser les services et ressources Azure à accéder à cet espace de travail*.
    1. Cliquez sur **+ Ajouter une adresse IP cliente**. Cette adresse IP est susceptible d’être modifiée. Il peut être nécessaire de répéter ce processus la prochaine fois que vous partagerez des données SQL à partir du portail Azure. Vous pouvez également ajouter une plage d’adresses IP.
    1. Cliquez sur **Enregistrer**. 

### <a name="receive-data-into-an-azure-data-explorer-cluster"></a>Recevez des données dans un cluster Azure Data Explorer : 

* Un cluster Azure Data Explorer dans le même centre de données Azure que le cluster Data Explorer du fournisseur de données : Si vous n’en avez pas déjà un, vous pouvez créer un [cluster Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal). Si vous ne connaissez pas le centre de données Azure du cluster du fournisseur de données, vous pouvez créer le cluster plus tard dans le processus.
* Autorisation d’écrire dans le cluster Azure Data Explorer, qui est présente dans *Microsoft.Kusto/clusters/write*. Cette autorisation existe dans le rôle Contributeur. 
* Autorisation d’ajouter l’attribution de rôle au cluster Azure Data Explorer, qui est présente dans *Microsoft.Authorization/role assignments/write*. Cette autorisation existe dans le rôle Propriétaire. 

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="open-invitation"></a>Ouvrir l’invitation

### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Vous pouvez ouvrir une invitation à partir d’un e-mail, ou directement à partir du portail Azure. 

   Pour ouvrir une invitation à partir d’un e-mail, recherchez dans votre boîte de réception une invitation émanant de votre fournisseur de données. L’invitation provient de Microsoft Azure et s’intitule **Invitation Azure Data Share de <yourdataprovider@domain.com>** . Cliquez sur **Afficher l’invitation** pour voir votre invitation dans Azure. 

   Pour ouvrir l’invitation directement à partir du portail Azure, recherchez **Invitations Azure Data Share** dans le portail Azure. Cette action vous dirige vers la liste des invitations Data Share.

   ![Liste des invitations](./media/invitations.png "Liste des invitations") 

1. Sélectionnez le partage que vous souhaitez voir. 

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Préparez votre environnement Azure CLI, puis affichez vos invitations.

Commencez par préparer votre environnement pour Azure CLI :

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Exécutez la commande [az datashare consumer invitation list](/cli/azure/ext/datashare/datashare/consumer/invitation#ext_datashare_az_datashare_consumer_invitation_list) pour voir vos invitations actuelles :

```azurecli
az datashare consumer invitation list --subscription 11111111-1111-1111-1111-111111111111
```

Copiez l’ID de votre invitation pour l’utiliser dans la section suivante.

---

## <a name="accept-invitation"></a>Accepter l’invitation

### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Veillez à passer en revue tous les champs, notamment les **Conditions d’utilisation**. Si vous acceptez les conditions d’utilisation, vous devez cocher la case indiquant que vous donnez votre accord. 

   ![Conditions d’utilisation](./media/terms-of-use.png "Conditions d’utilisation") 

1. Sous *Target Data Share Account* (Compte Data Share cible), sélectionnez l’abonnement et le groupe de ressources à utiliser pour le déploiement de votre partage de données. 

   Pour le champ **Data Share Account** (Compte Data Share), sélectionnez **Create new** (Créer) si vous n’avez pas de compte Data Share existant. Sinon, sélectionnez un compte Data Share existant dans lequel vous souhaitez accepter votre partage de données. 

   Pour le champ **Nom de partage reçu**, vous pouvez conserver la valeur par défaut spécifiée par le fournisseur de données ou spécifier le nouveau nom du partage reçu. 

   Une fois que vous avez accepté les conditions d’utilisation et spécifié un compte Data Share pour gérer votre partage reçu, sélectionnez **Accepter et configurer**. Un abonnement de partage est créé. 

   ![Options d’acceptation](./media/accept-options.png "Options d’acceptation") 

   Cette action vous dirige vers le partage reçu dans votre compte Data Share. 

   Si vous ne souhaitez pas accepter l’invitation, sélectionnez *Reject* (Rejeter). 

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Utilisez la commande [az datashare consumer share-subscription create](/cli/azure/ext/datashare/datashare/consumer/share-subscription#ext_datashare_az_datashare_consumer_share_subscription_create) pour créer le partage Data Share.

```azurecli
az datashare consumer share-subscription create --resource-group share-rg \
  --name "Fabrikam Solutions" --account-name FabrikamDataShareAccount \
  --invitation-id 89abcdef-0123-4567-89ab-cdef01234567 \
  --source-share-location "East US 2" --subscription 11111111-1111-1111-1111-111111111111
```

---

## <a name="configure-received-share"></a>Configurer un partage reçu

### <a name="portal"></a>[Portail](#tab/azure-portal)

Suivez les étapes ci-dessous pour configurer l’emplacement où vous souhaitez recevoir les données.

1. Sélectionnez l’onglet **Jeux de données**. Cochez la case en regard du jeu de données auquel vous souhaitez affecter une destination. Sélectionnez **+ Mapper à la cible** pour choisir un magasin de données cible. 

   ![Mapper sur cible](./media/dataset-map-target.png "Mapper sur cible") 

1. Sélectionnez un type de magasin de données cible dans lequel vous souhaitez que les données arrivent. Tous les fichiers de données ou les tables dans le magasin de données cible ayant le même chemin et le même nom seront remplacés. 

   Pour les partages sur place, sélectionnez un magasin de données à l’emplacement spécifié. L’emplacement est le centre de données Azure où se trouve le magasin de données source du fournisseur de données. Une fois le jeu de données mappé, vous pouvez suivre le lien dans le chemin cible pour accéder aux données.

   ![Compte de stockage cible](./media/dataset-map-target-sql.png "Stockage cible") 

1. Pour le partage basé sur un instantané, si le fournisseur de données a créé une planification d’instantané pour fournir une mise à jour régulière des données, vous pouvez également activer la planification des instantanés en sélectionnant l’onglet **Planification d’instantanés**. Cochez la case en regard de la planification d’instantané et sélectionnez **+ Activer**.

   ![Activer la planification d’instantané](./media/enable-snapshot-schedule.png "Activer la planification d’instantané")

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Utilisez les commandes suivantes pour configurer l’emplacement où vous souhaitez recevoir les données.

1. Exécutez la commande [az datashare consumer share-subscription list-source-dataset](/cli/azure/ext/datashare/datashare/consumer/share-subscription#ext_datashare_az_datashare_consumer_share_subscription_list_source_dataset) pour récupérer l’ID du jeu de données :

   ```azurecli
   az datashare consumer share-subscription list-source-dataset \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "[0].dataSetId"
   ```

1. Exécutez la commande [az storage account create](/cli/azure/storage/account#az_storage_account_create) pour créer un compte de stockage pour ce partage Data Share :

   ```azurecli
   az storage account create --resource-group "share-rg" --name "FabrikamDataShareAccount" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Utilisez la commande [az storage account show](/cli/azure/storage/account#az_storage_account_show) pour récupérer l’ID du compte de stockage :

   ```azurecli
   az storage account show --resource-group "share-rg" --name "FabrikamDataShareAccount" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "id"
   ```

1. Utilisez la commande suivante pour récupérer l’ID du principal du compte :

   ```azurecli
   az datashare account show --resource-group "share-rg" --name "cli_test_consumer_account" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "identity.principalId"
   ```

1. Utilisez la commande [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) pour créer une attribution de rôle pour le principal du compte :

   ```azurecli
   az role assignment create --role "01234567-89ab-cdef-0123-456789abcdef" \
     --assignee-object-id 6789abcd-ef01-2345-6789-abcdef012345 
     --assignee-principal-type ServicePrincipal --scope 456789ab-cdef-0123-4567-89abcdef0123 \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Créez une variable pour le mappage en fonction de l’ID de jeu de données :

   ```azurecli
   $mapping='{\"data_set_id\":\"' + $dataset_id + '\",\"container_name\":\"newcontainer\",
     \"storage_account_name\":\"datashareconsumersa\",\"kind\":\"BlobFolder\",\"prefix\":\"consumer\"}'
   ```

1. Utilisez la commande [az datashare consumer dataset-mapping create](/cli/azure/ext/datashare/datashare/consumer/dataset-mapping#ext_datashare_az_datashare_consumer_dataset_mapping_create) pour créer le mappage de jeu de données :

   ```azurecli
   az datashare consumer dataset-mapping create --resource-group "share-rg" \
     --name "consumer-data-set-mapping" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" --mapping $mapping \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Exécutez la commande [az datashare consumer share-subscription synchronization start](/cli/azure/ext/datashare/datashare/consumer/share-subscription/synchronization#ext_datashare_az_datashare_consumer_share_subscription_synchronization_start) pour démarrer la synchronisation du jeu de données.

   ```azurecli
   az datashare consumer share-subscription synchronization start \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount"  \
     --share-subscription-name "Fabrikam Solutions" --synchronization-mode "Incremental" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

   Exécutez la commande [az datashare consumer share-subscription synchronization list](/cli/azure/ext/datashare/datashare/consumer/share-subscription/synchronization#ext_datashare_az_datashare_consumer_share_subscription_synchronization_list) pour afficher une liste de vos synchronisations :

   ```azurecli
   az datashare consumer share-subscription synchronization list \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

   Utilisez la commande [az datashare consumer share-subscription list-source-share-synchronization-setting](/cli/azure/ext/datashare/datashare/consumer/share-subscription#ext_datashare_az_datashare_consumer_share_subscription_list_source_share_synchronization_setting) pour voir les paramètres de synchronisation définis sur votre partage.

   ```azurecli
   az datashare consumer share-subscription list-source-share-synchronization-setting \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" --subscription 11111111-1111-1111-1111-111111111111
   ```

---

## <a name="trigger-a-snapshot"></a>Déclencher une capture instantanée

### <a name="portal"></a>[Portail](#tab/azure-portal)

Ces étapes s’appliquent seulement au partage basé sur des instantanés.

1. Vous pouvez déclencher un instantané en sélectionnant l’onglet **Détails** puis **Déclencher un instantané**. Ici, vous pouvez déclencher une capture instantanée complète ou incrémentielle de vos données. Si vous recevez des données de votre fournisseur de données pour la première fois, sélectionnez l’option de copie complète. 

   ![Déclencher un instantané](./media/trigger-snapshot.png "Déclencher un instantané") 

1. Une fois que le dernier état d’exécution est *Réussi*, accédez au magasin de données cible pour voir les données reçues. Sélectionnez **Jeux de données**, puis cliquez sur le lien dans le chemin cible. 

   ![Jeux de données consommateur](./media/consumer-datasets.png "Mappage de jeu de données consommateur") 

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Exécutez la commande [az datashare consumer trigger create](/cli/azure/ext/datashare/datashare/consumer/trigger#ext_datashare_az_datashare_consumer_trigger_create) pour déclencher un instantané :

```azurecli
az datashare consumer trigger create --resource-group "share-rg" \
  --name "share_test_trigger" --account-name "FabrikamDataShareAccount" \
  --share-subscription-name "Fabrikam Solutions" --recurrence-interval "Day" \
  --synchronization-time "2020-04-23 18:00:00 +00:00" --kind ScheduleBased \
  --subscription 11111111-1111-1111-1111-111111111111
```

> [!NOTE]
> Utilisez cette commande uniquement pour le partage basé sur un instantané.

---

## <a name="view-history"></a>Afficher l’historique
Cette étape s’applique uniquement au partage basé sur un instantané. Pour afficher l’historique de vos instantanés, sélectionnez l’onglet **Historique**. Vous trouverez ici un historique de tous les instantanés générés au cours des 30 derniers jours.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque la ressource n’est plus nécessaire, accédez à la page **Vue d’ensemble du partage de données**, puis sélectionnez **Supprimer** pour la supprimer.

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez découvert comment accepter et recevoir un partage Azure Data Share. Pour en savoir plus sur les concepts liés à Azure Data Share, consultez la terminologie Azure Data Share.

> [!div class="nextstepaction"]
> [Concepts d’Azure Data Share](terminology.md)