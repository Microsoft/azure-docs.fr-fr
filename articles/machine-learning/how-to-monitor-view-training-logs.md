---
title: Surveiller et consulter les journaux d’exécution et les métriques de Machine Learning
titleSuffix: Azure Machine Learning
description: Surveillez vos expériences ML et consultez les métriques d’exécution avec les widgets Jupyter et Azure Machine Learning Studio.
services: machine-learning
author: likebupt
ms.author: keli19
ms.reviewer: peterlu
ms.service: machine-learning
ms.subservice: core
ms.date: 07/30/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 845160d92100a27c32f16eddcd1f36e9e8624e80
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360596"
---
# <a name="monitor-and-view-ml-run-logs-and-metrics"></a>Surveiller et consulter les journaux d’exécution et les métriques de Machine Learning

Dans cet article, vous allez apprendre à surveiller les exécutions d’Azure Machine Learning et à en consulter les journaux. Avant de pouvoir afficher les journaux, vous devez les activer. Pour plus d’informations, consultez [Activer la journalisation dans les exécutions de formation d’Azure ML](how-to-track-experiments.md).

Les journaux peuvent vous aider à diagnostiquer les erreurs et les avertissements, ou à effectuer le suivi des métriques de performances, telles que les paramètres et la précision du modèle. Cet article vous montre comment consulter les journaux à l’aide des méthodes suivantes :

> [!div class="checklist"]
> * Surveiller les exécutions dans le studio
> * Surveiller les exécutions avec le widget Jupyter Notebook
> * Surveiller les exécutions de Machine Learning automatisé
> * Afficher les journaux de sortie une fois l’opération terminée
> * Afficher les journaux de sortie dans le studio

Pour obtenir des informations générales sur la gestion de vos expériences, consultez [Démarrer, surveiller et annuler des exécutions de formation](how-to-manage-runs.md).

## <a name="monitor-runs-in-the-studio"></a>Surveiller les exécutions dans le studio

Pour surveiller les exécutions d’une cible de calcul spécifique dans votre navigateur, procédez comme suit :

1. Dans le [studio Azure Machine Learning](https://ml.azure.com/), sélectionnez votre espace de travail, puis sélectionnez __Calcul__ sur le côté gauche de la page.

1. Sélectionnez __Clusters d’entraînement__ pour afficher la liste des cibles de calcul utilisées pour l’entraînement. Ensuite, sélectionnez le cluster.

    ![Sélectionner le cluster d’entraînement](./media/how-to-track-experiments/select-training-compute.png)

1. Sélectionnez __Exécutions__. La liste des exécutions qui utilisent ce cluster s’affiche. Pour voir les détails d’une exécution, utilisez le lien qui se trouve dans la colonne __Exécution__. Pour voir les détails d’une expérience, utilisez le lien qui se trouve dans la colonne __Expérience__.

    ![Sélectionner des exécutions pour le cluster d’entraînement](./media/how-to-track-experiments/show-runs-for-compute.png)
    
    > [!TIP]
    > Étant donné que les cibles de calcul de formation sont des ressources partagées, elles peuvent avoir plusieurs exécutions mises en file d’attente ou actives à un moment donné.
    > 
    > Une exécution peut contenir des exécutions enfants. De fait, une tâche d’entraînement peut créer plusieurs entrées.

Une fois l’exécution terminée, elle n’est plus affichée dans cette page. Pour voir des informations sur les exécutions terminées, consultez la section __Expériences__ dans le studio, puis sélectionnez l’expérience et l’exécution. Pour plus d’informations, consultez la section [Consulter les métriques pour les exécutions terminées](#view-the-experiment-in-the-web-portal).

## <a name="monitor-runs-using-the-jupyter-notebook-widget"></a>Surveiller les exécutions avec le widget Jupyter Notebook

Lorsque vous utilisez la méthode **ScriptRunConfig** pour envoyer des exécutions, vous pouvez vérifier la progression de l’exécution avec un [widget Jupyter](/python/api/azureml-widgets/azureml.widgets?preserve-view=true&view=azure-ml-py). Comme l’envoi de l’exécution, le widget est asynchrone et fournit des mises à jour automatiques toutes les 10 à 15 secondes jusqu’à ce que la tâche soit terminée.

Affichez le widget Jupyter en attendant la fin de l’exécution.
    
```python
from azureml.widgets import RunDetails
RunDetails(run).show()
```

![Capture d’écran du widget de notebook Jupyter](./media/how-to-track-experiments/run-details-widget.png)

Vous pouvez également obtenir un lien vers le même affichage dans votre espace de travail.

```python
print(run.get_portal_url())
```

## <a name="monitor-automated-machine-learning-runs"></a>Surveiller les exécutions de Machine Learning automatisé

Pour les exécutions de Machine Learning automatisé, pour accéder aux graphiques d’une exécution précédente, remplacez `<<experiment_name>>` par le nom de l’expérience appropriée :

```python
from azureml.widgets import RunDetails
from azureml.core.run import Run

experiment = Experiment (workspace, <<experiment_name>>)
run_id = 'autoML_my_runID' #replace with run_ID
run = Run(experiment, run_id)
RunDetails(run).show()
```

![Widget de notebook Jupyter pour le Machine Learning automatisé](./media/how-to-track-experiments/azure-machine-learning-auto-ml-widget.png)

## <a name="show-output-upon-completion"></a>Afficher la sortie une fois l’opération terminée

Quand vous utilisez **ScriptRunConfig**, vous pouvez employer ```run.wait_for_completion(show_output = True)``` pour indiquer quand l’entraînement du modèle est terminé. L’indicateur ```show_output``` vous donne une sortie détaillée. Pour plus d’informations, consultez la section ScriptRunConfig de [Procédure d’activation de la journalisation](how-to-track-experiments.md#scriptrun-logs).

<a id="queryrunmetrics"></a>
## <a name="query-run-metrics"></a>Interroger les métriques d’exécution

Vous pouvez afficher les métriques d’un modèle entraîné à l’aide de ```run.get_metrics()```. Par exemple, vous pouvez l’utiliser avec l’exemple ci-dessus pour déterminer le meilleur modèle en recherchant le modèle avec la valeur d’erreur quadratique moyenne (MSE) la plus faible.

<a name="view-the-experiment-in-the-web-portal"></a>
## <a name="view-run-records-in-the-studio"></a>Consulter les enregistrements d’exécution dans le studio

Vous pouvez parcourir les enregistrements d’exécution terminés, notamment les mesures journalisées, dans le [studio Azure Machine Learning](https://ml.azure.com).

Accédez à l’onglet **Expériences**. Pour afficher toutes les exécutions de votre espace de travail dans Expériences, sélectionnez l’onglet **Toutes les exécutions**. Vous pouvez explorer les exécutions au niveau du détail pour des Expériences spécifiques en appliquant le filtre Expérience dans la barre de menus supérieure. 

Pour l’affichage individuel de l’Expérience, sélectionnez l’onglet **Toutes les expériences**. Dans le tableau de bord d’exécution d’expérience, vous pouvez voir les journaux et les métriques suivis pour chaque exécution. 

Vous pouvez descendre dans la hiérarchie jusqu’à une exécution spécifique pour en afficher les sorties ou les journaux, ou télécharger la capture instantanée de l’expérience afin de pouvoir partager le dossier de l’expérience avec d’autres utilisateurs.

Vous pouvez également modifier la table de la liste d’exécution pour sélectionner plusieurs exécutions et afficher la valeur la plus récente, la valeur minimale ou la valeur maximale du journal pour vos exécutions. Personnalisez vos graphiques pour comparer les valeurs des mesures journalisées et les agrégats sur plusieurs exécutions.

![Exécution des détails dans Azure Machine Learning Studio](media/how-to-track-experiments/experimentation-tab.gif)

### <a name="format-charts-in-the-studio"></a>Mettre en forme les graphiques dans le studio

Utilisez les méthodes suivantes dans les API de journalisation pour influencer la visualisation de vos métriques par le studio.

|Valeur connectée|Exemple de code| Format dans le portail|
|----|----|----|
|Journaliser un tableau de valeurs numériques| `run.log_list(name='Fibonacci', value=[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89])`|graphique en courbes à variable unique|
|Journaliser une valeur numérique avec le même nom de métrique utilisé à plusieurs reprises (comme à l’intérieur d’une boucle for)| `for i in tqdm(range(-10, 10)):    run.log(name='Sigmoid', value=1 / (1 + np.exp(-i))) angle = i / 2.0`| Graphique en courbes à variable unique|
|Journaliser une ligne avec 2 colonnes numériques à plusieurs reprises|`run.log_row(name='Cosine Wave', angle=angle, cos=np.cos(angle))   sines['angle'].append(angle)      sines['sine'].append(np.sin(angle))`|Graphique en courbes à deux variables|
|Journaliser un table avec 2 colonnes numériques|`run.log_table(name='Sine Wave', value=sines)`|Graphique en courbes à deux variables|


## <a name="next-steps"></a>Étapes suivantes

Essayez de suivre les étapes suivantes pour savoir comment utiliser Azure Machine Learning :

* Découvrez comment [suivre les expériences et activer les journaux dans le concepteur Azure Machine Learning](how-to-track-designer-experiments.md).

* Examinez un exemple d’inscription et de déploiement du meilleur modèle dans le tutoriel [Entraîner un modèle de classification d’images avec Azure Machine Learning](tutorial-train-models-with-aml.md).