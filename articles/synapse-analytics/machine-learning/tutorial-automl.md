---
title: 'Tutoriel : Entraîner un modèle avec le Machine Learning automatisé'
description: Tutoriel sur la façon d’entraîner un modèle Machine Learning sans code dans Azure Synapse à l’aide d’Apache Spark et du Machine Learning automatisé.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 11/20/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: 4967d5305b4b438f3baa6fca078d7b3169612590
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093398"
---
# <a name="tutorial-train-a-machine-learning-model-code-free-in-azure-synapse-with-apache-spark-and-automated-ml"></a>Tutoriel : Entraîner un modèle Machine Learning sans code dans Azure Synapse à l’aide d’Apache Spark et du Machine Learning automatisé

Découvrez comment facilement enrichir vos données dans des tables Spark avec de nouveaux modèles Machine Learning que vous entraînez à l’aide du [Machine Learning automatisé dans Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-automated-ml).  Dans Synapse, un utilisateur peut simplement sélectionner une table Spark dans l’espace de travail Azure Synapse à utiliser comme jeu de données d’entraînement pour la génération de modèles Machine Learning dans une expérience sans code.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> - Entraîner des modèles Machine Learning à l’aide d’une expérience sans code dans Azure Synapse Studio qui utilise le Machine Learning automatisé dans Azure Machine Learning. Le type de modèle que vous entraînez dépend du problème que vous essayez de résoudre.

Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit avant de commencer](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Prérequis

- [Espace de travail Synapse Analytics](../get-started-create-workspace.md) avec un compte de stockage ADLS Gen2 configuré comme stockage par défaut. Vous devez être le **contributeur de données Blob du stockage** du système de fichiers ADLS Gen2 que vous utilisez.
- Pool Spark dans votre espace de travail Azure Synapse Analytics. Pour plus d’informations, consultez [Créer un pool Spark dans Azure Synapse](../quickstart-create-sql-pool-studio.md).
- Service lié Azure Machine Learning dans votre espace de travail Azure Synapse Analytics. Pour plus de détails, consultez [Créer un service lié Azure Machine Learning dans Azure Synapse](quickstart-integrate-azure-machine-learning.md).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/)

## <a name="create-a-spark-table-for-training-dataset"></a>Créer une table Spark pour un jeu de données d’entraînement

Vous aurez besoin d’une table Spark pour ce tutoriel. Le notebook suivant va créer une table Spark.

1. Téléchargez le notebook [Create-Spark-Table-NYCTaxi- Data.ipynb](https://go.microsoft.com/fwlink/?linkid=2149229).

1. Importez le notebook dans Azure Synapse Studio.
![Importer un notebook](media/tutorial-automl-wizard/tutorial-automl-wizard-00a.png)

1. Sélectionnez le pool Spark que vous voulez utiliser, puis cliquez sur `Run all`. Exécuter ce notebook permettra d’obtenir les données des obtiendra des données sur les taxis de New York à partir d’un jeu de données ouvert et les enregistrera dans votre base de données Spark par défaut.
![Exécuter tout](media/tutorial-automl-wizard/tutorial-automl-wizard-00b.png)

1. Une fois l’exécution du notebook terminée, une nouvelle table Spark est créée sous la base de données Spark par défaut. Accédez au hub de données, puis recherchez la table nommée avec `nyc_taxi`.
![Tables Spark](media/tutorial-automl-wizard/tutorial-automl-wizard-00c.png)

## <a name="launch-automated-ml-wizard-to-train-a-model"></a>Lancer l’Assistant Machine Learning automatisé pour entraîner un modèle

Cliquez avec le bouton droit sur la table Spark créée à l’étape précédente. Sélectionnez « Machine Learning -> Enrichir avec un nouveau modèle » pour ouvrir l’Assistant.
![Lancer l’Assistant Machine Learning automatisé](media/tutorial-automl-wizard/tutorial-automl-wizard-00d.png)

Un panneau de configuration s’affiche et vous êtes invité à fournir des détails de configuration pour créer une exécution d’expérience de Machine Learning automatisé dans Azure Machine Learning. Cette exécution entraîne plusieurs modèles et le meilleur modèle d’une exécution réussie est inscrit dans le registre de modèles Azure Machine Learning :

![Configurer l’exécution - Étape 1](media/tutorial-automl-wizard/tutorial-automl-wizard-configure-run-00a.png)

- **Espace de travail Azure Machine Learning** : un espace de travail Azure Machine Learning est nécessaire pour la création de l’exécution de l’expérience de Machine Learning automatisé. Vous devez également lier votre espace de travail Azure Synapse à l’espace de travail Azure Machine Learning à l’aide d’un [service lié](quickstart-integrate-azure-machine-learning.md). Une fois tous les prérequis remplis, vous pouvez spécifier l’espace de travail Azure Machine Learning que vous voulez utiliser pour cette exécution de Machine Learning automatisé.

- **Nom de l’expérience** : spécifiez le nom de l’expérience. Quand vous soumettez une exécution de Machine Learning automatisé, vous fournissez un nom d’expérience. Les informations concernant l’exécution sont stockées sous cette expérience dans l’espace de travail Azure Machine Learning. Cette expérience crée par défaut une expérience et génère un nom proposé, mais vous pouvez également fournir le nom d’une expérience existante.

- **Meilleur modèle** : spécifiez le nom du meilleur modèle de l’exécution de Machine Learning automatisé. Ce nom est attribué au meilleur modèle qui est automatiquement enregistré dans le registre de modèles Azure Machine Learning après cette exécution. Une exécution de Machine Learning automatisé crée de nombreux modèles Machine Learning. En fonction de la métrique principale que vous allez sélectionner dans une étape ultérieure, ces modèles peuvent être comparés et le meilleur modèle peut être sélectionné.

- **Colonne cible** : il s’agit de ce que le modèle est entraîné à prédire. Choisissez la colonne que vous voulez prédire.

- **Pool Spark** : pool Spark que vous voulez utiliser pour l’exécution de l’expérience de Machine Learning automatisé. Les calculs sont exécutés sur le pool que vous spécifiez.

- **Détails de configuration Spark** : en plus du pool Spark, vous avez également la possibilité de fournir des détails sur la configuration de session.

Dans ce tutoriel, nous allons sélectionner la colonne numérique `fareAmount` comme colonne cible.

Cliquez sur « Continuer ».

## <a name="choose-task-type"></a>Choisir le type de tâche

Sélectionnez le type de modèle Machine Learning pour l’expérience en fonction de la question à laquelle vous tentez de répondre. Étant donné que nous avons sélectionné `fareAmount` comme colonne cible et qu’il s’agit d’une valeur numérique, nous allons sélectionner *Régression*.

Cliquez sur « Continuer » pour configurer des paramètres supplémentaires.

![Sélection du type de tâche](media/tutorial-automl-wizard/tutorial-automl-wizard-configure-run-00b.png)

## <a name="additional-configurations"></a>Configurations supplémentaires

Si vous sélectionnez le type *Classification* ou *Régression*, les configurations supplémentaires sont les suivantes :

- **Métrique principale** : Métrique utilisée pour mesurer les performances du modèle. Il s’agit de la métrique qui sera utilisée pour comparer les différents modèles créés lors de l’exécution de Machine Learning automatisé et déterminer quel modèle a le mieux fonctionné.

- **Durée du travail de formation (heures)**  : durée maximale, en heures, pour l’exécution et l’entraînement des modèles lors d’une expérience. Notez que vous pouvez également fournir des valeurs inférieures à 1. Par exemple, `0.5`.

- **Nombre maximal d'itérations simultanées** : Représente le nombre maximal d’itérations exécutées en parallèle.

- **Compatibilité du modèle ONNX** : Si cette option est activée, les modèles entraînés par le Machine Learning automatisé sont convertis au format ONNX. Cela s’avère particulièrement utile si vous voulez utiliser le modèle pour le scoring dans des pools SQL Azure Synapse.

Ces paramètres ont tous une valeur par défaut que vous pouvez personnaliser.
![Configurations supplémentaires](media/tutorial-automl-wizard/tutorial-automl-wizard-configure-run-00c.png)

> Notez que si vous sélectionnez l’option « Prévision de séries chronologiques », des configurations supplémentaires sont nécessaires. Les prévisions ne prennent pas non plus en charge la compatibilité du modèle ONNX.

Une fois toutes les configurations obligatoires effectuées, vous pouvez démarrer l’exécution du Machine Learning automatisé.

Il existe deux façons de démarrer une exécution du Machine Learning automatisé dans Azure Synapse. Pour une expérience sans code, vous pouvez choisir de **créer une exécution** directement. Si vous préférez du code, vous pouvez sélectionner **Ouvrir dans un notebook**, ce qui vous permet de voir le code qui crée l’exécution et d’exécuter le notebook.

### <a name="create-run-directly"></a>Créer directement une exécution

Cliquez sur « Démarrer l’exécution » pour démarrer directement l’exécution de Machine Learning automatisé. Une notification indique que l’exécution de Machine Learning automatisé démarre.

Une fois que l’exécution de Machine Learning automatisé a démarré, une autre notification de réussite s’affiche. Vous pouvez également cliquer sur le bouton de notification pour vérifier l’état de l’envoi de l’exécution.
Azure Machine Learning en cliquant sur le lien dans la notification de réussite.
![Notification de réussite](media/tutorial-automl-wizard/tutorial-automl-wizard-configure-run-00d.png)

### <a name="create-run-with-notebook"></a>Créer une exécution avec un notebook

Sélectionnez *Ouvrir dans un notebook* pour générer un notebook. Cliquez sur *Exécuter tout* pour exécuter le notebook.
Cela vous donne également la possibilité d’ajouter des paramètres supplémentaires à votre exécution de Machine Learning automatisé.

![Ouvrir un notebook](media/tutorial-automl-wizard/tutorial-automl-wizard-configure-run-00e.png)

Une fois l’exécution du notebook correctement envoyée, un lien vers l’exécution de l’expérience s’affiche dans l’espace de travail Azure Machine Learning, dans la sortie du notebook. Vous pouvez cliquer sur ce lien pour superviser votre exécution du Machine Learning automatisé dans Azure Machine Learning.
![Notebook - Exécuter tout](media/tutorial-automl-wizard/tutorial-automl-wizard-configure-run-00f.png))

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Scoring de modèle Machine Learning dans des pools SQL dédiés Azure Synapse](tutorial-sql-pool-model-scoring-wizard.md).
- [Démarrage rapide : Créer un service lié Azure Machine Learning dans Azure Synapse](quickstart-integrate-azure-machine-learning.md)
- [Fonctionnalités de Machine Learning dans Azure Synapse Analytics](what-is-machine-learning.md)
