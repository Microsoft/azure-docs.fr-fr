---
title: Configurer une connexion à un compte de stockage à l’aide d’une identité managée
titleSuffix: Azure Cognitive Search
description: Découvrez comment configurer une connexion d’indexeur à un compte Stockage Azure à l’aide d’une identité managée
manager: luisca
author: markheff
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: f26ca04955dfa854a8ee17b7aa255a6ed991b8df
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358369"
---
# <a name="set-up-a-connection-to-an-azure-storage-account-using-a-managed-identity"></a>Configurer une connexion à un compte Stockage Azure à l’aide d’une identité managée

Cette page explique comment configurer une connexion d’indexeur à un compte Stockage Azure à l’aide d’une identité managée au lieu de fournir des informations d’identification dans la chaîne de connexion de l’objet source de données.

Avant d’en apprendre plus sur cette fonctionnalité, il est recommandé de comprendre ce qu’est un indexeur et savoir comment configurer un indexeur pour votre source de données. Pour plus d’informations, consultez les liens suivants :
* [Présentation de l’indexeur](search-indexer-overview.md)
* [Indexeur de blobs Azure](search-howto-indexing-azure-blob-storage.md)
* [Indexeur Azure Data Lake Storage Gen2](search-howto-index-azure-data-lake-storage.md)
* [Indexeur de tables Azure](search-howto-indexing-azure-tables.md)

## <a name="set-up-the-connection"></a>Configurer la connexion

### <a name="1---turn-on-system-assigned-managed-identity"></a>1 – Activer l’identité managée affectée par le système

Quand une identité managée affectée par le système est activée, Azure crée une identité pour votre service de recherche qui peut être utilisée pour vous authentifier auprès d’autres services Azure au sein du même locataire et du même abonnement. Vous pouvez ensuite utiliser cette identité dans les attributions de contrôle d’accès en fonction du rôle Azure (Azure RBAC) qui autorisent l’accès aux données pendant l’indexation.

![Turn on system assigned managed identity](./media/search-managed-identities/turn-on-system-assigned-identity.png "Activer l’identité managée affectée par le système")

Après avoir sélectionné **Enregistrer** , vous verrez un ID d’objet qui a été attribué à votre service de recherche.

![ID d’objet](./media/search-managed-identities/system-assigned-identity-object-id.png "ID de l'objet")
 
### <a name="2---add-a-role-assignment"></a>2 – Ajouter une attribution de rôle

Au cours de cette étape, vous allez accorder à votre service Recherche cognitive Azure l’autorisation de lire les données de votre compte de stockage.

1. Dans le portail Azure, accédez au compte de stockage qui contient les données que vous souhaitez indexer.
2. Sélectionnez **Contrôle d’accès (IAM)**
3. Sélectionnez **Ajouter** , puis **Ajouter une attribution de rôle**.

    ![Ajouter une attribution de rôle](./media/search-managed-identities/add-role-assignment-storage.png "Ajouter une attribution de rôle")

4. Sélectionnez les rôles appropriés en fonction du type de compte de stockage que vous souhaitez indexer :
    1. Stockage Blob Azure nécessite que vous ajoutiez votre service de recherche au rôle **Lecteur des données blob du stockage**.
    1. Azure Data Lake Storage Gen2 nécessite que vous ajoutiez votre service de recherche au rôle **Lecteur des données blob du stockage**.
    1. Stockage Table Azure nécessite que vous ajoutiez votre service de recherche au rôle **Lecteur et accès aux données**.
5.  Laissez **Attribuer l’accès à** sur **Utilisateur, groupe ou principal de service Azure AD**.
6.  Recherchez votre service de recherche, sélectionnez-le, puis sélectionnez **Enregistrer**.

    Exemple pour Stockage Blob Azure et Azure Data Lake Storage Gen2 :

    ![Add Storage Blob Data Reader role assignment](./media/search-managed-identities/add-role-assignment-storage-blob-data-reader.png "Ajouter une attribution de rôle Lecteur des données blob du stockage")

    Exemple pour Stockage Table Azure :

    ![Add reader and data access role assignment](./media/search-managed-identities/add-role-assignment-reader-and-data-access.png "Ajouter une attribution de rôle Lecteur et accès aux données")

### <a name="3---create-the-data-source"></a>3 – Créer la source de données

L’[API REST](/rest/api/searchservice/create-data-source), le portail Azure et le [Kit de développement logiciel (SDK) .NET](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection) prennent également en charge la chaîne de connexion des identités managées. Voici un exemple de création d’une source de données pour indexer des données à partir d’un compte de stockage à l’aide de l’[API REST](/rest/api/searchservice/create-data-source) et d’une chaîne de connexion d’identité gérée. Le format de chaîne de connexion d’identité managée est le même pour l’API REST, le kit de développement logiciel (SDK) .NET et le portail Azure.

Lors de l’indexation d’un compte de stockage, la source de données doit avoir les propriétés requises suivantes :

* **name** est le nom unique de la source de données au sein de votre service de recherche.
* **type**
    * Stockage Blob Azure : `azureblob`
    * Stockage Table Azure : `azuretable`
    * Azure Data Lake Storage Gen2 : **type** est fourni une fois que vous vous êtes inscrit à la préversion à l’aide de [ce formulaire](https://aka.ms/azure-cognitive-search/mi-preview-request).
* **credentials**
    * Le format d’ **informations d’identification** est différent selon que vous utilisez ou non une identité managée. Vous indiquerez ici un ResourceId qui n’a pas de clé de compte ni de mot de passe. Le ResourceId doit inclure l’ID d’abonnement du compte de stockage, le groupe de ressources du compte de stockage et le nom du compte de stockage.
    * Format de l’identité managée : 
        * *ResourceId=/subscriptions/ **votre ID d’abonnement** /resourceGroups/ **le nom de votre groupe de ressources** /providers/Microsoft.Storage/storageAccounts/ **le nom de votre compte de stockage** /;*
* **container** spécifie le nom d’un conteneur ou d’une table dans votre compte de stockage. Par défaut, tous les objets blob du conteneur sont récupérables. Si vous souhaitez indexer uniquement les objets blob dans un répertoire virtuel particulier, vous pouvez spécifier ce répertoire à l’aide du paramètre facultatif **query**.

Exemple de création d’un objet source de données blob à l’aide de l’[API REST](/rest/api/searchservice/create-data-source) :

```
POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "blob-datasource",
    "type" : "azureblob",
    "credentials" : { "connectionString" : "ResourceId=/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name/;" },
    "container" : { "name" : "my-container", "query" : "<optional-virtual-directory-name>" }
}   
```

### <a name="4---create-the-index"></a>4 – Créer l’index

L’index spécifie les champs d’un document, les attributs et d’autres constructions qui façonnent l’expérience de recherche.

Voici comment créer un index avec un champ `content` pouvant faire l'objet d'une recherche afin de stocker le texte extrait d'objets blob :   

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }
```

Pour plus d’informations sur la création d’index, consultez [Création d'un index](/rest/api/searchservice/create-index)

### <a name="5---create-the-indexer"></a>5 – Créer l’indexeur

Un indexeur connecte une source de données à un index de recherche cible et fournit une planification afin d’automatiser l’actualisation des données.

Une fois l’index et la source de données créés, vous êtes prêt à créer l’indexeur.

Exemple de définition d’indexeur pour un indexeur de blobs :

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "blob-indexer",
      "dataSourceName" : "blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" }
    }
```

Cet indexeur s’exécutera toutes les deux heures (intervalle de planification défini sur « PT2H »). Pour exécuter un indexeur toutes les 30 minutes, définissez l’intervalle sur « PT30M ». Le plus court intervalle pris en charge est de 5 minutes. La planification est facultative : en cas d’omission, un indexeur ne s’exécute qu’une seule fois lorsqu’il est créé. Toutefois, vous pouvez à tout moment exécuter un indexeur à la demande.   

Pour plus d’informations sur l’API Créer un indexeur, consultez [Créer un indexeur](/rest/api/searchservice/create-indexer).

Pour plus d’informations sur la définition des planifications de l’indexeur, consultez [Comment planifier des indexeurs pour la Recherche cognitive Azure](search-howto-schedule-indexers.md).

## <a name="accessing-secure-data-in-storage-accounts"></a>Accès aux données sécurisées dans les comptes de stockage

Les comptes de stockage Azure peuvent être sécurisés davantage à l’aide de pare-feu et de réseaux virtuels. Si vous souhaitez indexer le contenu d’un compte de stockage Blob ou d’un compte de stockage Data Lake Gen2 qui est sécurisé par un pare-feu ou un réseau virtuel, suivez les instructions fournies dans [Accès aux données dans les comptes de stockage de manière sécurisée via une exception de service approuvé](search-indexer-howto-access-trusted-service-exception.md).

## <a name="see-also"></a>Voir aussi

En savoir plus sur les indexeurs de Stockage Azure :

* [Indexeur de blobs Azure](search-howto-indexing-azure-blob-storage.md)
* [Indexeur Azure Data Lake Storage Gen2](search-howto-index-azure-data-lake-storage.md)
* [Indexeur de tables Azure](search-howto-indexing-azure-tables.md)