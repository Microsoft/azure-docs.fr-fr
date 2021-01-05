---
title: Gérer les informations d’identification dans la bibliothèque de client de bases de données élastiques
description: Définition du niveau d’informations d’identification, de celui d’administrateur à celui de la lecture seule, pour les applications de base de données élastique.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/03/2019
ms.openlocfilehash: 5b91986d4f94861b87e413c9ff781107c3ed04a3
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92786596"
---
# <a name="credentials-used-to-access-the-elastic-database-client-library"></a>Informations d’identification utilisées pour accéder à la bibliothèque cliente de la base de données élastique
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

La [bibliothèque cliente de la base de données élastique](elastic-database-client-library.md) utilise trois types d’informations d’identification différents pour accéder au [gestionnaire des cartes de partitions](elastic-scale-shard-map-management.md). En fonction du besoin, utilisez les informations d’identification avec le niveau d’accès le plus faible possible.

* **Informations d'identification d'administration** : pour créer ou manipuler un gestionnaire des cartes de partitions. (Voir le [glossaire](elastic-scale-glossary.md).)
* **Informations d’identification d’accès** : pour accéder à un gestionnaire des cartes de partitions existant afin d’obtenir des informations sur les partitions.
* **Informations d'identification de connexion** : pour se connecter à des partitions.

Consultez également la section [Gestion des bases de données et des connexions dans la base de données Azure SQL](logins-create-manage.md)

## <a name="about-management-credentials"></a>À propos des informations d'identification d'administration

Les informations d’identification de gestion sont utilisées lors de la création d’un objet **ShardMapManager** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager)) pour les applications qui manipulent des cartes de partitions. (Par exemple, consultez [Ajout d’une partition à l’aide des outils de base de données élastique](elastic-scale-add-a-shard.md) et [Routage dépendant des données](elastic-scale-data-dependent-routing.md)). L’utilisateur de la bibliothèque cliente de l’infrastructure élastique crée les utilisateurs et les connexions SQL nécessaires et s’assure qu’ils sont dotés des autorisations en lecture/écriture pour la base de données de cartes de partitions globale et pour toutes les bases de données de partitions. Ces informations d'identification sont utilisées pour mettre à jour la carte de partitions globale et les cartes de partitions locales lorsque des modifications sont apportées à la carte de partitions. Par exemple, utilisez les informations d'identification de gestion pour créer l'objet de gestionnaire des cartes de partitions (en utilisant **GetSqlShardMapManager** , [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.getsqlshardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager)) :

```java
// Obtain a shard map manager.
ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(smmAdminConnectionString,ShardMapManagerLoadPolicy.Lazy);
```

La variable **smmAdminConnectionString** est une chaîne de connexion qui comporte les informations d’identification de gestion. L'identifiant utilisateur et le mot de passe fournissent l'accès en lecture/écriture à la base de données de cartes de partitions et aux partitions individuelles. La chaîne de connexion de gestion inclut également le nom du serveur et le nom de la base de données afin d'identifier la base de données de cartes de partitions globale. Voici une chaîne de connexion classique à cet effet :

```java
"Server=<yourserver>.database.windows.net;Database=<yourdatabase>;User ID=<yourmgmtusername>;Password=<yourmgmtpassword>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;”
```

N’utilisez pas de valeurs de type « username@server » ; utilisez simplement la valeur « nom d’utilisateur » à la place.  Cela vient du fait que les informations d’identification doivent fonctionner avec la base de données du gestionnaire de cartes de partitions et les partitions, qui peuvent se trouver sur différents serveurs.

## <a name="access-credentials"></a>Informations d’identification d’accès

Lorsque vous créez un gestionnaire des cartes de partitions dans une application qui ne gère pas les cartes de partitions, utilisez les informations d'identification dotées des autorisations en lecture seule pour la carte de partitions globale. Les informations extraites de la carte de partitions globale avec ces informations d’identification sont utilisées pour le [routage dépendant des données](elastic-scale-data-dependent-routing.md) et pour remplir le cache de la carte de partitions sur le client. Les informations d’identification sont fournies via le même modèle d’appel à **GetSqlShardMapManager** :

```java
// Obtain shard map manager.
ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(smmReadOnlyConnectionString, ShardMapManagerLoadPolicy.Lazy);  
```

Notez l’utilisation de **smmReadOnlyConnectionString** afin de refléter l’utilisation d’informations d’identification différentes pour l’accès des utilisateurs **non-administrateurs**  : ces informations d’identification ne doivent pas fournir d’autorisations d’écriture sur la carte de partitions globale.

## <a name="connection-credentials"></a>Informations d'identification de connexion

Des informations d'identification supplémentaires sont requises lorsque vous utilisez la méthode **OpenConnectionForKey** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper.listshardmapper.openconnectionforkey), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey)) pour accéder à une partition associée à une clé. Ces informations d'identification doivent fournir des autorisations d'accès en lecture seule aux tables de la carte de partitions locale résidant sur la partition. Cela est nécessaire pour effectuer une validation de connexion pour le routage dépendant des données sur la partition. Cet extrait de code permet l'accès à des données dans le contexte d’un routage dépendant des données :

```csharp
using (SqlConnection conn = rangeMap.OpenConnectionForKey<int>(targetWarehouse, smmUserConnectionString, ConnectionOptions.Validate))
```

Dans cet exemple, **smmUserConnectionString** comporte la chaîne de connexion correspondant aux informations d’identification de l’utilisateur. Pour Azure SQL Database, voici une chaîne de connexion classique pour les informations d’identification de l’utilisateur :

```java
"User ID=<yourusername>; Password=<youruserpassword>; Trusted_Connection=False; Encrypt=True; Connection Timeout=30;”  
```

Tout comme pour les informations d’identification de l’administrateur, n’utilisez pas de valeurs sous la forme « username@server ». Utilisez simplement « username ».  Notez également que la chaîne de connexion ne comporte pas de nom de serveur et de nom de base de données. En effet, l’appel de **OpenConnectionForKey** dirige automatiquement la connexion vers la partition appropriée en fonction de la clé. Par conséquent, il n'est pas nécessaire de fournir les noms de la base de données et du serveur.

## <a name="see-also"></a>Voir aussi

[Gestion des bases de données et des connexions dans Azure SQL Database](logins-create-manage.md)

[Sécurisation de votre base de données SQL](security-overview.md)

[Tâches de base de données élastique](elastic-jobs-overview.md)

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]