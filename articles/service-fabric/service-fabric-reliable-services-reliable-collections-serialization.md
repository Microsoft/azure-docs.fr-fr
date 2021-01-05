---
title: Sérialisation des objets Reliable Collection
description: Découvrez-en plus sur la sérialisation des objets Reliable Collection dans Azure Service Fabric, notamment la stratégie par défaut et la définition de la sérialisation personnalisée.
ms.topic: conceptual
ms.date: 5/8/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 1d29f5d3391bdea0b21997a392d4d2e053b7ec65
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89018618"
---
# <a name="reliable-collection-object-serialization-in-azure-service-fabric"></a>Sérialisation des objets Reliable Collections dans Azure Service Fabric
Les objets Reliable Collections répliquent et conserver leurs éléments pour permettre à ces derniers de résister aux pannes d’ordinateur et de courant.
La réplication et la conservation des éléments supposent au préalable d’effectuer une sérialisation.

Les objets Reliable Collections récupèrent le sérialiseur approprié pour un type donné à partir de Reliable State Manager.
Reliable State Manager contient des sérialiseurs intégrés et autorise l’inscription de sérialiseurs personnalisés pour un type donné.

## <a name="built-in-serializers"></a>Sérialiseurs intégrés

Reliable State Manager intègre un sérialiseur qui est utilisé pour certains types courants de manière à garantir par défaut une sérialisation efficace. Pour d’autres types, Reliable State Manager utilise le [DataContractSerializer](/dotnet/api/system.runtime.serialization.datacontractserializer?view=netcore-3.1).
Les sérialiseurs intégrés sont plus efficaces car ils savent que leurs types ne peuvent pas être modifiés et ils n’ont besoin d’inclure aucune information sur le type, par exemple son nom.

Reliable State Manager intègre un sérialiseur pour les types suivants : 
- Guid
- bool
- byte
- sbyte
- byte[]
- char
- string
- Décimal
- double
- float
- int
- uint
- long
- ulong
- short
- ushort

## <a name="custom-serialization"></a>Sérialisation personnalisée

Les sérialiseurs personnalisés sont couramment utilisés pour augmenter les performances ou pour chiffrer les données sur le réseau et sur le disque. Pour différentes raisons, les sérialiseurs personnalisés sont généralement plus efficaces que les sérialiseurs génériques, car ils n’ont pas besoin de sérialiser des informations relatives au type. 

[IReliableStateManager.TryAddStateSerializer\<T>](/dotnet/api/microsoft.servicefabric.data.ireliablestatemanager.tryaddstateserializer) permet d’inscrire un sérialiseur personnalisé pour le type T donné. Cette inscription doit intervenir dans la construction du StatefulServiceBase afin de s’assurer que, avant le début de la récupération, tous les objets Reliable Collections ont bien accès au sérialiseur approprié pour lire leurs données persistantes.

```csharp
public StatefulBackendService(StatefulServiceContext context)
  : base(context)
  {
    if (!this.StateManager.TryAddStateSerializer(new OrderKeySerializer()))
    {
      throw new InvalidOperationException("Failed to set OrderKey custom serializer");
    }
  }
```

> [!NOTE]
> Les sérialiseurs personnalisés prévalent sur les sérialiseurs intégrés. Par exemple, lorsqu’un sérialiseur personnalisé est inscrit pour int, il est utilisé pour sérialiser des entiers au lieu du sérialiseur intégré du type int.

### <a name="how-to-implement-a-custom-serializer"></a>Comment implémenter un sérialiseur personnalisé

Un sérialiseur personnalisé doit implémenter l’interface [IStateSerializer\<T>](/dotnet/api/microsoft.servicefabric.data.istateserializer-1).

> [!NOTE]
> IStateSerializer\<T> inclut une surcharge d’écriture et de lecture qui prend une valeur de base T supplémentaire. Cette API s’applique à la sérialisation différentielle. La fonctionnalité de sérialisation différentielle n’est pas exposée pour le moment. Par conséquent, ces deux surcharges ne sont pas appelées tant que la sérialisation différentielle n’est pas exposée et activée.

Voici un exemple de type personnalisé appelé OrderKey qui contient quatre propriétés

```csharp
public class OrderKey : IComparable<OrderKey>, IEquatable<OrderKey>
{
    public byte Warehouse { get; set; }

    public short District { get; set; }

    public int Customer { get; set; }

    public long Order { get; set; }

    #region Object Overrides for GetHashCode, CompareTo and Equals
    #endregion
}
```

Voici un exemple d’implémentation de IStateSerializer\<OrderKey>.
Notez que les surcharges de lecture et d’écriture qui prennent la valeur de base (baseValue) appellent leur surcharge respective pour assurer la compatibilité en aval.

```csharp
public class OrderKeySerializer : IStateSerializer<OrderKey>
{
  OrderKey IStateSerializer<OrderKey>.Read(BinaryReader reader)
  {
      var value = new OrderKey();
      value.Warehouse = reader.ReadByte();
      value.District = reader.ReadInt16();
      value.Customer = reader.ReadInt32();
      value.Order = reader.ReadInt64();

      return value;
  }

  void IStateSerializer<OrderKey>.Write(OrderKey value, BinaryWriter writer)
  {
      writer.Write(value.Warehouse);
      writer.Write(value.District);
      writer.Write(value.Customer);
      writer.Write(value.Order);
  }
  
  // Read overload for differential de-serialization
  OrderKey IStateSerializer<OrderKey>.Read(OrderKey baseValue, BinaryReader reader)
  {
      return ((IStateSerializer<OrderKey>)this).Read(reader);
  }

  // Write overload for differential serialization
  void IStateSerializer<OrderKey>.Write(OrderKey baseValue, OrderKey newValue, BinaryWriter writer)
  {
      ((IStateSerializer<OrderKey>)this).Write(newValue, writer);
  }
}
```

## <a name="upgradability"></a>Mise à niveau
Dans un [mise à niveau d'application propagée](service-fabric-application-upgrade.md), la mise à niveau est appliquée à un sous-ensemble de nœuds, à raison d'un domaine de mise à niveau à la fois. Pendant ce processus, certains domaines de mise à niveau se trouvent sur la version la plus récente de votre application, tandis que d'autres se trouvent sur la version antérieure. Pendant le déploiement, la nouvelle version de votre application doit être en mesure de lire l’ancienne version de vos données, tandis que l’ancienne version de votre application doit être à même de lire la nouvelle version de vos données. Si le format de données n’offre pas une compatibilité ascendante et descendante, la mise à niveau peut échouer ou, pire, des données peuvent être perdues ou endommagées.

Si vous utilisez le sérialiseur intégré, vous n’avez pas à vous soucier de la compatibilité.
Si toutefois vous utilisez un sérialiseur personnalisé ou le DataContractSerializer, les données doivent être indéfiniment compatibles en amont et en aval.
En d’autres termes, chaque version du sérialiseur doit être en mesure de sérialiser et désérialiser n’importe quelle version du type.

Les utilisateurs du contrat de données doivent suivre des règles de version bien définies pour l’ajout, la suppression et la modification de champs. En outre, le contrat de données prend en charge les champs inconnus, avec raccordement au processus de sérialisation et de désérialisation, ainsi que l’héritage de classe. Pour plus d'informations, consultez la page [Utilisation du contrat de données](/dotnet/framework/wcf/feature-details/using-data-contracts).

Les utilisateurs de sérialiseurs personnalisés doivent respecter les instructions du sérialiseur qu’ils utilisent afin de s’assurer de sa compatibilité en amont et en aval.
Une façon courante de prendre en charge toutes les versions consiste à ajouter au début des informations sur la taille et à ajouter uniquement les propriétés facultatives.
Chaque version peut ainsi lire un maximum d’informations et passer à la dernière partie du flux de données.

## <a name="next-steps"></a>Étapes suivantes
  * [Sérialisation et mise à niveau](service-fabric-application-upgrade-data-serialization.md)
  * [Référence du développeur pour les Collections fiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)
  * [mise à niveau de votre application à l’aide de Visual Studio](service-fabric-application-upgrade-tutorial.md) vous guide à travers une mise à niveau de l’application à l’aide de Visual Studio.
  * [mise à niveau de votre application à l’aide de PowerShell](service-fabric-application-upgrade-tutorial-powershell.md) vous guide à travers une mise à niveau de l’application à l’aide de PowerShell.
  * Contrôlez les mises à niveau de votre application à l'aide des [Paramètres de mise à niveau](service-fabric-application-upgrade-parameters.md).
  * Apprenez à utiliser les fonctionnalités avancées lors de la mise à niveau de votre application en consultant les [Rubriques avancées](service-fabric-application-upgrade-advanced.md).
  * Résolvez les problèmes courants de mise à niveau de l’application en vous reportant aux étapes de [Résolution des problèmes de mise à niveau des applications](service-fabric-application-upgrade-troubleshooting.md).
