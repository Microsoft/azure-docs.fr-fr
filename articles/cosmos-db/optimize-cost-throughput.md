---
title: Optimisation du coût du débit dans Azure Cosmos DB
description: Cet article explique comment optimiser le coût du débit pour les données stockées dans Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 02/07/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 86de3e1199b00dff4e03f3b4292f86e6c19ea491
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296537"
---
# <a name="optimize-provisioned-throughput-cost-in-azure-cosmos-db"></a>Optimiser le coût du débit approvisionné dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

En proposant un modèle avec débit approvisionné, Azure Cosmos DB offre des performances prévisibles à n’importe quelle échelle. La réservation ou l’approvisionnement du débit en amont élimine l’effet « voisin bruyant » pouvant affecter les performances. Vous spécifiez le débit exact dont vous avez besoin, et Azure Cosmos DB garantit par contrat de niveau de service le débit défini.

Vous pouvez commencer avec un débit minimum de 400 unité de requêtes (RU) par seconde, puis mettre à l’échelle jusqu'à des dizaines de millions de demandes par seconde voire plus. Chaque demande que vous exécutez sur votre conteneur ou base de données Azure Cosmos, par exemple une demande de lecture, une demande d’écriture, une demande de requête ou des procédures stockées, entraîne un coût correspondant qui est déduit de votre débit approvisionné. Si vous approvisionnez 400 RU/s et que vous émettez une requête qui coûte 40 RU, vous pourrez émettre 10 requêtes de ce type par seconde. Toute requête en dehors de ce cadre sera limitée et vous devez relancer la requête. Si vous utilisez des pilotes clients, ces pilotes prennent en charge la logique de nouvelle tentative automatique.

Vous pouvez configurer le débit sur des bases de données ou des conteneurs, et chaque stratégie peut vous aider à réduire les coûts en fonction du scénario.

## <a name="optimize-by-provisioning-throughput-at-different-levels"></a>Optimiser en configurant le débit à différents niveaux

* Si vous approvisionnez le débit sur une base de données, tous les conteneurs, par exemple les collections/tables/graphiques au sein de cette base de données, peuvent se partager le débit en fonction de la charge. Le débit réservé au niveau de la base de données est partagé inégalement sur un ensemble spécifique de conteneurs, selon la charge de travail.

* Si vous configurez le débit sur un conteneur, ce débit est garanti par contrat de niveau de service pour ce conteneur. Le choix d’une clé de partition logique est essentiel pour la répartition de la charge sur toutes les partitions logiques d’un conteneur. Consultez les articles [Partitionnement](partitioning-overview.md) et [Mise à l’échelle horizontale](partitioning-overview.md) pour plus d’informations.

Voici quelques indications pour choisir une stratégie de débit approvisionné :

**Provisionnez le débit sur une base de données Azure Cosmos (contenant un ensemble de conteneurs) si** :

1. Vous avez quelques dizaines de conteneurs Azure Cosmos et souhaitez partager le débit sur tout ou partie de ces conteneurs. 

2. Vous effectuez une migration à partir d’une base de données à locataire unique conçue pour s’exécuter sur des machines virtuelles hébergées sur IaaS ou localement, par exemple des bases de données NoSQL ou relationnelles, vers Azure Cosmos DB. Vous avez un grand nombre de collections/tables/graphiques et ne souhaitez pas modifier votre modèle de données. Notez que vous devrez accepter certains compromis concernant les avantages offerts par Azure Cosmos DB si vous ne mettez pas à jour votre modèle de données lors de la migration à partir d’une base de données locale. Il est recommandé d’accéder régulièrement à votre modèle de données pour optimiser les performances et les coûts. 

3. Vous souhaitez absorber les pics imprévus dans les charges de travail en regroupant le débit au niveau de la base de données qui subit un pic inattendu dans la charge de travail. 

4. Au lieu de définir un débit spécifique dans des conteneurs individuels, vous préférez répartir le débit d’agrégat sur un ensemble de conteneurs au sein de la base de données.

**Approvisionnez le débit sur un conteneur individuel si :**

1. Vous avez quelques conteneurs Cosmos Azure. Comme Azure Cosmos DB ne dépend pas d’un schéma spécifique, un conteneur peut contenir des éléments avec des schémas hétérogènes, sans forcer les clients à créer plusieurs types de conteneurs (un pour chaque entité). Choisissez cette option si vous estimez que le regroupement de 10 à 20 conteneurs dans un seul conteneur peut être bénéfique. Avec un minimum de 400 unités de requête pour les conteneurs, le regroupement de 10 à 20 conteneurs dans un seul conteneur peut être plus économique. 

2. Vous souhaitez contrôler le débit sur un conteneur spécifique et obtenir le débit garanti sur un conteneur donné avec une garantie par contrat de niveau de service.

**Optez pour une solution hybride combinant les deux stratégies ci-dessus :**

1. Comme mentionné précédemment, Azure Cosmos DB vous permet de combiner les deux stratégies ci-dessus : vous pouvez désormais avoir certains conteneurs au sein de la base de données Azure Cosmos, pouvant se partager le débit approvisionné sur la base de données, et d’autres conteneurs au sein de la même base de données, chacun avec un débit approvisionné dédié. 

2. Vous pouvez appliquer les stratégies ci-dessus afin d’obtenir une configuration hybride dans laquelle les deux niveaux de la base de données bénéficient d’un débit et certains conteneurs reçoivent un débit dédié.

Comme indiqué dans le tableau suivant, selon le choix de l’API, vous pouvez approvisionner un débit à différents niveaux de granularité.

|API|Pour un débit **partagé**, configurer |Pour un débit **dédié**, configurer |
|----|----|----|
|API SQL|Base de données|Conteneur|
|API d’Azure Cosmos DB pour MongoDB|Base de données|Collection|
|API Cassandra|Espace de clés|Table de charge de travail|
|API Gremlin|Compte de base de données|Graph|
|API de table|Compte de base de données|Table de charge de travail|

En approvisionnant le débit à différents niveaux, vous pouvez optimiser vos coûts selon les caractéristiques de votre charge de travail. Comme mentionné précédemment, vous pouvez par programmation et à tout moment réduire ou augmenter votre débit approvisionné pour des conteneurs individuels ou collectivement pour un ensemble de conteneurs. Grâce à cette mise à l'échelle flexible qui s’adapte à votre charge de travail, vous payez uniquement pour le débit que vous avez configuré. Si votre conteneur ou un ensemble de conteneurs est réparti dans plusieurs régions, la disponibilité du débit que vous configurez sur le conteneur ou l’ensemble de conteneurs est garantie dans toutes les régions.

## <a name="optimize-with-rate-limiting-your-requests"></a>Optimiser en limitant le débit de vos requêtes

Pour les charges de travail qui ne sont pas affectées par la latence, vous pouvez approvisionner un débit inférieur et permettre ainsi à l’application de gérer la limitation du débit lorsque le débit réel dépasse le débit approvisionné. Le serveur met fin à la requête de manière préventive avec `RequestRateTooLarge` (code d’état HTTP 429) et il retourne l’en-tête `x-ms-retry-after-ms` indiquant la durée, en millisecondes, pendant laquelle l’utilisateur doit attendre avant de réessayer. 

```html
HTTP Status 429, 
 Status Line: RequestRateTooLarge 
 x-ms-retry-after-ms :100
```

### <a name="retry-logic-in-sdks"></a>Logique de nouvelle tentative dans les kits de développement logiciel (SDK) 

Les kits de développement logiciel (SDK) natifs (.NET/.NET Core, Java, Node.js et Python) interceptent tous implicitement cette réponse, respectent l’en-tête retry-after spécifiée par le serveur, puis relancent la requête. La tentative suivante réussira toujours, sauf si plusieurs clients accèdent simultanément à votre compte.

Si plusieurs de vos clients opèrent simultanément et systématiquement au-delà du taux de requête, le nombre de nouvelles tentatives par défaut de 9 ne suffira peut-être pas. Dans ce cas, le client envoie à l’application une exception `RequestRateTooLargeException` avec le code d’état 429. Le nombre de nouvelles tentatives par défaut peut être modifié en définissant les `RetryOptions` sur l’instance ConnectionPolicy. Par défaut, l’`RequestRateTooLargeException` avec le code d’état 429 est retournée après un temps d’attente cumulé de 30 secondes si la requête continue à fonctionner au-dessus du taux de requête. Cela se produit même lorsque le nombre de nouvelles tentatives actuel est inférieur au nombre maximal de nouvelles tentatives, qu’il s’agisse de la valeur par défaut de 9 ou d’une valeur définie par l’utilisateur. 

[MaxRetryAttemptsOnThrottledRequests](/dotnet/api/microsoft.azure.documents.client.retryoptions.maxretryattemptsonthrottledrequests?preserve-view=true&view=azure-dotnet) a la valeur 3 ; dans ce cas, si une opération de requête est soumise à une restriction de taux car elle dépasse le débit réservé pour le conteneur, l’opération de requête réessaie trois fois avant d’envoyer l’exception à l’application. [MaxRetryWaitTimeInSeconds](/dotnet/api/microsoft.azure.documents.client.retryoptions.maxretrywaittimeinseconds?preserve-view=true&view=azure-dotnet#Microsoft_Azure_Documents_Client_RetryOptions_MaxRetryWaitTimeInSeconds) a la valeur 60 ; dans ce cas, si le délai d’attente de la nouvelle tentative cumulative depuis la première requête dépasse 60 secondes, l’exception est levée.

```csharp
ConnectionPolicy connectionPolicy = new ConnectionPolicy(); 
connectionPolicy.RetryOptions.MaxRetryAttemptsOnThrottledRequests = 3; 
connectionPolicy.RetryOptions.MaxRetryWaitTimeInSeconds = 60;
```

## <a name="partitioning-strategy-and-provisioned-throughput-costs"></a>Stratégie de partitionnement et coût du débit approvisionné

Une bonne stratégie de partitionnement est importante pour optimiser les coûts dans Azure Cosmos DB. Vérifiez dans les métriques de stockage qu’il n’y a pas de partitions asymétriques. Vérifiez dans les métriques de débit qu’il n’y a pas de débit asymétrique. Vérifiez qu’il n’y a pas d’asymétrie dans les clés de partition. Les clés dominantes du stockage sont représentées sous forme de métriques, mais la clé dépendra de votre modèle d’accès aux applications. Il est important de bien choisir la clé de partition logique appropriée. Une bonne clé de partition doit avoir les caractéristiques suivantes :

* Choisissez une clé de partition qui répartit la charge de travail de manière uniforme sur toutes les partitions et dans le temps. En d’autres termes, évitez une situation où certaines clés gèrent la majorité des données tandis que d’autres clés ont peu voire pas de données du tout. 

* Choisissez une clé de partition offrant des modèles d'accès répartis uniformément sur les partitions logiques. La charge de travail est raisonnablement répartie entre toutes les clés. En d’autres termes, la majeure partie de la charge de travail ne doit pas se concentrer sur quelques clés spécifiques. 

* Choisissez une clé de partition avec un large éventail de valeurs. 

L'idée de base est de répartir les données et l'activité de votre conteneur sur l'ensemble des partitions logiques afin que les ressources de débit et de stockage des données puissent être réparties sur les partitions logiques. Les candidats aux clés de partition peuvent inclure les propriétés qui apparaissent fréquemment en tant que filtre dans vos requêtes. Les requêtes peuvent être efficacement acheminées en incluant la clé de partition dans le prédicat de filtre. Cette stratégie de partitionnement facilite considérablement l’optimisation du débit approvisionné. 

### <a name="design-smaller-items-for-higher-throughput"></a>Conception d’éléments plus petits pour un débit plus élevé 

Les frais de requête ou le coût de traitement de requête d’une opération donnée sont directement liés à la taille de l’élément. Les opérations effectuées sur des éléments volumineux coûteront plus cher que sur des éléments plus petits. 

## <a name="data-access-patterns"></a>Modèles d’accès aux données 

Il est judicieux de séparer logiquement vos données en catégories logiques selon la fréquence à laquelle vous accédez aux données. En classant ces données par ordre d’importance, vous pouvez ajuster le stockage consommé et le débit requis. Selon la fréquence d’accès, vous pouvez placer les données dans des conteneurs distincts (par exemple, des tables, des graphiques et des collections) et ajuster le débit approvisionné sur ces éléments en fonction des besoins de ce segment de données. 

En outre, si vous utilisez Azure Cosmos DB et savez que vous n’effectuerez pas de recherches selon certaines valeurs de données ou que vous y accéderez rarement, vous devez stocker les valeurs compressées de ces attributs. Cette méthode vous permet de réaliser des économises au niveau de l’espace de stockage, de l’espace d’index et du débit approvisionné, entraînant ainsi une diminution des coûts.

## <a name="optimize-by-changing-indexing-policy"></a>Optimiser en modifiant la stratégie d’indexation 

Par défaut, Azure Cosmos DB indexe automatiquement chaque propriété de chaque enregistrement. Cette stratégie vise à faciliter le développement et à garantir d’excellentes performances dans différents types de requêtes ad-hoc. Si vos enregistrements volumineux contiennent des milliers de propriétés, mieux vaut éviter de payer le débit qu’entraînerait l’indexation de chaque propriété, surtout si vos recherches se limitent à 10 ou 20 de ces propriétés. À mesure que vous vous approchez de votre charge de travail finale, nous vous recommandons d’optimiser votre stratégie d’indexation. Vous trouverez plus d’informations sur la stratégie d’indexation Azure Cosmos DB [ici](index-policy.md). 

## <a name="monitoring-provisioned-and-consumed-throughput"></a>Surveillance du débit approvisionné et consommé 

Vous pouvez surveiller le nombre total d’unités de requête approvisionnées, le nombre de demandes de limitation de taux, ainsi que le nombre d’unités de requête que vous avez consommées dans le portail Azure. L’image suivante montre un exemple de métrique d’utilisation :

:::image type="content" source="./media/optimize-cost-throughput/monitoring.png" alt-text="Surveiller les unités de requête dans le Portail Azure":::

Vous pouvez également définir des alertes pour vérifier si le nombre de demandes de limitation de taux dépasse un seuil spécifique. Consultez l’article [Comment surveiller un compte Azure Cosmos DB](use-metrics.md) pour plus d’informations. Ces alertes peuvent envoyer un e-mail aux administrateurs de compte ou appeler un Webhook HTTP personnalisé ou une fonction Azure afin d’augmenter automatiquement le débit approvisionné. 

## <a name="scale-your-throughput-elastically-and-on-demand"></a>Faire évoluer votre débit en toute flexibilité et à la demande 

Dans la mesure où vous êtes facturé selon le débit approvisionné, adapter le débit approvisionné à vos besoins peut vous aider à éviter les frais qu’entraîne le débit inutilisé. Vous pouvez à tout moment ajuster à la hausse ou à la baisse votre débit approvisionné, en fonction de vos besoins. Si vos besoins en matière de débit sont très prévisibles, vous pouvez utiliser Azure Functions et avoir recours à un déclencheur à minuterie pour [augmenter ou réduire le débit selon un calendrier](scale-on-schedule.md). 

* La surveillance de la consommation de vos unités de requête et de vos demandes de limitation de taux peut révéler que vous n’avez pas besoin d’un débit approvisionné constant tout au long de la journée ou de la semaine. Votre trafic sera peut-être moindre pendant la nuit ou le week-end. En utilisant le portail Azure, les kits de développement logiciel Azure Cosmos DB natifs ou des API REST, vous pouvez faire évoluer votre débit approvisionné à tout moment. L’API REST Azure Cosmos DB fournit des points de terminaison pour mettre à jour par programmation le niveau de performance de vos conteneurs, ce qui permet d’ajuster facilement le débit à partir de votre code en fonction de l’heure de la journée ou du jour de la semaine. L’opération se déroule sans interruption du service et généralement en moins d’une minute. 

* Faites évoluer le débit lorsque vous ingérez des données dans Azure Cosmos DB, par exemple, lors de la migration de données. Une fois la migration effectuée, vous pouvez réduire le débit approvisionné pour gérer votre solution revenue à un état stable.  

* N’oubliez pas que la facturation est appliquée par tranche horaire : vous ne réaliserez donc aucune économie si vous modifiez votre débit approvisionné plus d’une fois par heure.

## <a name="determine-the-throughput-needed-for-a-new-workload"></a>Déterminer le débit nécessaire pour une nouvelle charge de travail 

Pour déterminer le débit approvisionné d’une nouvelle charge de travail, vous pouvez procéder comme suit : 

1. Effectuez une évaluation approximative initiale à l’aide de l’outil Capacity Planner et ajustez vos estimations avec l’explorateur Azure Cosmos DB dans le portail Azure. 

2. Il est recommandé de créer les conteneurs avec un débit plus élevé que prévu puis de diminuer ce débit en fonction des besoins. 

3. Il est recommandé d’utiliser un des kits de développement logiciel (SDK) Azure Cosmos DB natifs pour tirer parti des nouvelles tentatives automatiques avec les demandes à taux limité. Si vous utilisez une plateforme non prise en charge et l’API REST Azure Cosmos DB, implémentez votre propre stratégie de nouvelle tentative à l’aide de l’en-tête `x-ms-retry-after-ms`. 

4. Assurez-vous que votre code d’application est capable de gérer l’échec de toutes les tentatives. 

5. Vous pouvez configurer des alertes à partir du portail Azure afin de recevoir des notifications en cas de limitation du taux. Vous pouvez commencer par des limites conservatrices comme 10 requêtes à taux limité au cours des 15 dernières minutes, puis basculer vers des règles plus strictes une fois vous avez estimé votre consommation réelle. Vous pouvez également utiliser des limites de taux occasionnelles pour indiquer que vous testez différentes limites définies et que c’est exactement ce que vous voulez faire. 

6. Utilisez la surveillance pour comprendre le fonctionnement de votre modèle de trafic et déterminer si vous avez besoin d’ajuster dynamiquement votre provisionnement de débit sur la journée ou sur une semaine. 

7. Comparez régulièrement votre débit approvisionné et votre taux consommé pour vérifier que vous n’avez pas approvisionné plus de conteneurs et de bases de données que nécessaire. À des fins de contrôle, il est judicieux d’avoir un débit légèrement surapprovisionné.  

### <a name="best-practices-to-optimize-provisioned-throughput"></a>Meilleures pratiques pour optimiser le coût du débit approvisionné 

Les étapes suivantes vous aider à rendre vos solutions hautement évolutives et économiques lors de l’utilisation d’Azure Cosmos DB.  

1. Si vous avez considérablement surapprovisionné le débit sur les conteneurs et les bases de données, vous devez comparer les unités de requête approvisionnées et les unités de requêtes consommées afin d’ajuster les charges de travail.  

2. Une méthode permettant d’estimer la quantité de débit réservé requis par votre application consiste à enregistrer les frais d’unité de requête associés à l’exécution des opérations courantes sur un élément représentatif utilisé par votre application (un conteneur ou une base de données Azure Cosmos), puis à évaluer le nombre d’opérations que vous prévoyez d’effectuer chaque seconde. Veillez à mesurer et à inclure également les requêtes courantes et leur utilisation. Pour savoir comment estimer le coût des RU de requêtes par programme ou à l’aide du portail, voir [Optimiser le coût des requêtes](./optimize-cost-reads-writes.md). 

3. Une autre façon d’évaluer les opérations et leur coût en unités de requête consiste à activer les journaux d’activité Azure Monitor afin d’obtenir la répartition par opération/durée et les frais de chaque requête. Azure Cosmos DB applique des frais de requête pour chaque opération : ainsi, les frais de chaque opération peuvent être consignés dans la réponse à des fins d’analyse ultérieure. 

4. Vous pouvez augmenter ou réduire en toute flexibilité le débit approvisionné pour répondre aux besoins de votre charge de travail. 

5. Vous pouvez ajouter et supprimer des régions associées à votre compte Azure Cosmos selon vos besoins et contrôler ainsi vos coûts. 

6. Assurez-vous que vos données et charges de travail sont réparties de façon uniforme entre les partitions logiques de vos conteneurs. Si cette répartition des partitions n’est pas uniforme, vous risquez d’approvisionner un débit supérieur à ce qui est nécessaire. Si vous constatez que la répartition n’est pas uniforme, nous vous recommandons de redistribution la charge de travail uniformément entre les partitions ou de repartitionner les données. 

7. Si vous avez de nombreux conteneurs qui ne nécessitent pas de contrats de niveau de service, vous pouvez utiliser l’offre basée sur la base de données pour les cas où les contrats de niveau de service avec débit par conteneur ne s’appliquent pas. Vous devez identifier les conteneurs Azure Cosmos que vous souhaitez migrer vers l’offre de débit de niveau base de données inférieure puis migrer ces conteneurs à l’aide d’une solution basée sur des flux de modification. 

8. Vous pouvez utiliser « Cosmos DB Free Tier » (gratuit pendant un an), Try Cosmos DB (jusqu’à trois régions) ou l’émulateur téléchargeable Cosmos DB pour les scénarios de développement et de test. En utilisant ces options dans un environnement de développement et de test, vous pouvez considérablement réduire vos coûts.  

9. Vous pouvez effectuer d’autres optimisations spécifiques à votre charge de travail, par exemple augmenter la taille des lots, équilibrer la charge de travail des lectures dans plusieurs régions, et dédupliquer des données, le cas échéant.

10. Grâce à la capacité réservée Azure Cosmos DB, vous pouvez bénéficier de remises pouvant atteindre 65 % pendant trois ans. Le modèle de capacité réservée Azure Cosmos DB gère en amont les unités de requête qui seront nécessaires au fil du temps. Les remises sont accordées de telle sorte que plus vous utilisez des unités de requête sur longue période, plus votre remise sera importante. Ces remises sont appliquées immédiatement. Les unités de requête utilisées au-delà de vos valeurs approvisionnées sont facturées en fonction du coût de la capacité non réservé. Consultez la section [Capacité réservée Cosmos DB](cosmos-db-reserved-capacity.md)) pour plus d’informations. Vous pouvez également acheter une capacité réservée afin de réduire encore davantage le coût de votre débit approvisionné.  

## <a name="next-steps"></a>Étapes suivantes

Pour continuer à développer vos connaissances sur l’optimisation des coûts dans Azure Cosmos DB, consultez les articles suivants :

* En savoir plus sur l’[optimisation pour le développement et le test](optimize-dev-test.md)
* En savoir plus sur les [factures Azure Cosmos DB](understand-your-bill.md)
* En savoir plus sur l’[optimisation du coût de stockage](optimize-cost-storage.md)
* En savoir plus sur l’[optimisation du coût des lectures et écritures](optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des requêtes](./optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des comptes Azure Cosmos sur plusieurs régions](optimize-cost-regions.md)
