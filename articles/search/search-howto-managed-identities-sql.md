---
title: Configurer une connexion à Azure SQL Database à l’aide d’une identité managée
titleSuffix: Azure Cognitive Search
description: Découvrez comment configurer une connexion d’indexeur à Azure SQL Database à l’aide d’une identité managée
manager: luisca
author: markheff
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: b940da2cf754e7e1cac91df6b517ecebe55e8c40
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358420"
---
# <a name="set-up-an-indexer-connection-to-azure-sql-database-using-a-managed-identity"></a>Configurer une connexion d’indexeur à Azure SQL Database à l’aide d’une identité managée

Cette page explique comment configurer une connexion d’indexeur à Azure SQL Database à l’aide d’une identité managée au lieu de fournir des informations d’identification dans la chaîne de connexion de l’objet source de données.

Avant d’en apprendre plus sur cette fonctionnalité, il est recommandé de comprendre ce qu’est un indexeur et savoir comment configurer un indexeur pour votre source de données. Pour plus d’informations, consultez les liens suivants :

* [Présentation de l’indexeur](search-indexer-overview.md)
* [Indexeur Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)

## <a name="set-up-a-connection-using-a-managed-identity"></a>Configurer une connexion à l’aide d’une identité managée

### <a name="1---turn-on-system-assigned-managed-identity"></a>1 – Activer l’identité managée affectée par le système

Quand une identité managée affectée par le système est activée, Azure crée une identité pour votre service de recherche qui peut être utilisée pour vous authentifier auprès d’autres services Azure au sein du même locataire et du même abonnement. Vous pouvez ensuite utiliser cette identité dans les attributions de contrôle d’accès en fonction du rôle Azure (Azure RBAC) qui autorisent l’accès aux données pendant l’indexation.

![Turn on system assigned managed identity](./media/search-managed-identities/turn-on-system-assigned-identity.png "Activer l’identité managée affectée par le système")

Après avoir sélectionné **Enregistrer** , vous verrez un ID d’objet qui a été attribué à votre service de recherche.

![ID d’objet](./media/search-managed-identities/system-assigned-identity-object-id.png "ID de l'objet")

### <a name="2---provision-azure-active-directory-admin-for-sql-server"></a>2 – Approvisionner un administrateur Azure Active Directory pour SQL Server

Lors de la connexion à la base de données à l’étape suivante, vous devez vous connecter avec un compte Azure Active Directory (Azure AD) disposant d’un accès administrateur à la base de données afin de permettre à votre service de recherche d’y accéder.

Suivez les instructions indiquées [ici](../azure-sql/database/authentication-aad-configure.md?tabs=azure-powershell#provision-azure-ad-admin-sql-database) pour accorder à votre compte Azure AD un accès administrateur à la base de données.

### <a name="3---assign-the-search-service-permissions"></a>3 – Attribuer des autorisations au service de recherche

Suivez les étapes ci-dessous pour attribuer au service de recherche l’autorisation de lire la base de données.

1. Connectez-vous à Visual Studio.

    ![Connect to Visual Studio](./media/search-managed-identities/connect-with-visual-studio.png "Connectez-vous à Visual Studio.")

2. S'authentifier à l'aide du compte Azure AD

    ![Authentifier](./media/search-managed-identities/visual-studio-authentication.png "Authenticate")

3. Exécutez les commandes suivantes :

    Incluez les crochets autour du nom de votre service de recherche.
    
    ```
    CREATE USER [your search service name here] FROM EXTERNAL PROVIDER;
    EXEC sp_addrolemember 'db_datareader', [your search service name here];
    ```

    ![Nouvelle requête](./media/search-managed-identities/visual-studio-new-query.png "Nouvelle requête")

    ![Execute query](./media/search-managed-identities/visual-studio-execute-query.png "Exécuter la requête")

>[!NOTE]
> Si l’identité du service de recherche de l’étape 1 est modifiée après avoir terminé cette étape, vous devez supprimer l’appartenance au rôle et supprimer l’utilisateur dans la base de données SQL, puis rajouter les autorisations en effectuant à nouveau l’étape 3.
> La suppression de l’appartenance au rôle et de l’utilisateur peut être effectuée en exécutant les commandes suivantes :
> ```
> sp_droprolemember 'db_datareader', [your search service name];
> DROP USER IF EXISTS [your search service name];
> ```

### <a name="4---add-a-role-assignment"></a>4 – Ajouter une attribution de rôle

Au cours de cette étape, vous allez accorder à votre service Recherche cognitive Azure l’autorisation de lire les données de votre serveur SQL.

1. Dans le portail Azure, accédez à votre page Azure SQL Server.
2. Sélectionnez **Contrôle d’accès (IAM)**
3. Sélectionnez **Ajouter** , puis **Ajouter une attribution de rôle**.

    ![Ajouter une attribution de rôle](./media/search-managed-identities/add-role-assignment-sql-server.png "Ajouter une attribution de rôle")

4. Sélectionnez le rôle **Lecteur** approprié.
5. Laissez **Attribuer l’accès à** sur **Utilisateur, groupe ou principal de service Azure AD**.
6. Recherchez votre service de recherche, sélectionnez-le, puis sélectionnez **Enregistrer**.

    ![Add reader role assignment](./media/search-managed-identities/add-role-assignment-sql-server-reader-role.png "Ajouter une attribution de rôle Lecteur")

### <a name="5---create-the-data-source"></a>5 – Créer la source de données

L’[API REST](/rest/api/searchservice/create-data-source), le portail Azure et le [Kit de développement logiciel (SDK) .NET](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection) prennent également en charge la chaîne de connexion des identités managées. Voici un exemple de création d’une source de données pour indexer des données à partir d’une instance Azure SQL Database à l’aide de l’[API REST](/rest/api/searchservice/create-data-source) et d’une chaîne de connexion d’identité gérée. Le format de chaîne de connexion d’identité managée est le même pour l’API REST, le kit de développement logiciel (SDK) .NET et le portail Azure.

Lors de la création d’une source de données à l’aide de l’[API REST](/rest/api/searchservice/create-data-source), la source de données doit avoir les propriétés requises suivantes :

* **name** est le nom unique de la source de données au sein de votre service de recherche.
* **type** est `azuresql`
* **credentials**
    * Le format d’ **informations d’identification** est différent selon que vous utilisez ou non une identité managée. Vous indiquerez ici un nom Initial Catalog ou Database et un ResourceId qui n’a pas de clé de compte ni de mot de passe. Le ResourceId doit inclure l’ID d’abonnement d’Azure SQL Database, le groupe de ressources d’Azure SQL Database et le nom de la base de données SQL. 
    * Format de la chaîne de connexion de l’identité managée :
        * *Initial Catalog|Database= **nom de la base de données** ;ResourceId=/subscriptions/ **votre ID d’abonnement** /resourceGroups/ **votre nom de groupe de ressources** /providers/Microsoft.Sql/servers/ **votre nom de serveur SQL** /;Connection Timeout= **durée de la connexion** ;*
* **container** spécifie le nom de la table ou de l’affichage que vous souhaitez indexer.

Exemple de création d’un objet de source de données Azure SQL à l’aide de l’[API REST](/rest/api/searchservice/create-data-source) :

```
POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "sql-datasource",
    "type" : "azuresql",
    "credentials" : { "connectionString" : "Database=sql-database;ResourceId=/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azure-sql-resource-group/providers/Microsoft.Sql/servers/sql-server-search-demo;Connection Timeout=30;" },
    "container" : { "name" : "my-table" }
} 
```

### <a name="6---create-the-index"></a>6 – Créer l’index

L’index spécifie les champs d’un document, les attributs et d’autres constructions qui façonnent l’expérience de recherche.

Voici comment créer un index avec un champ `booktitle` pouvant faire l’objet d’une recherche :   

```
POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "my-target-index",
    "fields": [
    { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
    { "name": "booktitle", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
    ]
}
```

Pour plus d’informations sur la création d’index, consultez [Création d'un index](/rest/api/searchservice/create-index)

### <a name="7---create-the-indexer"></a>7 – Créer l’indexeur

Un indexeur connecte une source de données à un index de recherche cible et fournit une planification afin d’automatiser l’actualisation des données.

Une fois l’index et la source de données créés, vous êtes prêt à créer l’indexeur.

Exemple de définition d’indexeur pour un indexeur Azure SQL :

```
POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "sql-indexer",
    "dataSourceName" : "sql-datasource",
    "targetIndexName" : "my-target-index",
    "schedule" : { "interval" : "PT2H" }
}
```    

Cet indexeur s’exécutera toutes les deux heures (intervalle de planification défini sur « PT2H »). Pour exécuter un indexeur toutes les 30 minutes, définissez l’intervalle sur « PT30M ». Le plus court intervalle pris en charge est de 5 minutes. La planification est facultative : en cas d’omission, un indexeur ne s’exécute qu’une seule fois lorsqu’il est créé. Toutefois, vous pouvez à tout moment exécuter un indexeur à la demande.   

Pour plus d’informations sur l’API Créer un indexeur, consultez [Créer un indexeur](/rest/api/searchservice/create-indexer).

Pour plus d’informations sur la définition des planifications de l’indexeur, consultez [Comment planifier des indexeurs pour la Recherche cognitive Azure](search-howto-schedule-indexers.md).

## <a name="troubleshooting"></a>Dépannage

Si vous recevez une erreur lorsque l’indexeur tente de se connecter à la source de données qui indique que le client n’est pas autorisé à accéder au serveur, consultez les [erreurs courantes de l’indexeur](./search-indexer-troubleshooting.md).

## <a name="see-also"></a>Voir aussi

En savoir plus sur l’indexeur Azure SQL :
* [Indexeur Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)