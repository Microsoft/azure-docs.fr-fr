---
title: Résoudre des erreurs courantes dans l’API Cassandra dans Azure Cosmos DB
description: Ce document décrit les manières de résoudre des problèmes couramment rencontrés dans l’API Cassandra dans Azure Cosmos DB.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: troubleshooting
ms.date: 12/01/2020
ms.author: thvankra
ms.openlocfilehash: c969e4fac3ae30088cfe47a7b0edff22c578cb8b
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97802352"
---
# <a name="troubleshoot-common-issues-in-azure-cosmos-db-cassandra-api"></a>Résoudre des problèmes courants dans l’API Cassandra dans Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

L’API Cassandra dans Azure Cosmos DB est une couche de compatibilité qui assure la [prise en charge du protocole filaire](cassandra-support.md) pour la base de données Apache Cassandra open source populaire, et qui est optimisée par [Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/introduction). En tant que service natif Cloud entièrement managé, Azure Cosmos DB fournit des [garanties de disponibilité, de débit et de cohérence](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_3/) pour l’API Cassandra. Ces garanties ne sont pas possibles dans des implémentations héritées d’Apache Cassandra. L’API Cassandra facilite également les opérations de plateforme sans maintenance et les mises à jour correctives sans temps d’arrêt. Par conséquent, un grand nombre des ses opérations principales diffèrent de celles d’Apache Cassandra. Nous recommandons donc des paramètres et des approches spécifiques pour éviter les erreurs courantes. 

Cet article décrit les erreurs et solutions courantes pour les applications qui utilisent l’API Cassandra dans Azure Cosmos DB.

## <a name="common-errors-and-solutions"></a>Erreurs courantes et solutions

| Error               |  Description             | Solution  |
|---------------------|--------------------------|-----------|
| OverloadedException (Java) | Le nombre total d’unités de requête consommées est supérieur au nombre d’unités de requête approvisionnées sur l’espace de clés ou la table. Les requêtes sont donc limitées. | Envisagez de mettre à l’échelle le débit affecté à un espace de clés ou à une table à partir du portail Azure (voir [ici](manage-scale-cassandra.md) pour les opérations de mise à l’échelle dans l’API Cassandra). Vous pouvez également implémenter une stratégie de nouvelle tentative. Pour Java, consultez exemples de nouvelles tentatives pour le [pilote v3.x](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample) et le [pilote v4.x](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample-v4). Consultez également [Extensions Azure Cosmos Cassandra pour Java](https://github.com/Azure/azure-cosmos-cassandra-extensions). |
| OverloadedException (Java), même avec un débit suffisant | Le système semble limiter les demandes en dépit d’un débit suffisant pour être approvisionné pour le volume de demande et/ou le coût d’unité de requête consommé.  | L’API Cassandra implémente un budget de débit du système pour les opérations au niveau du schéma (CREATE TABLE, ALTER TABLE, DROP TABLE). Ce budget doit être suffisant pour les opérations de schéma dans un système de production. Toutefois, si vous avez un grand nombre d’opérations au niveau du schéma, il est possible que vous dépassiez cette limite. Ce budget n’étant pas contrôlé par l’utilisateur, vous devrez envisager de réduire le nombre d’opérations de schéma en cours d’exécution. Si cette action ne résout pas le problème, ou si elle n’est pas possible pour votre charge de travail, vous pouvez [créer une demande de support Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).|
| ClosedConnectionException (Java) | Après une période d’inactivité faisant suite à des connexions réussies, l’application ne peut pas se connecter.| Cette erreur peut être due au délai d’inactivité des équilibreurs de charge Azure, qui est de 4 minutes. Définissez le paramètre Keep Alive dans le pilote (voir ci-dessous) et augmentez les paramètres Keep Alive dans le système d’exploitation. Vous pouvez également [ajuster le délai d’inactivité dans Azure Load Balancer](../load-balancer/load-balancer-tcp-idle-timeout.md?tabs=tcp-reset-idle-portal). |
| Autres erreurs de connectivité intermittentes (Java) | La connexion s’interrompt ou expire de manière inattendue. | Les pilotes Apache Cassandra pour Java fournissent deux stratégies de reconnexion natives : `ExponentialReconnectionPolicy` et `ConstantReconnectionPolicy`. Par défaut, il s’agit de `ExponentialReconnectionPolicy`. Toutefois, pour l’API Cassandra dans Azure Cosmos DB, nous vous recommandons de `ConstantReconnectionPolicy` avec un délai de 2 secondes. Consultez la [documentation du pilote](https://docs.datastax.com/en/developer/java-driver/4.9/manual/core/reconnection/) pour le pilote Java v4.x, et voyez [ici](https://docs.datastax.com/en/developer/java-driver/3.7/manual/reconnection/) pour obtenir de l’aide sur Java 3.x (voir également les exemples ci-dessous).|

Si votre erreur n’est pas répertoriée ci-dessus et si vous rencontrez une erreur lors de l’exécution d’une [opération prise en charge dans l’API Cassandra](cassandra-support.md), où l’erreur est *absente lors de l’utilisation d’Apache Cassandra native*, [créez une demande de support Azure](../azure-portal/supportability/how-to-create-azure-support-request.md)

## <a name="configuring-reconnectionpolicy-for-java-driver"></a>Configuration de ReconnectionPolicy pour pilote Java

### <a name="version-3x"></a>Version 3.x

Pour la version 3.x du pilote Java, configurez la stratégie de reconnexion lors de la création d’un objet cluster :

```java
import com.datastax.driver.core.policies.ConstantReconnectionPolicy;

Cluster.builder()
  .withReconnectionPolicy(new ConstantReconnectionPolicy(2000))
  .build();
```

### <a name="version-4x"></a>Version 4.x

Pour la version 4.x du pilote Java, configurez la stratégie de reconnexion en remplaçant les paramètres dans le fichier `reference.conf` :

```xml
datastax-java-driver {
  advanced {
    reconnection-policy{
      # The driver provides two implementations out of the box: ExponentialReconnectionPolicy and
      # ConstantReconnectionPolicy. We recommend ConstantReconnectionPolicy for Cassandra API, with 
      # base-delay of 2 seconds.
      class = ConstantReconnectionPolicy
      base-delay = 2 second
    }
}
```

## <a name="enable-keep-alive-for-java-driver"></a>Activer Keep Alive pour pilote Java

### <a name="version-3x"></a>Version 3.x

Pour la version 3.x du pilote Java, définissez Keep Alive lors de la création d’un objet cluster, puis assurez-vous que l’option Keep Alive est [activée dans le système d’exploitation](https://knowledgebase.progress.com/articles/Article/configure-OS-TCP-KEEPALIVE-000080089) :

```java
import java.net.SocketOptions;
    
SocketOptions options = new SocketOptions();
options.setKeepAlive(true);
cluster = Cluster.builder().addContactPoints(contactPoints).withPort(port)
  .withCredentials(cassandraUsername, cassandraPassword)
  .withSocketOptions(options)
  .build();
```

### <a name="version-4x"></a>Version 4.x

Pour la version 4.x du pilote Java, définissez Keep Alive en remplaçant les paramètres dans `reference.conf`, puis assurez-vous que l’option Keep Alive est [activée dans le système d’exploitation](https://knowledgebase.progress.com/articles/Article/configure-OS-TCP-KEEPALIVE-000080089) :

```xml
datastax-java-driver {
  advanced {
    socket{
      keep-alive = true
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur les [fonctionnalités prises en charge](cassandra-support.md) dans l’API Cassandra dans Azure Cosmos DB.
- Découvrez comment [migrer d’Apache Cassandra natif vers l’API Cassandra dans Azure Cosmos DB](cassandra-migrate-cosmos-db-databricks.md).

