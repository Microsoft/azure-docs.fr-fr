---
title: Quotas du service Azure Cosmos DB
description: Quotas et limites par défaut du service Azure Cosmos DB sur différents types de ressources.
author: abhijitpai
ms.author: abpai
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/19/2020
ms.openlocfilehash: 9d351bc945c08c82e96791059b00dc94eb94b918
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368735"
---
# <a name="azure-cosmos-db-service-quotas"></a>Quotas du service Azure Cosmos DB

[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Cet article fournit une vue d’ensemble des quotas par défaut appliqués à différentes ressources dans Azure Cosmos DB.

## <a name="storage-and-database-operations"></a>Opérations de stockage et de base de données

Une fois que vous avez créé un compte Azure Cosmos dans votre abonnement, vous pouvez y gérer des données en [créant des bases de données, des conteneurs et des éléments](account-databases-containers-items.md).

### <a name="provisioned-throughput"></a>Débit approvisionné

Vous pouvez provisionner le débit au niveau d’un conteneur ou d’une base de données en termes d’[unités de requête (RU/s)](request-units.md). Le tableau suivant liste les limites de stockage et de débit par conteneur/base de données.

| Ressource | Limite par défaut |
| --- | --- |
| Nombre maximal d’unités de requête par conteneur ([mode provisionné avec débit dédié](account-databases-containers-items.md#azure-cosmos-containers)) | 1 000 000 par défaut. Vous pouvez l’augmenter en [soumettant un ticket de support Azure](create-support-request-quota-increase.md). |
| Nombre maximal d’unités de requête par base de données ([mode provisionné avec débit partagé](account-databases-containers-items.md#azure-cosmos-containers)) | 1 000 000 par défaut. Vous pouvez l’augmenter en [soumettant un ticket de support Azure](create-support-request-quota-increase.md). |
| Nombre maximal d’unités de requête par partition (logique) | 10 000 |
| Volume de stockage maximal sur tous les éléments par partition (logique) | 20 Go |
| Nombre maximal de clés de partition (logiques) distinctes | Illimité |
| Volume de stockage maximal par conteneur | Illimité |
| Volume de stockage maximal par base de données | Illimité |
| Taille maximale des pièces jointes par compte (la fonctionnalité de pièce jointe est déconseillée) | 2 Go |
| Unités de requête minimales/s requises par 1 Go | 10 RU/s<br>**Remarque :** si votre conteneur ou votre base de données contient plus de 1 To de données, votre compte peut être éligible à notre programme [« Stockage étendu/débit faible »](set-throughput.md#high-storage-low-throughput-program) |

> [!NOTE]
> Pour découvrir les meilleures pratiques en matière de gestion des charges de travail qui ont des clés de partition nécessitant des limites plus élevées de stockage ou de débit, voir [Créer une clé de partition synthétique](synthetic-partition-keys.md).

### <a name="minimum-throughput-limits"></a>Limites de débit minimal

Un conteneur Cosmos (ou une base de données à débit partagé) doit avoir un débit minimum de 400 RU/s. Au fur et à mesure que le conteneur augmente, Cosmos DB requiert un débit minimal pour s’assurer que la base de données ou le conteneur dispose de suffisamment de ressources pour ses opérations.

Les débits actuel et minimal d’un conteneur ou d’une base de données peuvent être récupérés à partir du portail Azure ou des SDK. Pour plus d’informations, consultez [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md). 

La valeur RU/s minimale réelle peut varier en fonction de la configuration de votre compte. Vous pouvez utiliser les [métriques Azure Monitor](monitor-cosmos-db.md#view-operation-level-metrics-for-azure-cosmos-db) pour voir l’historique du débit provisionné (RU/s) et du stockage sur une ressource. 

#### <a name="minimum-throughput-on-container"></a>Débit minimal sur un conteneur 

Pour estimer le débit minimal requis d’un conteneur avec un débit manuel, recherchez la valeur maximale :

* 400 RU/s 
* Stockage actuel en Go * 10 RU/s
* Valeur RU/s la plus élevée approvisionnée sur le conteneur / 100

Exemple : Supposons que vous disposiez d’un conteneur approvisionné avec 400 RU/s et un stockage de 0 Go. Vous augmentez le débit à 50 000 RU/s et importez 20 Go de données. La valeur RU/s minimale est maintenant `MAX(400, 20 * 10 RU/s per GB, 50,000 RU/s / 100)` = 500 RU/s. Au fil du temps, le stockage atteint 200 Go. La valeur RU/s minimale est maintenant `MAX(400, 200 * 10 RU/s per GB, 50,000 / 100)` = 2 000 RU/s. 

**Remarque :** Si votre conteneur ou votre base de données contient plus de 1 To de données, votre compte peut être éligible à notre [programme « Stockage étendu/débit faible »](set-throughput.md#high-storage-low-throughput-program).

#### <a name="minimum-throughput-on-shared-throughput-database"></a>Débit minimal sur une base de données à débit partagé 
Pour estimer le débit minimal requis d’une base de données à débit partagé avec un débit manuel, recherchez la valeur maximale :

* 400 RU/s 
* Stockage actuel en Go * 10 RU/s
* Valeur RU/s la plus élevée approvisionnée sur la base de données / 100
* 400 + MAX (nombre de conteneurs - 25, 0) * 100 RU/s

Exemple : Supposons que vous disposiez d’une base de données configurée avec 400 RU/s, 15 Go de stockage et 10 conteneurs. La valeur RU/s minimale est `MAX(400, 15 * 10 RU/s per GB, 400 / 100, 400 + 0 )` = 400 RU/s. En présence de 30 conteneurs dans la base de données, la valeur RU/s minimale serait `400 + MAX(30 - 25, 0) * 100 RU/s` = 900 RU/s. 

**Remarque :** Si votre conteneur ou votre base de données contient plus de 1 To de données, votre compte peut être éligible à notre [programme « Stockage étendu/débit faible »](set-throughput.md#high-storage-low-throughput-program).

En résumé, voici les limites minimales de RU provisionnées. 

| Ressource | Limite par défaut |
| --- | --- |
| Nombre minimal de RU/spar conteneur ([mode approvisionné avec débit dédié](./account-databases-containers-items.md#azure-cosmos-containers)) | 400 |
| Valeur RU/s minimale par base de données ([mode approvisionné avec débit partagé](./account-databases-containers-items.md#azure-cosmos-containers)) | 400 RU/s pour les 25 premiers conteneurs. 100 RU/s supplémentaires pour chaque conteneur ensuite. |

Cosmos DB prend en charge la mise à l’échelle par programmation du débit (RU/s) par conteneur ou base de données via les kits de développement logiciel (SDK) ou le portail.    

Selon les paramètres de valeur RU/s approvisionnée et de ressource, chaque ressource peut être mise à l’échelle de façon synchrone et immédiate entre la valeur RU/s minimale et jusqu’à 100x la valeur RU/s minimale. Si la valeur de débit demandée est en dehors de la plage, la mise à l’échelle est exécutée de façon asynchrone. La mise à l’échelle asynchrone peut prendre quelques minutes à quelques heures selon le débit demandé et la taille de stockage de données dans le conteneur.  

### <a name="serverless"></a>Sans serveur

[Serverless](serverless.md) vous permet d’utiliser vos ressources Azure Cosmos DB en fonction de la consommation. Le tableau suivant liste les limites de 95e centile du stockage et du débit par conteneur/base de données.

| Ressource | Limite |
| --- | --- |
| Nombre maximal d’unités de requête par partition (logique) | 5 000 |
| Volume de stockage maximal sur tous les éléments par partition (logique) | 20 Go |
| Nombre maximal de clés de partition (logiques) distinctes | Illimité |
| Volume de stockage maximal par conteneur | 50 Go |

## <a name="control-plane-operations"></a>Opérations du plan de contrôle

Vous pouvez [provisionner et gérer votre compte Azure Cosmos](how-to-manage-database-account.md) avec le portail Azure, Azure PowerShell, Azure CLI et des modèles Azure Resource Manager. Le tableau suivant liste les limites par abonnement, compte et nombre d’opérations.

| Ressource | Limite par défaut |
| --- | --- |
| Nombre maximal de comptes de base de données par abonnement | 50 par défaut. Vous pouvez l’augmenter en [soumettant un ticket de support Azure](create-support-request-quota-increase.md).|
| Nombre maximal de basculements régionaux | 1/heure par défaut. Vous pouvez l’augmenter en [soumettant un ticket de support Azure](create-support-request-quota-increase.md).|

> [!NOTE]
> Les basculements régionaux s’appliquent uniquement à des comptes d’écriture monorégions. Les comptes d’écriture multirégions ne requièrent pas ou n’ont pas de limite quant au changement de région d’écriture.

Cosmos DB sauvegarde automatiquement vos données à intervalles réguliers. Pour plus d’informations sur les fenêtres et intervalles de conservation des sauvegardes, consultez [Sauvegarde en ligne et restauration de données à la demande dans Azure Cosmos DB](online-backup-and-restore.md).

## <a name="per-account-limits"></a>Limites par compte

### <a name="provisioned-throughput"></a>Débit approvisionné

| Ressource | Limite par défaut |
| --- | --- |
| Nombre maximal de bases de données | Illimité |
| Nombre maximal de conteneurs par base de données avec un débit partagé |25 |
| Nombre maximal de conteneurs par base de données ou compte avec un débit dédié  |illimitée |
| Nombre maximal de régions | Aucune limite (toutes les régions Azure) |

### <a name="serverless"></a>Sans serveur

| Ressource | Limite |
| --- | --- |
| Nombre maximal de bases de données | Illimité |
| Nombre maximal de conteneurs par compte  | 100 |
| Nombre maximal de régions | 1 (n’importe quelle région Azure) |

## <a name="per-container-limits"></a>Limites par conteneur

En fonction de l’API que vous utilisez, un conteneur Azure Cosmos peut représenter une collection, une table ou un graphe. Les conteneurs prennent en charge la configuration de [contraintes de clé unique](unique-keys.md), de [procédures stockées, déclencheurs et fonctions définies par l’utilisateur](stored-procedures-triggers-udfs.md) et de [stratégies d’indexation](how-to-manage-indexing-policy.md). Le tableau suivant liste les limites propres aux configurations dans un conteneur. 

| Ressource | Limite par défaut |
| --- | --- |
| Longueur maximale du nom de la base de données ou du conteneur | 255 |
| Nombre maximal de procédures stockées par conteneur | 100 <sup>*</sup>|
| Nombre maximal de fonctions définies par l’utilisateur par conteneur | 25 <sup>*</sup>|
| Nombre maximal de chemins dans la stratégie d’indexation| 100 <sup>*</sup>|
| Nombre maximal de clés uniques par conteneur|10 <sup>*</sup>|
| Nombre maximal de chemins par contrainte de clé unique|16 <sup>*</sup>|
| Valeur TTL maximale |2147483647|

<sup>*</sup> Vous pouvez augmenter ces limites par conteneur en créant une [demande de support Azure](create-support-request-quota-increase.md).

## <a name="per-item-limits"></a>Limites par élément

En fonction de l’API que vous utilisez, un élément Azure Cosmos peut représenter un document d’une collection, une ligne de table, un nœud de graphe ou une arête de graphe. Le tableau suivant indique les limites par élément dans Cosmos DB. 

| Ressource | Limite par défaut |
| --- | --- |
| Taille maximale d’un élément | 2 Mo (longueur UTF-8 de la représentation JSON) |
| Longueur maximale de la valeur de clé de partition | 2 048 octets |
| Longueur maximale de la valeur d’ID | 1 023 octets |
| Nombre maximal de propriétés par élément | Aucune limite pratique |
| Longueur maximale du nom d’une propriété | Aucune limite pratique |
| Longueur maximale de la valeur d’une propriété | Aucune limite pratique |
| Longueur maximale d’une valeur de propriété de chaîne | Aucune limite pratique |
| Longueur maximale d’une valeur de propriété numérique | 64 bits double précision (norme IEEE754) |
| Niveau maximal d’imbrication des objets ou tableaux incorporés | 128 |
| Valeur TTL maximale |2147483647|

Aucune restriction ne pèse sur les charges utiles d’élément comme le nombre de propriétés et la profondeur d’imbrication, sauf en ce qui concerne la longueur des valeurs d’ID et de clé de partition, et la taille globale est limitée à 2 Mo. Vous devrez peut-être configurer une stratégie d’indexation pour les conteneurs qui ont des structures comportant des éléments grands ou complexes pour réduire la consommation de RU. Consultez [Modélisation des éléments dans Cosmos DB](how-to-model-partition-example.md) pour découvrir un exemple concret et des modèles permettant de gérer de grands éléments.

## <a name="per-request-limits"></a>Limites par demande

Azure Cosmos DB prend en charge les [opérations CRUD et de requête](/rest/api/cosmos-db/) sur des ressources telles que des conteneurs, des éléments et des bases de données. Il prend également en charge les [requêtes de lots transactionnels](/dotnet/api/microsoft.azure.cosmos.transactionalbatch) sur plusieurs éléments ayant la même clé de partition dans un conteneur.

| Ressource | Limite par défaut |
| --- | --- |
| Durée d’exécution maximale pour une seule opération (par exemple, l’exécution d’une procédure stockée ou la récupération d’une page de requête unique)| 5 secondes |
| Taille maximale de la demande (par exemple : procédure stockée, CRUD)| 2 Mo |
| Taille maximale de la réponse (par exemple, requête paginée) | 4 Mo |
| Nombre maximal d’opérations dans un lot transactionnel | 100 |

Dès qu’une opération telle qu’une requête atteint le délai d’expiration de l’exécution ou la taille limite de la réponse, elle retourne une page de résultats et un jeton de continuation au client pour reprendre l’exécution. Il n’existe aucune limite pratique quant à la durée de l’exécution d’une requête unique entre les continuations et les pages.

Cosmos DB utilise HMAC pour l’autorisation. Vous pouvez utiliser une clé primaire ou un [jeton de ressource](secure-access-to-data.md) pour un contrôle précis de l’accès aux ressources telles que les conteneurs, les clés de partition ou les éléments. Le tableau suivant liste les limites pour les jetons d’autorisation dans Cosmos DB.

| Ressource | Limite par défaut |
| --- | --- |
| Délai d’expiration maximal d’un jeton principal | 15 min  |
| Délai d’expiration minimal d’un jeton de ressource | 10 min  |
| Délai d’expiration maximal d’un jeton de ressource | 24 h par défaut. Vous pouvez l’augmenter en [soumettant un ticket de support Azure](create-support-request-quota-increase.md).|
| Variation d’horloge maximale pour l’autorisation de jeton| 15 min |

Cosmos DB prend en charge l’exécution de déclencheurs pendant les écritures. Le service prend en charge un maximum d’un pré-déclencheur et d’un post-déclencheur par opération d’écriture.

## <a name="metadata-request-limits"></a>Limites des requêtes de métadonnées

Azure Cosmos DB gère les métadonnées système pour chaque compte. Ces métadonnées vous permettent d’énumérer les collections, les bases de données, d’autres ressources Azure Cosmos DB et leurs configurations gratuitement.

| Ressource | Limite par défaut |
| --- | --- |
|Taux maximal de création de collections par minute|    100|
|Taux maximal de création de bases de données par minute|    100|
|Débit de mise à jour approvisionné maximal par minute|    5|

## <a name="limits-for-autoscale-provisioned-throughput"></a>Limites du débit approvisionné en mode de mise à l’échelle automatique

Pour plus de détails sur le débit et les limites de stockage avec mise à l’échelle automatique, consultez l’article [Mise à l’échelle automatique](provision-throughput-autoscale.md#autoscale-limits) et le [FAQ](autoscale-faq.md#lowering-the-max-rus).

| Ressource | Limite par défaut |
| --- | --- |
| Nombre maximal de RU/s auquel le système peut s’adapter |  `Tmax`, nombre maximal de RU/s de mise à l’échelle automatique défini par l’utilisateur|
| Nombre minimal de RU/s auxquelles le système peut adapter son échelle | `0.1 * Tmax`|
| Nombre actuel de RU/s auxquelles l’échelle du système est adaptée  |  `0.1*Tmax <= T <= Tmax`, selon l’utilisation|
| Nombre minimal de RU/s facturables par heure| `0.1 * Tmax` <br></br>La facturation est effectuée sur une base horaire, où vous êtes facturé pour le nombre maximal de RU/s auxquelles l’échelle du système est adaptée dans l’heure, ou bien `0.1*Tmax`, la valeur plus élevée étant retenue. |
| Nombre maximal de RU/s de mise à l’échelle automatique minimale pour un conteneur  |  `MAX(4000, highest max RU/s ever provisioned / 10, current storage in GB * 100)` arrondi au millier de RU/s le plus proche |
| Nombre maximal de RU/s de mise à l’échelle automatique minimale pour une base de données  |  `MAX(4000, highest max RU/s ever provisioned / 10, current storage in GB * 100,  4000 + (MAX(Container count - 25, 0) * 1000))` arrondi au millier de RU/s le plus proche <br></br>Notez que, si votre base de données comporte plus de 25 conteneurs, le système incrémente le nombre minimal de RU/S de mise à l’échelle automatique de 1000 RU/s par conteneur supplémentaire. Par exemple, si vous avez 30 conteneurs, le nombre maximal de RU/s de mise à l’échelle automatique la plus basse que vous pouvez définir est de 9000 RU/s (échelles comprises entre 900 et 9000 RU/s).

## <a name="sql-query-limits"></a>Limites de requête SQL

Cosmos DB prend en charge l’interrogation d’éléments à l’aide de [SQL](./sql-query-getting-started.md). Le tableau suivant décrit les restrictions dans les instructions de requête, par exemple en termes de nombre de clauses ou de longueur de requête.

| Ressource | Limite par défaut |
| --- | --- |
| Longueur maximale d’une requête SQL| 256 KB |
| Nombre maximal de jointures par requête| 5 <sup>*</sup>|
| Nombre maximal de fonctions définies par l’utilisateur par requête| 10 <sup>*</sup>|
| Nombre maximal de points par polygone| 4096 |
| Nombre maximal de chemins d'accès inclus par conteneur| 500 |
| Nombre maximal de chemins d'accès exclus par conteneur| 500 |
| Nombre maximal de propriétés dans un index composite| 8 |

<sup>*</sup> Vous pouvez augmenter le nombre maximal de requêtes SQL en créant une [demande de support Azure](create-support-request-quota-increase.md).

## <a name="mongodb-api-specific-limits"></a>Limites propres à l’API MongoDB

Cosmos DB prend en charge le protocole de transmission MongoDB pour les applications écrites par rapport à MongoDB. Vous trouverez les versions de protocole et les commandes prises en charge dans l’article [Fonctionnalités et syntaxe MongoDB prises en charge](mongodb-feature-support.md).

Le tableau suivant liste les limites propres à la prise en charge des fonctionnalités MongoDB. D’autres limites de service mentionnées pour l’API SQL (Core) s’appliquent également à l’API MongoDB.

| Ressource | Limite par défaut |
| --- | --- |
| Taille de mémoire maximale pour les requêtes MongoDB (cette limitation s'applique uniquement à la version 3.2 du serveur) | 40 Mo |
| Durée d’exécution maximale des opérations MongoDB| 30 s |
| Délai d’expiration de connexion inactive pour la fermeture de connexion côté serveur* | 30 minutes |

\* Nous recommandons que les applications clientes définissent le délai d’expiration de connexion inactive dans les paramètres du pilote sur 2-3 minutes, car le [délai d’expiration par défaut pour Azure LoadBalancer est de 4 minutes](../load-balancer/load-balancer-tcp-idle-timeout.md).  Ce délai d’expiration garantit que les connexions inactives ne sont pas fermées par un équilibreur de charge intermédiaire entre la machine cliente et Azure Cosmos DB.

## <a name="try-cosmos-db-free-limits"></a>Limites liées à l’essai gratuit de Cosmos DB

Le tableau suivant liste les limites liées à l’[essai gratuit d’Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/).

| Ressource | Limite par défaut |
| --- | --- |
| Durée de l’essai | 30 jours (une nouvelle évaluation peut être demandée après expiration) <br> Après expiration, les informations stockées sont supprimées. |
| Nombre maximal de conteneurs par abonnement (SQL, API Table, Gremlin) | 1 |
| Nombre maximal de conteneurs par abonnement (API MongoDB) | 3 |
| Débit maximal par conteneur | 5 000 |
| Débit maximal par base de données de débit partagé | 20000 |
| Volume total de stockage maximal par compte | 10 Go |

L’essai de Cosmos DB ne prend en charge la distribution mondiale que dans les régions USA Centre, Europe Nord et Asie Sud-Est. Vous ne pouvez pas créer de tickets de support Azure pour les comptes d’essai d’Azure Cosmos DB. Toutefois, un support est fourni pour les abonnés détenant des plans de support.

## <a name="free-tier-account-limits"></a>Limites du compte de niveau gratuit
Le tableau suivant liste les limites des [comptes du niveau gratuit d’Azure Cosmos DB](optimize-dev-test.md#azure-cosmos-db-free-tier).

| Ressource | Limite par défaut |
| --- | --- |
| Nombre de comptes du niveau gratuit par abonnement Azure | 1 |
| Durée de la remise du niveau gratuit | Durée de vie du compte. Acceptation requise au cours de la création du compte. |
| Nombre maximal de RU/s gratuites | 400 RU/s |
| Stockage maximal gratuit | 5 Go |
| Nombre maximal de bases de données à débit partagé | 5 |
| Nombre maximal de conteneurs dans une base de données à débit partagé | 25 <br>Dans les comptes du niveau gratuit, le nombre minimal de RU/s pour une base de données à débit partagé comptant jusqu’à 25 conteneurs est de 400 RU/s. |

  En plus de ce qui précède, les [Limites par compte](#per-account-limits) s’appliquent également aux comptes du niveau gratuit.

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus en détail les concepts de base de Cosmos DB de [distribution mondiale](distribute-data-globally.md), de [partitionnement](partitioning-overview.md) et de [débit provisionné](request-units.md).

Bien démarrer avec Azure Cosmos DB grâce à l’un de nos guides de démarrage rapide :

* [Prise en main de l’API SQL Azure Cosmos DB](create-sql-api-dotnet.md)
* [Bien démarrer avec l’API Azure Cosmos DB pour MongoDB](create-mongodb-nodejs.md)
* [Prise en main de l’API Cassandra Azure Cosmos DB](create-cassandra-dotnet.md)
* [Prise en main de l’API Gremlin Azure Cosmos DB](create-graph-dotnet.md)
* [Prise en main de l’API Table Azure Cosmos DB](create-table-dotnet.md)

> [!div class="nextstepaction"]
> [Essayez gratuitement Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)
