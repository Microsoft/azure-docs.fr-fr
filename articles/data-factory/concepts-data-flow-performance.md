---
title: Guide des performances et de réglage du flux de données de mappage
description: Découvrez des informations sur les facteurs clés ayant une incidence sur les performances des flux de données de mappage dans Azure Data Factory.
author: kromerm
ms.topic: conceptual
ms.author: makromer
ms.service: data-factory
ms.custom: seo-lt-2019
ms.date: 11/24/2020
ms.openlocfilehash: cc06f12317f5e30721452e07bd4dc5f50dfdb7ec
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022358"
---
# <a name="mapping-data-flows-performance-and-tuning-guide"></a>Guide des performances et du réglage du mappage de flux de données

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Les flux de données de mappage dans Azure Data Factory fournissent une interface sans code pour concevoir, déployer et orchestrer des transformations de données à grande échelle. Si vous n’êtes pas familiarisé avec les flux de données de mappage, consultez [Vue d’ensemble des flux de données de mappage](concepts-data-flow-overview.md). Cet article met en évidence différentes façons de régler et d’optimiser vos flux de données afin qu’ils soient conformes à vos points de référence en matière de performances.

Regardez la vidéo ci-dessous pour voir des exemples de minutages qui transforment les données avec des flux de données.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4rNxM]

## <a name="testing-data-flow-logic"></a>Test de la logique de flux de données

Lorsque vous concevez et testez des flux de données à partir de l’expérience utilisateur ADF, le mode débogage vous permet de tester de manière interactive par rapport à un cluster Spark en direct. Cela vous permet d’afficher un aperçu des données et d’exécuter vos flux de données sans attendre le préchauffage d’un cluster. Pour plus d’informations, consultez [Mode de débogage](concepts-data-flow-debug-mode.md).

## <a name="monitoring-data-flow-performance"></a>Supervision des performances de flux de données

Une fois que vous avez vérifié votre logique de transformation à l’aide du mode débogage, exécutez votre flux de travail de bout en bout en tant qu’activité dans un pipeline. Les flux de données sont mis en œuvre dans un pipeline à l’aide de l’[activité d’exécution d’un flux de données](control-flow-execute-data-flow-activity.md). L’activité de flux de travail offre une expérience de surveillance unique par rapport aux autres activités dans Azure Data Factory, qui affichent un plan d’exécution détaillé et un profil de performances de la logique de transformation. Pour afficher des informations de surveillance détaillées d’un flux de données, cliquez sur l’icône de lunettes dans la sortie d’exécution de l’activité d’un pipeline. Pour plus d’informations, consultez [Supervision des flux de données de mappage](concepts-data-flow-monitoring.md).

![Moniteur Data Flow](media/data-flow/monitoring-details.png "Moniteur Data Flow 2")

Lors de la surveillance des performances du flux de données, quatre goulots d’étranglement peuvent se présenter :

* temps de démarrage du cluster ;
* lecture à partir d’une source ;
* temps de transformation ;
* écriture dans un récepteur. 

![Surveillance du flux de données](media/data-flow/monitoring-performance.png "Moniteur Data Flow 3")

Le temps de démarrage du cluster est le temps nécessaire au lancement d’un cluster Apache Spark. Cette valeur se trouve dans l’angle supérieur droit de l’écran de surveillance. Les flux de données s’exécutent selon un modèle juste-à-temps, où chaque travail utilise un cluster isolé. Ce temps de démarrage est généralement de 3 à 5 minutes. Pour des travaux séquentiels, il est possible de réduire ce temps en activant une valeur de durée de vie. Pour plus d’informations, consultez [Optimisation d’Azure Integration Runtime](#ir).

Les flux de données utilisent un optimiseur Spark qui réorganise et exécute votre logique métier par « étapes » afin d’accélérer le processus. Pour chaque récepteur dans lequel votre flux de données écrit, la sortie de la surveillance indique la durée de chaque étape de transformation, ainsi que le temps nécessaire pour écrire les données dans le récepteur. La durée la plus important correspond probablement au goulot d’étranglement de votre flux de données. Si l’étape de transformation qui prend le plus de temps contient une source, vous souhaiterez peut-être optimiser davantage votre temps de lecture. Si une transformation prend beaucoup de temps, il se peut que vous deviez repartitionner ou augmenter la taille de votre runtime d’intégration. Si la durée de traitement du récepteur est importante, il se peut que vous deviez augmenter l’échelle de votre base de données ou vérifier que la sortie ne s’effectue pas dans un seul fichier.

Une fois que vous avez identifié le goulot d’étranglement de votre flux de données, utilisez les stratégies d’optimisation ci-dessous pour améliorer les performances.

## <a name="optimize-tab"></a>Onglet Optimiser

L’onglet **Optimiser** contient des paramètres pour configurer le schéma de partitionnement du cluster Spark. Cet onglet présent dans chaque transformation de flux de données spécifie si vous souhaitez repartitionner les données **après** la transformation. L’ajustement du partitionnement permet de contrôler la répartition de vos données entre les nœuds de calcul et les optimisations de la localisation des données qui peuvent avoir des effets tant positifs que négatifs sur les performances globales de vos flux de données.

![La capture d’écran montre l’onglet Optimiser, qui comprend l’option Partition, Type de partition et Nombre de partitions.](media/data-flow/optimize.png)

Par défaut, l’option *Utiliser le partitionnement actuel* est sélectionnée, qui indique à Azure Data Factory de conserver le partitionnement de sortie actuel de la transformation. Le repartitionnement des données prenant du temps, est recommandé d’*utiliser le partitionnement actuel* dans la plupart des scénarios. Les scénarios dans lesquels il peut être souhaitable de repartitionner vos données incluent des agrégats et des jointures qui entraînent une asymétrie considérable de vos données, ou l’utilisation d’un partitionnement de source sur une base de données SQL.

Pour changer le partitionnement de n’importe quelle transformation, sélectionnez l’onglet **Optimiser**, puis la case d’option **Définir le partitionnement**. Une série d'options de partitionnement vous est présentée. La méthode de partitionnement qui convient varie en fonction des volumes de données, des clés candidates, des valeurs null et de la cardinalité. 

> [!IMPORTANT]
> Une partition unique combine toutes les données distribuées dans une seule partition. Il s’agit d’une opération très lente qui affecte considérablement toutes les transformations et écritures en aval. Le service Azure Data Factory déconseille fortement d’utiliser cette option, sauf en présence d’objectifs métier explicites.

Les options de partitionnement suivantes sont disponibles dans chaque transformation :

### <a name="round-robin"></a>Tourniquet 

Un tourniquet (round robin) répartit équitablement les données entre les partitions. Utilisez un tourniquet (round robin) lorsque vous ne disposez pas des clés candidates adéquates pour implémenter une stratégie de partitionnement solide et intelligente. Vous pouvez définir le nombre de partitions physiques.

### <a name="hash"></a>Hachage

Azure Data Factory produit un hachage de colonnes pour obtenir des partitions uniformes, de sorte que des lignes contenant des valeurs similaires tombent dans la même partition. Lorsque vous utilisez l’option Hachage, effectuez un test pour détecter une éventuelle inclinaison de partition. Vous pouvez définir le nombre de partitions physiques.

### <a name="dynamic-range"></a>Plage dynamique

La plage dynamique utilise des plages dynamiques Spark basées sur les colonnes ou expressions que vous fournissez. Vous pouvez définir le nombre de partitions physiques. 

### <a name="fixed-range"></a>Plage fixe

Créez une expression qui fournit une plage fixe pour les valeurs figurant dans vos colonnes de données partitionnées. Pour éviter une inclinaison de partition, vous devez avoir une bonne compréhension de vos données avant d’utiliser cette option. Les valeurs que vous entrez pour l'expression sont utilisées dans le cadre d'une fonction de partition. Vous pouvez définir le nombre de partitions physiques.

### <a name="key"></a>Clé

Si vous avez une bonne compréhension de la cardinalité de vos données, une clé de partitionnement peut être une bonne stratégie. La clé de partitionnement crée des partitions pour chaque valeur unique de votre colonne. Vous ne pouvez pas définir le nombre de partitions car celui-ci dépend des valeurs uniques figurant dans les données.

> [!TIP]
> La définition manuelle du schéma de partitionnement a pour effet de remanier les données et risque d’annihiler les avantages de l’optimiseur Spark. La meilleure pratique consiste à ne pas définir manuellement le partitionnement, sauf si c’est indispensable.

## <a name="logging-level"></a>Niveau de journalisation

Si vous n’avez pas besoin que chaque exécution du pipeline de vos activités de flux de données journalise entièrement tous les journaux de télémétrie détaillés, vous pouvez éventuellement définir le niveau de journalisation sur « De base » ou « Aucun ». Lors de l’exécution de vos flux de données en mode « Verbose » (par défaut), vous demandez à ADF d’enregistrer entièrement l’activité à chaque niveau de partition individuel au cours de la transformation des données. Cela peut être une opération coûteuse. Par conséquent, n’activez l’option Verbose que lorsque la résolution des problèmes peut améliorer les performances globales du pipeline et du flux de données. Le mode « De base » ne consigne que les durées de transformation, tandis que le mode « Aucun » ne fournit qu’un résumé des durées.

![Niveau de journalisation](media/data-flow/logging.png "Définir le niveau de journalisation")

## <a name="optimizing-the-azure-integration-runtime"></a><a name="ir"></a> Optimisation d’Azure Integration Runtime

Les flux de données s’exécutent sur des clusters Spark qui sont lancés au moment de l’exécution. La configuration du cluster utilisé est définie dans le runtime d’intégration (IR) de l’activité. Trois aspects en lien avec les performances sont à prendre en considération lors de la définition de votre runtime d’intégration : type de cluster, taille de cluster et durée de vie.

Pour plus d’informations sur la création d’un runtime d’intégration, consultez [Runtime d’intégration dans Azure Data Factory](concepts-integration-runtime.md).

### <a name="cluster-type"></a>Type de cluster

Trois options sont disponibles pour le type de cluster Spark démarré : à usage général, à mémoire optimisée et optimisé pour le calcul.

Les clusters **à usage général** sont la sélection par défaut et sont idéaux pour la plupart des charges de travail de flux de données. Ils offrent généralement le meilleur compromis entre performances et coûts.

Si votre flux de données contient de nombreuses jointures et recherches, vous pouvez peut-être utiliser un cluster **à mémoire optimisée**. Les clusters à mémoire optimisée peuvent stocker davantage de données en mémoire et minimiser les erreurs dues à une mémoire insuffisante. Si le niveau de prix par cœur de l’option à mémoire optimisée est le plus élevé, celle-ci tend également à produire des pipelines plus performants. Si vous rencontrez des erreurs dues à une insuffisance de mémoire lors de l’exécution de vos flux de données, basculez vers une configuration d’Azure Integration Runtime à mémoire optimisée. 

L’option **optimisé pour le calcul** n’est pas idéale pour les flux de travail ETL et l’équipe Azure Data Factory la déconseille pour la plupart des charges de travail de production. Pour des transformations de données plus simples n’utilisant pas beaucoup de mémoire, telles que le filtrage de données ou l’ajout de colonnes dérivées, des clusters optimisés pour le calcul permettent d’abaisser le coût par cœur.

### <a name="cluster-size"></a>Taille du cluster

Les flux de données répartissent le traitement des données sur différents nœuds d’un cluster Spark pour effectuer des opérations en parallèle. Un cluster Spark avec davantage de cœurs augmente le nombre de nœuds dans l’environnement Compute. Des nœuds en plus grand nombre augmentent la puissance de traitement du flux de données. L’augmentation de la taille du cluster est souvent un moyen simple de réduire le temps de traitement.

La taille de cluster par défaut est de quatre nœuds de pilote et quatre nœuds worker.  Lorsque vous traitez davantage de données, des clusters plus grands sont recommandés. Vous trouverez ci-dessous les options de dimensionnement possibles :

| Cœurs de worker | Cœurs de pilote | Nombre total de cœurs | Notes |
| ------------ | ------------ | ----------- | ----- |
| 4 | 4 | 8 | Non disponible pour l’option optimisé pour le calcul |
| 8 | 8 | 16 | |
| 16 | 16 | 32 | |
| 32 | 16 | 48 | |
| 64 | 16 | 80 | |
| 128 | 16 | 144 | |
| 256 | 16 | 272 | |

Les flux de données sont tarifés sur la base de vCores/heure, mode de calcul intégrant la taille du cluster et le facteur de temps d’exécution. Lorsque vous augmentez l’échelle, le coût par minute de votre cluster augmente, mais le temps global d’exécution diminue.

> [!TIP]
> Il existe un plafond à la manière dont la taille d’un cluster affecte les performances d’un flux de données. Selon la taille de vos données, il existe un point au-delà duquel l’augmentation de la taille d’un cluster n’améliore plus les performances. Par exemple, si vous avez plus de nœuds que de partitions de données, l’ajout de nœuds n’est pas utile. Une bonne pratique consiste à commencer petit et à augmenter l’échelle pour répondre à vos besoins en matière de performances. 

### <a name="time-to-live"></a>Durée de vie

Par défaut, chaque activité de flux de données a pour effet de démarrer un nouveau cluster en fonction de la configuration du runtime d’intégration. Le démarrage du cluster prend quelques minutes et le traitement des données ne peut pas commencer tant que le processus de démarrage n’est pas terminé. Si vos pipelines contiennent plusieurs flux de données **séquentiels**, vous pouvez activer une valeur de durée de vie (TTL). La spécification d’une valeur de durée de vie a pour effet que le cluster reste actif pendant un certain temps après la fin de son exécution. Si un nouveau travail commence à utiliser le runtime d’intégration (IR) pendant la durée de vie (TTL), il va réutiliser le cluster existant et le temps de démarrage sera donc fortement réduit. Une fois le deuxième travail terminé, le cluster reste actif pendant la durée de vie.

Une seule tâche peut être exécutée sur un seul cluster à la fois. Si un cluster est disponible, mais que deux flux de données démarrent, un seul utilise par le cluster actif. Le deuxième travail démarre son propre cluster isolé.

Si la plupart de vos flux de données s’exécutent en parallèle, il n’est pas recommandé d’activer la durée de vie. 

> [!NOTE]
> La durée de vie n’est pas disponible lors de l’utilisation du runtime d’intégration de résolution automatique.

## <a name="optimizing-sources"></a>Optimisation des sources

Pour chaque source à l’exception d’Azure SQL Database, il est recommandé de conserver l’option **Utiliser le partitionnement actuel** sélectionnée. Lors de la lecture à partir de tous les autres systèmes sources, le flux de données partitionne automatiquement les données uniformément en fonction de la taille des données. Une nouvelle partition est créée pour chaque volume d’environ 128 Mo de données. Le nombre de partitions augmente à mesure que la taille des données augmente.

Tout partitionnement personnalisé se produit *après* que Spark a lu les données, et a un impact négatif sur les performances de votre flux de données. Comme les données étant partitionnées uniformément lors de la lecture, cela n’est pas recommandé. 

> [!NOTE]
> Les vitesses de lecture peuvent être limitées par le débit de votre système source.

### <a name="azure-sql-database-sources"></a>Sources d’Azure SQL Database

Azure SQL Database offre une option de partitionnement unique appelée partitionnement de la source. L’activation du partitionnement de la source peut améliorer vos temps de lecture à partir d’Azure SQL DB en activant des connexions parallèles sur le système source. Spécifiez le nombre de partitions et la manière de partitionner vos données. Utilisez une colonne de partition avec une cardinalité élevée. Vous pouvez également entrer une requête correspondant au schéma de partitionnement de votre table source.

> [!TIP]
> Pour le partitionnement de la source, les E/S de SQL Server constituent le goulot d’étranglement. L’ajout d’un trop grand nombre de partitions peut saturer votre base de données source. Généralement, quatre ou cinq partitions sont idéales lors de l’utilisation de cette option.

![Partitionnement de source](media/data-flow/sourcepart3.png "Partitionnement de source")

#### <a name="isolation-level"></a>Niveau d'isolation

Le niveau d’isolation de la lecture sur un système source SQL Azure a un impact sur les performances. L’option « Lecture non validée » permet d’obtenir des performances optimales et d’éviter tout verrouillage de la base de données. Pour en savoir plus sur les niveaux d’isolation SQL, consultez [Présentation des niveaux d’isolation](https://docs.microsoft.com/sql/connect/jdbc/understanding-isolation-levels).

#### <a name="read-using-query"></a>Lire à l’aide d’une requête

Vous pouvez lire à partir d’Azure SQL Database à l’aide d’une table ou d’une requête SQL. Si vous exécutez une requête SQL, celle-ci doit se terminer avant que la transformation puisse démarrer. Des requêtes SQL peuvent être utiles pour effectuer plus tôt des opérations susceptibles de s’exécuter plus rapidement et réduire la quantité de données lues à partir d’un serveur SQL Server, comme des instructions SELECT, WHERE et JOIN. En effectuant des opérations plus tôt, vous perdez la possibilité de suivre le lignage et les performances des transformations avant que les données entrent dans le flux de données.

### <a name="azure-synapse-analytics-sources"></a>Sources d’Azure Synapse Analytics

Quand vous utilisez Azure Synapse Analytics, un paramètre appelé **Activer le mode de préproduction** existe dans les options de source. Cela permet à ADF de lire à partir de Synapse à l’aide de ```Polybase```, ce qui améliore considérablement les performances de lecture. L’activation de ```Polybase``` nécessite que vous spécifiiez un Stockage Blob Azure ou un emplacement de préproduction Azure Data Lake Storage Gen2 dans les paramètres d’activité du flux de données.

![Activer le mode de préproduction](media/data-flow/enable-staging.png "Activer le mode de préproduction")

### <a name="file-based-sources"></a>Sources basées sur des fichiers

Si les flux de données prennent en charge un vaste éventail de types de fichiers, le service Azure Data Factory recommande d’utiliser le format Parquet natif de Spark pour optimiser les temps de lecture et d’écriture.

Si vous exécutez le même flux de données sur un ensemble de fichiers, nous vous recommandons de lire à partir d’un dossier en utilisant des chemins d’accès génériques ou en lisant à partir d’une liste de fichiers. Une exécution d’activité de flux de données unique peut traiter tous vos fichiers par lots. Vous trouverez plus d’informations sur la définition de ces paramètres dans la documentation du connecteur, par exemple dans [Stockage Blob Azure](connector-azure-blob-storage.md#source-transformation).

Autant que possible, évitez d’utiliser l’activité For-Each pour exécuter des flux de données sur un ensemble de fichiers. Cela aura pour effet que chaque itération de l’activité For-Each lance son propre cluster Spark, ce qui n’est souvent pas nécessaire et peut s’avérer onéreux. 

## <a name="optimizing-sinks"></a>Optimisation des récepteurs

Quand des flux de données écrivent dans des récepteurs, tout partitionnement personnalisé se produit immédiatement avant l’écriture. Comme pour la source, dans la plupart des cas, il est recommandé de conserver l’option de partition **Utiliser le partitionnement actuel** en tant qu’option de partition sélectionnée. L’écriture de données partitionnées est sensiblement plus rapide que l’écriture de données non partitionnées, même si votre destination n’est pas partitionnée. Vous trouverez ci-dessous des considérations spécifiques concernant les différents types de récepteurs. 

### <a name="azure-sql-database-sinks"></a>Récepteurs Azure SQL Database

Avec Azure SQL Database, le partitionnement par défaut doit fonctionner dans la plupart des cas. Il est possible que votre récepteur ait un trop nombre de partitions à gérer trop important pour votre base de données SQL. Si vous rencontrez ce problème, réduisez le nombre de partitions générées par votre récepteur SQL Database.

#### <a name="impact-of-error-row-handling-to-performance"></a>Impact de la gestion des lignes d’erreur sur les performances

Lorsque vous activez la gestion des lignes d’erreur (« continuer en cas d’erreur ») dans la transformation du récepteur, ADF effectue une étape supplémentaire avant d’écrire les lignes compatibles dans votre table de destination. Cette étape supplémentaire entraîne une faible baisse des performances qui peut être comprise dans la plage de 5 % pour cette étape, avec un faible impact supplémentaire sur les performances également ajouté si vous définissez l’option de consigner aussi avec les lignes incompatibles dans un fichier journal.

#### <a name="disabling-indexes-using-a-sql-script"></a>Désactivation des index à l’aide d’un script SQL

La désactivation des index avant un chargement dans une base de données SQL peut améliorer considérablement les performances d’écriture dans la table. Avant d’écrire sur votre récepteur SQL, exécutez la commande ci-dessous.

`ALTER INDEX ALL ON dbo.[Table Name] DISABLE`

Une fois l’écriture terminée, régénérez les index à l’aide de la commande suivante :

`ALTER INDEX ALL ON dbo.[Table Name] REBUILD`

Ces opérations peuvent être effectuées en mode natif à l’aide de scripts Pre-SQL et Post-SQL au sein d’une base de données SQL Azure ou d’un récepteur Synapse dans le mappage de flux de données.

![Désactiver les index](media/data-flow/disable-indexes-sql.png "Désactiver les index")

> [!WARNING]
> Lors de la désactivation des index, le flux de données prend le contrôle d’une base de données et les requêtes sont peu susceptibles d’aboutir à ce moment. Par conséquent, de nombreux travaux ETL sont déclenchés de nuit pour éviter ce conflit. Pour plus d’informations, découvrez les [contraintes de la désactivation d’index](https://docs.microsoft.com/sql/relational-databases/indexes/disable-indexes-and-constraints).

#### <a name="scaling-up-your-database"></a>Augmentation de l’échelle de votre base de données

Planifiez un redimensionnement de vos bases de données Azure SQL DB et Azure SQL DW de source et de récepteur avant l’exécution de votre pipeline pour augmenter le débit et réduire la limitation de requêtes Azure une fois que vous atteignez les limites DTU. Une fois l’exécution de votre pipeline terminée, redimensionnez vos bases de données à leur fréquence d’exécution normale.

### <a name="azure-synapse-analytics-sinks"></a>Récepteurs Azure Synapse Analytics

Lors de l’écriture dans Azure Synapse Analytics, assurez-vous que l’option **Activer le mode de préproduction** est définie sur true. Cela permet à ADF d’écrire à l’aide de [Polybase](/sql/relational-databases/polybase/polybase-guide) qui charge les données en bloc. Vous devrez référencer un référentiel Azure Data Lake Storage Gen2 ou un compte de Stockage Blob Azure pour la préproduction des données lors de l’utilisation de Polybase.

Outre Polybase, les mêmes meilleures pratiques s’appliquent à Azure Synapse Analytics qu’à Azure SQL Database.

### <a name="file-based-sinks"></a>Récepteurs basés sur des fichiers 

Si les flux de données prennent en charge un vaste éventail de types de fichiers, le service Azure Data Factory recommande d’utiliser le format Parquet natif de Spark pour optimiser les temps de lecture et d’écriture.

Si les données sont distribués uniformément, **Utiliser le partitionnement actuel** sera l’option de partitionnement la plus rapide pour l’écriture de fichiers.

#### <a name="file-name-options"></a>Options de nom de fichier

Lorsque vous écrivez des fichiers, vous avez le choix entre plusieurs options de nommage qui ont toutes une incidence sur les performances.

![Options du récepteur](media/data-flow/file-sink-settings.png "Options du récepteur")

La sélection de l’option **Par défaut** permet d’écrire le plus rapidement. Chaque partition correspond à un fichier avec le nom par défaut Spark. Cela est utile si vous lisez simplement à partir du dossier de données.

La définition d’un **modèle** de nommage a pour effet de renommer chaque fichier de partition en lui attribuant un nom plus convivial. Cette opération a lieu après l’écriture et est légèrement plus lente que le choix de la valeur par défaut. L’option Par partition vous permet de nommer chaque partition manuellement.

Si une colonne correspond à la sortie de données que vous souhaitez, vous pouvez sélectionner l’option **As data in column** (comme les données dans la colonne). Cela a pour effet de remanier les données et peut avoir un impact sur les performances si les colonnes ne sont pas distribuées uniformément.

La **sortie vers un fichier unique** combine toutes les données dans une seule partition. Cela se traduit par des temps d’écriture longs, en particulier pour des jeux de données volumineux. L’équipe Azure Data Factory **déconseille** fortement d’utiliser cette option, sauf en présence d’objectifs métier explicites.

### <a name="cosmosdb-sinks"></a>Récepteurs CosmosDB

Lors de l’écriture dans CosmosDB, la modification du débit et de la taille de lot lors de l’exécution du flux de données peut améliorer les performances. Ces modifications ne prennent effet que lors de l’exécution de l’activité de flux de données et les paramètres de collecte d’origine sont rétablis une fois l’exécution terminée. 

**Taille du lot :** calculez la taille de ligne approximative de vos données et vérifiez que le produit de la taille ligne multipliée par la taille de lot est inférieur à deux millions. Le cas échéant, augmentez la taille du lot pour obtenir un meilleur débit.

**Débit :** Définissez un paramètre de débit plus élevé ici pour permettre aux documents d’écrire plus rapidement sur CosmosDB. N’oubliez pas les coûts d’unité de requête supérieurs inhérents à un paramètre de débit élevé.

**Budget du débit d’écriture :** Utilisez une valeur inférieure au nombre total d’unités de requête par minute. Si vous avez un flux de données avec un grand nombre de partitions Spark, la définition d’un budget de débit permet d’équilibrer davantage ces partitions.

## <a name="optimizing-transformations"></a>Optimisation des transformations

### <a name="optimizing-joins-exists-and-lookups"></a>Optimisation des transformations Joins, Exists et Lookups

#### <a name="broadcasting"></a>Diffusion

Dans les transformations Joins, Exists et Lookups, si un ou les deux flux de données sont suffisamment petits pour tenir dans la mémoire de nœud worker, vous pouvez optimiser les performances en activant la **Diffusion**. Un diffusion a lieu quand vous envoyez de petites trames de données à tous les nœuds du cluster. Cela permet au moteur Spark d’effectuer une jointure sans remanier les données dans le grand flux. Par défaut, le moteur Spark détermine automatiquement s’il faut ou non diffuser un côté d’une jointure. Si vous êtes familiarisé avec vos données entrantes et savez qu’un flux sera beaucoup plus petit que l’autre, vous pouvez sélectionner une diffusion **Fixe**. Une diffusion fixe force Spark à diffuser le flux sélectionné. 

Si la taille des données diffusées est trop importante pour le nœud Spark, il se peut que vous rencontriez une erreur de mémoire insuffisante. Pour éviter les erreurs de mémoire insuffisante, utilisez des clusters **à mémoire optimisée**. Si vous rencontrez des délais d’expiration de diffusion au cours d’exécutions de flux de données, vous pouvez désactiver l’optimisation de la diffusion. Toutefois, cela se traduit par des flux de données plus lents.

![Optimisation de la transformation de jointure (Join)](media/data-flow/joinoptimize.png "Optimisation de la jointure")

#### <a name="cross-joins"></a>Jointures croisées

Si vous utilisez des valeurs littérales dans vos conditions de jointure ou si vous avez plusieurs correspondances des deux côtés d’une jointure, Spark exécute la jointure comme une jointure croisée. Une jointure croisée est un produit cartésien complet qui filtre les valeurs jointes. Cela est beaucoup plus lent que les autres types de jointures. Vérifiez que vous disposez de références de colonnes des deux côtés de vos conditions de jointure pour éviter l’impact sur les performances.

#### <a name="sorting-before-joins"></a>Tri avant les jointures

Contrairement à la jointure de fusion dans les outils tels que SSIS, la transformation de jointure n’est pas une opération de jointure de fusion obligatoire. Les clés de jointure ne nécessitent pas de tri avant la transformation. L’équipe Azure Data Factory ne recommande pas l’utilisation de transformations de tri (Sort) dans le mappage des flux de données.

### <a name="window-transformation-performance"></a>Performances de la transformation de fenêtre

La [transformation de fenêtre](data-flow-window.md) partitionne vos données par valeur dans les colonnes que vous sélectionnez dans le cadre de la clause ```over()``` dans les paramètres de transformation. Il existe un certain nombre de fonctions d’analyse et d’agrégation très populaires qui sont exposées dans la transformation Windows. Toutefois, si votre cas d’usage consiste à générer une fenêtre sur l’ensemble de votre jeu de données pour le classement ```rank()``` ou le numéro de ligne ```rowNumber()```, il est recommandé d’utiliser à la place la [transformation de classement (Rank)](data-flow-rank.md) et la [transformation de clé de substitution (Surrogate Key)](data-flow-surrogate-key.md). Ces transformations seront plus efficaces que les opérations sur des jeux de données complets utilisant ces fonctions.

### <a name="repartitioning-skewed-data"></a>Repartitionnement de données asymétriques

Certaines transformations, telles que des jointures et des agrégats, remanient vos partitions de données et peuvent entraîner une asymétrie des données. Une telle asymétrie a pour effet que les données ne sont pas réparties uniformément entre les partitions. Une asymétrie importante des données peut entraîner un ralentissement des transformations en aval et des écritures dans le récepteur. Vous pouvez vérifier l’asymétrie de vos données à tout moment pendant l’exécution d’un flux de données en cliquant sur la transformation dans l’affichage de la surveillance.

![Asymétrie et kurtosis](media/data-flow/skewness-kurtosis.png "Asymétrie et kurtosis")

L’affichage de la surveillance montre comment les données sont réparties sur chaque partition, ainsi que deux métriques, l’asymétrie et le kurtosis. **L’asymétrie** est une mesure de l’asymétrie des données. Elle peut avoir une valeur positive, nulle, négative ou indéfinie. Une asymétrie négative signifie que la queue gauche est plus longue que la droite. Le **kurtosis** mesure sur l’importance de la queue. Des valeurs de kurtosis élevées ne sont pas souhaitables. Les plages idéales d’asymétrie sont comprises entre -3 et +3, et les plages de kurtosis sont inférieures à 10. Un moyen simple d’interpréter ces nombres consiste à examiner le graphique de partition pour voir si la barre 1 est beaucoup plus grande que le reste.

Si vos données ne sont pas partitionnées uniformément après une transformation, vous pouvez utiliser l’[onglet Optimiser](#optimize-tab) pour repartitionner. Le remaniement des données prend du temps et peut ne pas améliorer les performances de celles-ci.

> [!TIP]
> Si vous repartitionnez vos données, mais avez des transformations en aval qui remanient vos données, utilisez un partitionnement de hachage sur une colonne utilisée comme clé de jointure.

## <a name="using-data-flows-in-pipelines"></a>Utilisation des flux de données dans des pipelines 

Lors de la création de pipelines complexes avec plusieurs flux de données, votre flux logique peut avoir un impact important sur le temps et le coût. Cette section décrit l’impact de différentes stratégies d’architecture.

### <a name="executing-data-flows-in-parallel"></a>Exécuter des flux de données en parallèle

Si vous exécutez plusieurs flux de données en parallèle, ADF crée des clusters Spark distincts pour chaque activité. Cela permet d’isoler chaque travail et de l’exécuter en parallèle, mais cela entraîne l’exécution simultanée de plusieurs clusters.

Si vos flux de données s’exécutent en parallèle, il est recommandé de ne pas activer la propriété de durée de vie Azure IR, car elle entraînerait l’utilisation de plusieurs pools à chaud inutilisés.

> [!TIP]
> Au lieu d’exécuter le même flux de données plusieurs fois pour chaque activité, mettez vos données dans un lac de données et utilisez des chemins d’accès génériques pour traiter les données dans un seul et même flux de données.

### <a name="execute-data-flows-sequentially"></a>Exécuter des flux de données séquentiellement

Si vous exécutez vos activités de flux de données dans l’ordre, il est recommandé de définir une durée de vie dans la configuration d’Azure IR. ADF réutilise les ressources de calcul, ce qui accélère le démarrage du cluster. Chaque activité sera toujours isolée pour recevoir un nouveau contexte Spark pour chaque exécution.

L’exécution des travaux dans l’ordre prendra probablement le plus de temps pour s’exécuter de bout en bout, mais elle fournit une séparation propre des opérations logiques.

### <a name="overloading-a-single-data-flow"></a>Surcharge d’un seul flux de données

Si vous placez toute votre logique au sein d’un même flux de données, ADF exécute l’intégralité du travail sur une seule instance Spark. Bien que cela puisse sembler être un moyen de réduire les coûts, cette approche combine des flux logiques différents et peut être difficile à surveiller et à déboguer. En cas de défaillance d’un composant, toutes les autres parties du travail échouent également. L’équipe Azure Data Factory recommande d’organiser les flux de données en flux indépendants de logique métier. Si votre flux de données devient trop volumineux, le fractionnement en composants distincts facilite la surveillance et le débogage. Bien qu’il n’y ait pas de limite inconditionnelle sur le nombre de transformations dans un flux de données, un trop grand nombre de transformations rend le travail complexe.

## <a name="next-steps"></a>Étapes suivantes

Consultez d’autres articles sur les flux de données consacrés aux performances :

- [Activité Data Flow](control-flow-execute-data-flow-activity.md)
- [Analyser les performances des flux de données](concepts-data-flow-monitoring.md)
