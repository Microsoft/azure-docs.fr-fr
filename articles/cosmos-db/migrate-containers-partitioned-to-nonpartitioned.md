---
title: Migrer des conteneurs Azure Cosmos non partitionnés vers des conteneurs partitionnés
description: Apprenez à migrer tous les conteneurs non partitionnés existants vers des conteneurs partitionnés.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 09/25/2019
ms.author: mjbrown
ms.custom: devx-track-csharp
ms.openlocfilehash: edb6114406922d55c439ae7426a2be933bba4aee
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334087"
---
# <a name="migrate-non-partitioned-containers-to-partitioned-containers"></a>Migrer des conteneurs non partitionnés vers des conteneurs partitionnés
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB prend en charge la création de conteneurs sans clé de partition. Actuellement, vous pouvez créer des conteneurs non partitionnés à l’aide d’Azure CLI et de kits SDK Azure Cosmos DB (.Net, Java, NodeJs) de version inférieure ou égale à 2.x. Vous ne pouvez pas créer de conteneurs non partitionnés à l’aide du Portail Azure. Toutefois, ces conteneurs non partitionnés ne sont pas élastiques et disposent d’une capacité de stockage fixe de 20 Go et d’un débit maximal de 10 000 RU/s.

Les conteneurs non partitionnés sont hérités, et vous devez migrer vos conteneurs non partitionnés existants vers des conteneurs partitionnés pour faire évoluer le stockage et le débit. Azure Cosmos DB fournit un mécanisme défini par le système pour migrer vos conteneurs non partitionnés vers des conteneurs partitionnés. Ce document explique la migration automatique de tous les conteneurs non partitionnés existants vers des conteneurs partitionnés. Vous pouvez tirer parti de la fonctionnalité de migration automatique uniquement si vous utilisez la version V3 des kits de développement logiciel dans tous les langages.

> [!NOTE]
> Actuellement, vous ne pouvez pas migrer les comptes d’API Azure Cosmos DB MongoDB et Gremlin à l’aide de la procédure décrite dans ce document.

## <a name="migrate-container-using-the-system-defined-partition-key"></a>Migrer un conteneur à l’aide de la clé de partition définie par le système

Pour prendre en charge la migration, Azure Cosmos DB fournit une clé de partition définie par le système, nommée `/_partitionkey`, sur tous les conteneurs qui n’ont pas de clé de partition. Vous ne pouvez pas modifier la définition de clé de partition après la migration des conteneurs. Par exemple, la définition d’un conteneur migré vers un conteneur partitionné se présentera comme suit :

```json
{
    "Id": "CollId" 
  "partitionKey": {
    "paths": [
      "/_partitionKey"
    ],
    "kind": "Hash"
  },
}
```

Une fois la migration du conteneur effectuée, vous pouvez créer des documents en remplissant la propriété `_partitionKey` et les autres propriétés du document. La propriété `_partitionKey` représente la clé de partition de vos documents.

Le choix de la clé de partition appropriée est important pour une utilisation optimale du débit approvisionné. Pour plus d’informations, consultez l’article [How to choose a partition key (Choisir une clé de partition)](partitioning-overview.md).

> [!NOTE]
> Vous pouvez tirer parti de la clé de partition définie par le système uniquement si vous utilisez la version V3/la dernière version des kits de développement logiciel dans tous les langages.

Voici un exemple de code permettant de créer un document avec la clé de partition définie par le système et de lire ce document :

**représentation JSON du document**

```csharp
DeviceInformationItem = new DeviceInformationItem
{
   "id": "elevator/PugetSound/Building44/Floor1/1",
   "deviceId": "3cf4c52d-cc67-4bb8-b02f-f6185007a808",
   "_partitionKey": "3cf4c52d-cc67-4bb8-b02f-f6185007a808"
} 

public class DeviceInformationItem
{
    [JsonProperty(PropertyName = "id")]
    public string Id { get; set; }

    [JsonProperty(PropertyName = "deviceId")]
    public string DeviceId { get; set; }

    [JsonProperty(PropertyName = "_partitionKey", NullValueHandling = NullValueHandling.Ignore)]
    public string PartitionKey {get {return this.DeviceId; set; }
}

CosmosContainer migratedContainer = database.Containers["testContainer"];

DeviceInformationItem deviceItem = new DeviceInformationItem() {
  Id = "1234",
  DeviceId = "3cf4c52d-cc67-4bb8-b02f-f6185007a808"
}

ItemResponse<DeviceInformationItem > response = 
  await migratedContainer.CreateItemAsync<DeviceInformationItem>(
    deviceItem.PartitionKey, 
    deviceItem
  );

// Read back the document providing the same partition key
ItemResponse<DeviceInformationItem> readResponse = 
  await migratedContainer.ReadItemAsync<DeviceInformationItem>( 
    partitionKey:deviceItem.PartitionKey, 
    id: device.Id
  );

```

Pour obtenir un exemple complet, consultez le référentiel GitHub [.Net samples][1].
                      
## <a name="migrate-the-documents"></a>Migrer les documents

Bien que la définition du conteneur ait été améliorée avec une propriété de clé de partition, les documents situés au sein du conteneur ne sont pas automatiquement migrés. Autrement dit, le chemin d’accès à la propriété de clé de partition système `/_partitionKey` n’est pas automatiquement ajouté aux documents existants. Vous devez repartitionner les documents existants en lisant les documents créés sans clé de partition et les réécrire avec la propriété `_partitionKey` dans les documents.

## <a name="access-documents-that-dont-have-a-partition-key"></a>Accéder aux documents sans clé de partition

Les applications peuvent accéder aux documents existants qui n’ont pas de clé de partition à l’aide de la propriété système spéciale appelée « PartitionKey.None » et qui correspond à la valeur des documents non migrés. Vous pouvez utiliser cette propriété dans toutes les opérations CRUD et de requête. Voici un exemple permettant de lire un document unique à partir de NonePartitionKey. 

```csharp
CosmosItemResponse<DeviceInformationItem> readResponse = 
await migratedContainer.Items.ReadItemAsync<DeviceInformationItem>( 
  partitionKey: PartitionKey.None, 
  id: device.Id
); 

```

Pour obtenir l’exemple complet sur la manière de repartitionner les documents, consultez le référentiel GitHub [.Net samples][1]. 

## <a name="compatibility-with-sdks"></a>Compatibilité avec les SDK

Les anciennes versions de SDK Azure Cosmos DB, comme V2.x.x et V1.x.x, ne prennent pas en charge la clé de propriété de partition définie par le système. Par conséquent, lorsque vous lisez la définition de conteneur à partir d’un SDK plus ancien, il n’y a pas de définition de clé de partition et ces conteneurs se comportent exactement comme avant. Les applications générées avec une ancienne version de SDK continuent de fonctionner avec des conteneurs non partitionnés comme si aucune modification n’avait été effectuée. 

Si un conteneur migré est utilisé par la version V3/dernière version du SDK et si vous commencez à remplir la clé de partition définie par le système dans les nouveaux documents, vous ne pouvez plus accéder (lecture, écriture, suppression, requête) à ces documents à partir de SDK plus anciens.

## <a name="known-issues"></a>Problèmes connus

**L’interrogation du nombre d’éléments insérés sans clé de partition à l’aide du SDK V3 peut impliquer une consommation de débit supérieure**

Si vous effectuez une requête à partir du kit SDK V3 pour les éléments insérés à l’aide du SDK V2 ou les éléments insérés à l’aide du SDK V3 avec le paramètre `PartitionKey.None`, la requête Count peut consommer plus de RU/s si le paramètre `PartitionKey.None` est fourni dans FeedOptions. Nous vous recommandons de ne pas spécifier le paramètre `PartitionKey.None` si aucun autre élément n’est inséré avec une clé de partition.

Si de nouveaux éléments sont insérés avec des valeurs différentes pour la clé de partition, l’interrogation de ce nombre d’éléments en passant la clé appropriée dans `FeedOptions` ne posera aucun problème. Après l’insertion de nouveaux documents avec la clé de partition, si vous devez interroger uniquement le nombre de documents sans la valeur de clé de partition, cette requête peut à nouveau entraîner une plus grande quantité de RU/s, de manière similaire aux collections partitionnées régulières.

## <a name="next-steps"></a>Étapes suivantes

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
* [Unités de requête dans Azure Cosmos DB](request-units.md)
* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md)
* [Utiliser un compte Azure Cosmos](./account-databases-containers-items.md)

[1]: https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/NonPartitionContainerMigration