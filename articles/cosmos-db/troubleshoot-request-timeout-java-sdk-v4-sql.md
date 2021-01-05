---
title: Résoudre les problèmes de dépassement de délai de demande ou HTTP 408 Azure Cosmos DB avec le SDK Java v4
description: Apprenez à diagnostiquer et corriger des exceptions de dépassement de délai de demande du SDK Java avec le SDK Java v4.
author: kushagrathapar
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 10/28/2020
ms.author: kuthapar
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: a805300ac62d0627c9b06188c9764a6887947afe
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94411284"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-java-v4-sdk-request-timeout-exceptions"></a>Diagnostiquer et résoudre les exceptions de dépassement de délai de demande avec le SDK Java v4 Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

L'erreur HTTP 408 se produit si le SDK n'a pas pu terminer la demande avant l'expiration du délai d'attente.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
La liste suivante répertorie les causes connues et les solutions pour les exceptions de dépassement de délai de demande.

### <a name="existing-issues"></a>Problèmes existants
Si vous constatez que les requêtes sont bloquées pendant une durée plus longue ou que le délai est plus fréquemment dépassé, mettez à niveau le Kit de développement logiciel (SDK) Java v4 vers la dernière version. REMARQUE :  Nous vous recommandons vivement d’utiliser la version 4.7.0 et les versions supérieures. Pour plus d’informations, consultez les [Notes de publication du Kit de développement logiciel (SDK) Java v4](sql-api-sdk-java-v4.md).

### <a name="high-cpu-utilization"></a>Utilisation élevée du processeur
L'utilisation élevée du processeur est le cas le plus courant. Pour une latence optimale, l'utilisation du processeur doit être d'environ 40 %. Utilisez 10 secondes comme intervalle pour superviser l'utilisation maximale (non moyenne) du processeur. Les pics d'utilisation du processeur sont plus courants avec les requêtes entre partitions où il peut y avoir plusieurs connexions pour une seule requête.

#### <a name="solution"></a>Solution :
L'application cliente qui utilise le SDK doit faire l'objet d'un scale-up ou d'un scale-out.

### <a name="connection-throttling"></a>Limitation de la connexion
La limitation de la connexion peut se produire en raison d’une Limite de connexion sur un ordinateur hôte ou d’une insuffisance de ports Azure SNAT (PAT).

### <a name="connection-limit-on-a-host-machine"></a>Limite de connexion sur un ordinateur hôte
Certains systèmes Linux, tels que Red Hat, ont une limite supérieure quant au nombre total de fichiers ouverts. Les sockets dans Linux étant implémentés en tant que fichiers, ce nombre limite aussi le nombre total de connexions. Exécutez la commande suivante :

```bash
ulimit -a
```

#### <a name="solution"></a>Solution :
La quantité maximale de fichiers ouverts autorisée, qui sont identifiés comme « nofile », doit être d’au moins 10 000. Pour plus d’informations, consultez les [conseils sur les performances](performance-tips-java-sdk-v4-sql.md) associés au kit SDK Java v4 Azure Cosmos DB.

### <a name="socket-or-port-availability-might-be-low"></a>La disponibilité du socket ou du port peut être faible
Quand ils s’exécutent dans Azure, les clients qui utilisent le SDK Java peuvent rencontrer un problème d’épuisement de ports Azure SNAT (PAT).

#### <a name="solution-1"></a>Solution 1 :
Si vous utilisez des machines virtuelles Azure, suivez le [guide consacré à l'insuffisance de ports SNAT](troubleshoot-java-sdk-v4-sql.md#snat).

#### <a name="solution-2"></a>Solution 2 :
Si vous utilisez Azure App Service, suivez le [guide de dépannage des erreurs de connexion](../app-service/troubleshoot-intermittent-outbound-connection-errors.md#cause) et [utilisez les diagnostics App Service](https://azure.github.io/AppService/2018/03/01/Deep-Dive-into-TCP-Connections-in-App-Service-Diagnostics.html).

#### <a name="solution-3"></a>Solution 3 :
Si vous utilisez Azure Functions, suivez la [recommandation Azure Functions](../azure-functions/manage-connections.md#static-clients) de gestion des clients statiques ou des singletons pour tous les services impliqués (y compris Cosmos DB). Vérifiez les [limites de service](../azure-functions/functions-scale.md#service-limits) en fonction du type et de la taille de votre hébergement Function App.

#### <a name="solution-4"></a>Solution 4 :
Si vous utilisez un proxy HTTP, vérifiez qu’il peut prendre en charge le nombre de connexions configuré dans SDK `GatewayConnectionConfig`. Sinon, vous serez confronté à des problèmes de connexion.

### <a name="create-multiple-client-instances"></a>Créer plusieurs instances clientes
La création de plusieurs instances clientes peut entraîner des problèmes de conflit et de dépassement de délai de connexion.

#### <a name="solution-1"></a>Solution 1 :
Suivez les [conseils en lien avec les performances](performance-tips-java-sdk-v4-sql.md#sdk-usage) et utilisez une seule instance CosmosClient pour l’ensemble de l’application.

#### <a name="solution-2"></a>Solution 2 :
S’il n’est pas possible d’avoir CosmosClient singleton dans une application, nous vous recommandons d’utiliser le partage de connexion entre plusieurs clients Cosmos via cette API `connectionSharingAcrossClientsEnabled(true)` dans CosmosClient. Si vous avez plusieurs instances du client Cosmos dans la même machine virtuelle Java qui interagissent avec plusieurs comptes Cosmos, l’activation de cette option permet le partage de connexion en mode Direct, si possible entre les instances du client Cosmos. Notez que lorsque vous définissez cette option, la configuration de la connexion (par exemple, configuration du délai d’attente du socket, configuration du délai d’inactivité) du premier client instancié sera utilisée pour toutes les autres instances du client.

### <a name="hot-partition-key"></a>Clé de partition à chaud
Azure Cosmos DB répartit le débit global approvisionné de manière uniforme entre les partitions physiques. Lorsqu'il existe une partition à chaud, une ou plusieurs clés de partition logique sur une partition physique consomment la totalité des unités de requête par seconde (RU/s) de la partition physique. Dans le même temps, les RU/s d'autres partitions physiques ne sont plus utilisées. Par conséquent, le nombre total de RU/s consommées sera inférieur au nombre global de RU/s approvisionnées dans la base de données ou le conteneur, mais vous verrez toujours une limitation (429s) sur les demandes par rapport à la clé de partition logique à chaud. Utilisez la [métrique Consommation d’unités de requête normalisée](monitor-normalized-request-units.md) pour voir si la charge de travail rencontre une partition à chaud. 

#### <a name="solution"></a>Solution :
Choisissez une clé de partition appropriée qui répartit uniformément le volume de demandes et le stockage. Découvrez comment [changer votre clé de partition](https://devblogs.microsoft.com/cosmosdb/how-to-change-your-partition-key/).

### <a name="high-degree-of-concurrency"></a>Degré élevé de simultanéité
L'application atteint un niveau élevé de simultanéité, ce qui peut entraîner des conflits sur le canal.

#### <a name="solution"></a>Solution :
L'application cliente qui utilise le SDK doit faire l'objet d'un scale-up ou d'un scale-out.

### <a name="large-requests-or-responses"></a>Demandes ou réponses volumineuses
Des demandes ou réponses volumineuses peuvent entraîner un blocage en tête de ligne sur le canal et exacerber les conflits, même avec un degré relativement faible de simultanéité.

#### <a name="solution"></a>Solution :
L'application cliente qui utilise le SDK doit faire l'objet d'un scale-up ou d'un scale-out.

### <a name="failure-rate-is-within-the-azure-cosmos-db-sla"></a>Le taux d'échec est compris dans le contrat de niveau de service Azure Cosmos DB
L’application doit être en mesure de gérer les défaillances temporaires et de réessayer si nécessaire. Il n'y a pas de nouvelle tentative pour les exceptions 408 car, lors de la création de chemins, il est impossible de savoir si le service a créé l'élément ou non. Le renvoi d’un même élément pour la création entraîne une exception de conflit. La logique métier des applications utilisateur peut avoir une logique personnalisée pour gérer les conflits, ce qui permet d'éviter l'ambiguïté d'un élément existant par rapport à un conflit suite à une nouvelle tentative de création.

### <a name="failure-rate-violates-the-azure-cosmos-db-sla"></a>Le taux d'échec n'est pas conforme au contrat de niveau de service Azure Cosmos DB
Contactez [Support Azure](https://aka.ms/azure-support).

## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-java-sdk-v4-sql.md) des problèmes lors de l'utilisation du SDK Java v4 Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour [Java v4](performance-tips-java-sdk-v4-sql.md).
