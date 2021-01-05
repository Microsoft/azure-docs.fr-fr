---
title: Analyser et surveiller la dérive des données sur les jeux de données (préversion)
titleSuffix: Azure Machine Learning
description: Créez des superviseurs de jeux de données Azure Machine Learning (préversion), surveillez la dérive des données dans les jeux de données et configurez des alertes.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: copeters
author: lostmygithubaccount
ms.date: 06/25/2020
ms.topic: conceptual
ms.custom: how-to, data4ml
ms.openlocfilehash: 04882c71a2d80e01029dd0a8b476f21a658e632b
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93359593"
---
# <a name="detect-data-drift-preview-on-datasets"></a>Détecter une dérive de données (préversion) sur des jeux de données


> [!IMPORTANT]
> La détection de la dérive de données sur les jeux de données est actuellement en préversion publique.
> La préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail en production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Apprenez à répondre au besoin de gouvernance et à l’équilibrer avec le besoin d’agilité.  

Avec les analyses de jeux de données Azure Machine Learning (version préliminaire), vous pouvez :
* **Analyser la dérive de vos données** pour comprendre comment elles évoluent au fil du temps.
* **Surveiller les données de modèle** pour identifier les différences entre les jeux de données de formation et de service.  Commencez par [la collecte des données de modèle à partir des modèles déployés](how-to-enable-data-collection.md).
* **Surveiller les nouvelles données** pour identifier les différences entre le jeu de données de référence et le jeu de données cible.
* **Profiler les caractéristiques dans les données** pour suivre la façon dont les propriétés statistiques évoluent au fil du temps.
* **Configurer des alertes sur la dérive des données** pour les premiers avertissements relatifs aux problèmes potentiels. 

Un [jeu de données Azure Machine learning](how-to-create-register-datasets.md) est utilisé pour créer l’analyse. Le jeu de données doit inclure une colonne timestamp.

Vous pouvez afficher les métriques de dérive des données avec le kit de développement logiciel (SDK) Python ou dans Azure Machine Learning Studio.  Les autres métriques et insights sont disponibles par le biais de la ressource [Azure Application Insights](../azure-monitor/app/app-insights-overview.md) associée à l’espace de travail Azure Machine Learning.

## <a name="prerequisites"></a>Prérequis

Pour créer et utiliser des analyses de jeux de données, vous avez besoin des éléments suivants :
* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.
* Un [espace de travail Azure Machine Learning](how-to-manage-workspace.md).
* Le [SDK Azure Machine Learning pour Python installé](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py), qui inclut le paquet azureml-datasets.
* Données structurées (tabulaires) avec un horodatage spécifié dans le chemin d’accès du fichier, le nom de fichier ou la colonne dans les données.

## <a name="what-is-data-drift"></a>Qu’est-ce qu’une dérive de données ?

La dérive de données est l’une des principales raisons pour lesquelles la précision du modèle se dégrade au fil du temps.  Dans le contexte de l’apprentissage automatique, la dérive de données est la modification des données d’entrée du modèle qui entraîne une dégradation des performances du modèle.  La surveillance de la dérive des données permet de détecter les problèmes de performances du modèle.

Les causes de la dérive des données sont les suivantes :

- Modifications de processus en amont, comme le remplacement d’un capteur, lequel ne prend plus les mesures en pouces mais en centimètres. 
- Problèmes de qualité des données, par exemple un capteur cassé indique toujours une valeur nulle.
- Dérive naturelle des données, comme la variation de la température moyenne au fil des saisons.
- Modification de la relation entre caractéristiques ou écart de covariable. 

Azure Machine Learning simplifie la détection de la dérive en calculant une mesure unique en faisant abstraction de la complexité des jeux de données comparés.  Ces jeux de données peuvent comporter des centaines de fonctionnalités et des dizaines de milliers de lignes. Une fois la dérive détectée, vous pouvez examiner les fonctionnalités qui sont à l’origine de la dérive.  Vous Inspectez ensuite les métriques au niveau des fonctionnalités pour déboguer et isoler la cause racine de la dérive.

Cette approche descendante permet de surveiller facilement les données qu’avec les techniques traditionnelles basées sur les règles. Les techniques basées sur des règles telles que la plage de données autorisée ou les valeurs uniques autorisées peuvent être longues et sujettes aux erreurs.

Dans Azure Machine Learning, vous utilisez des analyses de jeu de données pour détecter et alerter la dérive des données.
  
### <a name="dataset-monitors"></a>Analyses de jeu de données 

Avec un moniteur de jeu de données, vous pouvez :

* Détecter et signaler la dérive des données sur les nouvelles données d’un jeu de données.
* Analyser les données de l’historique de la dérive.
* Profiler les nouvelles données au fil du temps.

L’algorithme de dérive des données fournit une mesure globale des modifications apportées aux données et indique les caractéristiques qui sont responsables de l’investigation. Les superviseurs de jeu de données produisent un certain nombre d’autres métriques en profilant les nouvelles données dans le jeu de données `timeseries`. 

Des alertes personnalisées peuvent être configurées sur toutes les métriques générées par le superviseur via [Azure Application Insights](../azure-monitor/app/app-insights-overview.md). Les superviseurs de jeu de données permettent de détecter rapidement les problèmes des données et de réduire le temps nécessaire au débogage du problème en identifiant ses causes probables.  

Conceptuellement, il existe trois scénarios principaux pour la configuration des analyses de jeux de données dans Azure Machine Learning.

Scénario | Description
---|---
Surveiller des données de service d’un modèle afin de détecter toute dérive des données de formation du modèle | Les résultats de ce scénario sont le fruit de la surveillance d’un proxy en vue de vérifier l’exactitude du modèle, étant donné que celle-ci se dégrade si les données de service dérivent des données de formation.
Analyser un jeu de données de série chronologique pour la dérive d’une période précédente. | Ce scénario est plus général et peut être utilisé pour analyser les jeux de données impliqués en amont ou en aval de la génération de modèles.  Le jeu de données cible doit avoir une colonne timestamp. Le jeu de données de base peut être n’importe quel jeu de données tabulaires ayant des caractéristiques en commun avec le jeu de données cible.
Exécution de l’analyse des données passées. | Ce scénario peut permettre de comprendre les données historiques et éclairer les décisions de définition des paramètres des moniteurs de jeu de données.

Les analyses de jeu de données dépendent des services Azure suivants.

|Service Azure  |Description  |
|---------|---------|
| *Dataset* | La dérive utilise les jeux de données Machine Learning pour récupérer les données d’apprentissage et comparer les données pour la formation du modèle.  Le profil de génération des données est utilisé pour générer certaines des mesures rapportées telles que min, max, valeurs distinctes, comptage des valeurs distinctes. |
| *Pipeline et calcul Azureml* | La tâche de calcul de dérive est hébergée dans le pipeline azureml.  La tâche est déclenchée à la demande ou par planification pour s’exécuter sur un calcul configuré au moment de la création de l’analyse de dérive.
| *Application Insights*| La dérive émet des mesures pour Application Insights appartenant à l’espace de travail Machine Learning.
| *Stockage Blob Azure*| La dérive émet des métriques au format json vers le stockage d’objets blob Azure.

## <a name="how-dataset-monitors-data"></a>Comment le jeu de données surveille les données

Utilisez les jeux de données Machine Learning pour surveiller la dérive des données. Indiquez un jeu de données de référence (généralement le jeu de données d’entraînement d’un modèle). Un jeu de données cible (généralement constitué de données d’entrée de modèle) est comparé à votre jeu de données de référence au fil du temps. Cette comparaison signifie que votre jeu de données cible doit comporter une colonne d’horodatage (timestamp).

## <a name="create-target-dataset"></a>Créer un jeu de données cible

Le jeu de données cible doit avoir la caractéristique `timeseries` définie sur cette valeur en spécifiant la colonne timestamp à partir d’une colonne de données ou d’une colonne virtuelle dérivée du modèle de chemin d’accès des fichiers. Créez un jeu de données avec timestamp en utilisant le [SDK Python](#sdk-dataset) ou [Azure Machine Learning Studio](#studio-dataset). Une colonne représentant un « timestamp » doit être spécifiée pour ajouter une caractéristique `timeseries` au jeu de données. Si vos données sont partitionnées dans une structure de dossiers avec des informations temporelles, telles que « {yyyyyy/MM/dd} », vous pouvez créer une colonne virtuelle via le paramètre de modèle de chemin d’accès et la définir comme « timestamp de partition » afin de mettre l’accent sur les séries temporelles.

### <a name="python-sdk"></a><a name="sdk-dataset"></a>Kit SDK Python

Dans la classe [`Dataset`](/python/api/azureml-core/azureml.data.tabulardataset?preserve-view=true&view=azure-ml-py#&preserve-view=truewith-timestamp-columns-timestamp-none--partition-timestamp-none--validate-false----kwargs-), la méthode [`with_timestamp_columns()`](/python/api/azureml-core/azureml.data.tabulardataset?preserve-view=true&view=azure-ml-py#&preserve-view=truewith-timestamp-columns-timestamp-none--partition-timestamp-none--validate-false----kwargs-) définit la colonne timestamp du jeu de données.

```python 
from azureml.core import Workspace, Dataset, Datastore

# get workspace object
ws = Workspace.from_config()

# get datastore object 
dstore = Datastore.get(ws, 'your datastore name')

# specify datastore paths
dstore_paths = [(dstore, 'weather/*/*/*/*/data.parquet')]

# specify partition format
partition_format = 'weather/{state}/{date:yyyy/MM/dd}/data.parquet'

# create the Tabular dataset with 'state' and 'date' as virtual columns 
dset = Dataset.Tabular.from_parquet_files(path=dstore_paths, partition_format=partition_format)

# assign the timestamp attribute to a real or virtual column in the dataset
dset = dset.with_timestamp_columns('date')

# register the dataset as the target dataset
dset = dset.register(ws, 'target')
```

Pour obtenir un exemple complet d’utilisation de la caractéristique `timeseries` de jeux de données, consultez l’[exemple de notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/timeseries-datasets/tabular-timeseries-dataset-filtering.ipynb) ou la [documentation du SDK des jeux de données](/python/api/azureml-core/azureml.data.tabulardataset?preserve-view=true&view=azure-ml-py#&preserve-view=truewith-timestamp-columns-timestamp-none--partition-timestamp-none--validate-false----kwargs-).

### <a name="azure-machine-learning-studio"></a><a name="studio-dataset"></a>Azure Machine Learning Studio

Si vous créez votre jeu de données à l’aide de Azure Machine Learning Studio, assurez-vous que le chemin d’accès à vos données contient des informations d’horodatage, incluez tous les sous-dossiers avec des données et définissez le format de partition.

Dans l’exemple suivant, toutes les données sous le sous-dossier *NoaaIsdFlorida/2019* extraites, tandis que le format de partition spécifie l’année, le mois et le jour de l’horodatage.

[![Format de partition](./media/how-to-monitor-datasets/partition-format.png)](media/how-to-monitor-datasets/partition-format-expand.png)

Dans les paramètres **Schéma** , spécifiez la colonne Timestamp d’une colonne virtuelle ou réelle dans le jeu de données spécifié :

:::image type="content" source="media/how-to-monitor-datasets/timestamp.png" alt-text="Définir le timestamp":::

Si vos données sont partitionnées par date, comme c’est le cas ici, vous pouvez également spécifier le timestamp de partition.  Cela permet un traitement plus efficace des dates.

:::image type="content" source="media/how-to-monitor-datasets/timeseries-partitiontimestamp.png" alt-text="Timestamp de partition":::


## <a name="create-dataset-monitors"></a>Créer des analyses de jeu de données

Créer des analyses de jeu de données pour détecter et alerter la dérive des données sur un nouveau jeu de données.  Pour ce faire, vous pouvez utiliser le [SDK Python](#sdk-monitor) ou [Azure Machine Learning Studio](#studio-monitor).

### <a name="python-sdk"></a><a name="sdk-monitor"></a>Kit SDK Python

La [documentation de référence du SDK python sur la dérive des données](/python/api/azureml-datadrift/azureml.datadrift) fournir des informations complètes sur le sujet. 

Voici un exemple de création d’un moniteur analyse de jeu de données à l’aide du SDK Python

```python
from azureml.core import Workspace, Dataset
from azureml.datadrift import DataDriftDetector
from datetime import datetime

# get the workspace object
ws = Workspace.from_config()

# get the target dataset
dset = Dataset.get_by_name(ws, 'target')

# set the baseline dataset
baseline = target.time_before(datetime(2019, 2, 1))

# set up feature list
features = ['latitude', 'longitude', 'elevation', 'windAngle', 'windSpeed', 'temperature', 'snowDepth', 'stationName', 'countryOrRegion']

# set up data drift detector
monitor = DataDriftDetector.create_from_datasets(ws, 'drift-monitor', baseline, target, 
                                                      compute_target='cpu-cluster', 
                                                      frequency='Week', 
                                                      feature_list=None, 
                                                      drift_threshold=.6, 
                                                      latency=24)

# get data drift detector by name
monitor = DataDriftDetector.get_by_name(ws, 'drift-monitor')

# update data drift detector
monitor = monitor.update(feature_list=features)

# run a backfill for January through May
backfill1 = monitor.backfill(datetime(2019, 1, 1), datetime(2019, 5, 1))

# run a backfill for May through today
backfill1 = monitor.backfill(datetime(2019, 5, 1), datetime.today())

# disable the pipeline schedule for the data drift detector
monitor = monitor.disable_schedule()

# enable the pipeline schedule for the data drift detector
monitor = monitor.enable_schedule()
```

Pour obtenir un exemple complet de configuration d’un jeu de données et d’un détecteur de dérive de données `timeseries`, consultez notre [exemple de notebook](https://aka.ms/datadrift-notebook).

### <a name="azure-machine-learning-studio"></a><a name="studio-monitor"></a>Azure Machine Learning Studio

1. Accédez à la [page d’accueil du studio](https://ml.azure.com).
1. Sélectionnez l’onglet **Jeux de données** à gauche. 
1. Sélectionnez **Analyses de jeu de données**.
   ![Liste des analyses](./media/how-to-monitor-datasets/monitor-list.png)

1. Cliquez sur le bouton **+Créer une analyse** , puis passez à la prochaine étape de l’assistant en cliquant sur **Suivant**.  

:::image type="content" source="media/how-to-monitor-datasets/wizard.png" alt-text="Créer un assistant d'analyse":::

* **Sélectionner un jeu de données cible**.  Le jeu de données cible est un jeu de données tabulaires avec colonne timestamp qui sera analysé pour la dérive des données. Le jeu de données cible doit avoir des caractéristiques en commun avec le jeu de données de base et il doit s’agir d’un jeu de données `timeseries` auquel de nouvelles données sont ajoutées. Les données d’historique peuvent être analysées dans le jeu de données cible ou de nouvelles données peuvent être surveillées.

* **Sélectionner un jeu de données de référence.**  Sélectionnez le jeu de données tabulaires qui sera utilisé comme ligne de base pour la comparaison du jeu de données cible au fil du temps.  Le jeu de données de référence doit avoir des caractéristiques communes avec le jeu de données cible.  Sélectionnez une plage de temps pour utiliser une tranche du jeu de données cible, ou spécifiez un jeu de données distinct à utiliser comme ligne de base.

* **Paramètres d’analyse**.  Ces paramètres sont associés au pipeline de moniteur de jeu de données planifié qui va être créé. 

    | Paramètre | Description | Conseils | Mutable | 
    | ------- | ----------- | ---- | ------- |
    | Nom | Nom de l’analyse du jeu de données | | Non |
    | Fonctionnalités | Liste des caractéristiques qui seront analysées pour la dérive des données au fil du temps. | À définir sur la (ou les) caractéristique(s) dont vous souhaitez mesurer la dérive de concept. N’incluez pas les caractéristiques qui dérivent naturellement au fil du temps (mois, année, index, etc.). Vous pouvez effectuer un renvoi et démarrer une analyse de dérive de données existant après avoir ajusté la liste des fonctionnalités. | Oui | 
    | Cible de calcul | Cible de calcul Azure Machine Learning pour exécuter les travaux de l’analyse de jeu de données. | | Oui | 
    | Activer | Activer ou désactiver la planification du pipeline de l’analyse de jeu de données | Désactivez la planification pour analyser les données d’historique avec le paramètre de renvoi. Elle peut être activée après la création de l’analyse de jeu de données. | Oui | 
    | Fréquence | Fréquence qui permettra de planifier le travail de pipeline et d’analyser les données d’historique si vous exécutez un renvoi. Les options disponibles sont quotidienne, hebdomadaire ou mensuelle. | Chaque exécution compare les données du jeu de données cible en fonction de la fréquence : <li>Quotidienne : Comparer le jour complet le plus récent dans le jeu de données cible avec la ligne de base <li>Hebdomadaire : Comparer la semaine complète la plus récente (Lundi-Dimanche) dans le jeu de données cible avec la ligne de base <li>Mensuelle : Comparer le mois complet le plus récent dans le jeu de données cible avec la ligne de base | Non | 
    | Latence | Durée (en heures) nécessaire pour que les données parviennent au jeu de données. Par exemple, s’il faut trois jours pour que les données parviennent dans la base de données SQL encapsulée par le jeu de données, définissez la latence sur 72. | N’est plus modifiable après la création de l’analyse de jeu de données | Non | 
    | Adresses e-mail | Adresses de messagerie pour les alertes en fonction de la violation du seuil de pourcentage de dérive des données. | Les e-mails sont envoyés via Azure Monitor. | Oui | 
    | Seuil | Seuil du pourcentage de dérive des données pour les alertes envoyées par e-mail. | Il est possible de définir d’autres alertes et événements sur de nombreuses autres métriques dans la ressource Application Insights associée de l’espace de travail. | Oui |

Une fois l’Assistant terminé, l’analyse du jeu de données résultant s’affiche dans la liste. Sélectionnez-la pour accéder à sa page d’informations détaillées.

## <a name="understand-data-drift-results"></a>Comprendre les résultats de la dérive des données

Cette section vous montre les résultats de la surveillance d’un jeu de données, qui se trouvent dans la page **Jeux de données** / **Analyseurs de jeu de données** dans Azure Studio.  Vous pouvez mettre à jour les paramètres et analyser les données existantes pour une période spécifique sur cette page.  

Commencez par des insights de haut niveau sur l'ampleur de la dérive des données et un aperçu des caractéristiques à étudier plus en détail.

:::image type="content" source="media/how-to-monitor-datasets/drift-overview.png" alt-text="Vue d’ensemble de la dérive":::


| Métrique | Description | 
| ------ | ----------- | 
| Ampleur de la dérive de données | Pourcentage entre la ligne de base et le jeu de données cible au fil du temps. Comprise entre 0 et 100, 0, où 0 indique des jeux de données identiques et 100 indique que la fonctionnalité de dérive des données Azure Machine Learning peut distinguer complètement les deux jeux de données. Le bruit est pris en compte dans le calcul du pourcentage de précision en raison des techniques Machine Learning utilisées pour cette amplitude. | 
| Caractéristiques ayant le plus dérivé | Liste les caractéristiques du jeu de données présentant la plus forte dérive et contribuant le plus à la métrique d'amplitude de la dérive. En raison de l’évolution de la covariable, la distribution sous-jacente d’une fonctionnalité n’a pas nécessairement besoin d’être modifiée pour avoir une importance relativement élevée. |
| Seuil | L’amplitude de la dérive des données au-delà du seuil défini déclenchera des alertes. Il est possible de configurer cette analyse de deux manières. | 

### <a name="drift-magnitude-trend"></a>Tendance de l'amplitude de la dérive

Découvrez comment le jeu de données diffère du jeu de données cible au cours de la période spécifiée.  Plus on se rapproche des 100 %, plus les deux jeux de données sont différents.

:::image type="content" source="media/how-to-monitor-datasets/drift-magnitude.png" alt-text="Tendance de l'amplitude de la dérive":::

### <a name="drift-magnitude-by-features"></a>Amplitude de la dérive par caractéristiques

Cette section contient un aperçu de la modification de la distribution de l'élément sélectionné, ainsi que d'autres statistiques, au fil du temps.

Le jeu de données cible est également profilé au fil du temps. La distance statistique entre la distribution de ligne de base de chaque fonctionnalité est comparée au fil du jeu de données cible.  Conceptuellement, cela est similaire à l’amplitude de la dérive des données.  Toutefois, cette distance statistique concerne une fonctionnalité individuelle plutôt que toutes les fonctionnalités. Les valeurs minimum, maximum et moyenne sont également disponibles.

Dans Azure Machine Learning Studio, cliquez sur une barre dans le graphique pour afficher les détails de niveau de fonctionnalité de cette date. Par défaut, il affiche la distribution du jeu de données de référence et la distribution de la même caractéristique issue de la dernière exécution.

:::image type="content" source="media/how-to-monitor-datasets/drift-by-feature.gif" alt-text="Amplitude de la dérive par caractéristiques":::

Ces métriques peuvent également être récupérées dans le SDK Python par le biais de la méthode `get_metrics()` sur un objet `DataDriftDetector`.

### <a name="feature-details"></a>Détails de la caractéristique

Enfin, faites défiler l’écran pour afficher les détails de chaque fonctionnalité.  Utilisez les listes déroulantes au-dessus du graphique pour sélectionner la fonctionnalité, puis sélectionnez la métrique que vous souhaitez afficher.

:::image type="content" source="media/how-to-monitor-datasets/numeric-feature.gif" alt-text="Graphique des fonctionnalités numériques et comparaison":::

Les mesures dans le graphique dépendent du type de fonctionnalité.

* Caractéristiques numériques

    | Métrique | Description |  
    | ------ | ----------- |  
    | Distance Wasserstein | Quantité minimale de travail nécessaire à la transformation de la distribution de référence en distribution cible. |
    | Valeur moyenne | Valeur moyenne de la caractéristique. |
    | Valeur minimale | Valeur minimale de la caractéristique. |
    | Valeur maximale | Valeur maximale de la caractéristique. |

* Caractéristiques par catégorie
    
    | Métrique | Description |  
    | ------ | ----------- |  
    | Distance euclidienne     |  Calculée pour les colonnes catégorielles.  La distance euclidienne est calculée sur deux vecteurs, générés à partir de la distribution empirique de la même colonne catégorielle à partir de deux jeux de données.  0 indique qu’il n’y a aucune différence dans les distributions empiriques.    Plus la valeur s’éloigne de 0, plus la colonne a dérivé.  Les tendances peuvent être observées à partir d’un tracé de série chronologique de cette mesure et peuvent être utiles pour dévoiler une fonction de dérive.  |
    | Valeurs uniques | Nombre de valeurs uniques (cardinalité) de la caractéristique. |

Sur ce graphique, sélectionnez une seule date pour comparer la répartition des caractéristiques entre la cible et cette date pour la caractéristique affichée. Pour les caractéristiques numériques, cela montre deux distributions de probabilité.  Si la fonctionnalité est numérique, un graphique à barres est affiché.

:::image type="content" source="media/how-to-monitor-datasets/select-date-to-compare.gif" alt-text="Sélectionner une date à comparer à la cible":::

## <a name="metrics-alerts-and-events"></a>Métriques, alertes et événements

Les métriques peuvent être interrogées dans la ressource [Azure Application Insights](../azure-monitor/app/app-insights-overview.md) associée à votre espace de travail Machine Learning, Vous avez accès à toutes les fonctionnalités d’Application Insights, notamment la configuration de règles d’alerte personnalisées et de groupes d’actions pour déclencher une action, comme une notification E-mail/SMS/Push/Voix ou une fonction Azure. Pour plus d’informations, reportez-vous à la documentation complète d’Application Insights. 

Pour commencer, accédez au [Portail Azure](https://portal.azure.com) et sélectionnez la page **Vue d’ensemble** de votre espace de travail.  La ressource Application Insights associée se trouve à l’extrême droite :

[![Vue d’ensemble du portail Azure](./media/how-to-monitor-datasets/ap-overview.png)](media/how-to-monitor-datasets/ap-overview-expanded.png)

Sélectionnez Journaux (analytique) sous Supervision dans le volet gauche :

![Vue d’ensemble d’Application Insights](./media/how-to-monitor-datasets/ai-overview.png)

Les métriques du superviseur de jeu de données sont stockées en tant que `customMetrics`. Vous pouvez écrire et exécuter une requête après avoir configuré un moniteur de jeu de données pour les visualiser :

[![Requête Log Analytics](./media/how-to-monitor-datasets/simple-query.png)](media/how-to-monitor-datasets/simple-query-expanded.png)

Après avoir identifié les métriques pour configurer des règles d’alerte, créez une règle d’alerte :

![Nouvelle règle d’alerte](./media/how-to-monitor-datasets/alert-rule.png)

Vous pouvez utiliser un groupe d’actions existant ou en créer un pour définir l’action à entreprendre quand les conditions définies sont remplies :

![Nouveau groupe d’actions](./media/how-to-monitor-datasets/action-group.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour configurer une analyse de jeu de données, accédez à [Azure Machine Learning studio](https://ml.azure.com) ou au [notebook Python](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datadrift-tutorial/datadrift-tutorial.ipynb).
* Découvrez comment configurer la dérive de données sur les [modèles déployés sur Azure Kubernetes Service](./how-to-enable-data-collection.md).
* Configurez des superviseurs de dérive de jeu de données avec [Event Grid](how-to-use-event-grid.md). 
* Consultez ces [conseils de dépannage](resource-known-issues.md#data-drift) courants si vous rencontrez des problèmes.