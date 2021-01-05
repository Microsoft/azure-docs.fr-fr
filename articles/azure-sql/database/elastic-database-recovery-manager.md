---
title: Recovery Manager pour résoudre les problèmes de mappage de partitions
description: Utiliser la classe RecoveryManager pour résoudre les problèmes des mappages de partition
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/03/2019
ms.openlocfilehash: 91bcd998849c619a328a198c97bb8c977b9d8232
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792223"
---
# <a name="using-the-recoverymanager-class-to-fix-shard-map-problems"></a>Utilisation de la classe RecoveryManager pour résoudre les problèmes de correspondance de partitionnement
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

La classe [RecoveryManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager) permet aux applications ADO.NET de détecter et corriger facilement les incohérences entre le mappage de partitions global (GSM) et le mappage de partitions local (LSM) dans un environnement de base de données partitionné.

Le GSM et LSM assurent le suivi de mappage de chaque base de données dans un environnement partitionné. Il arrive qu’un arrêt se produise entre le GSM et le LSM. Dans ce cas, utilisez la classe RecoveryManager pour détecter et résoudre l’arrêt.

La classe RecoveryManager fait partie de la [Bibliothèque de client de base de données élastique](elastic-database-client-library.md).

![Mappage de partition][1]

Vous trouverez les définitions des termes évoqués ici dans la page [Glossaire des outils de base de données élastique](elastic-scale-glossary.md). Pour comprendre comment **ShardMapManager** est utilisé pour gérer les données dans une solution partitionnée, voir [gestion de mappage de partition](elastic-scale-shard-map-management.md).

## <a name="why-use-the-recovery-manager"></a>Pourquoi utiliser le Gestionnaire de récupération

Dans un environnement de base de données partitionnée, il existe un seul client par base de données et plusieurs bases de données par serveur. L'environnement peut également contenir plusieurs serveurs. Chaque base de données est mappée dans le mappage de partition, afin que les appels puissent être acheminés vers le serveur et la base de données corrects. Les bases de données sont suivies selon une **clé de partitionnement** et chaque partition se voit affecter une **plage de valeurs clés** . Par exemple, une clé de partitionnement peut représenter les noms de clients, de « D » à « F ». Le mappage de toutes les partitions (également connues sous le nom de bases de données) et leurs plages de mappage sont contenus dans le **mappage de partition global (GSM)** . Chaque base de données contient également un mappage des plages contenues dans la partition. Il est connu comme **mappage de partition locale (LSM)** . Lorsqu'une application se connecte à une partition, le mappage est mis en cache avec l'application pour une récupération rapide. Le LSM est utilisé pour valider des données placées en mémoire cache.

Le GSM et le LSM peuvent devenir désynchronisés pour les raisons suivantes :

1. La suppression d'un partitionnement dont on pense que la plage n'est plus utilisée ou l'attribution d'un nouveau nom au partitionnement. La suppression d’une partition  se traduit par un **mappage de partition orphelin** . De même, une base de données renommée peut causer un mappage de partition orphelin. En fonction de l'objectif de la modification, il peut être nécessaire de supprimer la partition ou de mettre à jour l'emplacement de partition. Pour récupérer une base de données supprimée, consultez [Restauration d’une base de données supprimée](recovery-using-backups.md).
2. Un événement de géo-basculement se produit. Pour continuer, il faut mettre à jour le nom du serveur et le nom de la base de données du gestionnaire de mappage de partition dans l’application, puis mettre à jour les détails de mappage de partition pour toutes les partitions d’un mappage de partition. En cas de basculement géographique, cette logique de récupération doit être automatisée au sein du flux de travail de basculement. L’automatisation des actions de récupération offre des possibilités de gestion sans friction pour les bases de données géolocalisées et évite les interventions manuelles. Pour en savoir plus sur les options de récupération d’une base de données en cas de panne du centre de données, voir les rubriques sur la [continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md) et la [récupération d’urgence](disaster-recovery-guidance.md).
3. Une partition ou une base de données ShardMapManager est restaurée vers une version antérieure. Pour en savoir plus sur la récupération jusqu`à une date et heure avec les sauvegardes, consultez [Récupération avec des sauvegardes](recovery-using-backups.md).

Pour plus d’informations sur les outils de base de données élastique, la géoréplication et la restauration d’Azure SQL Database, voir les rubriques suivantes :

* [Vue d’ensemble : Continuité de l’activité cloud et récupération d’urgence de base de données avec SQL Database](business-continuity-high-availability-disaster-recover-hadr-overview.md).
* [Prise en main des outils de base de données élastiques](elastic-scale-get-started.md)  
* [Gestion de ShardMap](elastic-scale-shard-map-management.md)

## <a name="retrieving-recoverymanager-from-a-shardmapmanager"></a>Récupération RecoveryManager à partir d’un ShardMapManager

Cette première sert à créer une instance de RecoveryManager. La [méthode GetRecoveryManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getrecoverymanager) retourne le Recovery Manager de l’instance [ShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager) en cours. Pour résoudre les incohérences dans le mappage du partitionnement, vous devez d’abord récupérer RecoveryManager pour ce mappage de partition particulier.

   ```java
    ShardMapManager smm = ShardMapManagerFactory.GetSqlShardMapManager(smmConnectionString,  
             ShardMapManagerLoadPolicy.Lazy);
             RecoveryManager rm = smm.GetRecoveryManager();
   ```

Dans cet exemple, RecoveryManager est initialisé à partir de ShardMapManager. Le ShardMapManager contenant un ShardMap également déjà initialisé.

Comme ce code d’application manipule la correspondance de mappage de partitionnement, les informations d’identification utilisées dans la méthode de fabrique (dans l’exemple précédent, smmConnectionString) doivent être des informations d’identification disposant des autorisations en lecture/écriture dans la base de données GSM référencée par la chaîne de connexion. Ces informations d’identification sont généralement différentes des informations d’identification utilisées pour ouvrir des connexions pour l’acheminement dépendant des données. Pour plus d’informations, consultez [Utilisation des informations d’identification dans le client de base de données élastique](elastic-scale-manage-credentials.md).

## <a name="removing-a-shard-from-the-shardmap-after-a-shard-is-deleted"></a>Suppression d’une partition du ShardMap après la suppression d’une partition

La [méthode DetachShard](/previous-versions/azure/dn842083(v=azure.100)) détache la partition donnée du mappage de partition et supprime les mappages associés à la partition.  

* Le paramètre d’emplacement est l’emplacement de partition, en particulier le nom du serveur et nom de la base de données de la partition qui est détachée.
* Le paramètre shardMapName correspond au nom de mappage de partition. Il est requis uniquement lorsque plusieurs mappages de partition sont gérés par le même gestionnaire de mappage de partition. facultatif.

> [!IMPORTANT]
> Utilisez cette technique seulement si vous êtes certain que la plage de mappage mis à jour est vide. Les méthodes ci-dessus ne vérifient pas les données de la plage déplacée, il est donc préférable d’inclure des vérifications dans votre code.

Cet exemple supprime les partitions du mappage de partition.

   ```java
   rm.DetachShard(s.Location, customerMap);
   ```

La carte de partitions reflète l’emplacement de la partition dans GSM avant sa suppression. Étant donné que la partition a été supprimée, on suppose que c’est intentionnel et la plage de clés de partitionnement n’est plus en cours d’utilisation. Sinon, vous pouvez exécuter une restauration à un moment donné. pour récupérer la partition à partir d’un point dans le temps ultérieur. (Dans ce cas, consultez la section suivante pour détecter les incohérences de partition.) Pour effectuer une récupération, consultez la rubrique sur la [récupération jusqu`à une date et heure](recovery-using-backups.md).

Si la suppression de base de données est intentionnelle, l’opération de nettoyage administratif final consiste à supprimer l’entrée de la partition dans le gestionnaire de table de partition. L’application évite ainsi d’écrire par inadvertance des informations pour une plage non prévue.

## <a name="to-detect-mapping-differences"></a>Pour détecter les différences de mappage

La [méthode DetectMappingDifferences](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.detectmappingdifferences) sélectionne et retourne un des mappages de partition (local ou global) comme source de vérité et rapproche les mappages sur les deux mappages de partition (GSM et LSM).

   ```java
   rm.DetectMappingDifferences(location, shardMapName);
   ```

* L *’emplacement* indique le nom du serveur et le nom de la base de données.
* Le paramètre *shardMapName* correspond au nom de mappage de partition. Il est requis uniquement si plusieurs mappages de partition sont gérés par le même gestionnaire de mappage de partition. facultatif.

## <a name="to-resolve-mapping-differences"></a>Pour résoudre les différences de mappage

La [méthode ResolveMappingDifferences](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.resolvemappingdifferences) sélectionne l’un des mappages de partition (local ou global) en tant que source de vérité et rapproche les mappages sur les deux cartes de partitionnement (GSM et LSM).

   ```java
   ResolveMappingDifferences (RecoveryToken, MappingDifferenceResolution.KeepShardMapping);
   ```

* Le paramètre *RecoveryToken* énumère les différences de mappage entre le GSM et le LSM pour la partition spécifique.
* L’ [énumération MappingDifferenceResolution](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.mappingdifferenceresolution) est utilisée pour indiquer la méthode de résolution de la différence entre les mappages de partition.
* **MappingDifferenceResolution.KeepShardMapping** est recommandé si LSM contient les correspondances exactes et que le mappage de la partition doit donc être utilisé. C’est généralement le cas lors d’un basculement : la partition se trouve désormais sur un nouveau serveur. Comme la partition doit d’abord être supprimée du GSM (à l’aide de la méthode RecoveryManager.DetachShard), il n’existe plus de mappage sur le GSM. Par conséquent, le LSM doit être utilisé pour rétablir le mappage de partition.

## <a name="attach-a-shard-to-the-shardmap-after-a-shard-is-restored"></a>Attachez une partition au ShardMap une fois la partition restaurée

La [méthode AttachShard](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.attachshard) attache la partition donnée à la table de partition. Il détecte des incohérences de mappage de partition et met à jour les mappages pour qu’ils correspondent à la partition au moment de la restauration de la partition. On suppose que la base de données est également renommée pour reprendre le nom de la base de données d’origine (avant la restauration de la partition), car la restauration à un point donné revient par défaut à une nouvelle base de données ajoutée avec un horodatage.

   ```java
   rm.AttachShard(location, shardMapName)
   ```

* Le paramètre *location* est le nom du serveur et le nom de la base de données de la partition qui est attachée.
* Le paramètre *shardMapName* correspond au nom de mappage de partition. Il est requis uniquement lorsque plusieurs mappages de partition sont gérés par le même gestionnaire de mappage de partition. facultatif.

Cet exemple ajoute une partition dans le mappage de la partition venant d'être restaurée à partir d'une restauration antérieure. Dans la mesure où la partition (à savoir le mappage pour la partition dans le LSM) a été restaurée, il est potentiellement incompatible avec l’entrée de partition dans le GSM. Hors de cet exemple de code, la partition a été restaurée et renommée avec le nom d’origine de la base de données. Comme elle a été restaurée, on suppose que le mappage présent dans le gestionnaire LSM est le mappage approuvé.

   ```java
   rm.AttachShard(s.Location, customerMap);
   var gs = rm.DetectMappingDifferences(s.Location);
      foreach (RecoveryToken g in gs)
       {
       rm.ResolveMappingDifferences(g, MappingDifferenceResolution.KeepShardMapping);
       }
   ```

## <a name="updating-shard-locations-after-a-geo-failover-restore-of-the-shards"></a>Mise à jour des emplacements de partition après un basculement géographique (restauration) des partitions

En cas de basculement géographique, la base de données secondaire est accessible en lecture et devient la nouvelle base de données primaire. Le nom du serveur et éventuellement la base de données (selon votre configuration), peut être différent de celui du site principal d’origine. Par conséquent, les entrées de mappage de la partition dans le GSM et LSM doivent être corrigées. De même, si la base de données est restaurée avec un autre nom ou  un emplacement, ou un  point antérieur dans le temps, ceci peut entraîner des incohérences dans les mappages de partition. Le Gestionnaire de mappage de partition gère la distribution des connexions ouvertes sur la base de données. La distribution est basée sur les données de la carte de partitions et la valeur de la clé de partitionnement cible de la demande d’application. Après un basculement géographique, cette information doit être mise à jour avec le bon nom de serveur, nom de base de données et mappage de partition de la base de données récupérée.

## <a name="best-practices"></a>Meilleures pratiques

Le basculement géographique et la récupération sont des opérations généralement gérées par un administrateur de cloud de l’application qui utilise intentionnellement les fonctionnalités de continuité d’activité d’Azure SQL Database. La planification de la continuité des activités requiert des processus, des procédures et des mesures garantissant que les opérations de l’entreprise peuvent continuer sans interruption. Les méthodes disponibles en tant que partie de la classe RecoveryManager doivent être utilisées dans ce flux de travail pour s’assurer que le GSM et LSM sont actualisés en fonction de l’opération de récupération exécutée. Il existe cinq opérations de base pour s’assurer que GSM et LSM reflètent bien les informations précises après un événement de basculement. Le code d’application servant à exécuter ces opérations peut être intégré dans des outils et de flux de travail existants.

1. Récupérer RecoveryManager à partir de ShardMapManager.
2. Détacher la partition ancienne du mappage de partition.
3. Attacher la nouvelle partition au mappage de partition, y compris le nouvel emplacement de la partition.
4. Détecter des incohérences dans le mappage entre le GSM et LSM.
5. Résoudre les différences entre le GSM et le LSM et approuver le LSM.

Cet exemple effectue les étapes suivantes :

1. Supprimer les partitions du mappage de partition qui reflètent les emplacements des partitions avant l’événement de basculement.
2. Joint des partitions au mappage de partition qui reflètent les nouveaux emplacements des partitions (le paramètre « Configuration.SecondaryServer » est le nouveau nom de serveur, mais le même nom de base de données).
3. Extrait des jetons de récupération en détectant des différences de mappage entre le GSM et le LSM pour chaque partition.
4. Résout les incohérences en approuvant le mappage LSM de chaque partition.

   ```java
   var shards = smm.GetShards();
   foreach (shard s in shards)
   {
     if (s.Location.Server == Configuration.PrimaryServer)
         {
          ShardLocation slNew = new ShardLocation(Configuration.SecondaryServer, s.Location.Database);
          rm.DetachShard(s.Location);
          rm.AttachShard(slNew);
          var gs = rm.DetectMappingDifferences(slNew);
          foreach (RecoveryToken g in gs)
            {
               rm.ResolveMappingDifferences(g, MappingDifferenceResolution.KeepShardMapping);
            }
        }
    }
   ```

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/elastic-database-recovery-manager/recovery-manager.png