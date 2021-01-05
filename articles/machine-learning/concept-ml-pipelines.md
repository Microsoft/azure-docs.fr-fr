---
title: Présentation des pipelines Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Découvrez comment les pipelines Machine Learning peuvent vous aider à créer, optimiser et gérer vos workflows Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: laobri
author: lobrien
ms.date: 08/17/2020
ms.custom: devx-track-python
ms.openlocfilehash: c29ee87ab177357f4289134bb39353c764a0d75b
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94535297"
---
# <a name="what-are-azure-machine-learning-pipelines"></a>Présentation des pipelines Azure Machine Learning

Dans cet article, vous allez apprendre comment les pipelines Machine Learning peuvent vous aider à créer, optimiser et gérer vos workflows Machine Learning. Ces workflows présentent un certain nombre d’avantages : 

+ Simplicité
+ Vitesse
+ Répétabilité
+ Souplesse
+ Contrôle de version et suivi
+ Modularité 
+ Assurance qualité
+ Contrôle des coûts

Ces avantages deviennent significatifs dès que votre projet de Machine Learning passe au-delà de l’exploration pure pour aller vers l’itération. Même les pipelines en une étape simples peuvent être utiles. Les projets Machine Learning sont souvent dans un état complexe, et cela peut soulager de savoir que l’exécution précise d’un workflow unique peut être un processus trivial.

<a name="compare"></a>
### <a name="which-azure-pipeline-technology-should-i-use"></a>Quelle technologie de pipeline Azure dois-je utiliser ?

Le cloud Azure offre plusieurs autres pipelines, chacun ayant un objectif différent. Le tableau suivant liste les différents pipelines et une explication de leur utilisation :

| Scénario | Personnage principal | Offre Azure | Offre OSS | Canal canonique | Forces | 
| -------- | --------------- | -------------- | ------------ | -------------- | --------- | 
| Orchestration de modèle (Machine Learning) | Scientifique des données | Pipelines Azure Machine Learning | Pipelines Kubeflow | Données -> Modèle | Distribution, mise en cache, orienté code, réutilisation | 
| Orchestration de données (préparation des données) | Ingénierie de données | [Azure Data Factory](../data-factory/concepts-pipelines-activities.md) | Apache Airflow | Données -> Données | Mouvement fortement typé, activités centrées sur les données |
| Orchestration de code et d’application (CI/CD) | Développeur d’applications/opérations | [Azure DevOps Pipelines](https://azure.microsoft.com/services/devops/pipelines/) | Jenkins | Code + modèle -> application/service | Prise en charge de l’activité la plus ouverte et flexible, files d’attente d’approbation, phases avec vérification | 

## <a name="what-can-azure-ml-pipelines-do"></a>Que peuvent faire les pipelines Azure ML ?

Un pipeline Azure Machine Learning est un workflow exécutable indépendamment d’une tâche Machine Learning complète. Les sous-tâches sont encapsulées sous la forme d’une série d’étapes au sein du pipeline. Un pipeline Azure Machine Learning peut être aussi simple qu’un pipeline qui appelle un script Python, de sorte qu’il _permet_ de faire à peu près n’importe quoi. Les pipelines _doivent_ se concentrer sur les tâches Machine Learning telles que :

+ Préparation des données, notamment l’importation, la validation et le nettoyage, la description et la transformation, la normalisation et la mise en lots
+ Configuration de la formation, notamment le paramétrage des arguments, les filePaths et les configurations de journalisation/création de rapports
+ Formation et validation efficaces et répétées. L’efficacité peut provenir de la spécification de sous-ensembles de données spécifiques, de différentes ressources de calcul matériel, du traitement distribué et de la surveillance de la progression
+ Déploiement, y compris le contrôle de version, la mise à l’échelle, l’approvisionnement et le contrôle d’accès

Ces étapes indépendantes permettent à plusieurs scientifiques de données de travailler en même temps sur le même pipeline sans surcharger les ressources de calcul. Des étapes distinctes facilitent également l'utilisation de différents types/tailles de calcul pour chaque étape.

Une fois conçu, le pipeline fait souvent l’objet de réglages précis au cours du cycle d’entraînement du pipeline. Quand vous réexécutez un pipeline, l’exécution passe directement aux étapes qui doivent être réexécutées, par exemple un script d’entraînement mis à jour. Les étapes qui n’ont pas besoin d’être réexécutées sont ignorées. 

Avec les pipelines, vous pouvez choisir d’utiliser un matériel différent pour différentes tâches. Azure effectue la coordination des différentes [cibles de calcul](concept-azure-machine-learning-architecture.md) utilisées afin que les données intermédiaires soient transmises de manière uniforme aux cibles de calcul en aval.

Vous pouvez [suivre les métrique de vos expériences de pipeline](./how-to-track-experiments.md) directement dans le Portail Azure ou dans la [page d’arrivée de votre espace de travail (préversion)](https://ml.azure.com). Après la publication d'un pipeline, vous pouvez configurer un point de terminaison REST qui vous permet de réexécuter le pipeline à partir de n'importe quelle plateforme ou pile.

En résumé, toutes les tâches complexes du cycle de vie Machine Learning peuvent être facilitées avec les pipelines. D’autres technologies de pipeline Azure ont leurs propres atouts. Les [pipelines Azure Data Factory](../data-factory/concepts-pipelines-activities.md) conviennent parfaitement pour exploiter les données, tandis qu’[Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) est l’outil approprié pour l’intégration et le déploiement continus. Toutefois, si votre objectif est le Machine Learning, les pipelines Azure Machine Learning sont susceptibles d’être le meilleur choix pour vos besoins en matière de workflow. 

### <a name="analyzing-dependencies"></a>Analyse des dépendances

De nombreux écosystèmes de programmation possèdent des outils qui orchestrent les dépendances de ressources, de bibliothèques ou de compilation. En règle générale, ces outils utilisent des timestamps de fichier pour calculer les dépendances. Lorsqu’un fichier est modifié, seul celui-ci et ses dépendants sont mis à jour (téléchargés, recompilés ou empaquetés). Les pipelines Azure ML étendent ce concept. Comme les outils de compilation traditionnels, les pipelines calculent les dépendances entre les étapes et effectuent uniquement les recalculs nécessaires. 

Toutefois, l’analyse des dépendances dans les pipelines Azure ML est plus sophistiquée que les simples timestamps. Chaque étape peut s’exécuter dans un environnement matériel et logiciel différent. La préparation des données peut être un processus long, mais il n’est pas nécessaire de l’exécuter sur du matériel avec des GPU puissantes, certaines étapes peuvent nécessiter des logiciels spécifiques au système d’exploitation, vous pouvez utiliser la formation distribuée, etc. 

Azure Machine Learning orchestre automatiquement toutes les dépendances entre les étapes de pipeline. Cette orchestration peut inclure l’ajout et la suppression d’images Docker, l’attachement et le détachement de ressources de calcul et le déplacement de données entre les étapes de manière cohérente et automatique.

### <a name="coordinating-the-steps-involved"></a>Coordination des étapes impliquées

Lorsque vous créez et exécutez un objet `Pipeline`, les étapes de haut niveau suivantes se produisent :

+ Pour chaque étape, le service calcule les conditions requises pour :
    + Ressources de calcul matérielles
    + Ressources du système d’exploitation (image(s) Docker)
    + Ressources logicielles (dépendances Conda / virtualenv)
    + Entrées de données 
+ Le service détermine les dépendances entre les étapes, ce qui aboutit à un graphique d’exécution dynamique
+ Quand chaque nœud dans le graphique d’exécution s’exécute :
    + Le service configure l’environnement matériel et logiciel requis (en réutilisant peut-être les ressources existantes)
    + L’étape s’exécute, en fournissant des informations de journalisation et de surveillance à l’objet `Experiment` qui les contient
    + Lorsque l’étape est terminée, ses sorties sont préparées comme entrées à l’étape suivante et/ou écrites dans le stockage
    + Les ressources qui ne sont plus nécessaires sont finalisées et détachées

![Étapes de pipeline](./media/concept-ml-pipelines/run_an_experiment_as_a_pipeline.png)

## <a name="building-pipelines-with-the-python-sdk"></a>Création de pipelines avec le kit de développement logiciel (SDK) Python

Dans le [Kit de développement logiciel (SDK) Azure Machine Learning Python ](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py), un pipeline est un objet Python défini dans le module `azureml.pipeline.core`. Un objet [Pipeline](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline%28class%29?preserve-view=true&view=azure-ml-py) contient une séquence ordonnée d’un ou plusieurs objets [PipelineStep](/python/api/azureml-pipeline-core/azureml.pipeline.core.builder.pipelinestep?preserve-view=true&view=azure-ml-py). La classe `PipelineStep` est abstraite et les étapes réelles sont des sous-classes telles que [EstimatorStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.estimatorstep?preserve-view=true&view=azure-ml-py), [PythonScriptStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.pythonscriptstep?preserve-view=true&view=azure-ml-py) ou [DataTransferStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep?preserve-view=true&view=azure-ml-py). La classe [ModuleStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.modulestep?preserve-view=true&view=azure-ml-py) contient une séquence réutilisable d’étapes qui peuvent être partagées entre les pipelines. Un `Pipeline` s’exécute dans le cadre d’une `Experiment`.

Un pipeline Azure ML est associé à un espace de travail Azure Machine Learning et une étape de pipeline est associée à une cible de calcul disponible dans cet espace de travail. Pour plus d’informations, consultez [Créer et gérer des espaces de travail Azure Machine Learning dans le Portail Azure](./how-to-manage-workspace.md) ou [Qu’est-ce qu’une cible de calcul dans Azure Machine Learning ?](./concept-compute-target.md).

### <a name="a-simple-python-pipeline"></a>Un pipeline Python simple

Cet extrait de code montre les objets et les appels nécessaires à la création et à l’exécution d’un `Pipeline` :

```python
ws = Workspace.from_config() 
blob_store = Datastore(ws, "workspaceblobstore")
compute_target = ws.compute_targets["STANDARD_NC6"]
experiment = Experiment(ws, 'MyExperiment') 

input_data = Dataset.File.from_files(
    DataPath(datastore, '20newsgroups/20news.pkl'))

output_data = PipelineData("output_data", datastore=blob_store)

input_named = input_data.as_named_input('input')

steps = [ PythonScriptStep(
    script_name="train.py",
    arguments=["--input", input_named.as_download(), "--output", output_data],
    inputs=[input_data],
    outputs=[output_data],
    compute_target=compute_target,
    source_directory="myfolder"
) ]

pipeline = Pipeline(workspace=ws, steps=steps)

pipeline_run = experiment.submit(pipeline)
pipeline_run.wait_for_completion()
```

L’extrait de code commence par les objets Azure Machine Learning communs, un `Workspace`, un `Datastore`, un [ComputeTarget](/python/api/azureml-core/azureml.core.computetarget?preserve-view=true&view=azure-ml-py) et un `Experiment`. Ensuite, le code crée les objets pour contenir `input_data` et `output_data`. Le tableau `steps` contient un élément unique, un `PythonScriptStep` qui utilisera les objets de données et s’exécutera sur le `compute_target`. Ensuite, le code instancie l’objet `Pipeline` lui-même, en transmettant l’espace de travail et le tableau d’étapes. L’appel à `experiment.submit(pipeline)` commence l’exécution du pipeline Azure ML. L’appel à `wait_for_completion()` se bloque jusqu’à ce que le pipeline soit terminé. 

Pour en savoir plus sur la connexion de votre pipeline à vos données, consultez les articles [Accès aux données dans Azure Machine Learning](concept-data.md) et [Déplacement de données au sein d’un pipeline ML et vers un autre pipeline ML (Python)](how-to-move-data-in-out-of-pipelines.md). 

## <a name="building-pipelines-with-the-designer"></a>Création de pipelines avec le concepteur

Les développeurs qui préfèrent une aire de conception visuelle peuvent utiliser le concepteur Azure Machine Learning pour créer des pipelines. Vous pouvez accéder à cet outil à partir de la sélection **Concepteur** sur la page d’accueil de votre espace de travail.  Le concepteur vous permet de glisser-déplacer des étapes sur l’aire de conception. 

Lorsque vous concevez visuellement des pipelines, les entrées et sorties d’une étape sont visibles. Vous pouvez faire glisser et déposer des connexions de données, ce qui vous permet de comprendre et de modifier rapidement le flux de données de votre pipeline.

![Exemple de concepteur Azure Machine Learning](./media/concept-designer/designer-drag-and-drop.gif)

## <a name="key-advantages"></a>Principaux avantages

Voici les principaux avantages de l’utilisation des pipelines pour des workflows de Machine Learning :

|Avantage principal|Description|
|:-------:|-----------|
|**Exécutions sans&nbsp; assistance**|Planifiez des étapes à exécuter en parallèle ou en séquence de façon fiable et sans assistance. La préparation et la modélisation des données peuvent prendre des jours voire des semaines, et les pipelines vous permettent de vous concentrer sur d’autres tâches pendant l’exécution du processus. |
|**Calcul hétérogène**|Utilisez plusieurs pipelines qui sont coordonnés de façon fiable entre des ressources de calcul et des emplacements de stockage hétérogènes et évolutifs. Utilisez efficacement les ressources de calcul disponibles en exécutant des étapes de pipeline individuelles sur différentes cibles de calcul, comme HDInsight, des machines virtuelles DSVM sur GPU et Databricks.|
|**Réutilisabilité**|Créez des modèles de pipeline pour des scénarios précis, comme le réentraînement et le scoring par lots. Déclenchez des pipelines publiés à partir de systèmes externes par de simples appels REST.|
|**Suivi et gestion de version**|Au lieu de suivre manuellement les données et les chemins d’accès aux résultats au fil des itérations, utilisez le kit SDK Pipelines pour nommer vos sources de données, vos entrées et vos sorties et en contrôler les versions. Vous pouvez aussi gérer séparément les scripts et les données pour augmenter la productivité.|
| **Modularité** | La séparation des sujets de préoccupation et l’isolation des modifications permet aux logiciels d’évoluer plus rapidement avec une qualité supérieure. | 
|**Collaboration**|Les pipelines permettent aux scientifiques des données de collaborer dans toutes les phases du processus de conception du Machine Learning tout en travaillant simultanément sur les étapes de pipeline.|

## <a name="next-steps"></a>Étapes suivantes

Les pipelines Azure ML sont une fonctionnalité puissante qui commence à générer de la valeur lors des premières étapes du développement. La valeur augmente à mesure que l’équipe et le projet s’étendent. Cet article a expliqué comment les pipelines sont spécifiés avec le Kit de développement logiciel (SDK) Azure Machine Learning Python et orchestrés sur Azure. Vous avez vu du code source simple et vous avez bénéficié d’une introduction à quelques-unes des classes `PipelineStep` disponibles. Vous devriez à présent savoir quand utiliser les pipelines Azure ML et comment Azure les exécute. 


+ Découvrez comment [créer votre premier pipeline](how-to-create-your-first-pipeline.md).

+ Découvrez comment [exécuter des prédictions par lots sur de grandes quantités de données](tutorial-pipeline-batch-scoring-classification.md ).

+ Consultez les documents de référence du Kit de développement logiciel (SDK) pour le [cœur du pipeline](/python/api/azureml-pipeline-core/?preserve-view=true&view=azure-ml-py) et les [étapes du pipeline](/python/api/azureml-pipeline-steps/?preserve-view=true&view=azure-ml-py).

+ Testez les exemples de notebooks Jupyter illustrant des [pipelines Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines). Découvrez comment [exécuter des notebooks pour explorer ce service](samples-notebooks.md).