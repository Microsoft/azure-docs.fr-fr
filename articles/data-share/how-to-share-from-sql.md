---
title: Partager et recevoir des données d’Azure SQL Database et d’Azure Synapse Analytics
description: Découvrez comment partager et recevoir des données d’Azure SQL Database et d’Azure Synapse Analytics
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 11/12/2020
ms.openlocfilehash: 87d6ca8ee69ca49cf52b61e6beddb56721658afa
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94593737"
---
# <a name="share-and-receive-data-from-azure-sql-database-and-azure-synapse-analytics"></a>Partager et recevoir des données d’Azure SQL Database et d’Azure Synapse Analytics

[!INCLUDE[appliesto-sql](includes/appliesto-sql.md)]

Azure Data Share prend en charge le partage basé sur des captures instantanées d’Azure SQL Database et d’Azure Synapse Analytics. Cet article explique comment partager et recevoir des données de ces sources.

Azure Data Share prend en charge le partage des tables et des vues d’Azure SQL Database et d’Azure Synapse Analytics (anciennement Azure SQL DW) et le partage des tables du pool SQL dédié Azure Synapse Analytics (espace de travail). Le partage à partir d’un pool SQL serverless Azure Synapse Analytics (espace de travail) n’est pas pris en charge actuellement. Les consommateurs de données peuvent choisir d’accepter les données dans Azure Data Lake Storage Gen2 ou Stockage Blob Azure en tant que fichier csv ou parquet, ainsi que dans Azure SQL Database et Azure Synapse Analytics en tant que tables.

Lors de l’acceptation de données dans Azure Data Lake Store Gen 2 ou Stockage Blob Azure, des captures instantanées complètes remplacent le contenu du fichier cible s’il existe déjà.
Lorsque les données sont reçues dans une table SQL, et si la table cible n’existe pas encore, Azure Data Share crée la table SQL avec le schéma source. S’il existe une table cible du même nom, celle-ci est remplacée par la dernière capture instantanée complète. Les captures instantanées incrémentielles ne sont actuellement pas prises en charge.

## <a name="share-data"></a>Partager des données

### <a name="prerequisites-to-share-data"></a>Conditions préalables pour partager des données

* Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* L’adresse e-mail de connexion Azure de vos destinataires (l’utilisation de leur alias de courrier ne fonctionnera pas).
* Si le magasin de données Azure source se trouve dans un autre abonnement Azure que celui que vous utiliserez pour créer une ressource de partage de données, inscrivez le [fournisseur de ressources Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) dans l’abonnement où se trouve le magasin de données Azure. 

### <a name="prerequisites-for-sql-source"></a>Conditions préalables pour une source SQL
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

* Pool SQL dédié Azure Synapse Analytics (espace de travail) avec les tables que vous souhaitez partager. Le partage de vue n’est actuellement pas pris en charge. Le partage depuis un pool SQL serverless n’est actuellement pas pris en charge.
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

### <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

### <a name="create-a-data-share-account"></a>Créer un compte Data Share

Créez une ressource Azure Data Share dans un groupe de ressources Azure.

1. Sélectionnez le bouton du menu en haut à gauche du portail, puis **Créer une ressource** (+).

1. Recherchez *Data Share*.

1. Sélectionnez Data Share, puis sélectionnez **Créer**.

1. Indiquez les détails de base de votre ressource Azure Data Share à partir des informations suivantes. 

     **Paramètre** | **Valeur suggérée** | **Description du champ**
    |---|---|---|
    | Abonnement | Votre abonnement | Sélectionnez l’abonnement Azure à utiliser pour votre compte de partage de données.|
    | Groupe de ressources | *test-resource-group* | Utilisez un groupe de ressources existant ou créez-en un. |
    | Emplacement | *USA Est 2* | Sélectionnez une région pour votre compte de partage de données.
    | Nom | *datashareaccount* | Spécifiez un nom pour votre compte de partage de données. |
    | | |

1. Sélectionnez **Vérifier + créer** puis **Créer** pour provisionner votre compte de partage de données. En règle générale, le provisionnement d’un nouveau compte de partage de données prend environ 2 minutes au maximum. 

1. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.

### <a name="create-a-share"></a>Créer un partage

1. Accédez à la page de présentation de votre partage Data Share.

    ![Partager vos données](./media/share-receive-data.png "Partager vos données") 

1. Sélectionnez **Start sharing your data** (Commencer à partager vos données).

1. Sélectionnez **Create** (Créer).   

1. Renseignez les détails de votre partage. Spécifiez un nom, un type de partage, une description du contenu du partage et des conditions d’utilisation (facultatif). 

    ![EnterShareDetails](./media/enter-share-details.png "Entrer les détails de partage") 

1. Sélectionnez **Continuer**.

1. Pour ajouter des jeux de données à votre partage, sélectionnez **Add Datasets** (Ajouter des jeux de données). 

    ![Ajouter des jeux de données à votre partage](./media/datasets.png "Groupes de données")

1. Sélectionnez le type de jeu de données à ajouter. Vous verrez une liste différente de types de jeux de données en fonction du type de partage (instantané ou sur place) que vous avez sélectionné à l’étape précédente. 

    ![AddDatasets](./media/add-datasets.png "Ajouter des jeux de données")    

1. Sélectionnez votre serveur SQL Server ou votre espace de travail Synapse, fournissez les informations d’identification si vous y êtes invité et sélectionnez **Suivant** pour accéder à l’objet que vous souhaitez partager, puis sélectionnez « Ajouter des jeux de données ». Vous pouvez sélectionner des tables et des vues d’Azure SQL Database et d’Azure Synapse Analytics (anciennement Azure SQL DW) ou des tables du pool SQL dédié Azure Synapse Analytics (espace de travail). 

    ![SelectDatasets](./media/select-datasets-sql.png "Sélectionner des jeux de données")    

1. Sous l’onglet Destinataires, entrez les adresses e-mail de votre consommateur de données en sélectionnant + Add Recipient (Ajouter un destinataire). L’adresse e-mail doit être l’e-mail de connexion Azure du destinataire.

    ![AddRecipients](./media/add-recipient.png "Ajouter des destinataires") 

1. Sélectionnez **Continuer**.

1. Si vous avez sélectionné le type de partage d’instantané, vous pouvez configurer la planification d’instantanés pour fournir des mises à jour de vos données à votre consommateur de données. 

    ![EnableSnapshots](./media/enable-snapshots.png "Activer les captures instantanées") 

1. Sélectionnez une heure de début et un intervalle de périodicité. 

1. Sélectionnez **Continuer**.

1. Sous l’onglet Review + Create (Revoir + Créer), passez en revue le contenu du package, les paramètres, les destinataires ainsi que les paramètres de synchronisation. Sélectionnez **Create** (Créer).

Votre partage Azure Data Share est désormais créé. Le destinataire de votre partage Data Share est maintenant prêt à accepter votre invitation. 

## <a name="receive-data"></a>Recevoir des données

### <a name="prerequisites-to-receive-data"></a>Conditions préalables pour recevoir des données
Avant de pouvoir accepter une invitation de partage de données, vous devez provisionner un certain nombre de ressources Azure listées ci-dessous. 

Vérifiez que tous les prérequis sont remplis avant d’accepter une invitation de partage de données. 

* Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Invitation Data Share : Invitation de Microsoft Azure dont l’objet est « Invitation Azure Data Share de **<yourdataprovider@domain.com>**  ».
* Inscrivez le [fournisseur de ressources Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) dans l’abonnement Azure où vous allez créer une ressource de partage de données et dans l’abonnement Azure où se trouvent vos magasins de données Azure cibles.

### <a name="prerequisites-for-target-storage-account"></a>Conditions préalables pour le compte de stockage cible
Si vous choisissez de recevoir des données dans le service Stockage Azure, vous trouverez ci-dessous la liste des conditions préalables.

* Compte Stockage Azure : Si vous n’en avez pas déjà, vous pouvez créer un [compte Stockage Azure](../storage/common/storage-account-create.md). 
* Autorisation d’écrire dans le compte de stockage, qui est présent dans *Microsoft.Storage/storageAccounts/write*. Cette autorisation existe dans le rôle **Contributeur**. 
* Autorisation d’ajouter l’attribution de rôle de l’identité managée de la ressource Data Share au compte de stockage, qui est présente dans *Microsoft.Authorization/role assignments/write*. Cette autorisation existe dans le rôle **Propriétaire**.  

### <a name="prerequisites-for-sql-target"></a>Conditions préalables pour une cible SQL
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

### <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

### <a name="open-invitation"></a>Ouvrir l’invitation

1. Vous pouvez ouvrir une invitation à partir d’un e-mail, ou directement à partir du portail Azure. 

   Pour ouvrir une invitation à partir d’un e-mail, recherchez dans votre boîte de réception une invitation émanant de votre fournisseur de données. L’invitation provient de Microsoft Azure et s’intitule **Invitation Azure Data Share de <yourdataprovider@domain.com>** . Cliquez sur **Afficher l’invitation** pour voir votre invitation dans Azure. 

   Pour ouvrir l’invitation directement à partir du portail Azure, recherchez **Invitations Azure Data Share** dans le portail Azure. Vous accédez à la liste des invitations Data Share.

   ![Liste des invitations](./media/invitations.png "Liste des invitations") 

1. Sélectionnez le partage que vous souhaitez voir. 

### <a name="accept-invitation"></a>Accepter l’invitation
1. Veillez à passer en revue tous les champs, notamment les **Conditions d’utilisation**. Si vous acceptez les conditions d’utilisation, vous devez cocher la case indiquant que vous donnez votre accord. 

   ![Conditions d’utilisation](./media/terms-of-use.png "Conditions d’utilisation") 

1. Sous *Target Data Share Account* (Compte Data Share cible), sélectionnez l’abonnement et le groupe de ressources à utiliser pour le déploiement de votre partage de données. 

   Pour le champ **Data Share Account** (Compte Data Share), sélectionnez **Create new** (Créer) si vous n’avez pas de compte Data Share existant. Sinon, sélectionnez un compte Data Share existant dans lequel vous souhaitez accepter votre partage de données. 

   Pour le champ **Nom de partage reçu**, vous pouvez conserver la valeur par défaut spécifiée par le fournisseur de données ou spécifier le nouveau nom du partage reçu. 

   Une fois que vous avez accepté les conditions d’utilisation et spécifié un compte Data Share pour gérer votre partage reçu, sélectionnez **Accepter et configurer**. Un abonnement de partage est créé. 

   ![Options d’acceptation](./media/accept-options.png "Options d’acceptation") 

   Vous accédez alors au partage reçu dans votre compte Data Share. 

   Si vous ne souhaitez pas accepter l’invitation, sélectionnez *Reject* (Rejeter). 

### <a name="configure-received-share"></a>Configurer un partage reçu
Suivez les étapes ci-dessous pour configurer l’emplacement où vous souhaitez recevoir les données.

1. Sélectionnez l’onglet **Jeux de données**. Cochez la case en regard du jeu de données auquel vous souhaitez affecter une destination. Sélectionnez **+ Mapper à la cible** pour choisir un magasin de données cible. 

   ![Mapper sur cible](./media/dataset-map-target.png "Mapper sur cible") 

1. Sélectionnez un de magasin de données cible dans lequel vous souhaitez que les données arrivent. Tous les fichiers de données ou les tables dans le magasin de données cible ayant le même chemin et le même nom seront remplacés. 

   ![Compte de stockage cible](./media/dataset-map-target-sql.png "Magasin de données cible") 

1. Pour le partage basé sur un instantané, si le fournisseur de données a créé une planification d’instantané pour fournir une mise à jour régulière des données, vous pouvez également activer la planification des instantanés en sélectionnant l’onglet **Planification d’instantanés**. Cochez la case en regard de la planification d’instantané et sélectionnez **+ Activer**.

   ![Activer la planification d’instantané](./media/enable-snapshot-schedule.png "Activer la planification d’instantané")

### <a name="trigger-a-snapshot"></a>Déclencher une capture instantanée
Ces étapes s’appliquent seulement au partage basé sur des instantanés.

1. Vous pouvez déclencher un instantané en sélectionnant l’onglet **Détails** puis **Déclencher un instantané**. Ici, vous pouvez déclencher une capture instantanée complète ou incrémentielle de vos données. Si vous recevez des données de votre fournisseur de données pour la première fois, sélectionnez l’option de copie complète. Pour les sources SQL, seule la capture instantanée complète est prise en charge.

   ![Déclencher un instantané](./media/trigger-snapshot.png "Déclencher un instantané") 

1. Une fois que le dernier état d’exécution est *Réussi*, accédez au magasin de données cible pour voir les données reçues. Sélectionnez **Jeux de données**, puis cliquez sur le lien dans le chemin cible. 

   ![Jeux de données consommateur](./media/consumer-datasets.png "Mappage de jeu de données consommateur") 

### <a name="view-history"></a>Afficher l’historique
Cette étape s’applique uniquement au partage basé sur un instantané. Pour afficher l’historique de vos instantanés, sélectionnez l’onglet **Historique**. Vous trouverez ici un historique de tous les instantanés générés au cours des 30 derniers jours. 

## <a name="supported-data-types"></a>Types de données pris en charge
Lorsque vous partagez des données à partir d’une source SQL, le mappage suivant est utilisé entre les types de données SQL Server et les types de données intermédiaires Azure Data Share pendant la création de l’instantané. 

| Type de données SQL Server | Type de données intermédiaires Azure Data Share |
|:--- |:--- |
| bigint |Int64 |
| binary |Byte[] |
| bit |Boolean |
| char |String, Char[] |
| Date |DateTime |
| Datetime |DateTime |
| datetime2 |DateTime |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[] |
| Float |Double |
| image |Byte[] |
| int |Int32 |
| money |Decimal |
| NCHAR |String, Char[] |
| ntext |String, Char[] |
| numeric |Decimal |
| NVARCHAR |String, Char[] |
| real |Unique |
| rowversion |Byte[] |
| smalldatetime |DateTime |
| SMALLINT |Int16 |
| SMALLMONEY |Decimal |
| sql_variant |Object |
| text |String, Char[] |
| time |TimeSpan |
| timestamp |Byte[] |
| TINYINT |Int16 |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| Xml |String |

>[!NOTE]
> 1. Pour les types de données mappés sur le type intermédiaire Décimal, la création d’un instantané prend en charge une précision maximale de 28. Si vous disposez de données qui nécessitent une précision supérieure à 28, vous pouvez les convertir en une chaîne. 
> 1.  Si vous partagez des données entre Azure SQL Database et Azure Synapse Analytics, sachez que tous les types de données ne sont pas pris en charge. Pour plus d’informations, consultez [Types de données de table dans un pool SQL dédié](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-data-types.md). 

## <a name="sql-always-encrypted-or-dynamic-data-masking"></a>SQL Always Encrypted ou Dynamic Data Masking
Actuellement, Azure Data Share ne prend pas en charge les bases de données SQL Azure avec Always Encrypted configuré. 

Pour les tables SQL sources avec masquage dynamique des données, les données sont masquées du côté du destinataire.

## <a name="sql-snapshot-performance"></a>Performances des instantanés SQL
Les performances des instantanés SQL sont affectées par un certain nombre de facteurs. Il est toujours recommandé d’effectuer vos propres tests de performances. Voici quelques exemples de facteurs qui ont un impact sur les performances.

* Configuration matérielle (par exemple, vCores, mémoire, DWU) des magasins de données SQL sources et cibles. 
* Accès simultané aux magasins de données sources et cibles. Si vous partagez plusieurs tables et vues à partir du même magasin de données SQL, ou si vous recevez plusieurs tables et vues dans le même magasin de données SQL, les performances seront affectées.   
* Emplacement des magasins de données sources et cibles. 

## <a name="troubleshoot-sql-snapshot-failure"></a>Résoudre les échecs d’instantanés SQL
La cause la plus courante de l’échec d’un instantané est que le partage de données n’a pas d’autorisation sur le magasin de données source ou cible. Afin d’accorder l’autorisation de partage de données à la source ou à la cible Azure SQL Database ou Azure Synapse Analytics (anciennement Azure SQL DW), vous devez exécuter le script SQL fourni lors de la connexion à la base de données SQL à l’aide de l’authentification Azure Active Directory. Pour résoudre les autres échecs d’instantanés SQL, reportez-vous à [Résoudre les problèmes d’instantanés](data-share-troubleshoot.md#snapshot-failed).

## <a name="next-steps"></a>Étapes suivantes
Vous avez appris à partager et recevoir des données à partir de sources SQL à l’aide du service Azure Data Share. Pour en savoir plus sur le partage à partir d’autres sources de données, passez à [Magasins de données pris en charge](supported-data-stores.md).