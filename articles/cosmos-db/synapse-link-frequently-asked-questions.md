---
title: Foire aux questions sur Azure Synapse Link pour Azure Cosmos DB
description: Recevez des réponses aux questions fréquemment posées sur Synapse Link pour Azure Cosmos DB dans des domaines comme la facturation, le magasin analytique, la sécurité et la durée de vie dans le magasin analytique.
author: Rodrigossz
ms.author: rosouz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/30/2020
ms.openlocfilehash: 82133f990c1714276aa13ff22c3f19d0993d16df
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488712"
---
# <a name="frequently-asked-questions-about-azure-synapse-link-for-azure-cosmos-db"></a>Foire aux questions sur Azure Synapse Link pour Azure Cosmos DB
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

Azure Synapse Link pour Azure Cosmos DB crée une intégration entre Azure Cosmos DB et Azure Synapse Analytics. Il permet aux clients d’exécuter des analyses quasiment en temps réel sur leurs données opérationnelles avec une isolation complète des performances de leurs charges de travail transactionnelles et sans pipeline ETL. Cet article répond aux questions fréquemment posées sur Azure Synapse Link pour Azure Cosmos DB.

## <a name="general-faq"></a>FAQ général

### <a name="is-azure-synapse-link-supported-for-all-azure-cosmos-db-apis"></a>Azure Synapse Link est-il pris en charge pour toutes les API Azure Cosmos DB ?

Azure Synapse Link est pris en charge pour l’API Azure Cosmos DB SQL (Core) et pour l’API Azure Cosmos DB pour MongoDB. 

### <a name="is-azure-synapse-link-supported-for-multi-region-azure-cosmos-db-accounts"></a>Azure Synapse Link est-il pris en charge pour les comptes Azure Cosmos DB à plusieurs régions ?

Oui, pour les comptes Azure Cosmos à plusieurs régions, les données stockées dans le magasin analytique sont également distribuées à l’échelle mondiale. Indépendamment d’une région d’écriture unique ou de plusieurs régions d’écriture, les requêtes analytiques effectuées à partir d’Azure Synapse Analytics peuvent être fournies à partir de la région la plus proche.

Lors de la planification de la configuration d’un compte Azure Cosmos DB à plusieurs régions avec prise en charge du magasin analytique, il est recommandé d’ajouter toutes les régions nécessaires au moment de la création du compte.

### <a name="can-i-choose-to-enable-azure-synapse-link-for-only-certain-region-and-not-all-regions-in-a-multi-region-account-set-up"></a>Puis-je choisir d’activer AzureSynapse Link pour une seule région et non dans toutes les régions d’un compte à plusieurs régions ?

Lorsqu’Azure Synapse Link est activé pour un compte à plusieurs régions, le magasin analytique est créé dans toutes les régions. Les données sous-jacentes sont optimisées pour le débit et la cohérence transactionnelle dans le magasin transactionnel.

### <a name="is-backup-and-restore-supported-for-azure-synapse-link-enabled-accounts"></a>La sauvegarde et la restauration sont-elles prises en charge par les comptes Azure Synapse Link ?

Pour les conteneurs avec le magasin analytique activé, la sauvegarde et la restauration automatiques de vos données dans le magasin analytique ne sont pas prises en charge pour l’instant. 

Lorsque Synapse Link est activé sur un compte de base de données, Azure Cosmos DB continue automatiquement [d’effectuer des sauvegardes](./online-backup-and-restore.md) de vos données dans le magasin transactionnel (uniquement) des conteneurs selon l’intervalle de sauvegarde planifié, comme toujours. Il est important de noter que lorsqu’un conteneur avec le magasin analytique activé est restauré vers un nouveau compte, le conteneur est restauré avec uniquement le magasin transactionnel et aucun magasin analytique activé. 

### <a name="can-i-disable-the-azure-synapse-link-feature-for-my-azure-cosmos-db-account"></a>Puis-je désactiver la fonctionnalité Azure Synapse Link de mon compte Azure Cosmos DB ?

Actuellement, une fois que la fonction Synapse Link est activée au niveau du compte, vous ne pouvez pas la désactiver. Sachez qu’il n’y aura aucune incidence sur la facturation si Synapse Link est activé au niveau du compte et qu’aucun conteneur n’est activé pour le magasin analytique.

Si vous devez désactiver la fonctionnalité, vous avez 2 possibilités. La première consiste à supprimer et à recréer un nouveau compte Azure Cosmos DB, en migrant les données si nécessaire. La deuxième consiste à ouvrir un ticket de support pour obtenir de l’aide sur la migration des données vers un autre compte.

### <a name="does-analytical-store-have-any-impact-on-cosmos-db-transactional-slas"></a>Le magasin analytique a-t-il un impact sur les SLA transactionnels Cosmos DB ?

Non, il n’y a aucun impact.

## <a name="azure-cosmos-db-analytical-store"></a>Magasin analytique Azure Cosmos DB

### <a name="can-i-enable-analytical-store-on-existing-containers"></a>Puis-je activer le magasin analytique sur les conteneurs existants ?

Actuellement, le magasin analytique ne peut être activé que pour les nouveaux conteneurs (à la fois dans les comptes nouveaux et existants).

### <a name="can-i-disable-analytical-store-on-my-azure-cosmos-db-containers-after-enabling-it-during-container-creation"></a>Puis-je désactiver le magasin analytique sur mes conteneurs Azure Cosmos DB après l’avoir activé lors de la création du conteneur ?

Actuellement, le magasin analytique ne peut pas être désactivé sur un conteneur Azure Cosmos DB une fois activé lors de la création du conteneur.

### <a name="is-analytical-store-supported-for-azure-cosmos-db-containers-with-autoscale-provisioned-throughput"></a>Le magasin analytique est-il pris en charge pour les conteneurs Azure Cosmos DB avec débit approvisionné par mise à l’échelle automatique ?

Oui, le magasin analytique peut être activé sur les conteneurs avec un débit approvisionné par mise à l’échelle automatique.

### <a name="is-there-any-effect-on-azure-cosmos-db-transactional-store-provisioned-rus"></a>Existe-t-il un effet sur les RU approvisionnées pour le magasin transactionnel Azure Cosmos DB ?

Azure Cosmos DB garantit l’isolation des performances entre les charges de travail transactionnelles et analytiques. L’activation du magasin analytique sur un conteneur n’a pas d’impact sur les RU/s configurées sur le magasin transactionnel Azure Cosmos DB. Les transactions (lecture et écriture) et les coûts de stockage du magasin analytique sont facturées séparément. Pour plus d’informations, consultez la [tarification du magasin analytique Azure Cosmos DB](analytical-store-introduction.md#analytical-store-pricing).

### <a name="are-delete-and-update-operations-on-the-transactional-store-reflected-in-the-analytical-store"></a>Les opérations de suppression et de mise à jour sur le magasin transactionnel sont-elles reflétées dans le magasin analytique ?

Oui, les suppressions et les mises à jour des données dans le magasin transactionnel sont reflétées dans le magasin analytique. Vous pouvez configurer la durée de vie (TTL) sur le conteneur pour inclure les données d’historique afin que le magasin analytique conserve toutes les versions des éléments qui répondent aux critères de durée de vie analytique. Pour plus d’informations, consultez [Vue d’ensemble de la durée de vie analytique](analytical-store-introduction.md#analytical-ttl).

### <a name="can-i-connect-to-analytical-store-from-analytics-engines-other-than-azure-synapse-analytics"></a>Puis-je me connecter à un magasin analytique à partir de moteurs d’analyse autres qu’Azure Synapse Analytics ?

Vous pouvez uniquement accéder aux requêtes et les exécuter sur le magasin analytique avec les runtimes fournis par Azure Synapse Analytics. Le magasin analytique peut être interrogé et analysé à l’aide des éléments suivants :

* Synapse Spark avec prise en charge complète de Scala, Python, SparkSQL et C#. Synapse Spark est essentiel à l’ingénierie des données et aux scénarios de science des données
* Pool SQL serverless avec langage T-SQL et la prise en charge des outils décisionnels courants (par exemple, Power BI Premium, etc.)

### <a name="can-i-connect-to-analytical-store-from-synapse-sql-provisioned"></a>Puis-je me connecter au magasin analytique à partir de Synapse SQL provisionné ?

À ce stade, le magasin analytique n’est pas accessible à partir de Synapse SQL provisionné.

### <a name="can-i-write-back-the-query-aggregation-results-from-synapse-back-to-the-analytical-store"></a>Puis-je réécrire les résultats de l’agrégation des requêtes de la requête Synapse dans le magasin analytique ?

Le magasin analytique est un magasin en lecture seule dans un conteneur Azure Cosmos DB. Par conséquent, vous ne pouvez pas réécrire directement les résultats de l’agrégation dans le magasin analytique, mais vous pouvez les écrire dans le magasin transactionnel Azure Cosmos DB d’un autre conteneur, qui peut ensuite être exploité comme une couche de service.

### <a name="is-the-autosync-replication-from-transactional-store-to-the-analytical-store-asynchronous-or-synchronous-and-what-are-the-latencies"></a>La réplication de synchronisation automatique entre le magasin transactionnel et le magasin d’analyse est-elle synchrone ou asynchrone et quelles sont les délais de latence ?

La latence de synchronisation automatique est généralement en moins de 2 minutes. Dans le cas d’une base de données de débit partagé avec un grand nombre de conteneurs, la latence de la synchronisation automatique des conteneurs individuels peut être supérieure et prendre jusqu’à 5 minutes. Nous aimerions en savoir plus, pour savoir si cette latence est adaptée à vos scénarios. Pour cela, contactez l’équipe [Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

### <a name="are-there-any-scenarios-where-the-items-from-the-transactional-store-are-not-automatically-propagated-to-the-analytical-store"></a>Existe-t-il des scénarios dans lesquels les éléments du magasin transactionnel ne sont pas propagés automatiquement dans le magasin analytique ?

Si certains éléments de votre conteneur enfreignent le [schéma bien défini pour l’analyse](analytical-store-introduction.md#analytical-schema), ils ne seront pas inclus dans le magasin analytique. Si vous vous trouvez face à scénario bloqué par un schéma bien défini pour l’analyse, envoyez un message à l’[équipe Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com) pour obtenir de l’aide.

### <a name="can-i-partition-the-data-in-analytical-store-differently-from-transactional-store"></a>Puis-je partitionner les données du magasin analytique différemment de la banque transactionnelle ?

Les données du magasin analytique sont partitionnées en fonction du partitionnement horizontal des partitions dans le magasin transactionnel. Actuellement, vous ne pouvez pas choisir une autre stratégie de partitionnement pour le magasin analytique.

### <a name="can-i-customize-or-override-the-way-transactional-data-is-transformed-into-columnar-format-in-the-analytical-store"></a>Puis-je personnaliser ou substituer la façon dont les données transactionnelles sont transformées en colonnes dans le magasin analytique ?

Actuellement, vous ne pouvez pas transformer les éléments de données lorsqu’ils sont propagés automatiquement du magasin transactionnel vers le magasin analytique. Si vous vous trouvez face à des scénarios bloqués par cette limitation, envoyez un message à l’[équipe Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

### <a name="is-analytical-store-supported-by-terraform"></a>Le magasin analytique est-il pris en charge par Terraform ?

Terraform ne prend pas en charge les conteneurs de magasin analytique. Pour plus d’informations, consultez les [problèmes liés à Terraform sur GitHub](https://github.com/hashicorp/terraform/issues).

## <a name="analytical-time-to-live-ttl"></a>Durée de vie (TTL) analytique

### <a name="is-ttl-for-analytical-data-supported-at-both-container-and-item-level"></a>La durée de vie des données analytiques est-elle prise en charge au niveau conteneur et au niveau élément ?

À ce stade, la durée de vie des données analytiques ne peut être configurée qu’au niveau du conteneur et il n’y a pas de prise en charge pour définir la durée de vie analytique au niveau de l’élément.

### <a name="after-setting-the-container-level--analytical-ttl-on-an-azure-cosmos-db-container-can-i-change-to-a-different-value-later"></a>Après avoir défini la durée de vie analytique au niveau conteneur sur un conteneur Azure Cosmos DB, puis-je utiliser ultérieurement une autre valeur ?

Oui, la durée de vie analytique peut être mise à jour et appliquer n’importe quelle valeur valide. Pour plus d’informations sur la durée de vie analytique, consultez l’article relatifs à la [durée de vie analytique](analytical-store-introduction.md#analytical-ttl).

### <a name="can-i-update-or-delete-an-item-from-the-analytical-store-after-it-has-been-ttld-out-from-the-transactional-store"></a>Puis-je mettre à jour ou supprimer un élément du magasin analytique après sa durée de vie dans le magasin transactionnel ?

Toutes les mises à jour et suppressions transactionnelles sont copiées dans le magasin analytique, mais si l’élément a été purgé du magasin transactionnel, il ne peut pas être mis à jour dans le magasin analytique. Pour plus d’informations, consultez l’article relatif à la [durée de vie analytique](analytical-store-introduction.md#analytical-ttl).

## <a name="billing"></a>Facturation

### <a name="what-is-the-billing-model-of-azure-synapse-link-for-azure-cosmos-db"></a>Quel est le modèle de facturation d’Azure Synapse Link pour Azure Cosmos DB ?

Le modèle de facturation d’Azure Synapse Link comprend les coûts engendrés par l’utilisation du magasin analytique Azure Cosmos DB et du runtime Synapse. Pour plus d’informations, consultez la [tarification du magasin analytique Azure Cosmos DB](analytical-store-introduction.md#analytical-store-pricing) et les articles sur la [tarification d’Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/synapse-analytics/).

### <a name="what-is-the-billing-impact-if-i-enable-synapse-link-in-my-azure-cosmos-db-database-account"></a>Quel est l’impact sur la facturation si j’active Synapse Link dans mon compte de base de données Azure Cosmos DB ?

Aucun. Vous ne serez facturé que lorsque vous créerez un conteneur de magasin analytique activé et que vous commencerez à charger des données.


## <a name="security"></a>Sécurité

### <a name="what-are-the-ways-to-authenticate-with-the-analytical-store"></a>Quelles sont les moyens d’authentification auprès du magasin analytique ?

L’authentification auprès du magasin analytique est identique à celle du magasin transactionnel. Pour une base de données spécifique, vous pouvez vous authentifier à l’aide de la clé primaire ou de la clé en lecture seule. Vous pouvez utiliser le service lié dans Azure Synapse Studio pour empêcher le collage des clés Azure Cosmos DB dans les notebooks Spark. L’accès à ce service lié est accessible à toute personne ayant accès à l’espace de travail.

## <a name="synapse-run-times"></a>Runtimes Synapse

### <a name="what-are-the-currently-supported-synapse-run-times-to-access-azure-cosmos-db-analytical-store"></a>Quelles sont les runtimes Synapse actuellement pris en charge pour accéder au magasin analytique Azure Cosmos DB ?

|Runtime Azure Synapse |Prise en charge actuelle |
|---------|---------|
|Pools Azure Synapse Spark | Lecture, écriture (via un magasin transactionnel), table, vue temporaire |
|Pool SQL serverless Azure Synapse    | Lire, afficher |
|Azure Synapse SQL approvisionné   |  Non disponible |

### <a name="do-my-azure-synapse-spark-tables-sync-with-my-azure-synapse-serverless-sql-pool-tables-the-same-way-they-do-with-azure-data-lake"></a>Mes tables Azure Synapse Spark se synchronisent-elles avec mes tables de pool SQL serverless Azure Synapse de la même manière qu’avec Azure Data Lake ?

Cette fonctionnalité n’est pas disponible pour l’instant.

### <a name="can-i-do-spark-structured-streaming-from-analytical-store"></a>Puis-je utiliser le flux structuré Spark à partir du magasin analytique ?

Actuellement, la prise en charge du flux structuré Spark pour Azure Cosmos DB est implémentée par la fonctionnalité de flux de modification du magasin transactionnel et n’est pas encore prise en charge à partir du magasin analytique.

## <a name="azure-synapse-studio"></a>Azure Synapse Studio

### <a name="in-the-azure-synapse-studio-how-do-i-recognize-if-im-connected-to-an-azure-cosmos-db-container-with-the-analytics-store-enabled"></a>Dans Azure Synapse Studio, comment puis-je savoir si je suis connecté à un conteneur Azure Cosmos DB avec le magasin d’analytique activé ?

Les conteneurs Azure Cosmos DB activés avec le magasin analytique présentent l’icône suivante :

:::image type="content" source="./media/synapse-link-frequently-asked-questions/analytical-store-icon.png" alt-text="Conteneur Azure Cosmos DB activé avec icône de magasin analytique":::

Le conteneur de magasin transactionnel est représenté par l’icône suivante :

:::image type="content" source="./media/synapse-link-frequently-asked-questions/transactional-store-icon.png" alt-text="Conteneur Azure Cosmos DB activé avec l’icône de magasin transactionnel":::
 
### <a name="how-do-you-pass-azure-cosmos-db-credentials-from-azure-synapse-studio"></a>Comment transmettre les informations d’identification Azure Cosmos DB depuis Azure Synapse Studio ?

Actuellement, les informations d’identification Azure Cosmos DB sont transmises lors de la création du service lié par l’utilisateur qui a accès aux bases de données Azure Cosmos DB. L’accès à ce magasin est possible pour les autres utilisateurs qui ont accès à l’espace de travail.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [avantages de d’Azure Synapse Link](synapse-link.md#synapse-link-benefits)

* En savoir plus sur l’intégration d’[Azure Synapse Link et Azure Cosmos DB](synapse-link.md#synapse-link-integration).
