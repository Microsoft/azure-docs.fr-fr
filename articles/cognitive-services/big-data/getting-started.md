---
title: Bien démarrer avec Cognitive Services pour le Big Data
description: Configurez votre pipeline MMLSpark avec Cognitive Services dans Azure Databricks et exécutez un exemple.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.topic: quickstart
ms.date: 07/06/2020
ms.author: marhamil
ms.openlocfilehash: 095f2c3ed17042bb616fb091d1af52a64c913709
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460495"
---
# <a name="getting-started"></a>Bien démarrer

La configuration de votre environnement est la première étape de la création d’un pipeline pour vos données. Une fois que votre environnement est prêt, l’exécution d’un exemple est simple et rapide.

Dans cet article, nous allons effectuer les étapes suivantes pour vous aider à démarrer :

1. [Créer une ressource Cognitive Services](#create-a-cognitive-services-resource)
1. [Créer un cluster Apache Spark](#create-an-apache-spark-cluster)
1. [Essayer un exemple](#try-a-sample)

## <a name="create-a-cognitive-services-resource"></a>Créer une ressource Cognitive Services

Pour utiliser Cognitive Services pour le Big Data, nous devons d’abord créer un service cognitif pour notre workflow. Il existe deux principaux types de services cognitifs : les services cloud hébergés dans Azure et les services conteneurisés gérés par les utilisateurs. Nous vous recommandons de commencer par les services cognitifs basée sur le cloud, qui sont plus simples.

### <a name="cloud-services"></a>Services cloud

Les services cognitifs basés sur le cloud sont des algorithmes intelligents hébergés dans Azure. Ces services sont prêts à l’emploi sans entraînement ; il vous suffit d’une connexion Internet. Vous pouvez [créer un service cognitif dans le portail Azure](../cognitive-services-apis-create-account.md?tabs=multiservice%2Cwindows) ou avec [Azure CLI](../cognitive-services-apis-create-account-cli.md?tabs=windows).

### <a name="containerized-services-optional"></a>Services conteneurisés (facultatif)

Si votre application ou charge de travail utilise des jeux de données volumineux, nécessite un réseau privé ou ne peut pas contacter le cloud, la communication avec les services cloud risque d’être impossible. Dans ce cas, les services cognitifs conteneurisés présentent les avantages suivants :

* **Faible connectivité** : vous pouvez déployer des services cognitifs conteneurisés dans n’importe quel environnement informatique, à la fois dans le cloud et hors cloud. Si votre application ne peut pas contacter le cloud, il peut être préférable de déployer des services cognitifs conteneurisés sur votre application.

* **Faible latence** : les services conteneurisés ne nécessitent pas la communication aller-retour vers/à partir du cloud, les réponses sont renvoyées avec des latences beaucoup plus faibles.

* **Confidentialité et sécurité des données** : vous pouvez déployer des services conteneurisés sur des réseaux privés, afin que les données sensibles ne quittent pas le réseau.

* **Scalabilité élevée** : les services conteneurisés n’ont pas de « limites de débit » et s’exécutent sur des ordinateurs gérés par l’utilisateur. Ainsi, vous pouvez mettre à l’échelle les services cognitifs comme bon vous semble afin de gérer des charges de travail plus volumineuses.

Suivez [ce guide](../cognitive-services-container-support.md?tabs=luis) pour créer un service cognitif conteneurisé.

## <a name="create-an-apache-spark-cluster"></a>Créer un cluster Apache Spark

[Apache Spark&trade;](http://spark.apache.org/) est une infrastructure informatique distribuée conçue pour le traitement des données Big Data. Les utilisateurs peuvent travailler avec Apache Spark dans Azure avec des services comme Azure Databricks, Azure Synapse Analytics, HDInsight et Azure Kubernetes Services. Pour utiliser Cognitive Services pour le Big Data, nous devons d’abord créer un cluster. Si vous disposez déjà d’un cluster Spark, n’hésitez pas à essayer un exemple.

### <a name="azure-databricks"></a>Azure Databricks

Azure Databricks est une plateforme d’analytique basée sur Apache Spark offrant une configuration en un clic, des workflows simplifiés et un espace de travail interactif. Elle est souvent utilisée pour la collaboration entre scientifiques de données, ingénieurs et analystes d’entreprise. Pour utiliser Cognitive Services pour le Big Data sur Azure Databricks, effectuez les étapes suivantes :

1. [Créer un espace de travail Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal#create-an-azure-databricks-workspace)
1. [Créer un cluster Spark dans Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal#create-a-spark-cluster-in-databricks)
1. Installer Cognitive Services pour le Big Data
    * Créez une bibliothèque dans votre espace de travail databricks.  
       <img src="media/create-library.png" alt="Create library" width="50%"/>
    * Entrez les coordonnées Maven suivantes :  `com.microsoft.ml.spark:mmlspark_2.11:1.0.0-rc3` Référentiel : `https://mmlspark.azureedge.net/maven`  
      <img src="media/library-coordinates.png" alt="Library Coordinates" width="50%"/>
    * Installez la bibliothèque sur un cluster  
      <img src="media/install-library.png" alt="Install Library on Cluster" width="50%"/>

### <a name="azure-synapse-analytics-optional"></a>Azure Synapse Analytics (facultatif)

Si vous le souhaitez, vous pouvez utiliser Synapse Analytics pour créer un cluster Spark. Azure Synapse Analytics regroupe l’entreposage des données d’entreprise et l’analytique de Big Data. Il vous donne la possibilité d’interroger les données avec votre propre vocabulaire, en utilisant des ressources serverless à la demande ou des ressources provisionnées, le tout à grande échelle. Pour bien démarrer avec Azure Synapse Analytics, effectuez les étapes suivantes :

1. [Créer un espace de travail Synapse (préversion)](../../synapse-analytics/quickstart-create-workspace.md).
1. [Créer un pool Apache Spark serverless (préversion) avec le portail Azure](../../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).

Dans Azure Synapse Analytics, Cognitive Services pour le Big Data est installé par défaut.

### <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Si vous utilisez des services cognitifs conteneurisés, l’une des options répandues pour déployer Spark avec des conteneurs est Azure Kubernetes Service.

Pour bien démarrer avec Azure Kubernetes Service, effectuez les étapes suivantes :

1. [Déployer un cluster AKS (Azure Kubernetes Service) à l’aide du portail Azure](../../aks/kubernetes-walkthrough-portal.md)
1. [Installer le chart Helm Apache Spark 2.4.0](https://hub.helm.sh/charts/microsoft/spark)
1. [Installer un conteneur de service cognitif à l’aide de Helm](../computer-vision/deploy-computer-vision-on-premises.md)

## <a name="try-a-sample"></a>Essayer un exemple

Une fois que vous avez configuré votre cluster Spark et votre environnement, vous pouvez exécuter un court exemple. Cette section montre comment utiliser Cognitive Services pour le Big Data dans Azure Databricks.

Tout d’abord, nous pouvons créer un notebook dans Azure Databricks. Pour les autres fournisseurs de cluster Spark, utilisez leurs notebooks ou Spark Submit.

1. Créez un notebook Databricks en choisissant **Nouveau notebook** dans le menu **Azure Databricks**.

    <img src="media/new-notebook.png" alt="Create a new notebook" width="50%"/>

1. Dans la boîte de dialogue **Créer un notebook**, entrez un nom, sélectionnez **Python** comme langage, puis sélectionnez le cluster Spark que vous avez créé précédemment.

    <img src="media/databricks-notebook-details.jpg" alt="New notebook details" width="50%"/>

    Sélectionnez **Create** (Créer).

1. Collez cet extrait de code dans votre nouveau notebook.

```python
from mmlspark.cognitive import *
from pyspark.sql.functions import col

# Add your subscription key from Text Analytics (or a general Cognitive Service key)
service_key = "ADD-SUBSCRIPTION-KEY-HERE"

df = spark.createDataFrame([
  ("I am so happy today, its sunny!", "en-US"),
  ("I am frustrated by this rush hour traffic", "en-US"),
  ("The cognitive services on spark aint bad", "en-US"),
], ["text", "language"])

sentiment = (TextSentiment()
    .setTextCol("text")
    .setLocation("eastus")
    .setSubscriptionKey(service_key)
    .setOutputCol("sentiment")
    .setErrorCol("error")
    .setLanguageCol("language"))

results = sentiment.transform(df)

# Show the results in a table
display(results.select("text", col("sentiment")[0].getItem("score").alias("sentiment")))

```

1. Récupérez votre clé d’abonnement à partir du menu **Clés et point de terminaison** de votre tableau de bord Analyse de texte dans le portail Azure.
1. Remplacez l’espace réservé de la clé d’abonnement dans le code de votre notebook Databricks par votre clé d’abonnement.
1. Sélectionnez le symbole de lecture (triangle) en haut à droite de la cellule de votre notebook pour exécuter l’exemple. Si vous le souhaitez, sélectionnez **Exécuter tout** en haut de votre notebook pour exécuter toutes les cellules. Les réponses s’afficheront sous la cellule dans un tableau.

### <a name="expected-results"></a>Résultats attendus

| texte                                      |   sentiment |
|:------------------------------------------|------------:|
| I am so happy today, its sunny! (Je suis trop content aujourd’hui, il fait beau !)           |   0,978959  |
| I am frustrated by this rush hour traffic (Cette circulation aux heures de pointe m’agace) |   0,0237956 |
| The cognitive services on spark aint bad (Les services Cognitive Services sur Spark ne sont pas mal)  |   0,888896  |

## <a name="next-steps"></a>Étapes suivantes

- [Courts exemples Python](samples-python.md)
- [Courts exemples Scala](samples-scala.md)
- [Recette : Maintenance prédictive](recipes/anomaly-detection.md)
- [Recette : Exploration intelligente d’images](recipes/art-explorer.md)