---
title: Gestion des états de Reliable Actors
description: Décrit la manière dont l’état de Reliable Actors est géré, conservé et répliqué pour garantir une haute disponibilité.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: badfc490f26b71881e7970c2c0be3472abfec25a
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575599"
---
# <a name="reliable-actors-state-management"></a>Gestion des états de Reliable Actors
Reliable Actors désignent des objets monothread capables d’encapsuler la logique et l’état. Étant donné que les acteurs s’exécutent sur Reliable Services, ils peuvent conserver leur état de façon fiable à l’aide des mêmes mécanismes de persistance et de réplication. De cette façon, les acteurs ne perdent pas leur état après des incidents, après une réactivation consécutive à un nettoyage de la mémoire, ou encore après leur déplacement entre des nœuds d’un cluster dans le cadre d’un équilibrage des ressources ou de mises à niveau.

## <a name="state-persistence-and-replication"></a>Persistance et réplication de l’état
Toutes les instances Reliable Actors sont considérées comme des instances *avec état* étant donné que chaque instance d’acteur est mappée à un identifiant unique. Autrement dit, les appels répétés au même ID d’acteur sont acheminés vers la même instance d’acteur. En revanche, dans un système sans état, rien ne peut garantir que les appels client sont acheminés vers le même serveur à chaque fois. Pour cette raison, les services d’acteur sont toujours des services avec état.

Même si les acteurs sont considérés comme des services avec état, cela ne signifie pas qu’ils doivent stocker l’état de manière fiable. Les acteurs peuvent choisir le niveau de persistance et de réplication de l’état en fonction de leurs exigences en matière de stockage de données :

* **État persistant** : l’état est conservé sur le disque et répliqué sur au moins trois réplicas. L’état persistant est l’option de stockage d’état la plus fiable, où l’état peut persister après une panne complète du cluster.
* **État volatil** : l’état est répliqué sur au moins trois réplicas et conservé uniquement en mémoire. L’état volatile garantit une résilience contre les défaillances de nœud et d’acteur, ainsi que pendant les mises à niveau et l’équilibrage des ressources. Toutefois, l’état n’est pas conservé sur le disque. Si tous les réplicas sont perdus en même temps, l’état est également perdu.
* **État non persistant** : l’état n’est ni répliqué, ni écrit sur le disque. Utilisez-le uniquement pour les acteurs qui n’ont pas besoin de conserver un état fiable.

Chaque niveau de persistance représente simplement une autre configuration du *fournisseur d’état* et de la *réplication* de votre service. Le fournisseur d’état (le composant Reliable Service conçu pour stocker l’état) détermine si l’état sera ou non écrit sur le disque. La réplication varie selon le nombre de réplicas avec lesquels est déployé un service. De la même manière que Reliable Services, le fournisseur d’état et le nombre de réplicas peuvent facilement être définis manuellement. L’infrastructure d’acteurs fournit un attribut, qui, lorsqu’il est utilisé sur un acteur, sélectionne automatiquement un fournisseur d’état par défaut et génère automatiquement des paramètres pour le nombre de réplicas afin d’obtenir un de ces trois paramètres de persistance. L’attribut StatePersistence n’est pas hérité par la classe dérivée, chaque type d’acteur doit fournir son niveau de StatePersistence.

### <a name="persisted-state"></a>État persistant
```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl  extends FabricActor implements MyActor
{
}
```  
Ce paramètre utilise un fournisseur d’état qui stocke les données sur disque et définit automatiquement le nombre de réplicas de service à 3.

### <a name="volatile-state"></a>État volatil
```csharp
[StatePersistence(StatePersistence.Volatile)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Volatile)
class MyActorImpl extends FabricActor implements MyActor
{
}
```
Ce paramètre utilise un fournisseur d’état uniquement en mémoire et définit le nombre de réplicas à 3.

### <a name="no-persisted-state"></a>État non persistant
```csharp
[StatePersistence(StatePersistence.None)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.None)
class MyActorImpl extends FabricActor implements MyActor
{
}
```
Ce paramètre utilise un fournisseur d’état uniquement en mémoire et définit le nombre de réplicas à 1.

### <a name="defaults-and-generated-settings"></a>Valeurs par défaut et paramètres générés
Lorsque vous utilisez l’attribut `StatePersistence`, un fournisseur d’état est automatiquement sélectionné pour vous lors de l’exécution au démarrage du service d’acteur. Toutefois, le nombre de réplicas est défini au moment de la compilation par les outils de génération d’acteurs Visual Studio. Ces outils génèrent automatiquement un *service par défaut* pour le service d’acteur dans ApplicationManifest.xml. Les paramètres sont créés pour la **taille minimale du jeu de réplicas** et la **taille cible du jeu de réplicas**.

Vous pouvez modifier ces paramètres manuellement. Cependant, chaque fois que l’attribut `StatePersistence` est modifié, les paramètres sont rétablis aux valeurs par défaut de taille de jeu de réplicas pour l’attribut `StatePersistence` sélectionné, ce qui remplace toutes les valeurs précédentes. En d’autres termes, les valeurs que vous définissez dans le fichier ServiceManifest.xml sont remplacées au moment de la génération *uniquement* quand vous modifiez la valeur d’attribut `StatePersistence`.

```xml
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application12Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Parameters>
      <Parameter Name="MyActorService_PartitionCount" DefaultValue="10" />
      <Parameter Name="MyActorService_MinReplicaSetSize" DefaultValue="3" />
      <Parameter Name="MyActorService_TargetReplicaSetSize" DefaultValue="3" />
   </Parameters>
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="MyActorPkg" ServiceManifestVersion="1.0.0" />
   </ServiceManifestImport>
   <DefaultServices>
      <Service Name="MyActorService" GeneratedIdRef="77d965dc-85fb-488c-bd06-c6c1fe29d593|Persisted">
         <StatefulService ServiceTypeName="MyActorServiceType" TargetReplicaSetSize="[MyActorService_TargetReplicaSetSize]" MinReplicaSetSize="[MyActorService_MinReplicaSetSize]">
            <UniformInt64Partition PartitionCount="[MyActorService_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
         </StatefulService>
      </Service>
   </DefaultServices>
</ApplicationManifest>
```

## <a name="state-manager"></a>Gestionnaire d’état
Chaque instance d’acteur possède son propre gestionnaire d’état, c’est-à-dire une structure de données de type dictionnaire qui stocke les paires clé/valeur de manière fiable. Le gestionnaire d’état est un wrapper autour d’un fournisseur d’état. Vous pouvez l’utiliser pour stocker des données, quel que soit le réglage de persistance utilisé. Il ne garantit pas qu’un service d’acteur en cours d’exécution puisse être modifié pour passer d’un paramètre d’état volatil (en mémoire uniquement) à un paramètre d’état persistant via une mise à niveau propagée, tout en conservant les données. Toutefois, il est possible de modifier le nombre de réplicas d’un service en cours d’exécution.

Les clés de gestionnaire d’état doivent être des chaînes. Les valeurs sont génériques et peuvent être de n’importe quel type, y compris de types personnalisés. Les valeurs stockées dans le gestionnaire d’état doivent être sérialisables en contrat de données, car elles peuvent être transmises sur le réseau vers d’autres nœuds pendant la réplication et peuvent être écrites sur le disque, en fonction du paramètre de persistance d’état d’un acteur.

Pour la gestion des états, le gestionnaire d’état expose des méthodes de dictionnaire courantes similaires à celles disponibles dans Reliable Dictionary.

Pour des exemples de gestion de l’état d’acteur, lire [Utiliser, enregistrer et supprimer l’état de Reliable Actors](service-fabric-reliable-actors-access-save-remove-state.md).

## <a name="best-practices"></a>Meilleures pratiques
Voici quelques pratiques et conseils de dépannage pour la gestion de l’état d’acteur.

### <a name="make-the-actor-state-as-granular-as-possible"></a>Rendre l’état d’acteur aussi précis que possible
Ceci est essentiel pour les performances et l’utilisation des ressources de votre application. À chaque écriture/mise à jour d’un « état nommé » pour un acteur, la valeur entière correspondant à cet « état nommé » est sérialisée et envoyée sur le réseau vers les réplicas secondaires.  Les réplicas secondaires écrivent sur le disque local et répondent au réplica principal. Lorsque le réplica principal reçoit les accusés de réception à partir d’un quorum de réplicas secondaires, il écrit l’état sur son disque local. Par exemple, supposons que la valeur est une classe qui compte 20 membres et a une taille de 1 Mo. Même si vous avez modifié un seul des membres de la classe qui a de taille de 1 Ko, vous finissez par payer les coûts de sérialisation et d’écriture sur le réseau/disque par pour 1 Mo. De même, si la valeur est une collection (par exemple, une liste, un tableau ou un dictionnaire), vous payez le coût de la collection complète, même si vous modifiez l’un de ses membres. L’interface de StateManager de la classe d’acteur est semblable à un dictionnaire. Vous devez toujours modéliser la structure de données qui représente un état d’acteur par dessus ce dictionnaire.
 
### <a name="correctly-manage-the-actors-life-cycle"></a>Gérer correctement le cycle de vie de l’acteur
Vous devez disposer d’une stratégie claire pour la gestion de la taille de l’état dans chaque partition d’un service d’acteur. Votre service d’acteur doit avoir un nombre fixe d’acteurs et les réutiliser aussi souvent que possible. Si vous créez sans cesse des acteurs, vous devez les supprimer une fois qu’ils ont terminé leur travail. L’infrastructure des acteurs stocke des métadonnées sur chaque acteur existant. La suppression de tous les états d’un acteur ne supprime pas les métadonnées associées. Vous devez supprimer l’acteur (voir [Suppression des acteurs et de leur état](service-fabric-reliable-actors-lifecycle.md#manually-deleting-actors-and-their-state)) pour supprimer toutes les informations stockées dans le système. Au titre de vérification supplémentaire, vous devez interroger le service d’acteur (voir [Énumération des acteurs](service-fabric-reliable-actors-enumerate.md)) de temps à autre pour vous assurer que le nombre d’acteurs s’inscrit dans la plage attendue.
 
Si la taille du fichier de base de données d’un service d’acteur augmente au-delà de la taille attendue, suivez les recommandations ci-dessus. Si vous suivez ces recommandations et que vous rencontrez toujours des problèmes de taille de fichier de base de données, vous devez [ouvrir un ticket de support](service-fabric-support.md) auprès de l'équipe produit pour obtenir de l'aide.

## <a name="next-steps"></a>Étapes suivantes

L’état stocké dans Reliable Actors doit être sérialisé avant d’être écrit sur le disque et répliqué pour une haute disponibilité. En savoir plus sur [Sérialisation du type d’acteur](service-fabric-reliable-actors-notes-on-actor-type-serialization.md).

Ensuite, en savoir plus sur l’[analyse des performances et des diagnostics des acteurs](service-fabric-reliable-actors-diagnostics.md).
