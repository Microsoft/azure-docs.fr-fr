---
title: Démarrer, analyser et annuler des exécutions de d’entraînement dans Python
titleSuffix: Azure Machine Learning
description: Apprenez à démarrer, à contrôler et à gérer vos exécutions d’expériences de Machine Learning avec le Kit de développement logiciel (SDK) Azure Machine Learning Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: roastala
author: rastala
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 12/04/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, devx-track-azurecli
ms.openlocfilehash: ec006636ed7e975b696aa32300b32089e3209bb5
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96600470"
---
# <a name="start-monitor-and-cancel-training-runs-in-python"></a>Démarrer, analyser et annuler des exécutions de d’entraînement dans Python

Le [Kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py), [l’interface de ligne de commande de Machine Learning](reference-azure-machine-learning-cli.md) et [Azure Machine Learning Studio](https://ml.azure.com) fournissent diverses méthodes permettant d’analyser, d’organiser et de gérer vos exécutions d’entraînement et d’expérimentation.

Cet article montre des exemples des tâches suivantes :

* Analyse des performances d’exécution.
* Annulation ou mise en échec d’exécutions.
* Création d’exécutions enfants.
* Identification et recherche d’exécutions.

> [!TIP]
> Si vous recherchez des informations sur la supervision du service Azure Machine Learning et des services Azure associés, consultez [Guide pratique pour superviser Azure Machine Learning](monitor-azure-machine-learning.md).
> Si vous recherchez des informations sur les modèles de supervision déployés en tant que services web ou modules IoT Edge, consultez [Collecter les données des modèles](how-to-enable-data-collection.md) et [Superviser avec Application Insights](how-to-enable-app-insights.md).

## <a name="prerequisites"></a>Prérequis

Vous devez disposer des éléments suivants :

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.

* Un [espace de travail Azure Machine Learning](how-to-manage-workspace.md).

* Le Kit de développement logiciel (SDK) Azure Machine Learning pour Python (version 1.0.21 ou ultérieure). Pour installer ou mettre à jour la dernière version du SDK, consultez [Installer ou mettre à jour le SDK](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).

    Pour installer votre version du Kit de développement logiciel (SDK) Azure Machine Learning, utilisez le code suivant :

    ```python
    print(azureml.core.VERSION)
    ```

* [Azure CLI](/cli/azure/?preserve-view=true&view=azure-cli-latest) et l’[extension CLI pour Azure Machine Learning](reference-azure-machine-learning-cli.md).

## <a name="monitor-run-performance"></a>Analyse des performances d’exécution

* Démarrer une exécution et son processus de journalisation

    # <a name="python"></a>[Python](#tab/python)
    
    1. Configurez votre expérience en important les classes Set up your experiment by importing the [Workspace](/python/api/azureml-core/azureml.core.workspace.workspace?preserve-view=true&view=azure-ml-py), [Experiment](/python/api/azureml-core/azureml.core.experiment.experiment?preserve-view=true&view=azure-ml-py), [Run](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py), et [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) depuis le package [azureml.core](/python/api/azureml-core/azureml.core?preserve-view=true&view=azure-ml-py).
    
        ```python
        import azureml.core
        from azureml.core import Workspace, Experiment, Run
        from azureml.core import ScriptRunConfig
        
        ws = Workspace.from_config()
        exp = Experiment(workspace=ws, name="explore-runs")
        ```
    
    1. Démarrez une exécution et son processus de journalisation à l’aide de la méthode [`start_logging()`](/python/api/azureml-core/azureml.core.experiment%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truestart-logging--args----kwargs-).
    
        ```python
        notebook_run = exp.start_logging()
        notebook_run.log(name="message", value="Hello from run!")
        ```
        
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    Pour démarrer une exécution de votre expérience, procédez comme suit :
    
    1. À partir d’un interpréteur de commandes ou d’une invite, utilisez Azure CLI pour vous identifier auprès de votre abonnement Azure :
    
        ```azurecli-interactive
        az login
        ```
        
        [!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 
    
    1. Attachez une configuration d’espace de travail au dossier qui contient votre script de d’entraînement. Remplacez `myworkspace` par votre espace de travail Azure Machine Learning. Remplacez `myresourcegroup` par le groupe de ressources Azure qui contient votre espace de travail :
    
        ```azurecli-interactive
        az ml folder attach -w myworkspace -g myresourcegroup
        ```
    
        Cette commande crée un sous-répertoire `.azureml` qui contient des exemples de fichiers d’environnement runconfig et conda. Il contient également un fichier `config.json` utilisé pour communiquer avec votre espace de travail Azure Machine Learning.
    
        Pour plus d’informations, consultez [az ml folder attach](/cli/azure/ext/azure-cli-ml/ml/folder?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-folder-attach).
    
    2. Pour démarrer l’exécution, utilisez la commande suivante : Lorsque vous utilisez cette commande, spécifiez le nom du fichier runconfig (le texte avant \*.runconfig si vous regardez le système de fichiers) avec le paramètre - c.
    
        ```azurecli-interactive
        az ml run submit-script -c sklearn -e testexperiment train.py
        ```
    
        > [!TIP]
        > La commande `az ml folder attach` crée un sous-répertoire `.azureml` qui contient deux exemples de fichiers runconfig.
        >
        > Si vous disposez d’un script Python qui crée un objet de configuration de série de tests par programmation, vous pouvez utiliser [RunConfig.save()](/python/api/azureml-core/azureml.core.runconfiguration?preserve-view=true&view=azure-ml-py#&preserve-view=truesave-path-none--name-none--separate-environment-yaml-false-) pour l’enregistrer dans un fichier runconfig.
        >
        > Pour plus d’exemples de fichiers runconfig, consultez [https://github.com/MicrosoftDocs/pipelines-azureml/](https://github.com/MicrosoftDocs/pipelines-azureml/).
    
        Pour plus d’informations, consultez [az ml run submit-script](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-submit-script).
    
    # <a name="studio"></a>[Studio](#tab/azure-studio)
    
    Pour soumettre une exécution de pipeline dans le concepteur, procédez comme suit :
    
    1. Définissez une cible de calcul par défaut pour votre pipeline.
    
    1. Sélectionnez l’option **Exécuter** située en haut du canevas du pipeline.
    
    1. Sélectionnez une expérience pour regrouper vos exécutions de pipeline.
    
    ---

* Surveiller l’état d’une exécution

    # <a name="python"></a>[Python](#tab/python)
    
    * Obtenir l’état d’une exécution avec la méthode [`get_status()`](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-status--).
    
        ```python
        print(notebook_run.get_status())
        ```
    
    * Pour obtenir l’ID d’exécution, le temps d’exécution et des détails supplémentaires sur l’exécution, utilisez la méthode [`get_details()`](/python/api/azureml-core/azureml.core.workspace.workspace?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-details--).
    
        ```python
        print(notebook_run.get_details())
        ```
    
    * Lorsque votre exécution se termine correctement, utilisez la méthode [`complete()`](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truecomplete--set-status-true-) pour la marquer comme terminé.
    
        ```python
        notebook_run.complete()
        print(notebook_run.get_status())
        ```
    
    * Si vous utilisez le modèle de conception `with...as` de Python, l’exécution se marque automatiquement comme étant terminée quand l’exécution est en dehors de l’étendue. Vous n’avez pas besoin de marquer manuellement l’exécution comme étant terminée.
        
        ```python
        with exp.start_logging() as notebook_run:
            notebook_run.log(name="message", value="Hello from run!")
            print(notebook_run.get_status())
        
        print(notebook_run.get_status())
        ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    * Pour afficher une liste des exécutions de votre expérience, utilisez la commande suivante. Remplacez `experiment` par le nom de votre expérience :
    
        ```azurecli-interactive
        az ml run list --experiment-name experiment
        ```
    
        Cette commande renvoie un document JSON qui répertorie des informations sur les exécutions de cette expérience.
    
        Pour plus d’informations, consultez [az ml experiment list](/cli/azure/ext/azure-cli-ml/ml/experiment?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-experiment-list).
    
    * Pour afficher des informations sur une exécution spécifique, utilisez la commande suivante. Remplacez `runid` par l’ID d’exécution.
    
        ```azurecli-interactive
        az ml run show -r runid
        ```
    
        Cette commande renvoie un document JSON qui répertorie des informations sur l’exécution.
    
        Pour plus d’informations, consultez [az ml run show](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-show).
    
    
    # <a name="studio"></a>[Studio](#tab/azure-studio)
    
    Pour afficher le nombre d’exécutions actives pour votre expérience dans le studio :
    
    1. Accédez à la section **Expériences**.
    
    1. Sélectionner une expérience.
    
        Dans la page d’expérience, vous pouvez voir le nombre de cibles de calcul actives et la durée de chaque exécution. 
    
    1. Personnalisez l’expérience en sélectionnant des exécutions à comparer, en ajoutant des graphiques ou en appliquant des filtres. Ces modifications peuvent être enregistrées sous la forme d’un **affichage personnalisé** afin que vous puissiez facilement revenir à votre travail. Les utilisateurs disposant d’autorisations d’espace de travail peuvent modifier ou consulter l’affichage personnalisé. Partagez également l’affichage personnalisé avec d’autres utilisateurs en copiant-collant l’URL dans le navigateur.  
    
        :::image type="content" source="media/how-to-manage-runs/custom-views.gif" alt-text="Capture d’écran : créer un affichage personnalisé":::
    
    1. Sélectionnez un numéro d’exécution spécifique.
    
    1. Dans l’onglet **Journaux d’activité**, vous trouverez des journaux de diagnostic et d’erreurs pour votre exécution de pipeline.
    
    ---
    
## <a name="cancel-or-fail-runs"></a>Annulation ou mise en échec d’exécutions

Si vous constatez une erreur ou si votre exécution prend trop de temps, vous pouvez l’annuler.

# <a name="python"></a>[Python](#tab/python)

Pour annuler une exécution à l’aide du Kit de développement logiciel (SDK), utilisez la méthode [`cancel()`](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truecancel--) :

```python
src = ScriptRunConfig(source_directory='.', script='hello_with_delay.py')
local_run = exp.submit(src)
print(local_run.get_status())

local_run.cancel()
print(local_run.get_status())
```

Si votre exécution se termine, mais contient une erreur (par exemple, utilisation d’un script d’entraînement incorrect), vous pouvez utiliser la méthode [`fail()`](/python/api/azureml-core/azureml.core.run%28class%29#fail-error-details-none--error-code-none---set-status-true-) la marquer comme ayant échoué.

```python
local_run = exp.submit(src)
local_run.fail()
print(local_run.get_status())
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour annuler une exécution à l’aide de l’interface de ligne de commande, utilisez la commande suivante : Remplacez `runid` par l’ID d’exécution.

```azurecli-interactive
az ml run cancel -r runid -w workspace_name -e experiment_name
```

Pour plus d’informations, consultez [az ml run cancel](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-cancel).

# <a name="studio"></a>[Studio](#tab/azure-studio)

Pour annuler une exécution dans le studio, procédez comme suit :

1. Accédez au pipeline en cours d’exécution dans la section **Expériences** ou **Pipelines**. 

1. Sélectionnez le numéro d’exécution du pipeline que vous souhaitez annuler.

1. Dans la barre d’outils, sélectionnez **Annuler**

---

## <a name="create-child-runs"></a>Créer des exécutions enfants

Créez des exécutions enfants pour regrouper les exécutions associées, comme pour différentes itérations de réglage des hyperparamètres.

> [!NOTE]
> Les exécutions enfants peuvent uniquement être créées à l’aide du Kit de développement logiciel (SDK).

Cet exemple de code utilise le script `hello_with_children.py` de création d’un lot de cinq exécutions enfants issues d’une exécution envoyée à l’aide de la méthode [`child_run()`](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truechild-run-name-none--run-id-none--outputs-none-) :

```python
!more hello_with_children.py
src = ScriptRunConfig(source_directory='.', script='hello_with_children.py')

local_run = exp.submit(src)
local_run.wait_for_completion(show_output=True)
print(local_run.get_status())

with exp.start_logging() as parent_run:
    for c,count in enumerate(range(5)):
        with parent_run.child_run() as child:
            child.log(name="Hello from child run", value=c)
```

> [!NOTE]
> Dès qu’elles sont hors de portée, les exécutions enfants sont automatiquement marquées comme étant terminées.

Pour créer efficacement de nombreuses exécutions enfants, utilisez la méthode [`create_children()`](/python/api/azureml-core/azureml.core.run.run?preserve-view=true&view=azure-ml-py#&preserve-view=truecreate-children-count-none--tag-key-none--tag-values-none-). Chaque création se traduisant par un appel réseau, la création d’un lot d’exécutions est plus efficace que la création d’exécutions une à une.

### <a name="submit-child-runs"></a>Envoyer des exécutions enfants

Les exécutions enfants peuvent également être envoyées à partir d’une exécution parente. Cela vous permet de créer des hiérarchies d’exécutions parentes et enfants. Vous ne pouvez pas créer une exécution enfant sans parent : même si l’exécution parente ne fait rien, mais que le lancement de l’enfant s’exécute, il est toujours nécessaire de créer la hiérarchie. L’état de toutes les exécutions est indépendant : un parent peut afficher l’état de réussite `"Completed"` même si une ou plusieurs exécutions enfants ont été annulées ou ont échoué.  

Vous pouvez souhaiter que vos exécutions enfants utilisent une configuration d’exécution différente de celle de l’exécution parente. Par exemple, vous pouvez utiliser une configuration basée sur l’UC et moins puissante pour le parent, tout en utilisant des configurations basées sur le GPU pour les enfants. Une autre volonté courante consiste à passer à chaque enfant des données et des arguments différents. Pour personnaliser une exécution enfant, créez un objet `ScriptRunConfig` pour l’exécution enfant. Le code ci-dessous effectue les actions suivantes :

- Récupère une ressource de calcul nommée `"gpu-cluster"` dans l’espace de travail `ws`
- Itère sur différentes valeurs d’argument à passer aux objets `ScriptRunConfig` des enfants
- Crée et envoie une nouvelle exécution enfant à l’aide de la ressource de calcul personnalisée et de l’argument
- Bloque jusqu’à ce que toutes les exécutions enfants soient terminées

```python
# parent.py
# This script controls the launching of child scripts
from azureml.core import Run, ScriptRunConfig

compute_target = ws.compute_targets["gpu-cluster"]

run = Run.get_context()

child_args = ['Apple', 'Banana', 'Orange']
for arg in child_args: 
    run.log('Status', f'Launching {arg}')
    child_config = ScriptRunConfig(source_directory=".", script='child.py', arguments=['--fruit', arg], compute_target=compute_target)
    # Starts the run asynchronously
    run.submit_child(child_config)

# Experiment will "complete" successfully at this point. 
# Instead of returning immediately, block until child runs complete

for child in run.get_children():
    child.wait_for_completion()
```

Pour créer efficacement de nombreuses exécutions enfants avec des configurations, des arguments et des entrées identiques, utilisez la méthode [`create_children()`](/python/api/azureml-core/azureml.core.run.run?preserve-view=true&view=azure-ml-py#&preserve-view=truecreate-children-count-none--tag-key-none--tag-values-none-). Chaque création se traduisant par un appel réseau, la création d’un lot d’exécutions est plus efficace que la création d’exécutions une à une.

Dans une exécution enfant, vous pouvez afficher l’ID d’exécution parente :

```python
## In child run script
child_run = Run.get_context()
child_run.parent.id
```

### <a name="query-child-runs"></a>Interroger les exécutions enfants

Pour interroger les exécutions enfants d’un parent spécifique, utilisez la méthode [`get_children()`](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-children-recursive-false--tags-none--properties-none--type-none--status-none---rehydrate-runs-true-). L’argument ``recursive = True`` vous permet d’interroger une arborescence imbriquée d’enfants et de petits-enfants.

```python
print(parent_run.get_children())
```

### <a name="log-to-parent-or-root-run"></a>Connexion à l’exécution parente ou racine

Vous pouvez utiliser le champ `Run.parent` pour accéder à l’exécution qui a lancé l’exécution enfant actuelle. Un cas d’utilisation courant est lorsque vous souhaitez consolider les résultats des journaux dans un emplacement unique. Notez que les exécutions enfants s’exécutent de façon asynchrone et qu’il n’y a aucune garantie de classement ou de synchronisation au-delà de la capacité du parent à attendre la fin de l’exécution de ses enfants.

```python
# in child (or even grandchild) run

def root_run(self : Run) -> Run :
    if self.parent is None : 
        return self
    return root_run(self.parent)

current_child_run = Run.get_context()
root_run(current_child_run).log("MyMetric", f"Data from child run {current_child_run.id}")

```


## <a name="tag-and-find-runs"></a>Identifier et rechercher des exécutions

Dans Azure Machine Learning, vous pouvez utiliser des balises et propriétés pour vous aider dans l’organisation et l’interrogation d’exécutions en vue de rechercher des informations importantes.

* Ajouter des propriétés et des balises

    # <a name="python"></a>[Python](#tab/python)
    
    Pour ajouter des métadonnées de recherche à vos exécutions, utilisez la méthode [`add_properties()`](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=trueadd-properties-properties-). Par exemple, le code suivant ajoute la propriété `"author"` à l’exécution :
    
    ```Python
    local_run.add_properties({"author":"azureml-user"})
    print(local_run.get_properties())
    ```
    
    Les propriétés sont immuables et créent enregistrement permanent à des fins d’audit. L’exemple de code suivant génère une erreur, car nous avons déjà ajouté `"azureml-user"` comme valeur de propriété `"author"` dans le code précédent :
    
    ```Python
    try:
        local_run.add_properties({"author":"different-user"})
    except Exception as e:
        print(e)
    ```
    
    Contrairement aux propriétés, les étiquettes sont modifiables. Pour ajouter des informations consultables et ayant du sens pour les utilisateurs de votre expérience, utilisez la méthode [`tag()`](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truetag-key--value-none-).
    
    ```Python
    local_run.tag("quality", "great run")
    print(local_run.get_tags())
    
    local_run.tag("quality", "fantastic run")
    print(local_run.get_tags())
    ```
    
    Vous pouvez également ajouter des balises de chaîne simples. Quand ces étiquettes apparaissent dans le dictionnaire des étiquettes, leur valeur est `None`.
    
    ```Python
    local_run.tag("worth another look")
    print(local_run.get_tags())
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    > [!NOTE]
    > À l’aide de l’interface CLI, vous pouvez uniquement ajouter ou mettre à jour des balises.
    
    Utilisez la commande suivante pour ajouter ou mettre à jour une balise :
    
    ```azurecli-interactive
    az ml run update -r runid --add-tag quality='fantastic run'
    ```
    
    Pour plus d’informations, consultez [az ml run update](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-update).
    
    # <a name="studio"></a>[Studio](#tab/azure-studio)
    
    Vous pouvez afficher les propriétés et les balises dans Studio, mais vous ne pouvez pas les y modifier.
    
    ---

* Interroger des propriétés et des balises

    Vous pouvez interroger des exécutions au sein d’une expérience pour renvoyer une liste d’exécutions qui correspondent aux propriétés et balises spécifiques.

    # <a name="python"></a>[Python](#tab/python)
    
    ```Python
    list(exp.get_runs(properties={"author":"azureml-user"},tags={"quality":"fantastic run"}))
    list(exp.get_runs(properties={"author":"azureml-user"},tags="worth another look"))
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    L’interface de ligne de commande Azure prend en charge les requêtes [JMESPath](http://jmespath.org) qui peuvent être utilisées pour filtrer les exécutions en fonction des balises et des propriétés. Pour utiliser une requête JMESPath avec l’interface de ligne de commande Azure, spécifiez-la avec le paramètre `--query`. Les exemples suivants montrent des requêtes utilisant des balises et propriétés :
    
    ```azurecli-interactive
    # list runs where the author property = 'azureml-user'
    az ml run list --experiment-name experiment [?properties.author=='azureml-user']
    # list runs where the tag contains a key that starts with 'worth another look'
    az ml run list --experiment-name experiment [?tags.keys(@)[?starts_with(@, 'worth another look')]]
    # list runs where the author property = 'azureml-user' and the 'quality' tag starts with 'fantastic run'
    az ml run list --experiment-name experiment [?properties.author=='azureml-user' && tags.quality=='fantastic run']
    ```
    
    Pour plus d’informations sur l’interrogation des résultats de l’interface de ligne de commande Azure, consultez [Interroger le résultat des commandes de l’interface de ligne de commande Azure](/cli/azure/query-azure-cli?preserve-view=true&view=azure-cli-latest).
    
    # <a name="studio"></a>[Studio](#tab/azure-studio)
    
    1. Accédez à la section **Pipelines**.
    
    1. Utilisez la barre de recherche pour filtrer les pipelines à partir de balises, de descriptions, de noms d’expériences et d’expéditeurs.
    
    ---

## <a name="example-notebooks"></a>Exemples de notebooks

Les notebooks suivants illustrent les concepts de cet article :

* Pour en savoir plus sur les API de journalisation, consultez [Notebook de l’API de journalisation](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/logging-api/logging-api.ipynb).

* Pour plus d’informations sur la gestion des exécutions avec le Kit de développement logiciel (SDK) Azure Machine Learning, consultez le [notebook sur la gestion des exécutions](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/manage-runs/manage-runs.ipynb).

## <a name="next-steps"></a>Étapes suivantes

* Pour savoir comment enregistrer les mesures de vos expériences, consultez [Enregistrer des métriques pendant les exécutions d’entraînement](how-to-track-experiments.md).
* Pour découvrir comment superviser les ressources et les journaux dans Azure Machine Learning, consultez [Supervision d’Azure Machine Learning](monitor-azure-machine-learning.md).