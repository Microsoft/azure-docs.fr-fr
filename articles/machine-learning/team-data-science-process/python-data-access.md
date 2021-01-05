---
title: Accéder à des jeux de données avec la bibliothèque cliente de Python - Team Data Science Process
description: Installez et utilisez la bibliothèque cliente Python pour accéder et gérer les données d'apprentissage automatique d'Azure en toute sécurité à partir d'un environnement Python local.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, devx-track-python, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 497b8f5598cf7aa7720f47863d465f5e29789b07
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321959"
---
# <a name="access-datasets-with-python-using-the-azure-machine-learning-python-client-library"></a>Accédez aux jeux de données avec Python grâce à la bibliothèque cliente Python d'Azure Machine Learning
L’aperçu de la bibliothèque cliente Python de Microsoft Azure Machine Learning offre un accès sécurisé à vos jeux de données Azure Machine Learning à partir d’un environnement Python local et permet la création et la gestion de jeux de données dans un espace de travail.

Cette rubrique fournit des instructions pour les procédures suivantes :

* installation de la bibliothèque cliente Python de Machine Learning
* accès et téléchargement des jeux de données, y compris des instructions sur l’obtention d’une autorisation d'accès aux jeux de données Azure Machine Learning depuis votre environnement Python local
* accès aux jeux de données intermédiaires à partir d'expériences
* utilisation de la bibliothèque cliente Python pour énumérer les jeux de données, accéder aux métadonnées, lire le contenu d’un jeu de données, créer de nouveaux jeux de données et mettre à jour des jeux de données existants

## <a name="prerequisites"></a><a name="prerequisites"></a>Configuration requise
La bibliothèque cliente Python a été testée dans les environnements suivants :

* Windows, Mac et Linux
* Python 2.7, 3.3 et 3.4

Il a une dépendance sur les packages suivants :

* requêtes
* python-dateutil
* pandas

Nous vous invitons à utiliser une distribution Python telle qu’[Anaconda](https://www.anaconda.com/) ou [Canopy](https://store.enthought.com/downloads/), qui est fournie avec Python, IPython et les trois packages listés ci-dessus installés. Bien que IPython n'est pas formellement requis, il s'agit d'un environnement idéal pour la manipulation et la visualisation interactive des données.

### <a name="how-to-install-the-azure-machine-learning-python-client-library"></a><a name="installation"></a>Installation de la bibliothèque cliente Python d'Azure Machine Learning
Installez la bibliothèque cliente Python d’Azure Machine Learning pour effectuer les tâches décrites dans cette rubrique. Cette bibliothèque est disponible depuis le [Python Package Index](https://pypi.python.org/pypi/azureml). Pour l'installer dans votre environnement Python, exécutez la commande suivante à partir de votre environnement Python local :

```console
pip install azureml
```

Vous pouvez également la télécharger et l’installer à partir des sources disponibles sur [GitHub](https://github.com/Azure/Azure-MachineLearning-ClientLibrary-Python).

```console
python setup.py install
```

Si vous avez installé git sur votre ordinateur, vous pouvez utiliser pip pour l'installer directement à partir du référentiel git :

```console
pip install git+https://github.com/Azure/Azure-MachineLearning-ClientLibrary-Python.git
```

## <a name="use-code-snippets-to-access-datasets"></a><a name="datasetAccess"></a>Utilisation des extraits de code pour accéder aux jeux de données
La bibliothèque cliente Python vous offre un accès par programme à vos jeux de données existants à partir des expériences qui ont été exécutées.

Depuis l’interface web Azure Machine Learning Studio (classique), vous pouvez générer des extraits de code qui incluent toutes les informations nécessaires pour télécharger et désérialiser des jeux de données en tant qu’objets DataFrame de pandas sur votre ordinateur local.

### <a name="security-for-data-access"></a><a name="security"></a>Sécurité relative à l'accès aux données
Les extraits de code fournis par Azure Machine Learning Studio (classique) pour une utilisation avec la bibliothèque de client Python incluent votre ID d’espace de travail et votre jeton d’autorisation. Ceux-ci vous permettent un accès complet à votre espace de travail et doivent être protégés, par exemple avec un mot de passe.

Pour des raisons de sécurité, la fonctionnalité d'extrait de code est uniquement disponible pour les utilisateurs qui ont leur rôle défini en tant que **Propriétaire** de l'espace de travail. Votre rôle s’affiche dans Azure Machine Learning Studio (classique) dans la page **UTILISATEURS** sous **Paramètres**.

![Capture d’écran montrant les paramètres de la page UTILISATEURS d’Azure Machine Learning Studio.][security]

Si votre rôle n’est pas défini en tant que **Propriétaire** , vous pouvez demander à être invité à nouveau en tant que propriétaire ou demander au propriétaire de l’espace de travail de vous fournir l’extrait de code.

Pour obtenir le jeton d’autorisation, vous pouvez choisir l’une des options suivantes :

* Demander un jeton à un propriétaire. Les propriétaires peuvent accéder à leurs jetons d’autorisation à partir de la page Paramètres de leur espace de travail dans Azure Machine Learning Studio (classique). Sélectionnez **Paramètres** dans le volet gauche puis cliquez sur **JETONS D’AUTORISATION** pour voir les jetons principaux et secondaires. Bien que les jetons d'autorisation principaux ou secondaires puissent être utilisés dans l'extrait de code, il est recommandé aux propriétaires de ne partager que les jetons d'autorisation secondaires.

   ![Jetons d’autorisation](./media/python-data-access/ml-python-access-settings-tokens.png)

* Demandez à être promu au rôle de propriétaire : un propriétaire actuel de l’espace de travail doit tout d’abord vous supprimer de l’espace de travail puis vous y inviter à nouveau en tant que propriétaire.

Une fois que les développeurs ont obtenu l’ID de l’espace de travail et le jeton d’autorisation, ils peuvent accéder à l’espace de travail à l’aide de l’extrait de code, indépendamment de leur rôle.

Les jetons d’autorisation sont gérés sur la page **JETONS D’AUTORISATION** sous **PARAMÈTRES**. Vous pouvez les régénérer, mais cette procédure entraîne la révocation de l’accès au jeton précédent.

### <a name="access-datasets-from-a-local-python-application"></a><a name="accessingDatasets"></a>Accès aux jeux de données depuis une application Python locale
1. Dans Machine Learning Studio (classique), cliquez sur **JEUX DE DONNÉES** dans la barre de navigation à gauche.
2. Sélectionnez le jeu de données auquel vous souhaitez accéder. Vous pouvez sélectionner un des jeux de données depuis la liste **MES JEUX DE DONNÉES** ou **EXEMPLES**.
3. Dans la barre d’outils inférieure, cliquez sur **Générer un code d’accès aux données**. Ce bouton est désactivé si les données sont dans un format incompatible avec la bibliothèque cliente Python.
   
    ![Capture d’écran montrant les jeux de données avec l’option GENERATE DATA ACCESS CODE (Générer le code d’accès aux données).][datasets]
4. Sélectionnez l'extrait de code dans la fenêtre qui s'affiche et copiez-le dans votre presse-papiers.
   
    ![Bouton Générer le fichier de code d’accès aux données][dataset-access-code]
5. Collez le code dans le bloc-notes de votre application Python locale.
   
    ![Collez le code dans le notebook][ipython-dataset]

## <a name="access-intermediate-datasets-from-machine-learning-experiments"></a><a name="accessingIntermediateDatasets"></a>Accès aux jeux de données intermédiaires à partir d'expériences de Machine Learning
Après l’exécution d’une expérience dans Machine Learning Studio (classique), il est possible d’accéder aux jeux de données intermédiaires depuis les nœuds de modules de sortie. Les jeux de données intermédiaires sont des données qui ont été créées et utilisées pour les étapes intermédiaires lorsqu'un outil de modèle a été exécuté.

Les jeux de données intermédiaires sont accessibles tant que le format de données est compatible avec la bibliothèque cliente Python.

Les formats suivants sont pris en charge (les constantes pour ces formats sont dans la classe `azureml.DataTypeIds`) :

* Texte brut
* CSV générique
* TSV générique
* CSV générique sans en-tête
* TSV générique sans en-tête

Vous pouvez déterminer le format en pointant sur un nœud de sortie de module. Celui-ci s'affiche avec le nom de nœud dans une infobulle.

Certains des modules, comme le module [Fractionner][split], ont un format de sortie appelé `Dataset`, qui n’est pas pris en charge par la bibliothèque de client Python.

![Format de jeu de données][dataset-format]

Vous devez utiliser un module de conversion, comme [Convertir au format CSV][convert-to-csv], afin d’obtenir une sortie dans un format pris en charge.

![Format CSV générique][csv-format]

Les étapes suivantes proposent un exemple qui créé une expérience, l'exécute et accède au jeu de données intermédiaire.

1. Création d'une nouvelle expérience.
2. Insérez un module **Jeu de données Adult Census Income Binary Classification** .
3. Insérez un module [Fractionner][split], puis connectez son entrée à la sortie du module de jeu de données.
4. Insérez un module [Convertir au format CSV][convert-to-csv], puis connectez son entrée à l’une des sorties du module [Fractionner][split].
5. Enregistrez l’expérience, exécutez-la et attendez que la tâche se termine.
6. Cliquez sur le nœud de sortie du module [Convertir au format CSV][convert-to-csv].
7. Lorsque le menu contextuel s’affiche, sélectionnez **Générer un code d’accès aux données**.
   
    ![Menu contextuel][experiment]
8. Sélectionnez l’extrait de code dans la fenêtre qui s’affiche et copiez-le dans votre presse-papiers.
   
    ![Générer le code d’accès à partir du menu contextuel][intermediate-dataset-access-code]
9. Collez le code dans votre bloc-notes.
   
    ![Collez le code dans le notebook][ipython-intermediate-dataset]
10. Vous pouvez visualiser les données à l'aide de matplotlib. Cela les affiche dans un histogramme pour la colonne âge :
    
    ![Histogramme][ipython-histogram]

## <a name="use-the-machine-learning-python-client-library-to-access-read-create-and-manage-datasets"></a><a name="clientApis"></a>Utilisation de la bibliothèque cliente Python de Machine Learning pour accéder, lire, créer et gérer des jeux de données
### <a name="workspace"></a>Espace de travail
L'espace de travail est le point d'entrée de la bibliothèque cliente Python. Il fournit la classe `Workspace` avec l’ID de votre espace de travail et le jeton d’autorisation pour créer une instance :

```python
ws = Workspace(workspace_id='4c29e1adeba2e5a7cbeb0e4f4adfb4df',
               authorization_token='f4f3ade2c6aefdb1afb043cd8bcf3daf')
```

### <a name="enumerate-datasets"></a>Énumérer les jeux de données
Pour énumérer tous les jeux de données dans un espace de travail donné :

```python
for ds in ws.datasets:
    print(ds.name)
```

Pour énumérer uniquement les jeux de données créés par l'utilisateur :

```python
for ds in ws.user_datasets:
    print(ds.name)
```

Pour énumérer uniquement les exemples de jeux de données :

```python
for ds in ws.example_datasets:
    print(ds.name)
```

Vous pouvez accéder à un jeu de données par son nom (qui respecte la casse) :

```python
ds = ws.datasets['my dataset name']
```

Vous pouvez également y accéder par l'index :

```python
ds = ws.datasets[0]
```

### <a name="metadata"></a>Métadonnées
Les jeux de données ont des métadonnées, en plus du contenu. (Les jeux de données intermédiaires sont une exception à cette règle et n'ont pas de métadonnées.)

Certaines valeurs de métadonnées sont affectées par l'utilisateur lors de la création :

* `print(ds.name)`
* `print(ds.description)`
* `print(ds.family_id)`
* `print(ds.data_type_id)`

D'autres sont des valeurs affectées par Azure ML :

* `print(ds.id)`
* `print(ds.created_date)`
* `print(ds.size)`

Consultez la classe `SourceDataset` pour plus d'informations sur les métadonnées disponibles.

### <a name="read-contents"></a>Lire le contenu
Les extraits de code fournis par Machine Learning Studio (classique) téléchargent et désérialisent automatiquement le jeu de données vers un objet DataFrame Pandas. Cette opération est effectuée à l'aide de la méthode `to_dataframe` :

```python
frame = ds.to_dataframe()
```

Si vous préférez télécharger les données brutes et procéder vous-même à la désérialisation, cela est possible. Pour le moment, il s'agit de la seule option pour les formats tels que « ARFF » que la bibliothèque cliente Python ne peut pas désérialiser.

Pour lire le contenu en texte :

```python
text_data = ds.read_as_text()
```

Pour lire le contenu en binaire :

```python
binary_data = ds.read_as_binary()
```

Vous pouvez également ouvrir un simple flux vers le contenu :

```python
with ds.open() as file:
    binary_data_chunk = file.read(1000)
```

### <a name="create-a-new-dataset"></a>Créer un nouveau jeu de données
La bibliothèque cliente Python vous permet de télécharger des jeux de données depuis votre programme Python. Ces jeux de données sont alors disponibles pour une utilisation dans votre espace de travail.

Si vous avez vos données dans un DataFrame de pandas, utilisez le code suivant :

```python
from azureml import DataTypeIds

dataset = ws.datasets.add_from_dataframe(
    dataframe=frame,
    data_type_id=DataTypeIds.GenericCSV,
    name='my new dataset',
    description='my description'
)
```

Si vos données sont déjà sérialisées, vous pouvez utiliser :

```python
from azureml import DataTypeIds

dataset = ws.datasets.add_from_raw_data(
    raw_data=raw_data,
    data_type_id=DataTypeIds.GenericCSV,
    name='my new dataset',
    description='my description'
)
```

La bibliothèque cliente Python est en mesure de sérialiser un DataFrame de pandas aux formats suivants (ces constantes sont dans la classe `azureml.DataTypeIds` ) :

* Texte brut
* CSV générique
* TSV générique
* CSV générique sans en-tête
* TSV générique sans en-tête

### <a name="update-an-existing-dataset"></a>Mettre à jour un jeu de données existant
Si vous essayez de télécharger un nouveau jeu de données avec un nom qui correspond à un jeu de données existant, vous devriez obtenir une erreur de conflit.

Pour mettre à jour un jeu de données existant, vous devez d'abord obtenir la référence d'un jeu de données existant :

```python
dataset = ws.datasets['existing dataset']

print(dataset.data_type_id) # 'GenericCSV'
print(dataset.name)         # 'existing dataset'
print(dataset.description)  # 'data up to jan 2015'
```

Utilisez ensuite `update_from_dataframe` pour sérialiser et remplacer le contenu du jeu de données sur Azure :

```python
dataset = ws.datasets['existing dataset']

dataset.update_from_dataframe(frame2)

print(dataset.data_type_id) # 'GenericCSV'
print(dataset.name)         # 'existing dataset'
print(dataset.description)  # 'data up to jan 2015'
```

Si vous souhaitez sérialiser les données dans un format différent, spécifiez une valeur pour le paramètre en option `data_type_id` .

```python
from azureml import DataTypeIds

dataset = ws.datasets['existing dataset']

dataset.update_from_dataframe(
    dataframe=frame2,
    data_type_id=DataTypeIds.GenericTSV,
)

print(dataset.data_type_id) # 'GenericTSV'
print(dataset.name)         # 'existing dataset'
print(dataset.description)  # 'data up to jan 2015'
```

Vous pouvez éventuellement définir une nouvelle description en spécifiant une valeur pour le paramètre `description` .

```python
dataset = ws.datasets['existing dataset']

dataset.update_from_dataframe(
    dataframe=frame2,
    description='data up to feb 2015',
)

print(dataset.data_type_id) # 'GenericCSV'
print(dataset.name)         # 'existing dataset'
print(dataset.description)  # 'data up to feb 2015'
```

Vous pouvez éventuellement définir un nouveau nom en spécifiant une valeur pour le paramètre `name` . À partir de maintenant, vous allez récupérer le jeu de données uniquement à l'aide du nouveau nom. Le code suivant met à jour les données, le nom et la description.

```python
dataset = ws.datasets['existing dataset']

dataset.update_from_dataframe(
    dataframe=frame2,
    name='existing dataset v2',
    description='data up to feb 2015',
)

print(dataset.data_type_id)                    # 'GenericCSV'
print(dataset.name)                            # 'existing dataset v2'
print(dataset.description)                     # 'data up to feb 2015'

print(ws.datasets['existing dataset v2'].name) # 'existing dataset v2'
print(ws.datasets['existing dataset'].name)    # IndexError
```

Les paramètres `data_type_id`, `name` et `description` sont facultatifs. Par défaut, ils indiquent leur valeur précédente. Le paramètre `dataframe` est toujours requis.

Si vos données sont déjà sérialisées, utilisez `update_from_raw_data` au lieu de `update_from_dataframe`. Si vous transmettez simplement `raw_data` au lieu de `dataframe`, cela fonctionne de la même manière.

<!-- Images -->
[security]:./media/python-data-access/security.png
[dataset-format]:./media/python-data-access/dataset-format.png
[csv-format]:./media/python-data-access/csv-format.png
[datasets]:./media/python-data-access/datasets.png
[dataset-access-code]:./media/python-data-access/dataset-access-code.png
[ipython-dataset]:./media/python-data-access/ipython-dataset.png
[experiment]:./media/python-data-access/experiment.png
[intermediate-dataset-access-code]:./media/python-data-access/intermediate-dataset-access-code.png
[ipython-intermediate-dataset]:./media/python-data-access/ipython-intermediate-dataset.png
[ipython-histogram]:./media/python-data-access/ipython-histogram.png


<!-- Module References -->
[convert-to-csv]: /azure/machine-learning/studio-module-reference/convert-to-csv
[split]: /azure/machine-learning/studio-module-reference/split-data