---
title: Migrer de Couchbase vers l’API SQL Azure Cosmos DB
description: Guide pas à pas pour effectuer une migration depuis Couchbase vers l’API SQL Azure Cosmos DB
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 02/11/2020
ms.author: mansha
author: manishmsfte
ms.custom: devx-track-java
ms.openlocfilehash: a15c6b5919f428b28daab86fea9c3b6473d19162
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606196"
---
# <a name="migrate-from-couchbase-to-azure-cosmos-db-sql-api"></a>Migrer de Couchbase vers l’API SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB est une base de données scalable, distribuée à l’échelle mondiale et complètement managée. Elle fournit un accès à faible latence garanti à vos données. Pour en savoir plus sur Azure Cosmos DB, consultez l’article de [présentation](introduction.md). Cet article fournit des instructions pour migrer des applications Java qui sont connectées à CouchBase vers un compte d’API SQL dans Azure Cosmos DB.

## <a name="differences-in-nomenclature"></a>Différences de nomenclature

Voici les principales fonctionnalités qui opèrent différemment dans Azure Cosmos DB et Couchbase :

| Couchbase | Azure Cosmos DB |
|--|--|
| Couchbase Server | Compte |
| Compartiment | Base de données |
| Compartiment | Conteneur/collection |
| Document JSON | Élément/document |

## <a name="key-differences"></a>Différences clés

* Azure Cosmos DB a un champ « ID » dans le document, alors que Couchbase a l’ID dans le cadre d’un compartiment. Le champ « ID » est unique dans la partition.

* Azure Cosmos DB évolue à l’aide de la technique de partitionnement. Cela signifie qu’il divise les données en plusieurs partitions. Ces partitions sont créées en fonction de la propriété de clé de partition que vous fournissez. Vous pouvez sélectionner la clé de partition pour optimiser les opérations de lecture et d’écriture ou les opérations de lecture/d’écriture optimisées également. Pour plus d’informations, consultez l’article sur le [partitionnement](./partitioning-overview.md).

* Dans Azure Cosmos DB, il n’est pas nécessaire que la hiérarchie de niveau supérieur désigne la collection, car le nom de celle-ci existe déjà. Cette fonctionnalité simplifie grandement la structure JSON. Voici un exemple qui montre les différences de modèle de données entre Couchbase et Azure Cosmos DB :

   **Couchbase** : ID de document = "99FF4444"

    ```json
    {
      "TravelDocument":
      {
       "Country":"India",
      "Validity" : "2022-09-01",
        "Person":
        {
          "Name": "Manish",
          "Address": "AB Road, City-z"
        },
        "Visas":
        [
          {
          "Country":"India",
          "Type":"Multi-Entry",
          "Validity":"2022-09-01"
          },
          {
          "Country":"US",
          "Type":"Single-Entry",
          "Validity":"2022-08-01"
          }
        ]
      }
    }
   ```

   **Azure Cosmos DB** : Fait référence à l’ID dans le document, comme indiqué ci-dessous.

    ```json
    {
      "id" : "99FF4444",
    
      "Country":"India",
       "Validity" : "2022-09-01",
        "Person":
        {
          "Name": "Manish",
          "Address": "AB Road, City-z"
        },
        "Visas":
        [
          {
          "Country":"India",
          "Type":"Multi-Entry",
          "Validity":"2022-09-01"
          },
          {
          "Country":"US",
          "Type":"Single-Entry",
          "Validity":"2022-08-01"
          }
        ]
      }
    
    ```
         
## <a name="java-sdk-support"></a>Support du SDK Java

Azure Cosmos DB a les SDK suivants pour prendre en charge différents frameworks Java :

* SDK Async
* SDK Spring Boot

Les sections suivantes décrivent quand utiliser chacun de ces SDK. Prenons un exemple dans lequel nous avons trois types de charges de travail :

## <a name="couchbase-as-document-repository--spring-data-based-custom-queries"></a>Couchbase en tant que dépôt de documents & requêtes personnalisées basées sur des données Spring

Si la charge de travail que vous migrez est basée sur le SDK Spring Boot, vous pouvez utiliser les étapes suivantes :

1. Ajoutez le parent au fichier POM.xml :

   ```java
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.1.5.RELEASE</version>
      <relativePath/>
   </parent>
   ```

1. Ajoutez des propriétés au fichier POM.xml :

   ```java
   <azure.version>2.1.6</azure.version>
   ```

1. Ajoutez des dépendances au fichier POM.xml :

   ```java
   <dependency>
       <groupId>com.microsoft.azure</groupId>
       <artifactId>azure-cosmosdb-spring-boot-starter</artifactId>
       <version>2.1.6</version>
   </dependency>
   ```

1. Ajoutez des propriétés d’application sous les ressources et spécifiez les éléments suivants. Veillez à remplacer les paramètres de l’URL, de la clé et du nom de la base de données :

   ```java
      azure.cosmosdb.uri=<your-cosmosDB-URL>
      azure.cosmosdb.key=<your-cosmosDB-key>
      azure.cosmosdb.database=<your-cosmosDB-dbName>
   ```

1. Définissez le nom de la collection dans le modèle. Vous pouvez également spécifier des annotations supplémentaires. Par exemple, l’ID et la clé de partition pour les désigner de manière explicite :

   ```java
   @Document(collection = "mycollection")
       public class User {
           @id
           private String id;
           private String firstName;
           @PartitionKey
           private String lastName;
       }
   ```

Voici les extraits de code pour les opérations CRUD :

### <a name="insert-and-update-operations"></a>Opérations d’insertion et de mise à jour

Où *_repo* est l’objet de dépôt et *doc* est l’objet de la classe POJO. Vous pouvez utiliser `.save` pour effectuer une insertion ou un upsert (si le document avec l’ID spécifié est trouvé). L’extrait de code suivant montre comment insérer ou mettre à jour un objet doc :

```_repo.save(doc);```

### <a name="delete-operation"></a>Opération de suppression

Examinez l’extrait de code suivant, où l’objet doc a un ID et une clé de partition indispensables à sa localisation et à sa suppression :

```_repo.delete(doc);```

### <a name="read-operation"></a>Opération de lecture

Vous pouvez lire le document en utilisant la clé de partition ou sans la spécifier. Si vous ne spécifiez pas la clé de partition, elle est traitée en tant que requête entre plusieurs partitions. Examinez les exemples de code suivants, dont le premier effectue une opération à l’aide d’un ID et d’un champ de clé de partition. Le deuxième exemple utilise un champ normal et ne spécifie pas le champ de clé de partition.

* ```_repo.findByIdAndName(objDoc.getId(),objDoc.getName());```
* ```_repo.findAllByStatus(objDoc.getStatus());```

Voilà, vous pouvez désormais utiliser votre application avec Azure Cosmos DB. L’exemple de code complet sur lequel repose l’exemple décrit dans ce document est disponible dans le dépôt GitHub [CouchbaseToCosmosDB-SpringCosmos](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/main/SpringCosmos).

## <a name="couchbase-as-a-document-repository--using-n1ql-queries"></a>Couchbase en tant que dépôt de documents et utilisation de requêtes N1QL

Les requêtes N1QL permettent de définir des requêtes dans Couchbase.

|Requête N1QL | Requête Azure CosmosDB|
|-------------------|-------------------|
|SELECT META(`TravelDocument`).id AS id, `TravelDocument`.* FROM `TravelDocument` WHERE `_type` = "com.xx.xx.xx.xxx.xxx.xxxx " and country = 'India’ and ANY m in Visas SATISFIES m.type == 'Multi-Entry' and m.Country IN ['India', Bhutan’] ORDER BY ` Validity` DESC LIMIT 25 OFFSET 0 | SELECT c.id,c FROM c JOIN m in  c.country=’India’ WHERE c._type = " com.xx.xx.xx.xxx.xxx.xxxx" and c.country = 'India' and m.type = 'Multi-Entry' and m.Country IN ('India', 'Bhutan') ORDER BY c.Validity DESC OFFSET 0 LIMIT 25 |

Vous pouvez remarquer les modifications suivantes dans vos requêtes N1QL :

* Vous n’avez pas besoin d’utiliser le mot clé META ou de faire référence au document de premier niveau. Au lieu de cela, vous pouvez créer votre propre référence au conteneur. Dans cet exemple, nous l’avons défini par « c » (nous aurions pu utiliser toute autre chose). Cette référence est utilisée comme préfixe pour tous les champs de premier niveau. Par exemple, c.id, c.country etc.

* Au lieu d’utiliser « ANY », vous pouvez maintenant effectuer une jointure sur un sous-document et y faire référence avec un alias dédié tel que « m ». Une fois créé l’alias d’un sous-document, vous devez l’utiliser. Par exemple, m.Country.

* La séquence OFFSET est différente dans la requête Azure Cosmos DB ; vous devez d’abord spécifier OFFSET, puis LIMIT. Nous vous recommandons de ne pas recourir au SDK Spring Data si vous utilisez le nombre maximal de requêtes définies personnalisées, car cela peut engendrer une surcharge inutile côté client lors de la transmission de la requête à Azure Cosmos DB. Au lieu de cela, nous disposons d’un SDK Async Java direct, qui peut être utilisé de manière plus efficace dans ce cas.

### <a name="read-operation"></a>Opération de lecture

Utilisez le SDK Async Java avec les étapes suivantes :

1. Configurez la dépendance suivante dans le fichier POM.xml :

   ```java
   <!-- https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb -->
   <dependency>
       <groupId>com.microsoft.azure</groupId>
       <artifactId>azure-cosmos</artifactId>
       <version>3.0.0</version>
   </dependency>
   ```

1. Créez un objet de connexion pour Azure Cosmos DB à l’aide de la méthode `ConnectionBuilder`, comme indiqué dans l’exemple suivant. Veillez à placer cette déclaration dans le bean, afin que le code suivant ne soit exécuté qu’une seule fois :

   ```java
   ConnectionPolicy cp=new ConnectionPolicy();
   cp.connectionMode(ConnectionMode.DIRECT);
    
   if(client==null)
      client= CosmosClient.builder()
         .endpoint(Host)//(Host, PrimaryKey, dbName, collName).Builder()
          .connectionPolicy(cp)
          .key(PrimaryKey)
          .consistencyLevel(ConsistencyLevel.EVENTUAL)
          .build();
   
   container = client.getDatabase(_dbName).getContainer(_collName);
   ```

1. Pour exécuter la requête, vous devez exécuter l’extrait de code suivant :

   ```java
   Flux<FeedResponse<CosmosItemProperties>> objFlux= container.queryItems(query, fo);
   ```

À présent, à l’aide de la méthode ci-dessus, vous pouvez passer plusieurs requêtes et les exécuter sans aucune difficulté. Si vous devez exécuter une grande requête pouvant être subdivisée en plusieurs requêtes, essayez l’extrait de code suivant plutôt que le précédent :

```java
for(SqlQuerySpec query:queries)
{
   objFlux= container.queryItems(query, fo);
   objFlux .publishOn(Schedulers.elastic())
         .subscribe(feedResponse->
            {
               if(feedResponse.results().size()>0)
               {
                  _docs.addAll(feedResponse.results());
               }
            
            },
            Throwable::printStackTrace,latch::countDown);
   lstFlux.add(objFlux);
}
                  
      Flux.merge(lstFlux);
      latch.await();
}
```

Avec le code précédent, vous pouvez exécuter des requêtes en parallèle et accroître l’optimisation des exécutions distribuées. Vous pouvez également exécuter les opérations d’insertion et de mise à jour :

### <a name="insert-operation"></a>Opération d'insertion

Pour insérer le document, exécutez le code suivant :

```java
Mono<CosmosItemResponse> objMono= container.createItem(doc,ro);
```

Abonnez-vous ensuite à Mono comme suit :

```java
CountDownLatch latch=new CountDownLatch(1);
objMono .subscribeOn(Schedulers.elastic())
        .subscribe(resourceResponse->
        {
           if(resourceResponse.statusCode()!=successStatus)
              {
                 throw new RuntimeException(resourceResponse.toString());
              }
           },
        Throwable::printStackTrace,latch::countDown);
latch.await();
```

### <a name="upsert-operation"></a>Opérations d’upsert

L’opération d’upsert nécessite que vous spécifiiez le document qui doit être mis à jour. Pour extraire le document complet, vous pouvez utiliser l’extrait de code mentionné sous le titre Opération de lecture, puis modifier le ou les champs obligatoires. L’extrait de code suivant effectue un upsert du document :

```java
Mono<CosmosItemResponse> obs= container.upsertItem(doc, ro);
```
Abonnez-vous ensuite à Mono. Reportez-vous à l’extrait de code d’abonnement à Mono dans la section Opération d’insertion.

### <a name="delete-operation"></a>Opération de suppression

L’extrait de code suivant effectue une opération de suppression :

```java
CosmosItem objItem= container.getItem(doc.Id, doc.Tenant);
Mono<CosmosItemResponse> objMono = objItem.delete(ro);
```

Ensuite, abonnez-vous à Mono (reportez-vous à l’extrait de code d’abonnement à Mono dans la section Opération d’insertion). L’exemple de code complet est disponible dans le dépôt GitHub [CouchbaseToCosmosDB-AsyncInSpring](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/main/AsyncInSpring).

## <a name="couchbase-as-a-keyvalue-pair"></a>Couchbase en tant que paire clé/valeur

Il s’agit d’un type simple de charge de travail dans lequel vous pouvez effectuer des recherches au lieu de requêtes. Pour les paires clé/valeur, procédez comme suit :

1. Envisagez « /ID » comme clé primaire ; vous avez ainsi l’assurance que vous pouvez effectuer une opération de recherche directement dans la partition spécifique. Créez une collection et spécifiez « /ID » comme clé de partition.

1. Désactivez complètement l’indexation. Étant donné que vous allez exécuter des opérations de recherche, toute surcharge d’indexation est inutile. Pour désactiver l’indexation, connectez-vous au portail Azure, puis accédez à Compte Azure Cosmos DB. Ouvrez l’**Explorateur de données**, puis sélectionnez votre  **Base de données** et le **Conteneur**. Ouvrez l’onglet **Mise à l’échelle et paramètres** et sélectionnez **Stratégie d’indexation**. La stratégie d’indexation actuelle se présente comme suit :
    
   ```json
   {
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/\"_etag\"/?"
        }
    ]
    }
   ````

   Remplacez la stratégie d’indexation ci-dessus par la stratégie suivante :

   ```json
   {
    "indexingMode": "none",
    "automatic": false,
    "includedPaths": [],
    "excludedPaths": []
    }
   ```

1. Utilisez l’extrait de code suivant pour créer l’objet de connexion. Objet de connexion (à placer dans @Bean ou à rendre statique) :

   ```java
   ConnectionPolicy cp=new ConnectionPolicy();
   cp.connectionMode(ConnectionMode.DIRECT);
   
   if(client==null)
      client= CosmosClient.builder()
         .endpoint(Host)//(Host, PrimaryKey, dbName, collName).Builder()
          .connectionPolicy(cp)
          .key(PrimaryKey)
          .consistencyLevel(ConsistencyLevel.EVENTUAL)
          .build();
    
   container = client.getDatabase(_dbName).getContainer(_collName);
   ```

Vous pouvez maintenant exécuter les opérations CRUD comme suit :

### <a name="read-operation"></a>Opération de lecture

Pour lire l’élément, utilisez l’extrait de code suivant :

```java        
CosmosItemRequestOptions ro=new CosmosItemRequestOptions();
ro.partitionKey(new PartitionKey(documentId));
CountDownLatch latch=new CountDownLatch(1);
      
var objCosmosItem= container.getItem(documentId, documentId);
Mono<CosmosItemResponse> objMono = objCosmosItem.read(ro);
objMono .subscribeOn(Schedulers.elastic())
        .subscribe(resourceResponse->
        {
           if(resourceResponse.item()!=null)
           {
              doc= resourceResponse.properties().toObject(UserModel.class);
           }
        },
        Throwable::printStackTrace,latch::countDown);
latch.await();
```

### <a name="insert-operation"></a>Opération d'insertion

Pour insérer un élément, vous pouvez exécuter le code suivant :

```java
Mono<CosmosItemResponse> objMono= container.createItem(doc,ro);
```

Abonnez-vous ensuite à Mono comme suit :

```java
CountDownLatch latch=new CountDownLatch(1);
objMono.subscribeOn(Schedulers.elastic())
      .subscribe(resourceResponse->
      {
         if(resourceResponse.statusCode()!=successStatus)
            {
               throw new RuntimeException(resourceResponse.toString());
            }
         },
      Throwable::printStackTrace,latch::countDown);
latch.await();
```

### <a name="upsert-operation"></a>Opérations d’upsert

Pour mettre à jour la valeur d’un élément, reportez-vous à l’extrait de code ci-dessous :

```java
Mono<CosmosItemResponse> obs= container.upsertItem(doc, ro);
```
Ensuite, abonnez-vous à Mono (reportez-vous à l’extrait de code d’abonnement à Mono dans la section Opération d’insertion).

### <a name="delete-operation"></a>Opération de suppression

Utilisez l’extrait de code suivant pour exécuter l’opération de suppression :

```java
CosmosItem objItem= container.getItem(id, id);
Mono<CosmosItemResponse> objMono = objItem.delete(ro);
```

Ensuite, abonnez-vous à Mono (reportez-vous à l’extrait de code d’abonnement à Mono dans la section Opération d’insertion). L’exemple de code complet est disponible dans le dépôt GitHub [CouchbaseToCosmosDB-AsyncKeyValue](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/main/AsyncKeyValue).

## <a name="data-migration"></a>Migration de données

Il existe deux façons de migrer des données.

* **Utiliser Azure Data Factory :** il s’agit de la méthode la plus recommandée pour migrer des données. Configurez la source en tant que Couchbase et le récepteur en tant qu’API SQL Azure Cosmos DB. Pour plus d’informations, consultez l’article sur le [connecteur Azure Cosmos DB Data Factory](../data-factory/connector-azure-cosmos-db.md).

* **Utiliser l’outil d’importation de données Azure Cosmos DB :** cette option est recommandée pour la migration à l’aide de machines virtuelles avec moins de données. Pour obtenir des instructions détaillées, consultez l’article sur l’[outil d’importation de données](./import-data.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour effectuer un test de performance, consultez l’article [Test des performances et de la mise à l’échelle avec Azure Cosmos DB](./performance-testing.md).
* Pour optimiser le code, consultez l’article [Conseils sur les performances pour Azure Cosmos DB](./performance-tips-async-java.md).
* Explorez le SDK Java Async v3 dans le dépôt GitHub des [informations de référence sur le SDK](https://github.com/Azure/azure-cosmosdb-java/tree/v3).
