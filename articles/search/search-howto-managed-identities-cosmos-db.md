---
title: Configurer une connexion à un compte Cosmos DB à l’aide d’une identité managée
titleSuffix: Azure Cognitive Search
description: Découvrez comment configurer une connexion d’indexeur à un compte Cosmos DB à l’aide d’une identité managée
manager: luisca
author: markheff
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 2a1744feedc3e0ffae6cf2cd45cd090a6c2f06d5
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422091"
---
# <a name="set-up-an-indexer-connection-to-a-cosmos-db-database-using-a-managed-identity"></a>Configurer une connexion d’indexeur à une base de données Cosmos DB à l’aide d’une identité managée

Cette page explique comment configurer une connexion d’indexeur à une base de données Azure Cosmos DB à l’aide d’une identité managée au lieu de fournir des informations d’identification dans la chaîne de connexion de l’objet source de données.

Avant d’en apprendre plus sur cette fonctionnalité, il est recommandé de comprendre ce qu’est un indexeur et savoir comment configurer un indexeur pour votre source de données. Pour plus d’informations, consultez les liens suivants :

* [Présentation de l’indexeur](search-indexer-overview.md)
* [Indexeur Azure Cosmos DB](search-howto-index-cosmosdb.md)

## <a name="set-up-a-connection-using-a-managed-identity"></a>Configurer une connexion à l’aide d’une identité managée

### <a name="1---turn-on-system-assigned-managed-identity"></a>1 – Activer l’identité managée affectée par le système

Quand une identité managée affectée par le système est activée, Azure crée une identité pour votre service de recherche qui peut être utilisée pour vous authentifier auprès d’autres services Azure au sein du même locataire et du même abonnement. Vous pouvez ensuite utiliser cette identité dans les attributions de contrôle d’accès en fonction du rôle Azure (Azure RBAC) qui autorisent l’accès aux données pendant l’indexation.

![Turn on system assigned managed identity](./media/search-managed-identities/turn-on-system-assigned-identity.png "Activer l’identité managée affectée par le système")

Après avoir sélectionné **Enregistrer** , vous verrez un ID d’objet qui a été attribué à votre service de recherche.

![ID d’objet](./media/search-managed-identities/system-assigned-identity-object-id.png "ID de l'objet")
 
### <a name="2---add-a-role-assignment"></a>2 – Ajouter une attribution de rôle

Au cours de cette étape, vous allez accorder à votre service Recherche cognitive Azure l’autorisation de lire les données de votre compte Cosmos DB.

1. Dans le portail Azure, accédez au compte Cosmos DB qui contient les données que vous souhaitez indexer.
2. Sélectionnez **Contrôle d’accès (IAM)**
3. Sélectionnez **Ajouter** , puis **Ajouter une attribution de rôle**.

    ![Ajouter une attribution de rôle](./media/search-managed-identities/add-role-assignment-cosmos-db.png "Ajouter une attribution de rôle")

4. Sélectionner le **rôle Lecteur de compte Cosmos DB**
5. Laissez **Attribuer l’accès à** sur **Utilisateur, groupe ou principal de service Azure AD**.
6. Recherchez votre service de recherche, sélectionnez-le, puis sélectionnez **Enregistrer**.

    ![Add reader and data access role assignment](./media/search-managed-identities/add-role-assignment-cosmos-db-account-reader-role.png "Ajouter une attribution de rôle Lecteur et accès aux données")

### <a name="3---create-the-data-source"></a>3 – Créer la source de données

L’[API REST](/rest/api/searchservice/create-data-source), le portail Azure et le [Kit de développement logiciel (SDK) .NET](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourcetype) prennent également en charge la chaîne de connexion des identités managées. Voici un exemple de création d’une source de données pour indexer des données à partir de Cosmos DB à l’aide de l’[API REST](/rest/api/searchservice/create-data-source) et d’une chaîne de connexion d’identité gérée. Le format de chaîne de connexion d’identité managée est le même pour l’API REST, le kit de développement logiciel (SDK) .NET et le portail Azure.

Lorsque vous utilisez des identités managées pour l’authentification, les **informations d’identification** n’incluent pas de clé de compte.

```
POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
Content-Type: application/json
api-key: [Search service admin key]

{
    "name": "cosmos-db-datasource",
    "type": "cosmosdb",
    "credentials": {
        "connectionString": "Database=sql-test-db;ResourceId=/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/cosmos-db-resource-group/providers/Microsoft.DocumentDB/databaseAccounts/my-cosmos-db-account/;"
    },
    "container": { "name": "myCollection", "query": null },
    "dataChangeDetectionPolicy": {
        "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName": "_ts"
    }
}
```    

Le corps de la requête contient la définition de la source de données, qui doit inclure les champs suivants :

| Champ   | Description |
|---------|-------------|
| **name** | Obligatoire. Choisissez un nom pour représenter votre objet source de données. |
|**type**| Obligatoire. Doit être `cosmosdb`. |
|**credentials** | Obligatoire. <br/><br/>Lors de la connexion à l’aide d’une identité managée, le format des **informations d’identification** doit être : *Database=[nom-base-de-données];ResourceId=[chaîne-id-ressource];(ApiKind=[type-api];)*<br/> <br/>Format de ResourceId : *ResourceId=/subscriptions/ **votre ID d’abonnement** /resourceGroups/ **le nom de votre groupe de ressources** /providers/Microsoft.DocumentDB/databaseAccounts/ **le nom de votre compte cosmos db** /;*<br/><br/>Pour les collections SQL, la chaîne de connexion ne requiert pas d’ApiKind.<br/><br/>Pour les collections MongoDB, ajoutez **ApiKind=MongoDb** à la chaîne de connexion. <br/><br/>Pour les graphes Gremlin et les tables Cassandra, inscrivez-vous à la [préversion de l’indexeur contrôlé](https://aka.ms/azure-cognitive-search/indexer-preview) pour accéder à la préversion et aux informations sur la façon de mettre en forme les informations d’identification.<br/>|
| **container** | Contient les éléments suivants : <br/>**nom**  : Obligatoire. Spécifiez l’ID de la collection de bases de données à indexer.<br/>**query** : facultatif. Vous pouvez spécifier une requête pour obtenir un schéma plat à partir d’un document JSON arbitraire de manière à ce qu’Azure Search puisse procéder à l’indexation.<br/>Pour l’API MongoDB, l’API Gremlin et l’API Cassandra, les requêtes ne sont pas prises en charge. |
| **dataChangeDetectionPolicy** | Recommandé |
|**dataDeletionDetectionPolicy** | Facultatif |

### <a name="4---create-the-index"></a>4 – Créer l’index

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

### <a name="5---create-the-indexer"></a>5 – Créer l’indexeur

Un indexeur connecte une source de données à un index de recherche cible et fournit une planification afin d’automatiser l’actualisation des données.

Une fois l’index et la source de données créés, vous êtes prêt à créer l’indexeur.

Exemple de définition d’indexeur :

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "cosmos-db-indexer",
      "dataSourceName" : "cosmos-db-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" }
    }
```

Cet indexeur s’exécutera toutes les deux heures (intervalle de planification défini sur « PT2H »). Pour exécuter un indexeur toutes les 30 minutes, définissez l’intervalle sur « PT30M ». Le plus court intervalle pris en charge est de 5 minutes. La planification est facultative : en cas d’omission, un indexeur ne s’exécute qu’une seule fois lorsqu’il est créé. Toutefois, vous pouvez à tout moment exécuter un indexeur à la demande.   

Pour plus d’informations sur l’API Créer un indexeur, consultez [Créer un indexeur](/rest/api/searchservice/create-indexer).

Pour plus d’informations sur la définition des planifications de l’indexeur, consultez [Comment planifier des indexeurs pour la Recherche cognitive Azure](search-howto-schedule-indexers.md).

## <a name="troubleshooting"></a>Dépannage

Si vous constatez que vous n’êtes pas en mesure d’indexer les données de Cosmos DB, prenez en compte les points suivants :

1. Si vous avez récemment alterné vos clés de compte Cosmos DB, vous devrez attendre jusqu’à 15 minutes pour que la chaîne de connexion de l’identité managée fonctionne.

1. Vérifiez si l’accès au compte Cosmos DB est limité à certains réseaux. Si c’est le cas, reportez-vous à [Accès de l’indexeur au contenu protégé par les fonctionnalités de sécurité réseau Azure](search-indexer-securing-resources.md).

## <a name="next-steps"></a>Étapes suivantes

* [Indexeur Azure Cosmos DB](search-howto-index-cosmosdb.md)