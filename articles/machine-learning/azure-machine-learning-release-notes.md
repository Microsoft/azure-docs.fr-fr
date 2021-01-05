---
title: Nouveautés de la version
titleSuffix: Azure Machine Learning
description: Découvrez les dernières mises à jour d’Azure Machine Learning et les kits SDK Python de Machine Learning et de préparation de données.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.author: jmartens
author: j-martens
ms.date: 09/10/2020
ms.openlocfilehash: b11b0b46e57247d09e576795079e20c679e519d5
ms.sourcegitcommit: 6cca6698e98e61c1eea2afea681442bd306487a4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/24/2020
ms.locfileid: "97760110"
---
# <a name="azure-machine-learning-release-notes"></a>Notes de publication d’Azure Machine Learning

Dans cet article, découvrez les versions d’Azure Machine Learning.  Pour obtenir le contenu complet de la référence SDK, consultez la page de référence du [**SDK principal pour Python**](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) d’Azure Machine Learning.

## <a name="2020-12-07"></a>2020-12-07

### <a name="azure-machine-learning-sdk-for-python-v1190"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.19.0
+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Ajout de la prise en charge expérimentale des données de test à AutoMLStep.
    + Ajout de l’implémentation principale initiale de la fonctionnalité d’ingestion de jeux de test.
    + Les références déplacées vers sklearn.externals.joblib dépendent directement de joblib.
    + Introduction d’un nouveau type de tâche AutoML « image-instance-segmentation ».
  + **azureml-automl-runtime**
    + Ajout de l’implémentation principale initiale de la fonctionnalité d’ingestion de jeux de test.
    + Lorsque toutes les chaînes d’une colonne de texte ont une longueur d’exactement un caractère, le caractériseur de mot-gramme TfIdf ne fonctionnera pas, car son générateur de jetons ignore les chaînes contenant moins de deux caractères. Cette modification du code permettra à AutoML de gérer ce cas d’usage.
    + Introduction d’un nouveau type de tâche AutoML « image-instance-segmentation ».
  + **azureml-contrib-automl-dnn-nlp**
    + Demande de tirage (pull request) initiale pour le nouveau package dnn-nlp.
  + **azureml-contrib-automl-dnn-vision**
    + Introduction d’un nouveau type de tâche AutoML « image-instance-segmentation ».
  + **azureml-contrib-automl-pipeline-steps**
    + Ce nouveau package est responsable de la création des étapes requises pour de nombreux scénarios de formation/d’inférence de modèles. Il déplace également le code de formation/d’inférence dans le package azureml.train.automl.runtime afin que tous les correctifs ultérieurs soient automatiquement disponibles par le biais des versions organisées de l’environnement.
  + **azureml-contrib-dataset**
    + Introduction d’un nouveau type de tâche AutoML « image-instance-segmentation ».
  + **azureml-core**
    + Ajout de l’implémentation principale initiale de la fonctionnalité d’ingestion de jeux de test.
    + Correction des avertissements XREF pour la documentation dans le package azureml-core.
    + Correction des chaînes doc pour la fonctionnalité de prise en charge de Command dans le Kit de développement logiciel (SDK).
    + Ajout de la propriété Command à RunConfiguration. Cette fonctionnalité permet aux utilisateurs d’exécuter une commande ou des exécutables réels sur le Kit de développement logiciel (SDK) AzureML.
    + Les utilisateurs peuvent supprimer une expérience vide en fonction de l’ID de cette expérience.
  + **azureml-dataprep**
    + Ajout de la prise en charge des jeux de données pour Spark générés avec Scala 2.12. Cela s’ajoute à la prise en charge existante de la version 2.11.
  + **azureml-mlflow**
    + AzureML-MLflow ajoute des protecteurs dans les scripts distants pour éviter l’arrêt prématuré des exécutions soumises.
  + **azureml-pipeline-core**
    + Correction d’un bogue lors de la définition d’un pipeline par défaut pour le point de terminaison du pipeline créé via l’interface utilisateur.
  + **azureml-pipeline-steps**
    + Ajout de la prise en charge expérimentale des données de test à AutoMLStep.
  + **azureml-tensorboard**
    + Correction des avertissements XREF pour la documentation dans le package azureml-core.
  + **azureml-train-automl-client**
    + Ajout de la prise en charge expérimentale des données de test à AutoMLStep.
    + Ajout de l’implémentation principale initiale de la fonctionnalité d’ingestion de jeux de test.
    + Introduction d’un nouveau type de tâche AutoML « image-instance-segmentation ».
  + **azureml-train-automl-runtime**
    + Ajout de l’implémentation principale initiale de la fonctionnalité d’ingestion de jeux de test.
    + Correction du calcul des explications brutes pour le meilleur modèle AutoML si les modèles AutoML sont formés à l’aide du paramètre validation_size.
    + Les références déplacées vers sklearn.externals.joblib dépendent directement de joblib.
  + **azureml-train-core**
    + HyperDriveRun.get_children_sorted_by_primary_metric() doit désormais se terminer plus rapidement.
    + Amélioration de la gestion des erreurs dans le Kit de développement logiciel (SDK) HyperDrive.
    +  Dépréciation de toutes les classes d’estimateur en faveur de l’utilisation de ScriptRunConfig pour configurer des exécutions d’expériences. Les classes déconseillées sont les suivantes :
        + MMLBaseEstimator
        + Estimateur
        + PyTorch 
        + TensorFlow 
        + Chainer 
        + SKLearn
    + Dépréciation de l’utilisation de Nccl et Gloo comme types d’entrée valides pour les classes Estimator en faveur de l’utilisation de PyTorchConfiguration avec ScriptRunConfig.
    + Dépréciation de l’utilisation de Mpi comme type d’entrée valide pour les classes Estimator en faveur de l’utilisation de MpiConfiguration avec ScriptRunConfig.
    + Ajout de la propriété Command à RunConfiguration. Cette fonctionnalité permet aux utilisateurs d’exécuter une commande ou des exécutables réels sur le Kit de développement logiciel (SDK) AzureML.

    +  Dépréciation de toutes les classes d’estimateur en faveur de l’utilisation de ScriptRunConfig pour configurer des exécutions d’expériences. Les classes déconseillées sont les suivantes : + MMLBaseEstimator + Estimator + PyTorch + TensorFlow + Chainer + SKLearn.
    + Dépréciation de l’utilisation de Nccl et Gloo comme type d’entrée valide pour les classes Estimator en faveur de l’utilisation de PyTorchConfiguration avec ScriptRunConfig. 
    + Dépréciation de l’utilisation de Mpi comme type d’entrée valide pour les classes Estimator en faveur de l’utilisation de MpiConfiguration avec ScriptRunConfig.



## <a name="2020-11-09"></a>09-11-2020

### <a name="azure-machine-learning-sdk-for-python-v1180"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.18.0
+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    +  Amélioration de la gestion des séries chronologiques en autorisant leur remplissage avec un bruit gaussien.
  + **azureml-automl-runtime**
    + Lève une exception ConfigException si une colonne DateTime contient une valeur OutOfBoundsDatetime
    + Amélioration de la gestion des séries chronologiques en autorisant leur remplissage avec un bruit gaussien.
    + S’assurer que chaque colonne de texte peut tirer parti de la transformation char-gram avec la plage n-gram en fonction de la longueur des chaînes de cette colonne de texte
    + Fournir des explications de fonctionnalités brutes pour le meilleur mode pour les expériences AutoML exécutées sur le calcul local de l’utilisateur
  + **azureml-core**
    + Épinglez le package : pyjwt pour éviter d’extraire des versions avec rupture dans les prochaines versions.
    + La création d’une expérience retournera l’expérience active ou la dernière expérience archivée avec le même nom donné, si une telle expérience existe, ou une nouvelle expérience.
    + L’appel de get_experiment par son nom retournera l’expérience active ou la dernière expérience archivée avec le nom donné.
    + Les utilisateurs ne peuvent pas renommer une expérience en cours de réactivation.
    + Message d’erreur amélioré pour inclure des correctifs potentiels lorsqu’un jeu de données est transmis incorrectement à une expérience (par exemple, ScriptRunConfig). 
    + Documentation améliorée pour `OutputDatasetConfig.register_on_complete` afin d’inclure le comportement de ce qui se produit lorsque le nom existe déjà.
    + La spécification des noms d’entrée et de sortie de jeu de données susceptibles d’entrer en conflit avec des variables d’environnement communes génère désormais un avertissement
    + Réaffectation du paramètre `grant_workspace_access` lors de l’inscription des magasins de données. Affectez-lui la valeur `True` pour accéder aux données derrière le réseau virtuel à partir de Machine Learning Studio.
      [En savoir plus](./how-to-enable-studio-virtual-network.md)
    + L’API de service liée est affinée. Au lieu de fournir l’ID de ressource, nous utilisons 3 paramètres distincts sub_id, rg, et name, définis dans la configuration.
    + Pour permettre aux clients de résoudre eux-mêmes les problèmes d’altération des jetons, activez la synchronisation des jetons de l’espace de travail en tant que méthode publique.
    + Cette modification permet d’utiliser une chaîne vide comme valeur pour script_param
  + **azureml-train-automl-client**
    +  Amélioration de la gestion des séries chronologiques en autorisant leur remplissage avec un bruit gaussien.
  + **azureml-train-automl-runtime**
    + Lève une exception ConfigException si une colonne DateTime contient une valeur OutOfBoundsDatetime
    + Ajout de la prise en charge pour fournir des explications de fonctionnalités brutes pour le meilleur mode pour les expériences AutoML exécutées sur le calcul local de l’utilisateur
    + Amélioration de la gestion des séries chronologiques en autorisant leur remplissage avec un bruit gaussien.
  + **azureml-train-core**
    + Cette modification permet d’utiliser une chaîne vide comme valeur pour script_param
  + **azureml-train-restclients-hyperdrive**
    + Le fichier LISEZ-MOI a été modifié pour fournir plus de contexte
  + **azureml-widgets**
    + Ajout de la prise en charge des chaînes à des graphiques/bibliothèque de coordonnées parallèles pour le widget.

## <a name="2020-11-05"></a>05-11-2020

### <a name="data-labeling-for-image-instance-segmentation-polygon-annotation-preview"></a>Étiquetage des données pour la segmentation d’instance d’image (annotation de polygone) (préversion)

Le type de projet de segmentation d’instance d’image (annotations de polygone) dans l’étiquetage des données est maintenant disponible. Les utilisateurs peuvent ainsi dessiner et annoter des polygones autour du contour des objets dans les images. Les utilisateurs pourront affecter une classe et un polygone à chaque objet pertinent dans une image.

En savoir plus sur [l’étiquetage de segmentation d’une instance d’image](how-to-label-images.md).



## <a name="2020-10-26"></a>2020-10-26

### <a name="azure-machine-learning-sdk-for-python-v1170"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.17.0
+ **nouveaux exemples**
  + Un nouveau référentiel d’exemples alimenté par la communauté est disponible à l’adresse https://github.com/Azure/azureml-examples
+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Correction d’un problème lié au déclenchement d’une erreur XGBoostError par get_output.
  + **azureml-automl-runtime**
    + Les fonctionnalités basées sur l’heure ou le calendrier créées par AutoML auront désormais le préfixe.
    + Correction d’une erreur IndexError qui se produit lors de la formation de StackEnsemble pour les jeux de données de classification comportant un grand nombre de classes et sur lesquels le sous-échantillonnage est activé.
    + Correction d’un problème lié à la délivrance de prédictions VotingRegressor inexactes après réajustement du modèle.
  + **azureml-core**
    + Ajout de détails supplémentaires sur la relation entre la configuration de déploiement AKS et les concepts d’Azure Kubernetes Service.
    + Prise en charge des étiquettes des clients d’environnement. L’utilisateur peut étiqueter les environnements et y faire référence par étiquette.
  + **azureml-dataprep**
    + Amélioration du message d’erreur lors de l’utilisation de Spark avec Scala 2.12, actuellement non pris en charge.
  + **azureml-explain-model**
    + Le package azureml-explain-model est officiellement déconseillé
  + **azureml-mlflow**
    + Résolution d’un bogue dans mlflow.projects.run sur le serveur principal azureml, lié à la gestion incorrecte de l’état En cours de finalisation.
  + **azureml-pipeline-core**
    + Ajout de la prise en charge de la création, du recensement et de l’obtention du calendrier de pipeline basé sur un point de terminaison de pipeline.
    +  Amélioration de la documentation de PipelineData.as_dataset avec un exemple d’utilisation non valide. L’utilisation incorrecte de PipelineData.as_dataset provoque désormais le déclenchement d’une erreur ValueException.
    + Modification du notebook de pipelines HyperDriveStep pour enregistrer le meilleur modèle dans un PipelineStep directement après l’exécution de HyperDriveStep.
  + **azureml-pipeline-steps**
    + Modification du notebook de pipelines HyperDriveStep pour enregistrer le meilleur modèle dans un PipelineStep directement après l’exécution de HyperDriveStep.
  + **azureml-train-automl-client**
    + Correction d’un problème lié au déclenchement d’une erreur XGBoostError par get_output.

### <a name="azure-machine-learning-studio-notebooks-experience-october-update"></a>Expérience Azure Machine Learning Studio avec les notebooks (mise à jour d’octobre)
+ **Nouvelles fonctionnalités**
  + [Prise en charge de réseau virtuel complet](./how-to-enable-studio-virtual-network.md)
  + [Mode Focus](./how-to-run-jupyter-notebooks.md#focus-mode)
  + Enregistrer les blocs-notes Ctrl-S
  + Numéros de ligne

+ **Résolutions de bogue et améliorations**
  + Amélioration de la vitesse et de la fiabilité du noyau
  + Mises à jour de l’interface utilisateur du widget Jupyter

## <a name="2020-10-12"></a>12-10-2020

### <a name="azure-machine-learning-sdk-for-python-v1160"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.16.0
+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + AKSWebservice et AKSEndpoints prennent désormais en charge les limites de ressources de processeur et de mémoire au niveau pod. Ces limites facultatives peuvent être utilisées en définissant les indicateurs `--cpu-cores-limit` et `--memory-gb-limit` dans les appels CLI applicables.
  + **azureml-core**
    + Épingler les versions majeures de dépendances directes d’azureml-core
    + AKSWebservice et AKSEndpoints prennent désormais en charge les limites de ressources de processeur et de mémoire au niveau pod. Pour en savoir plus, accédez à [Kubernetes Resources and Limits (Ressources et limites de Kubernetes)](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits).
    + Mise à jour de run.log_table pour autoriser la consignation de lignes individuelles.
    + Ajout de la méthode statique `Run.get(workspace, run_id)` pour récupérer une exécution uniquement à l’aide d’un espace de travail. 
    + Ajout de la méthode d’instance `Workspace.get_run(run_id)` pour récupérer une exécution dans l’espace de travail.
    + Introduction de la propriété de commande dans la configuration de l’exécution, ce qui permet aux utilisateurs d’envoyer des commandes au lieu de scripts et d’arguments.
  + **azureml-interpret**
    + Correction du comportement de l’indicateur is_raw client dans azureml-interpréter
  + **azureml-sdk**
    + `azureml-sdk` prend officiellement en charge Python 3.8.
  + **azureml-train-core**
    + Ajout de l’environnement organisé TensorFlow 2.3
    + Introduction de la propriété de commande dans la configuration de l’exécution, ce qui permet aux utilisateurs d’envoyer des commandes au lieu de scripts et d’arguments.
  + **azureml-widgets**
    + Modification de l’interface pour le widget d’exécution de script.


## <a name="2020-09-28"></a>28/09/2020

### <a name="azure-machine-learning-sdk-for-python-v1150"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.15.0
+ **Résolutions de bogue et améliorations**
  + **azureml-contrib-interpret**
    + Explicateur LIME déplacé du package azureml-contrib-interpret vers le package interpret-community et explicateur d’images supprimé du package azureml-contrib-interpret
    + Tableau de bord de visualisation supprimé du package azureml-contrib-interpret, client d’explication déplacé vers le package azureml-interpret et déconseillé dans le package azureml-contrib-interpret et notebooks mis à jour pour refléter l’amélioration de l’API
    + Correction des descriptions de packages PyPI pour les packages azureml-interpret, azureml-explain-model, azureml-contrib-interpret et azureml-tensorboard
  + **azureml-contrib-notebook**
    + Dépendance nbcovert épinglée à < 6 afin que Papermill 1.x continue à fonctionner.
  + **azureml-core**
    + Ajout de paramètres à TensorflowConfiguration et MpiConfiguration pour permettre une initialisation plus rationalisée des attributs de classe sans que l’utilisateur soit obligé de définir chaque attribut individuel. Ajout d’une classe PyTorchConfiguration pour la configuration des tâches PyTorch distribuées dans ScriptRunConfig.
    + Version du package azure-mgmt-resource épinglée pour corriger l’erreur d’authentification.
    + Prise en charge du déploiement Triton sans code.
    + Les répertoires de sortie spécifiés dans Run.start_logging() sont désormais suivis lors de l’utilisation de l’exécution dans des scénarios interactifs. Les fichiers suivis sont visibles sur ML Studio lorsque la fonction Run.complete() est appelée.
    + L’encodage de fichier peut maintenant être spécifié lors de la création du jeu de données avec `Dataset.Tabular.from_delimited_files` et `Dataset.Tabular.from_json_lines_files` en transmettant l’argument `encoding`. Les encodages pris en charge sont « utf8 », « iso88591 », « latin1 », « ascii », « utf16 », « utf32 », « utf8bom » et « windows1252 ».
    + Résolution de bogue où l’objet d’environnement n’est pas transmis au constructeur ScriptRunConfig.
    + Mise à jour de la fonction Run.cancel() pour autoriser l’annulation d’une exécution locale à partir d’un autre ordinateur.
  + **azureml-dataprep**
    +  Résolution des problèmes d’expiration du délai d’attente lors du montage du jeu de données.
  + **azureml-explain-model**
    + Correction des descriptions de packages PyPI pour les packages azureml-interpret, azureml-explain-model, azureml-contrib-interpret et azureml-tensorboard
  + **azureml-interpret**
    + Tableau de bord de visualisation supprimé du package azureml-contrib-interpret, client d’explication déplacé vers le package azureml-interpret et déconseillé dans le package azureml-contrib-interpret et notebooks mis à jour pour refléter l’amélioration de l’API
    + Mise à jour du package azureml-interpret pour dépendre du package interpret-community 0.15.0
    + Correction des descriptions de packages PyPI pour les packages azureml-interpret, azureml-explain-model, azureml-contrib-interpret et azureml-tensorboard
  + **azureml-pipeline-core**
    +  Résolution du problème de pipeline avec `OutputFileDatasetConfig` où le système peut cesser de répondre lorsque `register_on_complete` est appelé avec le paramètre `name` défini sur un nom de jeu de données préexistant.
  + **azureml-pipeline-steps**
    + Suppression des notebooks Databricks obsolètes.
  + **azureml-tensorboard**
    + Correction des descriptions de packages PyPI pour les packages azureml-interpret, azureml-explain-model, azureml-contrib-interpret et azureml-tensorboard
  + **azureml-train-automl-runtime**
    + Tableau de bord de visualisation supprimé du package azureml-contrib-interpret, client d’explication déplacé vers le package azureml-interpret et déconseillé dans le package azureml-contrib-interpret et notebooks mis à jour pour refléter l’amélioration de l’API
  + **azureml-widgets**
    + Tableau de bord de visualisation supprimé du package azureml-contrib-interpret, client d’explication déplacé vers le package azureml-interpret et déconseillé dans le package azureml-contrib-interpret et notebooks mis à jour pour refléter l’amélioration de l’API

## <a name="2020-09-21"></a>2020-09-21

### <a name="azure-machine-learning-sdk-for-python-v1140"></a>Kit SDK Azure Machine Learning pour Python v1.14.0
+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Le profilage de grille est supprimé du SDK et n’est plus pris en charge.
  + **azureml-accel-models**
    + Le package azureml-accel-models prend à présent en charge Tensorflow 2.x
  + **azureml-automl-core**
    + Ajout de la gestion des erreurs dans get_output dans les cas où les versions locales de pandas/sklearn ne correspondent pas à celles utilisées lors de l’apprentissage
  + **azureml-automl-runtime**
    + Correction d’un bogue dans lequel les itérations AutoArima échouent avec un PredictionException et le message suivant : « Une défaillance silencieuse s’est produite pendant la prédiction. »
  + **azureml-cli-common**
    + Le profilage de grille a été supprimé du SDK et n’est plus pris en charge.
  + **azureml-contrib-server**
    + Mise à jour de la description du package pour la page de présentation pypi.
  + **azureml-core**
    + Le profilage de grille a été supprimé du SDK et n’est plus pris en charge.
    + Réduction du nombre de messages d’erreur lors de l’échec de la récupération de l’espace de travail.
    + Ne pas afficher d’avertissement en cas d’échec de la récupération des métadonnées
    + Nouvelle étape Kusto et nouvelle cible de calcul Kusto.
    + Mise à jour du document pour le paramètre SKU. Suppression de la référence SKU de la fonctionnalité de mise à jour de l’espace de travail dans l’interface CLI et le SDK.
    + Mise à jour de la description du package pour la page de présentation Pypi.
    + Mise à jour de la documentation pour les environnements AzureML.
    + Exposition des paramètres des ressources managées par le service pour l’espace de travail AML dans le SDK.
  + **azureml-dataprep**
    + Activation de l’autorisation Execute sur les fichiers pour le montage du jeu de données.
  + **azureml-mlflow**
    + Mise à jour de la documentation AzureML MLflow et des exemples de notebooks 
    + Nouvelle prise en charge des projets MLflow avec le backend AzureML
    + Prise en charge du registre de modèles MLflow
    + Ajout de la prise en charge d’Azure RBAC pour les opérations AzureML-MLflow 
    
  + **azureml-pipeline-core**
    + Amélioration de la documentation des méthodes PipelineOutputFileDataset.parse_*.
    + Nouvelle étape Kusto et nouvelle cible de calcul Kusto.
    + La propriété Swaggerurl fournie pour l’entité de point de terminaison de pipeline via cet utilisateur peut voir la définition de schéma pour le point de terminaison de pipeline publié.
  + **azureml-pipeline-steps**
    + Nouvelle étape Kusto et nouvelle cible de calcul Kusto.
  + **azureml-telemetry**
    + Mise à jour de la description du package pour la page de présentation pypi.
  + **azureml-train**
    + Mise à jour de la description du package pour la page de présentation pypi.
  + **azureml-train-automl-client**
    + Ajout de la gestion des erreurs dans get_output dans les cas où les versions locales de pandas/sklearn ne correspondent pas à celles utilisées lors de l’apprentissage
  + **azureml-train-core**
    + Mise à jour de la description du package pour la page de présentation pypi.
    
## <a name="2020-08-31"></a>2020-08-31

### <a name="azure-machine-learning-sdk-for-python-v1130"></a>Kit SDK Azure Machine Learning pour Python v1.13.0
+ **Fonctionnalités préliminaires**
  + **azureml-core** Avec la nouvelle fonctionnalité de jeux de données de sortie, vous pouvez mettre à jour dans le stockage cloud, notamment les objets blob, ADLS Gen 1, ADLS Gen 2 et FileShare. Vous pouvez configurer l’emplacement de sortie des données, le mode de sortie des données (montage ou chargement), l’enregistrement des données de sortie pour une réutilisation ultérieure ainsi que le partage et la transmission des données intermédiaires entre les étapes de pipeline en toute transparence. Cela permet la reproductibilité, le partage, empêche la duplication des données et génère des gains de productivité et de rentabilité. [Découvrez son utilisation](/python/api/azureml-core/azureml.data.output_dataset_config.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py)
    
+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Ajout du fichier validated_{platform}_requirements.txt pour épingler toutes les dépendances PIP pour AutoML.
    + Cette version prend en charge des modèles de plus de 4 Go.
    + Mise à niveau des dépendances AutoML : `scikit-learn` (maintenant 0.22.1), `pandas` (maintenant 0.25.1) et `numpy` (Now 1.18.2).
  + **azureml-automl-runtime**
    + Définition de horovod pour que les DNN de texte utilisent toujours la compression fp16.
    + Cette version prend en charge des modèles de plus de 4 Go.
    + Correction du problème où AutoML échoue avec ImportError : impossible d’importer le nom `RollingOriginValidator`.
    + Mise à niveau des dépendances AutoML : `scikit-learn` (maintenant 0.22.1), `pandas` (maintenant 0.25.1) et `numpy` (Now 1.18.2).
  + **azureml-contrib-automl-dnn-forecasting**
    + Mise à niveau des dépendances AutoML : `scikit-learn` (maintenant 0.22.1), `pandas` (maintenant 0.25.1) et `numpy` (Now 1.18.2).
  + **azureml-contrib-fairness**
    + Brève description d’azureml-contrib-fairness.
  + **azureml-contrib-pipeline-steps**
    + Ajout d’un message indiquant que ce package est déprécié et que l’utilisateur devrait utiliser azureml-pipeline-steps à la place.
  + **azureml-core**
    + Ajout de la commande de clé de liste pour l’espace de travail.
    + Ajout d’un paramètre de balises dans le kit de développement logiciel (SDK) de l’espace de travail et l’interface CLI.
    + Correction du bogue dans lequel l’envoi d’une exécution enfant avec un jeu de données échoue en raison de `TypeError: can't pickle _thread.RLock objects`.
    + Ajout de la valeur par défaut/documentation de page_count pour la liste de modèles().
    + Modification de l’interface CLI et du kit de développement logiciel (SDK) pour prendre le paramètre adbworkspace et ajout d’un espace de travail adb lier/dissocier le testeur.
    + Correction du bogue dans Dataset.update qui entraînait la mise à jour de la version la plus récente du jeu de données et non de la version de la mise à jour du jeu de données qui était appelée. 
    + Correction du bogue dans Dataset.get_by_name qui affichait les balises pour la version la plus récente du jeu de données même lorsqu’une version antérieure spécifique était récupérée.
  + **azureml-interpret**
    + Ajout des sorties de probabilité aux explicateurs de scoring SHAP dans azureml-interpret en fonction du paramètre shap_values_output de l’explicateur d’origine.
  + **azureml-pipeline-core**
    + Documentation de `PipelineOutputAbstractDataset.register` améliorée.
  + **azureml-train-automl-client**
    + Mise à niveau des dépendances AutoML : `scikit-learn` (maintenant 0.22.1), `pandas` (maintenant 0.25.1) et `numpy` (Now 1.18.2).
  + **azureml-train-automl-runtime**
    + Mise à niveau des dépendances AutoML : `scikit-learn` (maintenant 0.22.1), `pandas` (maintenant 0.25.1) et `numpy` (Now 1.18.2).
  + **azureml-train-core**
    + Les utilisateurs doivent maintenant fournir un argument hyperparameter_sampling valide lors de la création d’une configuration HyperDriveConfig. De plus, la documentation de HyperDriveRunConfig a été modifiée pour informer les utilisateurs de la dépréciation de HyperDriveRunConfig.
    + Restauration de la version par défaut de PyTorch vers 1.4.
    + Ajout d’images PyTorch 1.6 et Tensorflow 2.2 et d’un environnement organisé.

### <a name="azure-machine-learning-studio-notebooks-experience-august-update"></a>Expérience Azure Machine Learning Studio avec les notebooks (mise à jour d’août)
+ **Nouvelles fonctionnalités**
  + Nouvelle page d’accueil de prise en main 
  
+ **Fonctionnalités préliminaires**
    + Fonctionnalité de regroupement dans les notebooks. Grâce à la fonctionnalité de [regroupement](./how-to-run-jupyter-notebooks.md#clean-your-notebook-preview), les utilisateurs peuvent désormais facilement nettoyer les notebooks. Le regroupement utilise une analyse de dépendance automatisée de votre notebook, ce qui garantit que le code essentiel est conservé, tout en supprimant les éléments non pertinents.

+ **Résolutions de bogue et améliorations**
  + Amélioration de la vitesse et de la fiabilité
  + Résolution des bogues en mode sombre
  + Résolution des bogues de défilement de sortie
  + L’exemple de recherche cherche désormais dans tout le contenu de la totalité des fichiers de l’exemples de référenties de notebooks Azure Machine Learning
  + Les cellules R à plusieurs lignes peuvent maintenant s’exécuter
  + « Je fais confiance au contenu de ce fichier » est maintenant vérifié automatiquement après la première exécution
  + Amélioration de la boîte de dialogue de résolution des conflits avec une option « Créer une copie »
  
## <a name="2020-08-17"></a>17/08/2020

### <a name="azure-machine-learning-sdk-for-python-v1120"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.12.0

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Ajout des paramètres image_name et image_label à Model.package() pour permettre le changement de nom de l’image de package générée.
  + **azureml-automl-core**
    + AutoML génère un nouveau code d’erreur à partir de dataprep lorsque le contenu est modifié lors de la lecture.
  + **azureml-automl-runtime**
    + Ajout d’alertes pour l’utilisateur lorsque les données contiennent des valeurs manquantes, mais que l’ingénierie de caractéristiques est désactivée.
    + Correction des échecs d’exécution enfant lorsque les données contiennent nan et que l’ingénierie de caractéristiques est désactivée.
    + AutoML génère un nouveau code d’erreur à partir de dataprep lorsque le contenu est modifié lors de la lecture.
    + Mise à jour de la normalisation pour les métriques de prévision qui se produisent par grain.
    + Amélioration du calcul des quantiles de prévision lorsque les fonctionnalités de recherche en arrière sont désactivées.
    + Correction de la gestion des matrices éparses booléennes lors du calcul des explications après AutoML.
  + **azureml-core**
    + La nouvelle méthode `run.get_detailed_status()` affiche à présent l’explication détaillée de l’état actuel de l’exécution. Elle n’indique actuellement qu’une explication de l’état `Queued`.
    + Ajout des paramètres image_name et image_label à Model.package() pour permettre le changement de nom de l’image de package générée.
    + Nouvelle méthode `set_pip_requirements()` permettant de définir toute la section PIP dans [`CondaDependencies`](/python/api/azureml-core/azureml.core.conda_dependencies.condadependencies?preserve-view=true&view=azure-ml-py) en une seule fois.
    + Activation de l’inscription du magasin de données ADLS Gen2 sans informations d’identification.
    + Amélioration du message d’erreur lors de la tentative de téléchargement ou de montage d’un type incorrect de jeu de données.
    + Mise à jour de l’exemple de notebook de filtrage des jeux de données de série chronologique avec d’autres exemples de partition_timestamp qui fournit l’optimisation de filtre.
    + Modification du SDK et de l’interface CLI pour accepter subscriptionId, resourceGroup, workspaceName, peConnectionName comme paramètres au lieu d’ArmResourceId lors de la suppression de la connexion au point de terminaison privé.
    + L’élément décoratif expérimental affiche le nom de la classe pour faciliter l’identification.
    + Les descriptions des ressources à l’intérieur des modèles ne sont plus générées automatiquement en fonction d’une exécution.
  + **azureml-datadrift**
    + L’API create_from_model dans DataDriftDetector est marquée comme étant bientôt déconseillée.
  + **azureml-dataprep**
    + Amélioration du message d’erreur lors de la tentative de téléchargement ou de montage d’un type incorrect de jeu de données.
  + **azureml-pipeline-core**
    + Correction d’un bogue lors de la désérialisation du graphique de pipeline qui contient des jeux de données inscrits.
  + **azureml-pipeline-steps**
    + RScriptStep prend en charge RSection à partir d’azureml.core.environment.
    + Suppression du paramètre passthru_automl_config de l’API publique `AutoMLStep` et conversion en paramètre interne uniquement.
  + **azureml-train-automl-client**
    + Suppression des exécutions de l’environnement managé asynchrone local d’AutoML. Toutes les exécutions locales sont exécutées dans l’environnement dans lequel l’exécution a été lancé.
    + Correction des problèmes d’instantané lors de l’envoi d’exécutions AutoML sans scripts fournis par l’utilisateur.
    + Correction des échecs d’exécution enfant lorsque les données contiennent nan et que l’ingénierie de caractéristiques est désactivée.
  + **azureml-train-automl-runtime**
    + AutoML génère un nouveau code d’erreur à partir de dataprep lorsque le contenu est modifié lors de la lecture.
    + Correction des problèmes d’instantané lors de l’envoi d’exécutions AutoML sans scripts fournis par l’utilisateur.
    + Correction des échecs d’exécution enfant lorsque les données contiennent nan et que l’ingénierie de caractéristiques est désactivée.
  + **azureml-train-core**
    + Ajout de la prise en charge de la spécification des options PIP (par exemple, --extra-index-url) dans le fichier d’exigences PIP transmis à un [`Estimator`](/python/api/azureml-train-core/azureml.train.estimator.estimator?preserve-view=true&view=azure-ml-py) par le biais du paramètre `pip_requirements_file`.


## <a name="2020-08-03"></a>03-08-2020

### <a name="azure-machine-learning-sdk-for-python-v1110"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.11.0

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Correction de l’infrastructure de modèle et infrastructure du modèle non passée dans l’objet d’exécution dans le chemin d’inscription du modèle CLI
    + Correction de la commande d’affichage d’identité amlcompute de l’interface CLI pour afficher l’ID de locataire et l’ID de principal 
  + **azureml-train-automl-client**
    + Ajout de get_best_child() à AutoMLRun pour extraire la meilleure exécution d’enfant pour une exécution d’AutoML sans télécharger le modèle associé.
    + Ajout de l’objet ModelProxy ajouté qui autorise l’exécution de prédictions ou de prévisions sur un environnement de formation à distance sans télécharger le modèle localement.
    + Les exceptions non gérées dans AutoML pointent désormais vers une page HTTP de problèmes connus, contenant des informations supplémentaires sur les erreurs.
  + **azureml-core**
    + Les noms de modèles peuvent compter 255 caractères.
    + Modification du type d’objet de retour Environment.get_image_details(). La classe `DockerImageDetails` a remplacé `dict`, les détails de l’image sont disponibles dans les propriétés de la nouvelle classe. Les modifications sont à compatibilité descendante.
    + Correction de bogue pour Environment.from_pip_requirements() afin de conserver la structure des dépendances
    + Correction d’un bogue où log_list échouait si un int et double étaient inclus dans la même liste.
    + Lorsque vous activez un lien privé sur un espace de travail existant, notez que, si des cibles de calcul sont associées à l’espace de travail, elles ne fonctionneront pas si elles ne se trouvent pas derrière le même réseau virtuel que le point de terminaison privé de l’espace de travail.
    + `as_named_input` rendu facultatif lors de l’utilisation de jeux de données dans des expériences et ajout de `as_mount` et `as_download` à `FileDataset`. Le nom d’entrée est généré automatiquement en cas d’appel de `as_mount` ou `as_download`.
  + **azureml-automl-core**
    + Les exceptions non gérées dans AutoML pointent désormais vers une page HTTP de problèmes connus, contenant des informations supplémentaires sur les erreurs.
    + Ajout de get_best_child() à AutoMLRun pour extraire la meilleure exécution d’enfant pour une exécution d’AutoML sans télécharger le modèle associé.
    + Ajout de l’objet ModelProxy ajouté qui autorise l’exécution de prédictions ou de prévisions sur un environnement de formation à distance sans télécharger le modèle localement.
  + **azureml-pipeline-steps**
    + Ajout des indicateurs `enable_default_model_output` et `enable_default_metrics_output` à `AutoMLStep`. Ces indicateurs permettent d’activer/désactiver les sorties par défaut.


## <a name="2020-07-20"></a>20-07-2020

### <a name="azure-machine-learning-sdk-for-python-v1100"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.10.0

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Lors de l’utilisation de AutoML, si un chemin d’accès est passé à l’objet AutoMLConfig et qu’il n’existe pas déjà, il est automatiquement créé.
    + Les utilisateurs peuvent désormais spécifier une fréquence de série chronologique pour les tâches de prévision à l’aide du paramètre `freq`.
  + **azureml-automl-runtime**
    + Lors de l’utilisation de AutoML, si un chemin d’accès est passé à l’objet AutoMLConfig et qu’il n’existe pas déjà, il est automatiquement créé.
    + Les utilisateurs peuvent désormais spécifier une fréquence de série chronologique pour les tâches de prévision à l’aide du paramètre `freq`.
    + Les prévisions d’AutoML prennent désormais en charge la propagation d’évaluation qui s’applique quand la longueur d’un jeu de test ou de validation dépasse l’horizon d’entrée, et que la valeur y_pred connue est utilisée comme contexte de prévision.
  + **azureml-core**
    + Des messages d’avertissement sont imprimés si aucun fichier n’a été téléchargé à partir du magasin de données dans une exécution.
    + Ajout de documentation pour `skip_validation` à la `Datastore.register_azure_sql_database method`.
    + Les utilisateurs doivent opérer une mise à niveau vers le Kit de développement logiciel (SDK) v1.10.0 ou version ultérieure pour créer un point de terminaison privé approuvé automatiquement. Cela inclut la ressource Blocs-notes utilisable derrière le réseau virtuel.
    + Exposer NotebookInfo dans la réponse à la commande Obtenir l’espace de travail.
    + Modifications apportées pour que les appels répertorient les cibles de calcul et que l’obtention de la cible de calcul réussisse sur une exécution à distance. Les fonctions du Kit de développement logiciel (SDK) pour obtenir une cible de calcul et répertorier les cibles de cible de calcul de l’espace de travail fonctionnent désormais dans des exécutions à distance.
    + Ajout de messages de dépréciation aux descriptions de classe pour les classes azureml.core.image.
    + Levée d’exception et nettoyage d’espace de travail et de ressources dépendantes si la création de point de terminaison privé d’espace de travail échoue.
    + Prise en charge de la mise à niveau de référence (SKU) d’espace de travail dans la méthode de mise à jour d’espace de travail.
  + **azureml-datadrift**
    + Mise à jour de la version de matplotlib de 3.0.2 à 3.2.1 pour prendre en charge Python 3.8.
  + **azureml-dataprep**
    + Ajout de la prise en charge de sources de données URL web avec une demande `Range` ou `Head`. 
    + Amélioration de la stabilité améliorée pour le montage et le téléchargement de jeux de données de fichiers.
  + **azureml-train-automl-client**
    + Résolution des problèmes liés à la suppression de `RequirementParseError` de setuptools.
    + Utilisation de Docker au lieu de Conda pour les exécutions locales soumises à l’aide de « compute_target=’local’ »
    + Correction de la durée d’itération imprimée sur la console. Auparavant, la durée de l’itération était parfois imprimée comme heure de fin d’exécution moins l’heure de création de l’exécution. Elle a été corrigée en heure de fin d’exécution moins l’heure de début de l’exécution.
    + Lors de l’utilisation de AutoML, si un chemin d’accès est passé à l’objet AutoMLConfig et qu’il n’existe pas déjà, il est automatiquement créé.
    + Les utilisateurs peuvent désormais spécifier une fréquence de série chronologique pour les tâches de prévision à l’aide du paramètre `freq`.
  + **azureml-train-automl-runtime**
    + Amélioration de la sortie de la console lorsque les meilleures explications du modèle échouent.
    + Changement de nom du paramètre d’entrée « backlist_models » en « blocked_models ».
      + Changement de nom du paramètre d’entrée « whitelist_models » en « allowed_models ».
    + Les utilisateurs peuvent désormais spécifier une fréquence de série chronologique pour les tâches de prévision à l’aide du paramètre `freq`.

  
## <a name="2020-07-06"></a>2020-07-06

### <a name="azure-machine-learning-sdk-for-python-v190"></a>Kit SDK Azure Machine Learning pour Python v1.9.0

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Remplacement de get_model_path() par la variable d'environnement AZUREML_MODEL_DIR dans le script de scoring généré automatiquement par AutoML. Ajout des données de télémétrie pour suivre les défaillances lors d’init().
    + Suppression de la possibilité de spécifier `enable_cache` dans le cadre d’AutoMLConfig
    + Correction d’un bogue où les exécutions pouvaient échouer avec des erreurs de service pendant des exécutions de prévisions spécifiques
    + Amélioration de la gestion des erreurs pour des modèles spécifiques pendant `get_output`
    + Correction de l’appel à fitted_model.fit(X, y) pour la classification avec le transformateur y
    + Activation de l’imputation de remplissage prédictif personnalisée pour les tâches de prévision
    + Une nouvelle classe ForecastingParameters sera utilisée à la place des paramètres de prévision dans un format dict
    + Détection automatique du retard de la cible améliorée
    + Ajout d’une disponibilité limitée de caractérisation à plusieurs nœuds distribuée sur plusieurs GPU avec BERT
  + **azureml-automl-runtime**
    + Prophet effectue désormais une modélisation saisonnière additive plutôt que multiplicative.
    + Résolution du problème lorsque les grains courts, avec des fréquences différentes de celles des grains longs, entraînaient des échecs d’exécution.
  + **azureml-contrib-automl-dnn-vision**
    + Collecter les statistiques système/GPU et les moyennes des journaux pour l’apprentissage et le scoring
  + **azureml-contrib-mir**
    + Ajout de la prise en charge de l’indicateur enable-app-insights dans ManagedInferencing
  + **azureml-core**
    + Un paramètre de validation pour ces API en permettant d’ignorer la validation lorsque la source de données n’est pas accessible depuis le calcul actuel.
      + TabularDataset.time_before(end_time, include_boundary=True, validate=True)
      + TabularDataset.time_after(start_time, include_boundary=True, validate=True)
      + TabularDataset.time_recent(time_delta, include_boundary=True, validate=True)
      + TabularDataset.time_between(start_time, end_time, include_boundary=True, validate=True)
    + Ajout de la prise en charge du filtrage des frameworks pour la liste de modèles et ajout de l’exemple NCD AutoML dans le bloc-notes
    + Pour Datastore.register_azure_blob_container et Datastore.register_azure_file_share (les seules les options qui prennent en charge le jeton SAS), nous avons mis à jour les chaînes de documentation du champ `sas_token` pour inclure les exigences d’autorisation minimales pour les scénarios de lecture et d’écriture typiques.
    + Dépréciation du paramètre _with_auth dans ws.get_mlflow_tracking_uri()
  + **azureml-mlflow**
    + Ajout de la prise en charge du déploiement de modèles file:// locaux avec AzureML-MLflow
    + Dépréciation du paramètre _with_auth dans ws.get_mlflow_tracking_uri()
  + **azureml-opendatasets**
    + Les jeux de données de suivi du COVID-19 récemment publiés sont désormais disponibles avec le kit de développement logiciel (SDK)
  + **azureml-pipeline-core**
    + Avertissement de déconnexion lorsque « azureml-defaults » n’est pas inclus dans le cadre de pip-dependency
    + Amélioration du rendu des notes.
    + Ajout de la prise en charge des sauts de ligne entre guillemets lors de l’analyse des fichiers délimités dans PipelineOutputFileDataset.
    + La classe PipelineDataset est déconseillée. Pour plus d’informations, consultez https://aka.ms/dataset-deprecation. Découvrez comment utiliser le jeu de données avec le pipeline ; consultez https://aka.ms/pipeline-with-dataset.
  + **azureml-pipeline-steps**
    + Mises à jour de la documentation pour azureml-pipeline-steps.
    +  Ajout de la possibilité dans le `load_yaml()` de ParallelRunConfig pour les utilisateurs de définir des environnements inlined avec le reste de la configuration ou dans un fichier distinct
  + **azureml-train-automl-client**.
    + Suppression de la possibilité de spécifier `enable_cache` dans le cadre d’AutoMLConfig
  + **azureml-train-automl-runtime**
    + Ajout d’une disponibilité limitée de caractérisation à plusieurs nœuds distribuée sur plusieurs GPU avec BERT.
    + Ajout de la gestion des erreurs pour les packages incompatibles dans les exécutions de Machine Learning automatisées basées sur ADB.
  + **azureml-widgets**
    + Mises à jour de la documentation pour azureml-widgets.

  
## <a name="2020-06-22"></a>2020-06-22

### <a name="azure-machine-learning-sdk-for-python-v180"></a>Kit SDK Azure Machine Learning pour Python v1.8.0
  
  + **Fonctionnalités préliminaires**
    + **azureml-contrib-fairness** Le package `azureml-contrib-fairness` offre une intégration entre l’évaluation d’équité open source et le package d’atténuation du manque d’équité [Fairlearn](https://fairlearn.github.io) et Azure Machine Learning Studio. En particulier, le package permet de charger des tableaux de bord d’évaluation d’équité du modèle dans le cadre d’une exécution AzureML et de les afficher dans Azure Machine Learning Studio

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Prise en charge de l’obtention des journaux du conteneur init.
    + Ajout de nouvelles commandes CLI pour gérer ComputeInstance
  + **azureml-automl-core**
    + Les utilisateurs peuvent à présent activer l’itération d’ensemble d’empilement pour les tâches de série chronologique avec un avertissement indiquant qu’un dépassement est possible.
    + Ajout d’un nouveau type d’exception utilisateur qui est déclenché si le contenu du magasin de cache a été falsifié
  + **azureml-automl-runtime**
    + Le balayage d’équilibrage de classe n’est plus activé si l’utilisateur désactive la caractérisation.  
  + **azureml-contrib-itp**
    + Le type de calcul CmAks est pris en charge. Vous pouvez attacher votre propre cluster AKS à l’espace de travail pour le travail de formation.
  + **azureml-contrib-notebook**
    + Améliorations apportées à la documentation du package azureml-contrib-notebook.
  + **azureml-contrib-pipeline-steps**
    + Améliorations apportées à la documentation du package azureml-contrib--pipeline-steps.
  + **azureml-core**
    + Ajout des fonctions set_connection, get_connection, list_connections et delete_connection pour que le client opère sur la ressource de connexion de l’espace de travail
    + Mises à jour de la documentation du package zureml-coore/azureml.exceptions.
    + Mises à jour de la documentation du package azureml-core.
    + Mises à jour de la documentation de la classe ComputeInstance.
    + Améliorations à la documentation du package azureml-core/azureml.core.compute.
    + Améliorations à la documentation des classes liées à webservice dans azureml-core.
    + Prise en charge du magasin de données sélectionné par l’utilisateur pour stocker les données de profilage
    + Ajout des propriétés expand et page_count pour l’API de liste de modèles
    + Correction du bogue où la suppression de la propriété overwrite entraînait l’échec de l’exécution envoyée avec une erreur de désérialisation.
    + Correction de la structure de dossiers incohérente lors du téléchargement ou du montage d’un FileDataset référençant un seul fichier.
    + Le chargement d’un jeu de données de fichiers parquet sur to_spark_dataframe est désormais plus rapide et prend en charge tous les types de données SQL parquet et Spark.
    + Prise en charge de l’obtention des journaux du conteneur init.
    + Les exécutions AutoML sont désormais marquées comme étant des exécutions enfants de l’étape d’exécution parallèle.
  + **azureml-datadrift**
    + Améliorations apportées à la documentation du package azureml-contrib-notebook.
  + **azureml-dataprep**
    + Le chargement d’un jeu de données de fichiers parquet sur to_spark_dataframe est désormais plus rapide et prend en charge tous les types de données SQL parquet et Spark.
    + Meilleure gestion de la mémoire en cas de problème OutOfMemory pour to_pandas_dataframe.
  + **azureml-interpret**
    + Mise à niveau d'azureml-interpret pour utiliser interpret-community version 0.12.*
  + **azureml-mlflow**
    + Améliorations apportées à la documentation de azureml-mlflow.
    + Ajout de la prise en charge du registre de modèle AML avec MLFlow.
  + **azureml-opendatasets**
    + Ajout de la prise en charge de Python 3.8
  + **azureml-pipeline-core**
    + Mise à jour de la documentation de `PipelineDataset` pour clarifier qu’il s’agit d’une classe interne.
    + Mises à jour à ParallelRunStep pour accepter plusieurs valeurs pour un argument, par exemple : « --group_column_names », « col1 », « col2 », « col3 »
    + Suppression de l’exigence passthru_automl_config pour l’utilisation des données intermédiaires avec AutoMLStep dans les pipelines.
  + **azureml-pipeline-steps**
    + Améliorations apportées à la documentation du package azureml-pipeline-steps.
    + Suppression de l’exigence passthru_automl_config pour l’utilisation des données intermédiaires avec AutoMLStep dans les pipelines.
  + **azureml-telemetry**
    + Améliorations apportées à la documentation de azureml-telemetry.
  + **azureml-train-automl-client**
    + Correction d’un bogue à cause duquel `experiment.submit()` appelait deux fois sur un objet `AutoMLConfig` avec un comportement différent.
    + Les utilisateurs peuvent maintenant activer l’itération d’ensemble d’empilement pour les tâches de séries chronologiques avec un avertissement indiquant qu’un dépassement est possible.
    + Modification du comportement d’exécution d’AutoML pour déclencher UserErrorException si le service lève une erreur utilisateur
    + Résout un bogue à cause duquel azureml_automl.log n’était pas généré ou ne contenait pas les journaux lors de l’exécution d’une expérience AutoML sur une cible de calcul à distance.
    + Pour les jeux de données de classification avec des classes déséquilibrées, nous appliquons l’équilibrage de poids ; si le nettoyeur de fonctionnalités détermine un déséquilibre pour les données sous-échantillonnées, l’équilibrage de poids améliore les performances de la tâche de classification d’un certain seuil.
    + Les exécutions AutoML sont désormais marquées comme étant des exécutions enfants de l’étape d’exécution parallèle.
  + **azureml-train-automl-runtime**
    + Modification du comportement d’exécution d’AutoML pour déclencher UserErrorException si le service lève une erreur utilisateur
    + Les exécutions AutoML sont désormais marquées comme étant des exécutions enfants de l’étape d’exécution parallèle.

  
## <a name="2020-06-08"></a>2020-06-08

### <a name="azure-machine-learning-sdk-for-python-v170"></a>Kit SDK Azure Machine Learning pour Python v1.7.0

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Suppression du profilage de modèle de la contribution Mir en nettoyant les commandes CLI et les dépendances de package ; le profilage de modèle est disponible dans core.
    + Mise à niveau de la version minimale de l’interface Azure CLI vers 2.3.0
  + **azureml-automl-core**
    + Message d’exception plus clair à l’étape de caractérisation fit_transform() en raison de paramètres de transformateur personnalisés.
    + Ajoutez la prise de plusieurs langages pour les modèles de transformateur de Deep Learning tels que BERT dans le Machine Learning automatisé.
    + Suppression du paramètre lag_length déconseillé de la documentation.
    + La documentation sur les paramètres de prévision a été améliorée. Le paramètre lag_length a été déconseillé.
  + **azureml-automl-runtime**
    + Correction de l’erreur déclenchée lorsque l’une des colonnes catégoriques est vide dans l’heure de prévision/test.
    + Résolution des échecs d’exécution lorsque les fonctionnalités lookback sont activées et que les données contiennent des grains courts.
    + Correction du problème lié à un message d’erreur d’index de temps dupliqué lorsque les retards ou fenêtres dynamiques étaient définis sur « auto ».
    + Correction du problème avec les modèles Prophet et ARIMA sur les jeux de données contenant les fonctionnalités lookback.
    + Ajout de la prise en charge des dates antérieures à 1677-09-21 et ultérieures à 2262-04-11 dans les colonnes autres que la date et l’heure dans les tâches de prévision. Amélioration des messages d’erreur.
    + La documentation sur les paramètres de prévision a été améliorée. Le paramètre lag_length a été déconseillé.
    + Message d’exception plus clair à l’étape de caractérisation fit_transform() en raison de paramètres de transformateur personnalisés.
    + Ajoutez la prise de plusieurs langages pour les modèles de transformateur de Deep Learning tels que BERT dans le Machine Learning automatisé.
    + Les opérations de cache qui aboutissent à des OSErrors génèrent une erreur utilisateur.
    + Ajout de vérifications pour s’assurer que les données de formation et de validation ont le même nombre et le même ensemble de colonnes
    + Correction du problème avec le script de notation AutoML généré automatiquement lorsque les données contiennent des guillemets
    + Activation des explications pour AutoML Prophet et des modèles d’ensemble qui contiennent le modèle Prophet.
    + Un problème client récent a révélé un bogue de site dans lequel nous consignions les messages dans le cadre de l’équilibrage des classes, même lorsque la logique d’équilibrage de classe n’était pas correctement activée. Suppression de ces journaux/messages avec cette demande de tirage.
  + **azureml-cli-common**
    + Suppression du profilage de modèle de la contribution Mir en nettoyant les commandes CLI et les dépendances de package ; le profilage de modèle est disponible dans core.
  + **azureml-contrib-reinforcementlearning**
    + Outil de test de charge
  + **azureml-core**
    + Modifications apportées à la documentation de Script_run_config.py
    + Résolution d’un bogue avec l’impression de la sortie de la ligne de commande submit-pipeline
    + Améliorations de la documentation pour azureml-core/azureml.data
    + Corrige le problème lors de la récupération du compte de stockage à l’aide de la commande hdfs getconf
    + Amélioration de la documentation pour register_azure_blob_container et register_azure_file_share
  + **azureml-datadrift**
    + Amélioration de l’implémentation pour la désactivation et l’activation des superviseurs de dérive de données
  + **azureml-interpret**
    + Dans le client d’explication, suppression des valeurs NaN ou Infs avant la sérialisation JSON lors du chargement à partir des artefacts
    + Mise à jour vers la dernière version d’interpret-community pour améliorer les erreurs de mémoire insuffisante pour les explications globales avec de nombreuses fonctionnalités et classes
    + Ajout du paramètre facultatif true_ys au téléchargement d’explication pour activer des fonctionnalités supplémentaires dans l’interface utilisateur de Studio
    + Amélioration des performances de download_model_explanations() et list_model_explanations()
    + Petites modifications apportées aux blocs-notes, pour faciliter le débogage
  + **azureml-opendatasets**
    + azureml-opendatasets nécessite azureml-dataprep version 1.4.0 ou ultérieure. Ajout d’un avertissement si une version inférieure est détectée
  + **azureml-pipeline-core**
    + Cette modification permet à l’utilisateur de fournir un runconfig facultatif à moduleVersion lors de l’appel à module.Publish_python_script.
    + Le compte de nœud peut être un paramètre de pipeline dans ParallelRunStep dans azureml.pipeline.steps
  + **azureml-pipeline-steps**
    + Cette modification permet à l’utilisateur de fournir un runconfig facultatif à moduleVersion lors de l’appel à module.Publish_python_script.
  + **azureml-train-automl-client**
    + Ajoutez la prise de plusieurs langages pour les modèles de transformateur de Deep Learning tels que BERT dans le Machine Learning automatisé.
    + Suppression du paramètre lag_length déconseillé de la documentation.
    + La documentation sur les paramètres de prévision a été améliorée. Le paramètre lag_length a été déconseillé.
  + **azureml-train-automl-runtime**
    + Activation des explications pour AutoML Prophet et des modèles d’ensemble qui contiennent le modèle Prophet.
    + Mises à jour de la documentation des packages azureml-train-automl-*.
  + **azureml-train-core**
    + Prise en charge de la version 2.1 de TensorFlow dans l’estimateur PyTorch
    + Améliorations apportées au package azureml-train-core.
  
## <a name="2020-05-26"></a>2020-05-26

### <a name="azure-machine-learning-sdk-for-python-v160"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.6.0

+ **Nouvelles fonctionnalités**
  + **azureml-automl-runtime**
    + La prévision AutoML prend désormais en charge la prévision des clients au-delà de l’horizon maximal spécifié, sans nouvel apprentissage du modèle. Lorsque la destination de la prévision se situe au-delà de l’horizon maximal spécifié, la fonction forecast () continue d’effectuer des prédictions de points à la date ultérieure à l’aide d’un mode d’opération récursif. Pour une illustration de la nouvelle fonctionnalité, consultez la section « Prévision au-delà de l’horizon maximal » du notebook « forecasting-forecast-function » dans le [dossier](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning).
  
  + **azureml-pipeline-steps**
    + Désormais publié, ParallelRunStep fait partie du package **azureml-pipeline-steps**. La version de ParallelRunStep présente dans **azureml-contrib-pipeline-steps** est déconseillée. Modifications par rapport à la préversion publique :
      + Ajout d'un paramètre `run_max_try` configurable facultatif pour contrôler le nombre maximal d’appels visant à exécuter la méthode pour un lot donné ; la valeur par défaut est 3.
      + Aucun PipelineParameters n’est plus généré automatiquement. Les valeurs configurables suivantes peuvent être définies comme PipelineParameter de manière explicite.
        + mini_batch_size
        + node_count
        + process_count_per_node
        + Niveau de journalisation
        + run_invocation_timeout
        + run_max_try
      + La valeur par défaut de process_count_per_node est désormais 1. L’utilisateur doit ajuster cette valeur pour obtenir de meilleures performances. La meilleure pratique consiste à opter pour nombre de GPU ou d’UC dont dispose un nœud.
      + ParallelRunStep n’injecte aucun package, l’utilisateur doit inclure des packages **azureml-core** et **azureml-dataprep [pandas, fuse]** dans la définition de l’environnement. Si l’image Docker personnalisée est utilisée avec user_managed_dependencies, l’utilisateur doit installer Conda sur l’image.
      
+ **Dernières modifications**
  + **azureml-pipeline-steps**
    + Utilisation d’azureml.dprep.Dataflow déconseillée comme type d’entrée valide pour AutoMLConfig
  + **azureml-train-automl-client**
    + Utilisation d’azureml.dprep.Dataflow déconseillée comme type d’entrée valide pour AutoMLConfig

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Correction du bogue à cause duquel un avertissement pouvait être imprimé pendant `get_output` qui demandait à l’utilisateur de passer à une version antérieure du client.
    + Mac mis à jour pour utiliser cudatoolkit= 9.0 car il n’est pas encore disponible dans la version 10.
    + Suppression des restrictions sur les modèles phrophet et xgboost lorsqu’ils sont entraînés sur un cacul distant.
    + Amélioration de la journalisation dans AutoML
    + La gestion des erreurs liées à la caractérisation personnalisée dans les tâches de prévision a été améliorée.
    + Ajout d’une fonctionnalité permettant aux utilisateurs d’inclure des fonctions décalées pour générer des prévisions.
    + Mise à jour du message d’erreur afin d’afficher correctement l’erreur de l’utilisateur.
    + Prise en charge de cv_split_column_names à utiliser avec training_data
    + Mise à jour de la journalisation du message d’exception et traceback.
  + **azureml-automl-runtime**
    + Activez des garde-fous pour prévoir les imputations de valeurs manquantes.
    + Amélioration de la journalisation dans AutoML
    + Ajout d’une gestion des erreurs affinée pour les exceptions liées à la préparation des données
    + Suppression des restrictions sur les modèles phrophet et xgboost lorsqu’ils sont formés sur un cacul distant.
    + `azureml-train-automl-runtime` et `azureml-automl-runtime` ont mis à jour des dépendances pour `pytorch`, `scipy` et `cudatoolkit`. Nous prenons désormais en charge `pytorch==1.4.0`, `scipy>=1.0.0,<=1.3.1` et `cudatoolkit==10.1.243`.
    + La gestion des erreurs liées à la caractérisation personnalisée dans les tâches de prévision a été améliorée.
    + Le mécanisme de détection de la fréquence du jeu de données de prévision a été amélioré.
    + Correction d’un problème lié à l’apprentissage du modèle Prophet sur certains jeux de données.
    + La détection automatique de l’horizon maximum lors de la prévision a été améliorée.
    + Ajout d’une fonctionnalité permettant aux utilisateurs d’inclure des fonctions décalées pour générer des prévisions.
    +  Ajoute des fonctionnalités à la fonction de prévision pour permettre des prévisions au-delà de l’horizon formé, sans nouvelle apprentissage du modèle de prévision.
    + Prise en charge de cv_split_column_names à utiliser avec training_data
  + **azureml-contrib-automl-dnn-forecasting**
    + Amélioration de la journalisation dans AutoML
  + **azureml-contrib-mir**
    + Ajout d’une prise en charge des services Windows dans ManagedInferencing
    + Suppression des anciens workflows MIR comme celui consistant à attacher le calcul MIR, la classe SingleModelMirWebservice - Profilage du modèle de nettoyage placé dans le package contrib-mir
  + **azureml-contrib-pipeline-steps**
    + Correctif mineur pour la prise en charge YAML
    + ParallelRunStep est publié en disponibilité générale - azureml.contrib.pipeline.steps comporte un avis de désapprobation et est déplacé vers azureml.pipeline.steps
  + **azureml-contrib-reinforcementlearning**
    + Outil de test de charge RL
    + L’estimateur RL propose des valeurs par défaut intelligentes
  + **azureml-core**
    + Suppression des anciens workflows MIR comme celui consistant à attacher le calcul MIR, la classe SingleModelMirWebservice - Profilage du modèle de nettoyage placé dans le package contrib-mir
    + Correction des informations fournies à l’utilisateur en cas d’échec du profilage : ID de demande inclus et reformulation du message pour qu’il soit plus explicite. Ajout d’un nouveau workflow de profilage aux exécuteurs de profilage
    + Amélioration du texte d’erreur en cas d’échec d’exécution d’un jeu de données.
    + Ajout d’une prise en charge CLI de liaison privée d’espace de travail.
    + Ajout d’un paramètre facultatif `invalid_lines` à `Dataset.Tabular.from_json_lines_files` permettant de spécifier la manière de gérer les lignes contenant un JSON non valide.
    + Nous déconseillerons la création basée sur l’exécution d’un calcul dans la prochaine version. Nous vous recommandons de créer un cluster Amlcompute réel en tant que cible de calcul persistante et d’utiliser le nom du cluster en tant que cible de calcul dans votre configuration d’exécution. Voir l’exemple de notebook disponible ici : aka.ms/amlcomputenb
    + Amélioration des messages d’erreur en cas d’échec d’exécution d’un jeu de données.
  + **azureml-dataprep**
    + Avertissement pour mettre à niveau la version pyarrow de manière plus explicite.
    + Amélioration de la gestion des erreurs et du message renvoyé en cas d’échec d’exécution du dataflow.
  + **azureml-interpret**
    + Mises à jour de la documentation du package azureml-interpret.
    + Correction des packages d’interprétabilité et des notebooks à des fins de compatibilité avec la dernière mise à jour sklearn
  + **azureml-opendatasets**
    + Renvoie None si aucune donnée n’est renvoyée.
    + Amélioration des performances de to_pandas_dataframe.
  + **azureml-pipeline-core**
    + Correctif rapide pour ParallelRunStep en cas de chargement interrompu à partir de YAML
    + ParallelRunStep est publié en disponibilité générale - azureml.contrib.pipeline.steps comporte un avis de désapprobation et est déplacé vers azureml.pipeline.steps - nouvelles fonctionnalités incluant : 1. Jeux de données en tant que PipelineParameter 2. Nouveau paramètre run_max_retry 3. Nom du fichier de sortie append_row configurable
  + **azureml-pipeline-steps**
    + azureml.dprep.Dataflow désormais déconseillé en tant que type valide pour les données d'entrée.
    + Correctif rapide pour ParallelRunStep en cas de chargement interrompu à partir de YAML
    + ParallelRunStep est publié en disponibilité générale - azureml.contrib.pipeline.steps comporte un avis de désapprobation et est déplacé vers azureml.pipeline.steps - nouvelles fonctionnalités incluant :
      + Jeux de données en tant que PipelineParameter
      + Nouveau paramètre run_max_retry
      + Nom du fichier de sortie append_row configurable
  + **azureml-telemetry**
    + Mise à jour de la journalisation du message d’exception et traceback.
  + **azureml-train-automl-client**
    + Amélioration de la journalisation dans AutoML
    + Mise à jour du message d’erreur afin d’afficher correctement l’erreur de l’utilisateur.
    + Prise en charge de cv_split_column_names à utiliser avec training_data
    + azureml.dprep.Dataflow désormais déconseillé en tant que type valide pour les données d'entrée.
    + Mac mis à jour pour utiliser cudatoolkit= 9.0 car il n’est pas encore disponible dans la version 10.
    + Suppression des restrictions sur les modèles phrophet et xgboost lorsqu’ils sont formés sur un cacul distant.
    + `azureml-train-automl-runtime` et `azureml-automl-runtime` ont mis à jour des dépendances pour `pytorch`, `scipy` et `cudatoolkit`. Nous prenons désormais en charge `pytorch==1.4.0`, `scipy>=1.0.0,<=1.3.1` et `cudatoolkit==10.1.243`.
    + Ajout d’une fonctionnalité permettant aux utilisateurs d’inclure des fonctions décalées pour générer des prévisions.
  + **azureml-train-automl-runtime**
    + Amélioration de la journalisation dans AutoML
    + Ajout d’une gestion des erreurs affinée pour les exceptions liées à la préparation des données
    + Suppression des restrictions sur les modèles phrophet et xgboost lorsqu’ils sont formés sur un cacul distant.
    + `azureml-train-automl-runtime` et `azureml-automl-runtime` ont mis à jour des dépendances pour `pytorch`, `scipy` et `cudatoolkit`. Nous prenons désormais en charge `pytorch==1.4.0`, `scipy>=1.0.0,<=1.3.1` et `cudatoolkit==10.1.243`.
    + Mise à jour du message d’erreur afin d’afficher correctement l’erreur de l’utilisateur.
    + Prise en charge de cv_split_column_names à utiliser avec training_data
  + **azureml-train-core**
    + Ajout d’un nouvel ensemble d’exceptions spécifiques à HyperDrive. azureml.train.hyperdrive lève désormais des exceptions détaillées.
  + **azureml-widgets**
    + Les widgets AzureML ne s’affichent pas dans JupyterLab
  

## <a name="2020-05-11"></a>2020-05-11

### <a name="azure-machine-learning-sdk-for-python-v150"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.5.0

+ **Nouvelles fonctionnalités**
  + **Fonctionnalités préliminaires**
    + **azureml-contrib-reinforcementlearning**
        + Azure Machine Learning prend en charge la version préliminaire de l’apprentissage par renforcement à l’aide de l’infrastructure [Ray](https://ray.io). Le `ReinforcementLearningEstimator` permet la formation d’agents d’apprentissage par renforcement sur les cibles de calcul de GPU et UC dans Azure Machine Learning.

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Résout un journal des avertissements laissé par inadvertance dans ma PR précédente. Le journal a été utilisé pour le débogage et a été accidentellement oublié.
    + Résolution de bogue : informer les clients de l’échec partiel pendant le profilage
  + **azureml-automl-core**
    + Accélérez le modèle Prophet/AutoArima dans la prévision AutoML en activant l’ajustement parallèle pour la série chronologique lorsque les jeux de données ont plusieurs séries chronologiques. Pour tirer parti de cette nouvelle fonctionnalité, il est recommandé de définir « max_cores_per_iteration = -1 » (c’est-à-dire, à l’aide de tous les cœurs de processeur disponibles) dans AutoMLConfig.
    + Corriger l’erreur de clé liée à l’impression de garde-fous dans l’interface de la console
    + Correction du message d’erreur pour experimentation_timeout_hours
    + Modèles Tensorflow déconseillés pour AutoML.
  + **azureml-automl-runtime**
    + Correction du message d’erreur pour experimentation_timeout_hours
    + Correction de l’exception non classifiée lors de la tentative de désérialisation à partir du magasin de cache
    + Accélérez le modèle Prophet/AutoArima dans la prévision AutoML en activant l’ajustement parallèle pour la série chronologique lorsque les jeux de données ont plusieurs séries chronologiques.
    + Correction de la prévision avec la fenêtre dynamique activée sur les jeux de données où le test ou la prédiction défini(e) ne contient pas l’un des grains du jeu d’apprentissage.
    + Amélioration de la gestion des données manquantes
    + Correction du problème lié aux intervalles de prédiction lors des prévisions sur les jeux de données, contenant des séries chronologiques qui ne sont pas alignées dans le temps.
    + Amélioration de la validation de la forme des données pour les tâches de prévision.
    + Amélioration de la détection de fréquence.
    + Création d’un meilleur message d’erreur si les plis de validation croisée pour les tâches de prévision ne peuvent pas être générés.
    + Correction de l’interface de la console pour imprimer correctement le garde-fou de valeur manquante.
    + Application des vérifications de type de données sur la saisie de cv_split_indices dans AutoMLConfig.
  + **azureml-cli-common**
    + Résolution de bogue : informer les clients de l’échec partiel pendant le profilage
  + **azureml-contrib-mir**
    + Ajoute une classe azureml.contrib.mir.RevisionStatus qui relaie des informations sur la révision MIR actuellement déployée et la version la plus récente spécifiée par l’utilisateur. Cette classe est incluse dans l’objet MirWebservice sous l’attribut « deployment_status ».
    + Active la mise à jour sur les Webservices de type MirWebservice et sa classe enfant SingleModelMirWebservice.
  + **azureml-contrib-reinforcementlearning**
    + Ajout de la prise en charge de Ray 0.8.3
    + AmlWindowsCompute prend uniquement en charge Azure Files en tant que stockage monté
    + Health_check_timeout renommé en health_check_timeout_seconds
    + Correction de certaines descriptions de classe/méthode.
  + **azureml-core**
    + Activation des conversions WASB -> Blob dans les clouds Azure Government et China.
    + Corrige le bogue pour permettre aux rôles de lecteur d’utiliser des commandes CLI az ml run pour accéder aux informations d’exécution
    + Suppression de la journalisation inutile pendant les exécutions à distance d’Azure ML avec des jeux de données d’entrée.
    + RCranPackage prend désormais en charge le paramètre « version » pour la version du package CRAN.
    + Résolution de bogue : informer les clients de l’échec partiel pendant le profilage
    + Ajout de la gestion de virgule flottante de style européen pour azureml-core.
    + Activation des fonctionnalités de liaison privée d’espace de travail dans le kit de développement logiciel (SDK) Azure ML.
    + Lorsque vous créez un TabularDataset à l’aide de `from_delimited_files`, vous pouvez spécifier si les valeurs vides doivent être chargées en tant que None ou en tant que chaîne vide en définissant l’argument booléen `empty_as_string`.
    + Ajout de la gestion de virgule flottante de style européen pour les jeux de données.
    + Amélioration des messages d’erreur sur les échecs de montage du jeu de données.
  + **azureml-datadrift**
    + La requête de résultats de dérive de données du Kit de développement logiciel présentait un bogue lors duquel aucune distinction n’était opérée entre les métriques de fonctionnalités minimales, maximales et moyennes, entraînant ainsi des valeurs en double. Nous avons résolu ce bogue en faisant précéder la cible ou la ligne de base des noms de mesures. Avant: min, max, mean en double. Après : target_min, target_max, target_mean, baseline_min, baseline_max, baseline_mean.
  + **azureml-dataprep**
    + Amélioration de la gestion des environnements Python en écriture restreints lorsque vous garantissez les dépendances .NET requises pour la livraison des données.
    + Correction de la création de dataflow sur le fichier avec des enregistrements vides de début.
    + Ajout d’options de gestion des erreurs pour `to_partition_iterator` similaires à `to_pandas_dataframe`.
  + **azureml-interpret**
    + Réduction des limites de longueur de chemin d’accès pour réduire la probabilité de dépasser la limite de Windows
    + Correction de bogue pour les explications éparses créées avec l’explicatif d’imitation à l’aide d’un modèle de substitution linéaire.
  + **azureml-opendatasets**
    + Résolution du problème des colonnes de MNIST analysées sous la forme d’une chaîne qui doit être une valeur entière.
  + **azureml-pipeline-core**
    + Autorisation de l’option regenerate_outputs lors de l’utilisation d’un module incorporé dans un ModuleStep.
  + **azureml-train-automl-client**
    + Modèles Tensorflow déconseillés pour AutoML.
    + Correction des algorithmes non pris en charge de liste verte d’utilisateurs en mode local
    + Corrections de la documentation dans AutoMLConfig.
    + Application des vérifications de type de données sur la saisie de cv_split_indices dans AutoMLConfig.
    + Correction du problème lié à l’échec de l’exécution d’AutoML dans show_output
  + **azureml-train-automl-runtime**
    + Correction d’un bogue dans les itérations Ensemble qui empêchait le délai de téléchargement du modèle de s’exécuter correctement.
  + **azureml-train-core**
    + Correction d’une faute de frappe dans la classe azureml.train.dnn.Nccl.
    + Prise en charge de la version 1.5 de PyTorch dans l’estimateur PyTorch
    + Correction du problème où l’image de l’infrastructure ne pouvait pas être extraite dans la région Azure Government lors de l’utilisation des estimateurs d’infrastructure de formation

  
## <a name="2020-05-04"></a>2020-05-04
**Nouvelle expérience de Notebook**

Vous pouvez désormais créer, modifier et partager des notebooks et des fichiers de Machine Learning directement dans l’expérience Web Studio de Azure Machine Learning. Vous pouvez utiliser toutes les classes et méthodes disponibles dans [le kit de développement logiciel (SDK) Python du Machine Learning](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) à l’intérieur de ces blocs-notes. Démarrez [ici](./how-to-run-jupyter-notebooks.md)

**Nouvelles fonctionnalités introduites :**

+ Éditeur amélioré (éditeur Monaco) utilisé par VS Code 
+ Améliorations UI/UX
+ Barre d'outils de cellules
+ Nouvelle barre d’outils du notebook et contrôles de calcul
+ Barre d’états du notebook 
+ Basculement de noyau inclus
+ Support R
+ Améliorations de l’accessibilité et de la localisation
+ Palette de commandes
+ Raccourcis clavier supplémentaires
+ Enregistrement automatique
+ Amélioration de la fiabilité et des performances

Accédez aux outils de création web suivants à partir de Studio :
    
| Outil basé sur le web  |     Description  |
|---|---|
| Notebooks Azure ML Studio   |     Première création de classe pour les fichiers de notebook et support de toutes les opérations disponibles dans le kit de développement logiciel (SDK) Python Azure ML. | 

## <a name="2020-04-27"></a>2020-04-27

### <a name="azure-machine-learning-sdk-for-python-v140"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.4.0

+ **Nouvelles fonctionnalités**
  + Les clusters AmlCompute prennent désormais en charge la configuration d’une identité managée sur le cluster au moment de l’approvisionnement. Il vous suffit de spécifier si vous souhaitez utiliser une identité affectée par le système ou une identité affectée par l’utilisateur, et de fournir un identityId dans ce dernier cas. Vous pouvez ensuite configurer les autorisations d’accès à diverses ressources, telles que Stockage ou ACR, de manière à ce que l’identité du calcul soit utilisée pour accéder en toute sécurité aux données, au lieu de l’approche basée sur les jetons qu’AmlCompute utilise aujourd’hui. Pour plus d’informations sur les paramètres, consultez notre documentation de référence sur le Kit de développement logiciel (SDK).
  

+ **Dernières modifications**
  + Les clusters AmlCompute prenaient en charge une fonctionnalité d’évaluation autour de la création basée sur l’exécution, que nous prévoyons de déprécier dans deux semaines. Vous pouvez continuer à créer des cibles de calcul persistantes, comme toujours, à l’aide de la classe Amlcompute, mais l’approche spécifique consistant à spécifier l’identificateur « amlcompute » en tant que cible de calcul dans la configuration de l’exécution ne sera pas prise en charge dans un avenir proche. 

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-runtime**
    + Active la prise en charge du type non hachable lors du calcul du nombre de valeurs uniques dans une colonne.
  + **azureml-core**
    + Amélioration de la stabilité lors de la lecture à partir de Stockage Blob Azure à l’aide d’un TabularDataset.
    + Amélioration de la documentation pour le paramètre `grant_workspace_msi` pour `Datastore.register_azure_blob_store`.
    + Correction du bogue lié à `datastore.upload` lors de la prise en charge de l’argument `src_dir` se terminant par un `/` ou un `\`.
    + Ajout d’un message d’erreur actionnable lors d’une tentative de chargement vers un magasin de données Stockage Blob Azure qui ne dispose pas d’une clé d’accès ou d’un jeton SAS.
  + **azureml-interpret**
    + Ajout d’une limite supérieure à la taille de fichier pour les données de visualisation sur les explications chargées.
  + **azureml-train-automl-client**
    + Vérification explicite des paramètres label_column_name et weight_column_name pour qu’AutoMLConfig soit de type chaîne.
  + **azureml-contrib-pipeline-steps**
    + ParallelRunStep prend désormais en charge le jeu de données comme paramètre de pipeline. L’utilisateur peut construire un pipeline avec un exemple de jeu de données et modifier le jeu de données d’entrée du même type (fichier ou tabulaire) pour l’exécution du nouveau pipeline.

  
## <a name="2020-04-13"></a>13/04/2020

### <a name="azure-machine-learning-sdk-for-python-v130"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.3.0

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Ajout de données de télémétrie supplémentaires pour les opérations post-apprentissage.
    + Accélère l'apprentissage automatique ARIMA en utilisant l'apprentissage CSS (somme des carrés conditionnelle) pour les séries de longueur supérieure à 100. La longueur utilisée est stockée en tant que constante ARIMA_TRIGGER_CSS_TRAINING_LENGTH dans la classe TimeSeriesInternal sous /src/azureml-automl-core/azureml/automl/core/shared/constants.py
    + La journalisation utilisateur des exécutions de prévisions a été améliorée. Le journal présente désormais davantage d'informations sur la phase en cours.
    + Interdiction de définir target_rolling_window_size sur une valeur inférieure à 2
  + **azureml-automl-runtime**
    + Amélioration du message d'erreur affiché lorsque des timestamps en double sont trouvés.
    + Interdiction de définir target_rolling_window_size sur une valeur inférieure à 2
    + Correction de l'échec d'imputation du décalage. Le problème était dû au nombre insuffisant d'observations nécessaires pour décomposer une série de façon saisonnière. Les données « désaisonnalisées » sont utilisées pour calculer une fonction d'autocorrélation partielle (PACF) afin de déterminer la longueur du décalage.
    + Activation de la personnalisation de la caractérisation de l'objet des colonnes pour les tâches de prévision par la configuration de la caractérisation. Les objets de colonne Numérique et Catégorique sont maintenant pris en charge pour les tâches de prévision.
    + Activation de la personnalisation de la caractérisation de la suppression des colonnes pour les tâches de prévision par la configuration de la caractérisation.
    + Activation de la personnalisation de l'imputation pour les tâches de prévision par la configuration de la caractérisation. L'imputation en valeur constante pour la colonne cible et l'imputation en valeur moyenne, médiane, la plus fréquente et constante pour les données d'apprentissage sont désormais prises en charge.
  + **azureml-contrib-pipeline-steps**
    + Accepter que les noms de calcul des chaînes soient transmis à ParallelRunConfig
  + **azureml-core**
    +  Ajout de l'API Environment.clone(new_name) pour créer une copie de l'objet Environment
    +  Environment.docker.base_dockerfile accepte le chemin d'accès aux fichiers. S'il est en mesure de résoudre un fichier, le contenu est lu dans la propriété d'environnement base_dockerfile
    + Réinitialisation automatique des valeurs mutuellement exclusives pour base_image et base_dockerfile lorsque l'utilisateur définit manuellement une valeur dans Environment.docker
    + Ajout dans RSection de l'indicateur user_managed qui indique si l'environnement est géré par l'utilisateur ou par AzureML.
    + Jeu de données : correction de l’échec du téléchargement du jeu de données si le chemin d’accès aux données contient des caractères Unicode.
    + Jeu de données : amélioration du mécanisme de mise en cache du montage du jeu de données pour respecter l’espace disque minimum requis dans Capacité de calcul Azure Machine Learning. Le nœud reste ainsi utilisable et le travail n’est pas annulé.
    + Jeu de données : Ajout d’un index pour la colonne de série chronologique lorsque vous accédez à un jeu de données de série chronologique sous la forme de dataframe Pandas, utilisé pour accélérer l’accès aux données de série chronologique.  Auparavant, l'index portait le même nom que la colonne timestamp, ce qui ne permettait pas aux utilisateurs de faire la distinction entre la colonne timestamp réelle et l'index. Nous ne donnons plus de nom spécifique à l'index car il ne doit pas être utilisé en tant que colonne. 
    + Jeu de données : correction du problème d’authentification des jeux de données dans le cloud souverain.
    + Jeu de données : correction de l’échec `Dataset.to_spark_dataframe` concernant les jeux de données créés à partir de magasins de données Azure PostgreSQL.
  + **azureml-interpret**
    + Ajout de scores globaux à la visualisation si les valeurs d'importance locale sont partiellement remplies
    + Mise à jour d'azureml-interpret pour utiliser interpret-community 0.9.*
    + Résolution d'un problème lié au téléchargement des explications contenant des données d'évaluation partiellement remplies
    + Ajout de la prise en charge du format Partiellement rempli de l'objet d'explication dans AutoML
  + **azureml-pipeline-core**
    + Prise en charge de ComputeInstance comme cible de calcul dans les pipelines
  + **azureml-train-automl-client**
    + Ajout de données de télémétrie supplémentaires pour les opérations post-apprentissage.
    + Correction de la régression lors d'un arrêt précoce
    + azureml.dprep.Dataflow désormais déconseillé en tant que type valide pour les données d'entrée.
    +  Délai d'expiration de l'expérience AutoML par défaut remplacé par six jours.
  + **azureml-train-automl-runtime**
    + Ajout de données de télémétrie supplémentaires pour les opérations post-apprentissage.
    + Ajout de la prise en charge de bout en bout d’AutoML
  + **azureml-opendatasets**
    + Ajout de données de télémétrie supplémentaires pour le moniteur de service.
    + Activer l’instance Front Door pour les objets blob afin d'améliorer la stabilité 

## <a name="2020-03-23"></a>2020-03-23

### <a name="azure-machine-learning-sdk-for-python-v120"></a>Kit SDK Azure Machine Learning pour Python v1.2.0

+ **Dernières modifications**
  + Arrêt de la prise en charge de Python 2.7

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Ajoute « --subscription-ID » au commandes `az ml model/computetarget/service` dans l’interface CLI
    + Ajout de la prise en charge de la transmission de la clé gérée par le client (CMK) vault_url, key_name et key_version pour le déploiement ACI
  + **azureml-automl-core** 
    + Activation de l’imputation personnalisée avec une valeur constante pour les tâches de prévision de données X et y.
    + Résolution du problème lié à l’affichage de messages d’erreur à l’utilisateur.    
  + **azureml-automl-runtime**
    + Résolution du problème lié à la prévision sur les jeux de données contenant des grains avec une seule ligne
    + Réduction de la quantité de mémoire requise par les tâches de prévision.
    + Ajout de meilleurs messages d’erreur si la colonne de temps a un format incorrect.
    + Activation de l’imputation personnalisée avec une valeur constante pour les tâches de prévision de données X et y.
  + **azureml-core**
    + Ajout de la prise en charge du chargement de ServicePrincipal à partir de variables d’environnement : AZUREML_SERVICE_PRINCIPAL_ID, AZUREML_SERVICE_PRINCIPAL_TENANT_ID, and AZUREML_SERVICE_PRINCIPAL_PASSWORD
    + Introduction d’un nouveau paramètre `support_multi_line` pour `Dataset.Tabular.from_delimited_files` : Par défaut (`support_multi_line=False`), tous les sauts de ligne, y compris dans des valeurs de champ entre guillemets, sont interprétés comme une coupure d’enregistrement. La lecture de données de cette façon est plus rapide et plus optimisée pour une exécution en parallèle sur plusieurs cœurs de processeur. Toutefois, elle peut entraîner la production silencieuse de davantage d’enregistrements avec des valeurs de champ mal alignées. Elle doit être définie sur `True` lorsque les fichiers délimités sont connus pour contenir des sauts de ligne entre guillemets.
    + Ajout de la possibilité d’inscrire ADLS Gen2 dans l’interface CLI Azure Machine Learning
    + Changement de nom du paramètre « fine_grain_timestamp » en « timestamp » et du paramètre « coarse_grain_timestamp » en « partition_timestamp » pour la méthode with_timestamp_columns() dans TabularDataset afin de mieux refléter l’utilisation des paramètres.
    + Augmentation de la longueur maximale du nom de l’expérience à 255.
  + **azureml-interpret**
    + Mise à jour d’azureml-interpret en interpret-community 0.7.*
  + **azureml-sdk**
    + Modification des dépendances avec la version compatible Tilde pour la prise en charge des mises à jour correctives dans les versions préliminaires et les mises en production stables.


## <a name="2020-03-11"></a>2020-03-11

### <a name="azure-machine-learning-sdk-for-python-v115"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.1.5

+ **Dépréciation des fonctionnalités**
  + **Python 2.7**
    + Version la plus récente prenant en charge Python 2.7

+ **Dernières modifications**
  + **Gestion sémantique de version 2.0.0**
    + Depuis la version 1.1, le SDK Python Azure Machine Learning a adopté la gestion sémantique de version 2.0.0. [En savoir plus ici](https://semver.org/). Toutes les versions ultérieures suivront le nouveau schéma de numérotation et le contrat de gestion sémantique de version. 

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Modifiez le nom de la commande CLI du point de terminaison « az ml endpoint aks » en « az ml endpoint realtime » pour garantir la cohérence.
    + Mise à jour des instructions d’installation de l’interface CLI pour une branche CLI stable et expérimentale.
    + Le profilage d’instance unique a été corrigé pour produire une recommandation et a été mis à disposition dans le kit SDK Core.
  + **azureml-automl-core**
    + Activation de l’inférence du mode batch (prise de plusieurs lignes en une fois) pour les modèles ONNX AutoML
    + Amélioration de la détection de la fréquence sur les jeux de données, des données manquantes ou contenant des points de données irréguliers
    + Ajout de la possibilité de supprimer des points de données qui ne respectent pas la fréquence dominante.
    + Modification de l’entrée du constructeur pour prendre une liste d’options permettant d’appliquer les options d’imputation pour les colonnes correspondantes.
    + La journalisation des erreurs a été améliorée.
  + **azureml-automl-runtime**
    + Correction du problème qui provoquait la levée d’une erreur si le grain qui n’était pas présent dans le jeu d’entraînement apparaissait dans le jeu de test
    + Suppression de l’exigence liée à y_query lors de la notation du service de prévision
    + Correction du problème avec les prévisions lorsque le jeu de données contient des fragments réduits avec de longs laps de temps.
    + Correction du problème lorsque l’horizon maximal automatique est activé et que la colonne de date contient des dates sous forme de chaînes. Des messages d’erreur et de conversion appropriés ont été ajoutés pour les cas où la conversion en date n’est pas possible.
    + Utilisation de NumPy et SciPy en mode natif pour la sérialisation et la désérialisation des données intermédiaires pour FileCacheStore (utilisé pour les exécutions AutoML locales)
    + Correction d’un bogue où les exécutions enfants ayant échoué pouvaient rester bloquées dans l’état d’exécution.
    + Augmentation de la vitesse de caractérisation.
    + Correction de la vérification de fréquence lors du scoring. Désormais, les tâches de prévision ne nécessitent pas d’équivalence stricte de fréquence entre le jeu de formation et celui de test.
    + Modification de l’entrée du constructeur pour prendre une liste d’options permettant d’appliquer les options d’imputation pour les colonnes correspondantes.
    + Correction des erreurs liées à la sélection du type de latence.
    + Correction de l’erreur non classifiée déclenchée sur les jeux de données, où des grains se trouvent avec la ligne unique.
    + Résolution du problème de lenteur lors de la détection de fréquence.
    + Correction d’un bogue dans la gestion des exceptions AutoML qui faisait que la véritable raison de l’échec de la formation était remplacée par un AttributeError.
  + **azureml-cli-common**
    + Le profilage d’instance unique a été corrigé pour produire une recommandation et a été mis à disposition dans le kit SDK Core.
  + **azureml-contrib-mir**
    + Ajout d’une fonctionnalités dans la classe MirWebservice pour récupérer le jeton d’accès
    + Utilisez l’authentification par jeton pour MirWebservice par défaut pendant MirWebservice.run() call ; actualisez uniquement si l’appel échoue.
    + Le déploiement du service web Mir requiert désormais les références SKU appropriées [Standard_DS2_v2, Standard_F16 et Standard_A2_v2] au lieu des références [Ds2v2, A2v2 et F16], respectivement.
  + **azureml-contrib-pipeline-steps**
    + Paramètre facultatif side_inputs ajouté à ParallelRunStep. Ce paramètre peut être utilisé pour monter le dossier sur le conteneur. Les types actuellement pris en charge sont DataReference et PipelineData.
    + Les paramètres passés dans ParallelRunConfig peuvent désormais être remplacés en passant des paramètres de pipeline. De nouveaux paramètres de pipeline sont pris en charge : aml_mini_batch_size, aml_error_threshold, aml_logging_level, aml_run_invocation_timeout (aml_node_count et aml_process_count_per_node faisaient déjà partie de la précédente version).
  + **azureml-core**
    + Les services web AzureML déployés utilisent désormais par défaut la journalisation `INFO`. Vous pouvez contrôler cela en définissant la variable d’environnement `AZUREML_LOG_LEVEL` dans le service déployé.
    + Le SDK Python utilise le service de découverte pour utiliser le point de terminaison « api » au lieu de « pipelines ».
    + Basculement vers les nouveaux itinéraires dans tous les appels du Kit de développement logiciel (SDK).
    + Modification du routage des appels à ModelManagementService vers une nouvelle structure unifiée.
      + La méthode de mise à jour de l’espace de travail est désormais publiquement disponible.
      + Ajout du paramètre image_build_compute dans la méthode de mise à jour de l’espace de travail afin de permettre à l’utilisateur de mettre à jour le calcul pour la génération d’image.
    + Ajout de messages de dépréciation à l’ancien workflow de profilage Correction des limites de mémoire et d’UC du profilage.
    + Ajout de RSection dans le cadre de l’environnement pour exécuter les travaux R.
    + Ajout de la validation à `Dataset.mount` pour déclencher une erreur lorsque la source du jeu de données n’est pas accessible ou ne contient pas de données.
    + Ajout de `--grant-workspace-msi-access` en tant que paramètre supplémentaire pour l’interface CLI de magasin de données pour l’inscription du conteneur de blobs Azure, ce qui vous permet d’inscrire un conteneur de blobs qui se trouve derrière un réseau virtuel.
    + Le profilage d’instance unique a été corrigé pour produire une recommandation et a été mis à disposition dans le kit SDK Core.
    + Correction du problème dans aks.py_deploy.
    + Valide l’intégrité des modèles en cours de chargement afin d’éviter les échecs de stockage en mode silencieux.
    + L’utilisateur peut désormais spécifier une valeur pour la clé d’authentification lorsqu’il regénère les clés de services web.
    + Correction du bogue où les lettres majuscules ne peuvent pas être utilisées comme nom d’entrée du jeu de données.
  + **azureml-defaults**
    + `azureml-dataprep` est maintenant installé dans le cadre de `azureml-defaults`. Il n’est plus nécessaire d’installer data prep[fuse] manuellement sur les cibles de calcul pour monter des jeux de données.
  + **azureml-interpret**
    + Mise à jour d’azureml-interpret vers interpret-community 0.6*
    + Mise à jour d’azureml-interpret pour dépendre désormais d’interpret-community 0.5.0
    + Ajout d’exceptions azureml-style à azureml-interpret
    + Correction de la sérialisation DeepScoringExplainer pour les modèles Keras
  + **azureml-mlflow**
    + Ajout de la prise en charge des clouds souverains à azureml.mlflow
  + **azureml-pipeline-core**
    + Le notebook de scoring par lots du pipeline utilise désormais ParallelRunStep
    + Correction d’un bogue entraînant une réutilisation incorrecte des résultats PythonScriptStep malgré la modification de la liste d’arguments
    + Ajout de la possibilité de définir le type des colonnes lors de l’appel des méthodes parse_* sur `PipelineOutputFileDataset`
  + **azureml-pipeline-steps**
    + Déplacement de `AutoMLStep` dans le package `azureml-pipeline-steps`. Dépréciation d’`AutoMLStep` dans `azureml-train-automl-runtime`.
    + Ajout d’un exemple de documentation pour le jeu de données sous la forme d’une entrée PythonScriptStep
  + **azureml-tensorboard**
    + Mise à jour d’azureml-tensorboard pour prendre en charge TensorFlow 2.0
    + Affiche le numéro de port correct lors de l’utilisation d’un port Tensorboard personnalisé sur une instance de calcul
  + **azureml-train-automl-client**
    + Résolution d’un problème où certains packages pouvaient être installés à des versions incorrectes sur des exécutions distantes.
    + Correction du problème de substitution FeaturizationConfig qui filtre la configuration de caractérisation personnalisée.
  + **azureml-train-automl-runtime**
    + Résolution du problème de détection de fréquence dans les exécutions distantes
    + Déplacement d’`AutoMLStep` dans le package `azureml-pipeline-steps`. Dépréciation d’`AutoMLStep` dans `azureml-train-automl-runtime`.
  + **azureml-train-core**
    + Prise en charge de la version 1.4 de PyTorch dans l’estimateur PyTorch
  
## <a name="2020-03-02"></a>2020-03-02

### <a name="azure-machine-learning-sdk-for-python-v112rc0-pre-release"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.1.2rc0 (préversion)

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Activation de l’inférence du mode batch (prise de plusieurs lignes en une fois) pour les modèles ONNX AutoML
    + Amélioration de la détection de la fréquence sur les jeux de données, des données manquantes ou contenant des points de données irréguliers
    + Ajout de la possibilité de supprimer des points de données qui ne respectent pas la fréquence dominante.
  + **azureml-automl-runtime**
    + Correction du problème qui provoquait la levée d’une erreur si le grain qui n’était pas présent dans le jeu d’entraînement apparaissait dans le jeu de test
    + Suppression de l’exigence liée à y_query lors de la notation du service de prévision
  + **azureml-contrib-mir**
    + Ajout d’une fonctionnalités dans la classe MirWebservice pour récupérer le jeton d’accès
  + **azureml-core**
    + Les services web AzureML déployés utilisent désormais par défaut la journalisation `INFO`. Vous pouvez contrôler cela en définissant la variable d’environnement `AZUREML_LOG_LEVEL` dans le service déployé.
    + Correction de l’itération sur `Dataset.get_all` pour retourner tous les jeux de données inscrits auprès de l’espace de travail.
    + Amélioration du message d’erreur quand un type non valide est passé à l’argument `path` des API de création de jeu de données.
    + Le SDK Python utilise le service de découverte pour utiliser le point de terminaison « api » au lieu de « pipelines ».
    + Basculement vers les nouvelles routes dans tous les appels du SDK
    + Modification du routage des appels à ModelManagementService vers une nouvelle structure unifiée
      + La méthode de mise à jour de l’espace de travail est désormais publiquement disponible.
      + Ajout du paramètre image_build_compute dans la méthode de mise à jour de l’espace de travail afin de permettre à l’utilisateur de mettre à jour le calcul pour la génération d’image
    +  Ajout de messages de dépréciation à l’ancien workflow de profilage Correction des limites de mémoire et de processeur du profilage
  + **azureml-interpret**
    + Mise à jour d’azureml-interpret vers interpret-community 0.6.*
  + **azureml-mlflow**
    + Ajout de la prise en charge des clouds souverains à azureml.mlflow
  + **azureml-pipeline-steps**
    + Déplacement de `AutoMLStep` vers `azureml-pipeline-steps package`. Dépréciation d’`AutoMLStep` dans `azureml-train-automl-runtime`.
  + **azureml-train-automl-client**
    + Résolution d’un problème où certains packages pouvaient être installés à des versions incorrectes sur des exécutions distantes.
  + **azureml-train-automl-runtime**
    + Résolution du problème de détection de fréquence dans les exécutions distantes
    + Déplacement de `AutoMLStep` vers `azureml-pipeline-steps package`. Dépréciation d’`AutoMLStep` dans `azureml-train-automl-runtime`.
  + **azureml-train-core**
    + Déplacement de `AutoMLStep` vers `azureml-pipeline-steps package`. Dépréciation d’`AutoMLStep` dans `azureml-train-automl-runtime`.

## <a name="2020-02-18"></a>18-02-2020

### <a name="azure-machine-learning-sdk-for-python-v111rc0-pre-release"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.1.1rc0 (préversion)

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Le profilage d’instance unique a été corrigé pour produire une recommandation et a été mis à disposition dans le kit SDK Core.
  + **azureml-automl-core**
    + La journalisation des erreurs a été améliorée.
  + **azureml-automl-runtime**
    + Correction du problème avec les prévisions lorsque le jeu de données contient des fragments réduits avec de longs laps de temps.
    + Correction du problème lorsque l’horizon maximal automatique est activé et que la colonne de date contient des dates sous forme de chaînes. Nous avons ajouté une conversion correcte et une erreur raisonnable si la conversion en date n’est pas possible
    + Utilisation de NumPy et SciPy en mode natif pour la sérialisation et la désérialisation des données intermédiaires pour FileCacheStore (utilisé pour les exécutions AutoML locales)
    + Correction d’un bogue où les exécutions enfants ayant échoué pouvaient rester bloquées dans l’état d’exécution.
  + **azureml-cli-common**
    + Le profilage d’instance unique a été corrigé pour produire une recommandation et a été mis à disposition dans le kit SDK Core.
  + **azureml-core**
    + Ajout de `--grant-workspace-msi-access` en tant que paramètre supplémentaire pour l’interface CLI de magasin de données pour l’inscription du conteneur de blobs Azure, ce qui vous permet d’inscrire un conteneur de blobs qui se trouve derrière un réseau virtuel
    + Le profilage d’instance unique a été corrigé pour produire une recommandation et a été mis à disposition dans le kit SDK Core.
    + Correction du problème dans aks.py _deploy
    + Valide l’intégrité des modèles en cours de chargement afin d’éviter les échecs de stockage en mode silencieux.
  + **azureml-interpret**
    + Ajout d’exceptions de style azureml à azureml-interpret
    + Correction de la sérialisation DeepScoringExplainer pour les modèles Keras
  + **azureml-pipeline-core**
    + Le notebook de scoring par lots du pipeline utilise désormais ParallelRunStep
  + **azureml-pipeline-steps**
    + Déplacement d’`AutoMLStep` dans le package `azureml-pipeline-steps`. Dépréciation d’`AutoMLStep` dans `azureml-train-automl-runtime`.
  + **azureml-contrib-pipeline-steps**
    + Paramètre facultatif side_inputs ajouté à ParallelRunStep. Ce paramètre peut être utilisé pour monter le dossier sur le conteneur. Les types actuellement pris en charge sont DataReference et PipelineData.
  + **azureml-tensorboard**
    + Mise à jour d’azureml-tensorboard pour prendre en charge TensorFlow 2.0
  + **azureml-train-automl-client**
    + Correction du problème de substitution FeaturizationConfig qui filtre la configuration de caractérisation personnalisée.
  + **azureml-train-automl-runtime**
    + Déplacement d’`AutoMLStep` dans le package `azureml-pipeline-steps`. Dépréciation d’`AutoMLStep` dans `azureml-train-automl-runtime`.
  + **azureml-train-core**
    + Prise en charge de la version 1.4 de PyTorch dans l’estimateur PyTorch
  
## <a name="2020-02-04"></a>2020-02-04

### <a name="azure-machine-learning-sdk-for-python-v110rc0-pre-release"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.1.0rc0 (préversion)

+ **Dernières modifications**
  + **Gestion sémantique de version 2.0.0**
    + Depuis la version 1.1, le SDK Python Azure Machine Learning a adopté la gestion sémantique de version 2.0.0. [En savoir plus ici](https://semver.org/). Toutes les versions ultérieures suivront le nouveau schéma de numérotation et le contrat de gestion sémantique de version. 
  
+ **Résolutions de bogue et améliorations**
  + **azureml-automl-runtime**
    + Augmentation de la vitesse de caractérisation.
    + Correction de la vérification de fréquence lors du scoring. Désormais, dans les tâches de prévision, nous n’exigeons plus une équivalence stricte de fréquence entre l’entraînement et le jeu de test.
  + **azureml-core**
    + L’utilisateur peut désormais spécifier une valeur pour la clé d’authentification lorsqu’il regénère les clés de services web.
  + **azureml-interpret**
    + Mise à jour d’azureml-interpret pour dépendre désormais d’interpret-community 0.5.0
  + **azureml-pipeline-core**
    + Correction d’un bogue entraînant une réutilisation incorrecte des résultats PythonScriptStep malgré la modification de la liste d’arguments
  + **azureml-pipeline-steps**
    + Ajout d’un exemple de documentation pour le jeu de données sous la forme d’une entrée PythonScriptStep
  + **azureml-contrib-pipeline-steps**
    + Les paramètres passés dans ParallelRunConfig peuvent désormais être remplacés en passant des paramètres de pipeline. De nouveaux paramètres de pipeline sont pris en charge : aml_mini_batch_size, aml_error_threshold, aml_logging_level, aml_run_invocation_timeout (aml_node_count et aml_process_count_per_node faisaient déjà partie de la précédente version).
  
## <a name="2020-01-21"></a>2020-01-21

### <a name="azure-machine-learning-sdk-for-python-v1085"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.85

+ **Nouvelles fonctionnalités**
  + **azureml-core**
    + Obtenir l’utilisation des cœurs actuelle et la limite de quota pour les ressources AmlCompute dans un espace de travail et un abonnement donnés
  
  + **azureml-contrib-pipeline-steps**
    + Permettre à l’utilisateur de passer le jeu de données tabulaire en tant que résultat intermédiaire de l’étape précédente à parallelrunstep

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-runtime**
    + Suppression de l’exigence de colonne y_query dans la demande envoyée au service de prévision déployé. 
    + Le « y_query » a été supprimé de la section de la demande de service notebook Orange Juice de Dominick.
    + Correction du bogue empêchant les prévisions sur les modèles déployés opérant sur des jeux de données avec des colonnes de date et d’heure.
    + Ajout du coefficient de corrélation Matthews en tant que métrique de classification, pour la classification binaire et multiclasse.
  + **azureml-contrib-interpret**
    + Suppression des explications de texte d’azureml-contrib-interpret, car une explication du texte a été déplacée vers le référentiel interpret-text qui sera bientôt publié.
  + **azureml-core**
    + Jeu de données : les utilisations du jeu de données de fichier ne dépendent plus de l’installation de numpy et pandas dans l’environnement Python.
    + Modification de LocalWebservice.wait_for_deployment() pour vérifier l’état du conteneur Docker local avant d’essayer d’effectuer un test ping de son point de terminaison d’intégrité, ce qui réduit considérablement le temps nécessaire pour signaler un échec de déploiement.
    + Correction de l’initialisation d’une propriété interne utilisée dans LocalWebservice.reload() lors de la création de l’objet service à partir d’un déploiement existant à l’aide du constructeur LocalWebservice().
    + Modification du message d’erreur pour clarification.
    + Ajout d’une nouvelle méthode appelée get_access_token() à AksWebservice, qui retourne l’objet AksServiceAccessToken contenant un jeton d’accès, une actualisation après horodatage, une expiration sur horodatage et un type de jeton. 
    + Abandon de la méthode get_token() existante dans AksWebservice, car la nouvelle méthode retourne toutes les informations que cette méthode retourne.
    + Modification de la sortie de la commande az ml service get-access-token. Changement de nom de token en accessToken et de refreshBy en refreshAfter. Ajout des propriétés expiryOn et tokenType.
    + Correction de Fixed get_active_runs
  + **azureml-explain-model**
    + mise à jour de shap vers 0.33.0 et d’interpret-community vers 0.4.*
  + **azureml-interpret**
    + mise à jour de shap vers 0.33.0 et d’interpret-community vers 0.4.*
  + **azureml-train-automl-runtime**
    + Ajout du coefficient de corrélation Matthews en tant que métrique de classification, pour la classification binaire et multiclasse.
    + Abandon de l’indicateur de prétraitement du code et remplacement par caractérisation (caractérisation activée par défaut)

## <a name="2020-01-06"></a>2020-01-06

### <a name="azure-machine-learning-sdk-for-python-v1083"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.83

+ **Nouvelles fonctionnalités**
  + Jeu de données : ajoutez deux options `on_error` et `out_of_range_datetime` pour que `to_pandas_dataframe` échoue lorsque les données ont des valeurs d’erreur au lieu de les remplir avec `None`.
  + Espace de travail : Ajout de l’indicateur `hbi_workspace` pour les espaces de travail avec des données sensibles qui permettent un chiffrement supplémentaire et désactivent les diagnostics avancés sur les espaces de travail. Nous avons également ajouté la prise en charge de l’ajout de vos propres clés pour l’instance Cosmos DB associée, en spécifiant les paramètres `cmk_keyvault` et `resource_cmk_uri` lors de la création d’un espace de travail, ce qui crée une instance Cosmos DB dans votre abonnement lors de l’approvisionnement de votre espace de travail. [Apprenez-en davantage ici](./concept-data-encryption.md#azure-cosmos-db).

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-runtime**
    + Correction d’une régression qui provoquait le déclenchement de TypeError lors de l’exécution d’AutoML sur les versions de Python inférieures à 3.5.4.
  + **azureml-core**
    + Correction du bogue dans `datastore.upload_files` où un chemin d’accès relatif ne commençant pas par `./` n’était pas utilisable.
    + Ajout de messages de dépréciation pour tous les chemins du code de la classe Image.
    + Correction de la construction de l’URL Gestion des modèles pour la région Azure China 21Vianet.
    + Correction du problème selon lequel il n’était pas possible de packager les modèles qui utilisent source_dir pour Azure Functions.    
    + Ajout d’une option à [Environment.build_local()](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py) pour pouvoir envoyer (push) une image dans le registre de conteneurs de l’espace de travail AzureML.
    + Mise à jour du kit de développement logiciel (SDK) de façon à utiliser la nouvelle bibliothèque de jetons sur Azure Synapse en compatibilité descendante.
  + **azureml-interpret**
    + Correction du bogue selon lequel None était retourné quand aucune explication n’était téléchargeable. Lève désormais une exception, ce qui correspond aux autres comportements.
  + **azureml-pipeline-steps**
    + Ajout de l’interdiction de passer des `DatasetConsumptionConfig` au paramètre `inputs` de `Estimator` lorsque `Estimator` est utilisé dans `EstimatorStep`.
  + **azureml-sdk**
    + Ajout du client AutoML au package azureml-sdk, ce qui permet d’envoyer les exécutions AutoML distantes sans installer le package AutoML complet.
  + **azureml-train-automl-client**
    + Correction de l’alignement sur la sortie de la console pour les exécutions AutoML.
    + Correction du bogue selon lequel une mauvaise version de pandas était parfois installée sur amlcompute distant.

## <a name="2019-12-23"></a>2019-12-23

### <a name="azure-machine-learning-sdk-for-python-v1081"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.81

+ **Résolutions de bogue et améliorations**
  + **azureml-contrib-interpret**
    + Report de la dépendance Shap sur interpret-community à partir de azureml-interpret.
  + **azureml-core**
    + Ajout de la possibilité d’indiquer la cible de calcul comme paramètre des objets de configuration de déploiement correspondants. Il s’agit en particulier du nom de la cible de calcul sur laquelle le déploiement est effectué, et non de l’objet du kit SDK.
    + Ajout d’informations CreatedBy aux objets Model et Service, Accessibles avec .created_by
    + Correction de ContainerImage.Run(), qui ne configurait pas correctement le port HTTP du conteneur Docker.
    + Possibilité de rendre `azureml-dataprep` facultatif pour la commande CLI `az ml dataset register`.
    + Correction du bogue selon lequel `TabularDataset.to_pandas_dataframe` revenait à tort sur un autre lecteur et imprimait un avertissement.
  + **azureml-explain-model**
    + Report de la dépendance Shap sur interpret-community à partir de azureml-interpret.
  + **azureml-pipeline-core**
    + Ajout d’une nouvelle étape de pipeline `NotebookRunnerStep`, consistant à exécuter un bloc-notes local.
    + Suppression des fonctions get_all dépréciées pour PublishedPipelines, Schedules et PipelineEndpoints.
  + **azureml-train-automl-client**
    + Début de la dépréciation de data_script comme entrée d’AutoML.


## <a name="2019-12-09"></a>09-12-2019

### <a name="azure-machine-learning-sdk-for-python-v1079"></a>SDK Azure Machine Learning pour Python v1.0.79

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Suppression de la journalisation de featurizationConfig
      + Journalisation modifiée de manière à journaliser uniquement "auto"/"off"/"customized"
  + **azureml-automl-runtime**
    + Ajout de la prise en charge de pandas.Series et de pandas.Categorical pour la détection du type de données de la colonne. Avant, seul numpy.ndarray était pris en charge
      + Ajout de modifications relatives au code permettant de gérer correctement les types de données de catégorie.
    + L’interface de la fonction Forecast a été améliorée : le paramètre y_pred est désormais facultatif. Les docstrings ont été améliorés.
  + **azureml-contrib-dataset**
    + Correction d’un bogue dans lequel les jeux de données étiquetés ne pouvaient pas être montés.
  + **azureml-core**
    + Résolution d’un bogue concernant `Environment.from_existing_conda_environment(name, conda_environment_name)`. L’utilisateur peut créer une instance d’environnement qui soit le réplica exact de l’environnement local
    + Remplacement des méthodes de jeux de données relatives à la série chronologique par `include_boundary=True` par défaut.
  + **azureml-train-automl-client**
    + Résolution d’un problème lors duquel les résultats de validation n’étaient pas affichés quand l’option d’affichage de la sortie était définie sur false.


## <a name="2019-11-25"></a>2019-11-25

### <a name="azure-machine-learning-sdk-for-python-v1076"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.76

+ **Dernières modifications**
  + Problèmes de mise à niveau d’Azureml-Train-AutoML
    + La mise à niveau vers azureml-train-automl>=1.0.76 à partir de azureml-train-automl<1.0.76 peut provoquer des installations partielles, entraînant l’échec de certaines importations AutoML. Pour résoudre ce risque, vous pouvez exécuter le script d’installation qui se trouve sur https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/automl_setup.cmd. Ou, si vous utilisez pip directement, vous pouvez :
      + « pip install --upgrade azureml-train-automl »
      + « pip install --ignore-installed azureml-train-automl-client »
    + ou vous pouvez désinstaller l’ancienne version avant la mise à niveau
      + « pip uninstall azureml-train-automl »
      + « pip install azureml-train-automl »

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-runtime**
    + AutoML prend désormais en compte les deux classes true et false lors du calcul des métriques scalaires moyennes pour les tâches de classification binaires.
    + Déplacement du Machine Learning et du code de formation dans AzureML-AutoML-Core vers un nouveau package AzureML-AutoML-Runtime.
  + **azureml-contrib-dataset**
    + Lorsque vous appelez `to_pandas_dataframe` sur un jeu de données étiqueté avec l’option de téléchargement, vous pouvez choisir de remplacer les fichiers existants.
    + Lors de l’appel de `keep_columns` ou `drop_columns` qui entraîne la suppression d’une colonne TimeSeries, d’étiquette ou d’image, les fonctionnalités correspondantes seront également supprimées pour le jeu de données.
    + Correction d’un problème avec le chargeur pytorch pour la tâche de détection d’objets.
  + **azureml-contrib-interpret**
    + Suppression du widget de tableau de bord d’explication d’azureml-contrib-interpret, package modifié pour référencer le nouveau package dans interpret_community
    + Version mise à jour d’interpret-community vers 0.2.0
  + **azureml-core**
    + Amélioration des performances de `workspace.datasets`.
    + Ajout de la possibilité d’inscrire des magasins de données Azure SQL Database à l’aide de l’authentification par nom d’utilisateur et mot de passe
    + Correctif pour le chargement RunConfigurations à partir de chemins d’accès relatifs.
    + Lors de l’appel de `keep_columns` ou `drop_columns` qui entraîne la suppression d’une colonne TimeSeries, les fonctionnalités correspondantes seront également supprimées pour le jeu de données.
  + **azureml-interpret**
    + version mise à jour d’interpret-community à 0.2.0
  + **azureml-pipeline-steps**
    + Valeurs prises en charge documentées pour `runconfig_pipeline_params` pour les étapes de pipeline Machine Learning Azure.
  + **azureml-pipeline-core**
    + Ajout de l’option CLI pour télécharger la sortie au format json pour les commandes de pipeline.
  + **azureml-train-automl**
    + Fractionner AzureML-train-AutoML en deux packages, un package client AzureML-Train-AutoML-Client et un package de formation ML AzureML-Train-AutoML-Runtime
  + **azureml-train-automl-client**
    + Ajout d’un client léger pour l’envoi d’expériences AutoML sans avoir à installer de dépendances Machine Learning localement.
    + Journalisation fixe des décalages détectés automatiquement, des tailles de fenêtre dynamiques et des horizons maximum dans les exécutions distantes.
  + **azureml-train-automl-runtime**
    + Ajout d’un nouveau package AutoML pour isoler les composants de Machine Learning et d’exécution du client.
  + **azureml-contrib-train-rl**
    + Ajout de la prise en charge de l’apprentissage de renforcement dans le SDK.
    + Ajout de la prise en charge d’AmlWindowsCompute dans RL SDK.


## <a name="2019-11-11"></a>2019-11-11

### <a name="azure-machine-learning-sdk-for-python-v1074"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.74

  + **Fonctionnalités préliminaires**
    + **azureml-contrib-dataset**
      + Après l’importation de azureml-contrib-dataset, vous pouvez appeler `Dataset.Labeled.from_json_lines` au lieu de `._Labeled` pour créer un jeu de données étiqueté.
      + Lorsque vous appelez `to_pandas_dataframe` sur un jeu de données étiqueté avec l’option de téléchargement, vous pouvez choisir de remplacer les fichiers existants.
      + Lors de l’appel de `keep_columns` ou `drop_columns` qui entraîne la suppression d’une colonne TimeSeries, d’étiquette ou d’image, les fonctionnalités correspondantes seront également supprimées pour le jeu de données.
      + Résolution des problèmes liés au chargeur PyTorch lors de l’appel de `dataset.to_torchvision()`.

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Ajout du profilage de modèle à l’interface CLI en préversion.
    + Corrige le changement cassant de Stockage Azure provoquant l’échec de l’interface CLI AzureML.
    + Ajout du type d’équilibrage de charge à MLC pour les types AKS
  + **azureml-automl-core**
    + Correction du problème de détection de l’horizon maximal sur la série chronologique, avec des valeurs manquantes et des niveaux de précision multiples.
    + Correction du problème lié aux échecs de génération des découpages de validation croisée.
    + Remplacez cette section par un message sous forme de balisage Markdown qui apparaîtra dans les notes de publication : -Amélioration de la gestion des formats réduits dans les jeux de données de prévision.
    + Correction du problème de masquage de certaines informations utilisateur lors de la journalisation. -Amélioration de la journalisation des erreurs lors des exécutions de prévisions.
    + Ajout de psutil en tant que dépendance Conda au fichier de déploiement yml généré automatiquement.
  + **azureml-contrib-mir**
    + Corrige le changement cassant de Stockage Azure provoquant l’échec de l’interface CLI AzureML.
  + **azureml-core**
    + Correction d’un bogue dans lequel les modèles déployés sur Azure Functions produisaient des erreurs de type 500.
    + Correction d’un problème dans lequel le fichier amlignore n’était pas appliqué sur les captures instantanées.
    + Ajout d’une nouvelle API amlcompute. get_active_runs qui renvoie un générateur pour l’exécution et les exécutions en file d’attente sur un amlcompute donné.
    + Ajout du type d’équilibrage de charge à MLC pour les types AKS.
    + Ajout du paramètre booléen append_prefix à download_files dans run.py et download_artifacts_from_prefix dans artifacts_client. Cet indicateur est utilisé pour aplatir sélectivement le chemin d’accès d’origine de sorte que seul le nom du fichier ou du dossier est ajouté au output_directory
    + Résolvez le problème de désérialisation pour `run_config.yml` avec l’utilisation du jeu de données.
    + Lors de l’appel de `keep_columns` ou `drop_columns` qui entraîne la suppression d’une colonne TimeSeries, les fonctionnalités correspondantes seront également supprimées pour le jeu de données.
  + **azureml-interpret**
    + Mise à jour d’interpret-community version sur 0.1.0.3
  + **azureml-train-automl**
    + Correction d’un problème où automl_step peut ne pas imprimer les problèmes de validation.
    + Correction de register_model pour qu’il aboutisse, même si les dépendances sont manquantes localement dans l’environnement du modèle.
    + Résolution d’un problème où certaines exécutions à distance n’étaient pas activées pour le docker.
    + Ajoutez la journalisation de l’exception qui provoque l’échec prématuré d’une exécution locale.
  + **azureml-train-core**
    + Envisagez l’exécution de resume_from dans le cadre du calcul de la meilleure exécution des enfants de paramétrage automatisé.
  + **azureml-pipeline-core**
    + Correction de la gestion des paramètres dans la construction d’arguments du pipeline.
    + Ajout de la description du pipeline et du paramètre de type d’étape YAML.
    + Nouveau format YAML pour l’étape de pipeline et ajout d’un avertissement de désapprobation pour l’ancien format.



## <a name="2019-11-04"></a>04-11-2019

### <a name="web-experience"></a>Expérience web

La page d’accueil de l’espace de travail collaboratif à l’adresse [https://ml.azure.com](https://ml.azure.com) a été améliorée et rebaptisée « Azure Machine Learning Studio ».

À partir de Studio, vous pouvez effectuer l’apprentissage, le test, le déploiement et la gestion de ressources Azure Machine Learning comme des jeux de données, des pipelines, des modèles, des points de terminaison, etc.

Accédez aux outils de création web suivants à partir de Studio :

| Outil basé sur le web | Description | 
|-|-|-|
| Machines virtuelles de notebooks (préversion) | Station de travail cloud entièrement managée | 
| [Machine Learning automatisé](tutorial-first-experiment-automated-ml.md) (préversion) | Aucune expérience de code pour l’automatisation du développement de modèles Machine Learning | 
| [Designer](concept-designer.md) | Outil de modélisation Machine Learning par glisser-déplacer précédemment appelé interface visuelle | 


### <a name="azure-machine-learning-designer-enhancements"></a>Améliorations du concepteur Azure Machine Learning

+ Anciennement appelé l’interface visuelle 
+    11 nouveaux [modules](algorithm-module-reference/module-reference.md) notamment des utilitaires de recommandation, de classement et de formation, avec des fonctionnalités d’ingénierie, de validation croisée et de transformation des données.

### <a name="r-sdk"></a>SDK R 
 
Les scientifiques des données et les développeurs d’IA utilisent le [SDK Azure Machine Learning pour R](tutorial-1st-r-experiment.md) afin de générer et d’exécuter des workflows de machine learning avec Azure Machine Learning.

Le SDK Azure Machine Learning pour R utilise le package `reticulate` pour effectuer une liaison au SDK Python. En se liant directement à Python, le SDK pour R vous permet d’accéder aux objets et méthodes principaux implémentés dans le SDK Python à partir de tout environnement R que vous choisissez.

Les principales fonctionnalités du SDK sont les suivantes :

+    Gérez les ressources cloud pour la surveillance, la journalisation et l’organisation de vos expériences de Machine Learning.
+    Formez des modèles à l’aide de ressources cloud, y compris la formation du modèle avec accélération GPU.
+    Déployez vos modèles en tant que services web sur Azure Container Instances (ACI) et Azure Kubernetes Service (AKS).

Pour obtenir une documentation complète, consultez le [site web du package](https://azure.github.io/azureml-sdk-for-r).

### <a name="azure-machine-learning-integration-with-event-grid"></a>Intégration entre Azure Machine Learning et Event Grid 

Azure Machine Learning est désormais un fournisseur de ressources pour Event Grid. Il est possible de configurer des événements Machine Learning sur le Portail Azure ou avec Azure CLI. Les utilisateurs peuvent créer des événements pour la fin d’une exécution, l’inscription de modèles, le déploiement de modèles et la détection d’une dérive de données. Ces événements peuvent être routés vers des gestionnaires d’événements pris en charge par Event Grid à des fins de consommation. Pour plus d’informations, voir les articles [schema](../event-grid/event-schema-machine-learning.md) et [tutorial](how-to-use-event-grid.md) sur les événements Machine Learning.

## <a name="2019-10-31"></a>2019-10-31

### <a name="azure-machine-learning-sdk-for-python-v1072"></a>SDK Azure Machine Learning pour Python v1.0.72

+ **Nouvelles fonctionnalités**
  + Ajout de superviseurs de jeu de données via le package [**azureml-datadrift**](/python/api/azureml-datadrift), permettant la supervision des jeux de données de série chronologique pour détecter la dérive de données ou d’autres modifications statistiques au fil du temps. Des alertes et des événements peuvent être déclenchés si une dérive est détectée ou si d’autres conditions sur les données sont remplies. Consultez [notre documentation](how-to-monitor-datasets.md) pour plus d’informations.
  + Annonce de deux nouvelles éditions (également appelées références SKU) dans Azure Machine Learning. Avec cette version, vous pouvez désormais créer un espace de travail Azure Machine Learning De base ou Entreprise. Tous les espaces de travail existants sont dirigés par défaut vers l’édition De base, et vous pouvez accéder au portail Azure ou à Studio pour mettre à niveau l’espace de travail à tout moment. Vous pouvez créer un espace de travail De base ou Entreprise à partir du portail Azure. Lisez [notre documentation](./how-to-manage-workspace.md) pour en savoir plus. À partir du SDK, l’édition de votre espace de travail peut être déterminée à l’aide de la propriété « sku » de votre objet espace de travail.
  + Nous avons également apporté des améliorations à la Capacité de calcul Azure Machine Learning : vous pouvez désormais visualiser les métriques de vos clusters (comme le nombre total de nœuds, les nœuds en cours d’exécution, le quota total de cœurs) dans Azure Monitor, en plus de la visualisation des journaux de diagnostic pour le débogage. En outre, vous pouvez également afficher les exécutions en cours ou en file d’attente sur votre cluster et des détails comme les adresses IP des différents nœuds de votre cluster. Vous pouvez les afficher dans le portail ou à l’aide des fonctions correspondantes dans le SDK ou l’interface CLI.

  + **Fonctionnalités préliminaires**
    + Nous publions la prise en charge de la préversion du chiffrement de disque pour votre disque SSD local dans la Capacité de calcul Azure Machine Learning. Créez un ticket de support technique pour obtenir que votre abonnement soit mis en liste verte afin d’utiliser cette fonctionnalité.
    + Préversion publique d’Azure Machine Learning Batch Inference. Azure Machine Learning Batch Inference cible des travaux d’inférence de grande taille qui ne sont pas sensibles au temps. Batch Inference fournit une mise à l’échelle de calcul d’inférence économique avec un débit inégalé pour les applications asynchrones. Ce système est optimisée pour les débits élevés et les inférences autonomes (fire and forget) sur des ensembles de données volumineux.
    + [**azureml-contrib-dataset**](/python/api/azureml-contrib-dataset)
        + Fonctionnalités activées pour le jeu de données étiqueté
        ```Python
        import azureml.core
        from azureml.core import Workspace, Datastore, Dataset
        import azureml.contrib.dataset
        from azureml.contrib.dataset import FileHandlingOption, LabeledDatasetTask

        # create a labeled dataset by passing in your JSON lines file
        dataset = Dataset._Labeled.from_json_lines(datastore.path('path/to/file.jsonl'), LabeledDatasetTask.IMAGE_CLASSIFICATION)

        # download or mount the files in the `image_url` column
        dataset.download()
        dataset.mount()

        # get a pandas dataframe
        from azureml.data.dataset_type_definitions import FileHandlingOption
        dataset.to_pandas_dataframe(FileHandlingOption.DOWNLOAD)
        dataset.to_pandas_dataframe(FileHandlingOption.MOUNT)

        # get a Torchvision dataset
        dataset.to_torchvision()
        ```

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + L’interface CLI prend désormais en charge le packaging de modèle.
    + Ajout d’interface CLI de jeu de données. Pour plus d’informations : `az ml dataset --help`
    + Ajout de la prise en charge du déploiement et du packaging des modèles pris en charge (ONNX, scikit-learn et TensorFlow) sans instance InferenceConfig.
    + Ajout de l’indicateur de remplacement pour le déploiement de service (ACI et AKS) dans le SDK et l’interface CLI. S’il est fourni, l’indicateur remplace le service existant si le service portant le nom existe déjà. Si le service n’existe pas, un nouveau service est créé.
    + Les modèles peuvent être inscrits avec deux nouveaux frameworks, Onnx et Tensorflow. - L’inscription de modèle accepte les exemples de données d’entrée, les exemples de données de sortie et la configuration des ressources pour le modèle.
  + **azureml-automl-core**
    + L’apprentissage d’une itération s’exécute dans un processus enfant uniquement lorsque des contraintes d’exécution sont définies.
    + Ajout d’une barrière de sécurité pour les tâches de prévision, afin de vérifier si une valeur max_horizon spécifiée entraînera un problème de mémoire sur l’ordinateur donné. Si c’est le cas, un message de barrière de sécurité s’affiche.
    + Ajout de la prise en charge de fréquences complexes comme deux ans et un 1 mois. -Ajout d’un message d’erreur compréhensible si la fréquence ne peut pas être déterminée.
    + Ajout d’azureml-defaults à la génération automatique conda env pour résoudre l’échec du déploiement du modèle
    + La conversion des données intermédiaires en jeu de données tabulaire est autorisée dans le pipeline Azure Machine Learning, ainsi que leur utilisation dans `AutoMLStep`.
    + Mise à jour de l’objectif de colonne implémentée pour le streaming.
    + Mise à jour du paramètre de transformateur implémentée pour Imputer et HashOneHotEncoder pour le streaming.
    + Ajout de la taille de données actuelle et de la taille de données minimale requise aux messages d’erreur de validation.
    + Mise à jour de la taille de données minimale requise pour la validation croisée afin de garantir un minimum de deux échantillons dans chaque pli de validation.
  + **azureml-cli-common**
    + L’interface CLI prend désormais en charge le packaging de modèle.
    + Les modèles peuvent être inscrits avec deux nouveaux frameworks, Onnx et Tensorflow.
    + L’inscription de modèle accepte les exemples de données d’entrée, les exemples de données de sortie et la configuration des ressources pour le modèle.
  + **azureml-contrib-gbdt**
    + Correction du canal de publication du Notebook
    + Ajout d’un avertissement pour la cible de calcul non AmlCompute que nous ne prenons pas en charge
    + Ajout de l’estimateur LightGMB au package azureml-contrib-gbdt
  + [**azureml-core**](/python/api/azureml-core)
    + L’interface CLI prend désormais en charge le packaging de modèle.
    + Ajout d’un avertissement d’obsolescence pour les API de jeu de données dépréciées. Consultez l’avis de modification de l’API de jeu de données sur https://aka.ms/tabular-dataset.
    + Modification de [`Dataset.get_by_id`](/python/api/azureml-core/azureml.core.dataset%28class%29#get-by-id-workspace--id-) pour qu’il retourne le nom et la version de l’inscription si le jeu de données est inscrit.
    + Correction d’un bogue qui fait que ScriptRunConfig avec le jeu de données en tant qu’argument ne peut pas être utilisé à plusieurs reprises pour soumettre une exécution expérimentale.
    + Les jeux de données récupérés lors d’une exécution feront l’objet d’un suivi et peuvent être visualisés dans la page des détails d’exécution ou en appelant [`run.get_details()`](/python/api/azureml-core/azureml.core.run%28class%29#get-details--) une fois l’exécution terminée.
    + La conversion des données intermédiaires en jeu de données tabulaire est autorisée dans le pipeline Azure Machine Learning, ainsi que leur utilisation dans [`AutoMLStep`](/python/api/azureml-train-automl-runtime/azureml.train.automl.runtime.automlstep).
    + Ajout de la prise en charge du déploiement et du packaging des modèles pris en charge (ONNX, scikit-learn et TensorFlow) sans instance InferenceConfig.
    + Ajout de l’indicateur de remplacement pour le déploiement de service (ACI et AKS) dans le SDK et l’interface CLI. S’il est fourni, l’indicateur remplace le service existant si le service portant le nom existe déjà. Si le service n’existe pas, un nouveau service est créé.
    +  Les modèles peuvent être inscrits avec deux nouveaux frameworks, Onnx et Tensorflow. L’inscription de modèle accepte les exemples de données d’entrée, les exemples de données de sortie et la configuration des ressources pour le modèle.
    + Ajout d’un nouveau magasin de données pour Azure Database pour MySQL. Ajout d’un exemple pour l’utilisation d’Azure Database pour MySQL dans DataTransferStep dans des pipelines Azure Machine Learning.
    + Ajout de fonctionnalités pour ajouter et supprimer des balises dans des expérimentations - Ajout de fonctionnalités pour supprimer des balises dans les exécutions
    + Ajout de l’indicateur de remplacement pour le déploiement de service (ACI et AKS) dans le SDK et l’interface CLI. S’il est fourni, l’indicateur remplace le service existant si le service portant le nom existe déjà. Si le service n’existe pas, un nouveau service est créé.
  + [**azureml-datadrift**](/python/api/azureml-datadrift)
    + Déplacé de `azureml-contrib-datadrift` vers `azureml-datadrift`
    + Ajout de la prise en charge de la surveillance des jeux de données TimeSeries pour la dérive et d’autres mesures statistiques
    + Nouvelles méthodes `create_from_model()` et `create_from_dataset()` ajoutées à la classe [`DataDriftDetector`](/python/api/azureml-datadrift/azureml.datadrift.datadriftdetector%28class%29). La méthode `create()` sera déconseillée.
    + Ajustements des visualisations dans Python et l’interface utilisateur dans Azure Machine Learning Studio.
    + Prise en charge de la planification des analyses hebdomadaires et mensuelles, en plus des analyses quotidiennes des jeux de données.
    + Prise en charge du renvoi des métriques du moniteur de données pour analyser les données historiques des analyses de jeu de données.
    + Divers correctifs de bogues
  + [**azureml-pipeline-core**](/python/api/azureml-pipeline-core)
    + azureml-dataprep n’est plus nécessaire pour soumettre une exécution de pipeline Azure Machine Learning à partir du fichier `yaml` du pipeline.
  + [**azureml-train-automl**](/python/api/azureml-train-automl-runtime/)
    + Ajout d’azureml-defaults à la génération automatique conda env pour résoudre l’échec du déploiement du modèle
    + La formation à distance AutoML comprend désormais les valeurs azureml-defaults pour permettre la réutilisation de l’environnement de formation pour l’inférence.
  + **azureml-train-core**
    + Ajout de la prise en charge de PyTorch 1.3 dans l’estimateur [`PyTorch`](/python/api/azureml-train-core/azureml.train.dnn.pytorch)

## <a name="2019-10-21"></a>2019-10-21

### <a name="visual-interface-preview"></a>Interface visuelle (préversion)

+ L’interface visuelle Azure Machine Learning (préversion) a été repensée pour s’exécuter sur [les pipelines Azure Machine Learning](concept-ml-pipelines.md). Les pipelines (auparavant appelés expériences) créés dans l’interface visuelle sont désormais entièrement intégrés à l’expérience de base Azure Machine Learning.
  + Expérience de gestion unifiée avec des ressources de kit de développement logiciel (SDK)
  + Contrôle de version et suivi des modèles d’interface visuels, des pipelines et des points de terminaison
  + Interface utilisateur repensée
  + Ajout du déploiement Batch Inference
  + Ajout de la prise en charge d’Azure Kubernetes service (AKS) pour les des cibles de calcul de l’inférence
  + Nouveau flux de travail de création de pipeline au niveau de Python
  + Nouvelle [page de destination](https://ml.azure.com) pour les outils de création visuelle

+ **Nouveaux modules**
  + Appliquer une opération mathématique
  + Appliquer une transformation SQL
  + Limiter les valeurs
  + Totaliser les données
  + Importer à partir de la base de données SQL

## <a name="2019-10-14"></a>2019-10-14

### <a name="azure-machine-learning-sdk-for-python-v1069"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.69

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Limitation des explications de modèle à exécuter au mieux au lieu de calculer les explications pour chaque exécution. Modification de ce comportement pour local, remote et ADB.
    + Ajout de la prise en charge des explications de modèle à la demande pour l’interface utilisateur
    + Ajout de psutil en tant que dépendance de `automl` et psutil inclus en tant que dépendance conda dans amlcompute.
    + Correction du problème lié aux décalages heuristiques et aux tailles de fenêtre dynamique sur les jeux de données de prévisions, dont certaines séries peuvent provoquer des erreurs algébriques linéaires
      + Ajout de l’impression pour les paramètres déterminés de manière heuristique dans les exécutions de prévision.
  + **azureml-contrib-datadrift**
    + Ajout de la protection lors de la création de métriques de sortie si la dérive du niveau de jeu de données ne figure pas dans la première section.
  + **azureml-contrib-interpret**
    + Le package azureml-contrib-explain-model a été renommé en azureml-contrib-interpret
  + **azureml-core**
    + API ajoutée pour annuler l’inscription des jeux de données. `dataset.unregister_all_versions()`
    + Le package azureml-contrib-explain-model a été renommé azureml-contrib-interpret.
  + **[azureml-core](/python/api/azureml-core)**
    + API ajoutée pour annuler l’inscription des jeux de données. dataset.[unregister_all_versions()](/python/api/azureml-core/azureml.data.abstract_datastore.abstractdatastore#unregister--).
    + Ajout de l’API de jeu données pour vérifier l’heure de modification des données. `dataset.data_changed_time`.
    + Pouvoir utiliser `FileDataset` et `TabularDataset` en tant qu’entrées pour `PythonScriptStep`, `EstimatorStep` et `HyperDriveStep` dans le pipeline Azure Machine Learning
    + Les performances de `FileDataset.mount` ont été améliorées pour les dossiers avec un grand nombre de fichiers
    + Pouvoir consommer [FileDataset](/python/api/azureml-core/azureml.data.filedataset) et [TabularDataset](/python/api/azureml-core/azureml.data.tabulardataset) comme entrées pour [PythonScriptStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep), [EstimatorStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.estimatorstep) et [HyperDriveStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.hyperdrivestep) dans le pipeline Azure Machine Learning.
    + Les performances de FileDataset.[mount()](/python/api/azureml-core/azureml.data.filedataset#mount-mount-point-none----kwargs-) ont été améliorées pour les dossiers avec un grand nombre de fichiers
    + Ajout de l’URL aux recommandations d’erreurs connues dans les détails de l’exécution.
    + Correction d’un bogue dans run.get_metrics lors duquel les demandes échouaient si une exécution avait un trop grand nombre d’enfants
    + Correction d’un bogue dans [run.get_metrics](/python/api/azureml-core/azureml.core.run.run#get-metrics-name-none--recursive-false--run-type-none--populate-false-) lors duquel les demandes échouaient si une exécution avait un trop grand nombre d’enfants
    + Ajout de la prise en charge de l’authentification sur le cluster Arcadia.
    + La création d’un objet Experiment permet d’obtenir ou de créer l’expérience dans l’espace de travail Azure Machine Learning pour le suivi de l’historique des exécutions. L’ID d’expérience et l’heure archivée sont renseignés dans l’objet Experiment lors de la création. Exemple : experiment = Experiment(workspace, "New Experiment") experiment_id = experiment.id archive() et reactivate() sont des fonctions qui peuvent être appelées dans une expérience pour masquer et restaurer l’expérience à partir de l’expérience utilisateur ou retournées par défaut dans un appel pour répertorier les expériences. Si une nouvelle expérience est créée avec le même nom qu’une expérience archivée, vous pouvez renommer l’expérience archivée lors de la réactivation en transmettant un nouveau nom. Il ne peut y avoir qu’une seule expérience active avec un nom donné. Exemple : experiment1 = Experiment(workspace, "Active Experiment") experiment1.archive() # Créez une nouvelle expérience active portant le même nom que celle archivée. experiment2. = Experiment(workspace, "Active Experiment") experiment1.reactivate(new_name="Previous Active Experiment") La liste de méthodes statiques() dans Experiment peut accepter un filtre de nom et un filtre ViewType. Les valeurs ViewType sont "ACTIVE_ONLY", "ARCHIVED_ONLY" et "ALL" Exemple : archived_experiments = Experiment.list(workspace, view_type="ARCHIVED_ONLY") all_first_experiments = Experiment.list(workspace, name="First Experiment", view_type="ALL")
    + Prise en charge de l’utilisation de l’environnement pour le déploiement de modèle et la mise à jour du service.
  + **azureml-datadrift**
    + L’attribut show de la classe DataDriftDector ne prend plus en charge l’argument facultatif « with_details ». L’attribut show affichera uniquement le coefficient de dérive des données et la contribution à la dérive des données des colonnes de caractéristiques.
    + Modifications du comportement de l’attribut DataDriftDetector « get_Output » :
      + les paramètres d’entrée start_time et end_time sont facultatifs au lieu d’obligatoires ;
      + Les entrée spécifiques start_time et/ou end_time avec un run_id spécifique dans le même appel entraînent une exception d’erreur de valeur, car elles s’excluent mutuellement ;
      + lorsque vous entrez des valeurs spécifiques pour start_time et/ou end_time, seuls les résultats des exécutions planifiées sont retournés ;
      + Le paramètre « daily_latest_only » est déconseillé.
    + Prise en charge de la récupération des sorties de dérive des données basées sur les jeux de données.
  + **azureml-explain-model**
    + Renomme le package AzureML-explain-model en AzureML-interpret, en conservant l’ancien package pour la compatibilité descendante pour le moment
    + Correction du bogue `automl` avec des explications brutes définies sur la tâche de classification au lieu de la régression par défaut lors d’un téléchargement à partir d’ExplanationClient
    + Ajout de la prise en charge de `ScoringExplainer` à créer directement à l’aide de `MimicWrapper`
  + **azureml-pipeline-core**
    + Amélioration des performances pour la création de pipelines volumineux
  + **azureml-train-core**
    + Ajout de la prise en charge de TensorFlow 2.0 dans l’estimateur TensorFlow
  + **azureml-train-automl**
    + La création d’un objet [Experiment](/python/api/azureml-core/azureml.core.experiment.experiment) permet d’obtenir ou de créer l’expérience dans l’espace de travail Azure Machine Learning pour le suivi de l’historique des exécutions. L’ID d’expérience et l’heure archivée sont renseignés dans l’objet Experiment lors de la création. Exemple :

        ```py
        experiment = Experiment(workspace, "New Experiment")
        experiment_id = experiment.id
        ```
        [archive()](/python/api/azureml-core/azureml.core.experiment.experiment#archive--) et [reactivate()](/python/api/azureml-core/azureml.core.experiment.experiment#reactivate-new-name-none-) sont des fonctions qui peuvent être appelées dans une expérience pour masquer et restaurer l’expérience à partir de l’expérience utilisateur ou retournées par défaut dans un appel pour répertorier les expériences. Si une nouvelle expérience est créée avec le même nom qu’une expérience archivée, vous pouvez renommer l’expérience archivée lors de la réactivation en transmettant un nouveau nom. Il ne peut y avoir qu’une seule expérience active avec un nom donné. Exemple :

        ```py
        experiment1 = Experiment(workspace, "Active Experiment")
        experiment1.archive()
        # Create new active experiment with the same name as the archived.
        experiment2 = Experiment(workspace, "Active Experiment")
        experiment1.reactivate(new_name="Previous Active Experiment")
        ```
        La méthode statique [list()](/python/api/azureml-core/azureml.core.experiment.experiment#list-workspace--experiment-name-none--view-type--activeonly---tags-none-) dans Experiment peut accepter un filtre de nom et un filtre ViewType. Les valeurs ViewType sont « ACTIVE_ONLY », « ARCHIVED_ONLY » et « ALL ». Exemple :

        ```py
        archived_experiments = Experiment.list(workspace, view_type="ARCHIVED_ONLY")
        all_first_experiments = Experiment.list(workspace, name="First Experiment", view_type="ALL")
        ```
    + Prise en charge de l’utilisation de l’environnement pour le déploiement de modèle et la mise à jour du service.
  + **[azureml-datadrift](/python/api/azureml-datadrift)**
    + L’attribut show de la classe [DataDriftDetector](/python/api/azureml-datadrift/azureml.datadrift.datadriftdetector.datadriftdetector) ne prend plus en charge l’argument facultatif « with_details ». L’attribut show affichera uniquement le coefficient de dérive des données et la contribution à la dérive des données des colonnes de caractéristiques.
    + Le comportement de la fonction DataDriftDetector [get_output]python/api/azureml-datadrift/azureml.datadrift.datadriftdetector.datadriftdetector#get-output-start-time-none--end-time-none--run-id-none-) change :
      + les paramètres d’entrée start_time et end_time sont facultatifs au lieu d’obligatoires ;
      + Les entrée spécifiques start_time et/ou end_time avec un run_id spécifique dans le même appel entraînent une exception d’erreur de valeur, car elles s’excluent mutuellement ;
      + lorsque vous entrez des valeurs spécifiques pour start_time et/ou end_time, seuls les résultats des exécutions planifiées sont retournés ;
      + Le paramètre « daily_latest_only » est déconseillé.
    + Prise en charge de la récupération des sorties de dérive des données basées sur les jeux de données.
  + **azureml-explain-model**
    + Ajout de la prise en charge de [ScoringExplainer](/python/api/azureml-interpret/azureml.interpret.scoring.scoring_explainer.scoringexplainer?view=azure-ml-py&preserve-view=true) pour une création directe à l’aide de MimicWrapper
  + **[azureml-pipeline-core](/python/api/azureml-pipeline-core)**
    + Amélioration des performances pour la création de pipelines volumineux.
  + **[azureml-train-core](/python/api/azureml-train-core)**
    + Ajout de la prise en charge de TensorFlow 2.0 dans l’estimateur [TensorFlow](/python/api/azureml-train-core/azureml.train.dnn.tensorflow).
  + **[azureml-train-automl](/python/api/azureml-train-automl-runtime/)**
    + L’exécution parente n’est plus en échec en cas d’échec de l’itération du programme d’installation, car l’orchestration s’en charge déjà.
    + Ajout de la prise en charge de local-docker et local-conda pour les expériences AutoML
    + Ajout de la prise en charge de local-docker et local-conda pour les expériences AutoML.


## <a name="2019-10-08"></a>2019-10-08

### <a name="new-web-experience-preview-for-azure-machine-learning-workspaces"></a>Nouvelle expérience web (préversion) pour les espaces de travail Azure Machine Learning

L’onglet Expérience dans le [nouveau portail des espaces de travail](https://ml.azure.com) a été mis à jour pour que les scientifiques des données puissent superviser leurs expériences de manière plus performante. Vous pouvez explorer les fonctionnalités suivantes :
+ Métadonnées d’expérience pour filtrer et trier facilement votre liste d’expériences
+ Pages de détails des expériences simplifiées et performantes pour vous permettre de visualiser et de comparer vos exécutions
+ Nouvelle conception des pages de détails des exécutions pour comprendre et superviser vos exécutions d’entraînement

## <a name="2019-09-30"></a>30-09-2019

### <a name="azure-machine-learning-sdk-for-python-v1065"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.65

  + **Nouvelles fonctionnalités**
    + Ajout d’environnements organisés. Ces environnements ont été préconfigurés avec des bibliothèques pour des tâches de Machine Learning courantes, et ont été précompilés et mis en cache en tant qu’images Docker afin d’accélérer l’exécution. Celles-ci s’affichent par défaut dans la liste d’environnements de l’espace de travail, avec le préfixe « AzureML ».
    + Ajout d’environnements organisés. Ces environnements ont été préconfigurés avec des bibliothèques pour des tâches de Machine Learning courantes, et ont été précompilés et mis en cache en tant qu’images Docker afin d’accélérer l’exécution. Celles-ci s’affichent par défaut dans la liste d’environnements de [l’espace de travail](/python/api/azureml-core/azureml.core.workspace%28class%29), avec le préfixe « AzureML ».

  + **azureml-train-automl**
  + **[azureml-train-automl](/python/api/azureml-train-automl-runtime/)**
    + Ajout de la prise en charge de la conversion ONNX pour ADB et HDI

+ **Fonctionnalités préliminaires**
  + **azureml-train-automl**
  + **[azureml-train-automl](/python/api/azureml-train-automl-runtime/)**
    + BERT et BiLSTM pris en charge en tant que caractériseur de texte (préversion uniquement)
    + Personnalisation de caractérisation prise en charge pour les paramètres d’objet de colonne et de transformateur (préversion uniquement)
    + Explications brutes prises en charge lorsque l’utilisateur active l’explication du modèle pendant l’apprentissage (préversion uniquement)
    + Ajout de Prophet pour les prévisions `timeseries` en tant que pipeline apte à l’apprentissage (préversion uniquement)

  + **azureml-contrib-datadrift**
    + Déplacement de packages d’azureml-contrib-datadrift vers azureml-datadrift ; le package `contrib` sera supprimé dans une mise en production ultérieure

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Introduction de FeaturizationConfig dans AutoMLConfig et AutoMLBaseSettings
    + Introduction de FeaturizationConfig dans [AutoMLConfig](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig) et AutoMLBaseSettings
      + Remplacement de l’objectif de la colonne pour caractérisation avec une colonne et un type de caractéristique donnés
      + Remplacement des paramètres du transformateur
    + Ajout d’un message de dépréciation pour explain_model() et retrieve_model_explanations()
    + Ajout de Prophet en tant que pipeline apte à l’apprentissage (préversion uniquement)
    + Ajout d’un message de dépréciation pour explain_model() et retrieve_model_explanations().
    + Ajout de Prophet en tant que pipeline apte à l’apprentissage (préversion uniquement).
    + Ajout de la prise en charge de la détection automatique des décalages cibles, de la taille de fenêtre dynamique et de l’horizon maximal. Si l’une des valeurs target_lags, target_rolling_window_size ou max_horizon est définie sur « auto », les heuristiques sont appliquées pour estimer la valeur du paramètre correspondant en fonction des données d’apprentissage.
    + Correction des prévisions lorsque le jeu de données contient une colonne de grain ; ce grain est de type numérique et il y a un écart entre l’apprentissage et le jeu de test
    + Correction du message d’erreur relatif à l’index dupliqué dans la série distante des tâches de prévision
    + Correction des prévisions lorsque le jeu de données contient une colonne de grain ; ce grain est de type numérique et il y a un écart entre l’apprentissage et le jeu de test.
    + Correction du message d’erreur relatif à l’index dupliqué dans la série distante des tâches de prévision.
    + Ajout d’une barrière de sécurité pour vérifier si un jeu de données est déséquilibré. Si c’est le cas, un message de barrière de sécurité s’affiche sur la console.
  + **azureml-core**
    + Ajout de la possibilité de récupérer l’URL SAP du modèle dans le stockage via l’objet de modèle Ex. : model.get_sas_url()
    + Introduction de `run.get_details()['datasets']` pour obtenir des jeux de données associés à l’exécution envoyée
    + Ajout de l’API `Dataset.Tabular.from_json_lines_files` pour créer un TabularDataset à partir de fichiers Lignes JSON. Pour en savoir plus sur ces données tabulaires dans les fichiers de Lignes JSON sur TabularDataset, consultez [cet article](how-to-create-register-datasets.md) pour accéder à la documentation.
    + Ajout de champs de taille de machine virtuelle (disque du système d’exploitation, nombre de GPU) à la fonction supported_vmsizes ()
    + Ajout de champs à la fonction list_nodes () pour afficher l’exécution, les adresses IP privée et publique, le port, etc.
    + Possibilité de spécifier un nouveau champ pendant l’approvisionnement du cluster, --remotelogin_port_public_access, qui peut être défini sur activé ou désactivé selon que vous souhaitez laisser le port SSH ouvert ou fermé au moment de la création du cluster. Si vous ne le spécifiez pas, le service ouvre ou ferme intelligemment le port selon que vous déployez ou non le cluster à l’intérieur d’un réseau virtuel.
  + **azureml-explain-model**
  + **[azureml-core](/python/api/azureml-core/azureml.core)**
    + Ajout de la possibilité de récupérer l’URL SAP du modèle dans le stockage via l’objet de modèle Ex. : model.[get_sas_url()](/python/api/azureml-core/azureml.core.model.model#get-sas-urls--)
    + Introduction de run.[get_details](/python/api/azureml-core/azureml.core.run%28class%29#get-details--)['datasets'] pour obtenir des jeux de données associés à l’exécution envoyée
    + Ajout de l’API `Dataset.Tabular`.[from_json_lines_files()](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#from-json-lines-files-path--validate-true--include-path-false--set-column-types-none--partition-format-none-) pour créer un TabularDataset à partir de fichiers Lignes JSON. Pour découvrir ces données tabulaires dans les fichiers Lignes JSON sur TabularDataset, visitez la page https://aka.ms/azureml-data pour accéder à la documentation.
    + Ajout de champs de taille de machine virtuelle (disque du système d’exploitation, nombre de GPU) à la fonction [supported_vmsizes()](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute#supported-vmsizes-workspace--location-none-)
    + Ajout de champs à la fonction [list_nodes()](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute#list-nodes--) pour afficher l’exécution, les adresses IP privée et publique, le port, etc.
    + Possibilité de spécifier un nouveau champ pendant le [provisionnement](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute#provisioning-configuration-vm-size-----vm-priority--dedicated---min-nodes-0--max-nodes-none--idle-seconds-before-scaledown-none--admin-username-none--admin-user-password-none--admin-user-ssh-key-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--tags-none--description-none--remote-login-port-public-access--notspecified--) du cluster, qui peut être défini sur activé ou désactivé pour laisser le port SSH ouvert ou fermé au moment de la création du cluster. Si vous ne le spécifiez pas, le service ouvre ou ferme intelligemment le port selon que vous déployez ou non le cluster à l’intérieur d’un réseau virtuel.
  + **azureml-explain-model**
    + Amélioration de la documentation pour les sorties d’explication dans le scénario de classification.
    + Ajout de la possibilité de charger les valeurs y prédites sur l’explication des exemples d’évaluation. Déverrouille des visualisations plus utiles.
    + Ajout de la propriété explainer à MimicWrapper pour permettre l’obtention du MimicExplainer sous-jacent.
  + **azureml-pipeline-core**
    + Ajout d’un bloc-notes pour décrire Module, ModuleVersion et ModuleStep
  + **azureml-pipeline-steps**
    + Ajout de RScriptStep pour prendre en charge l’exécution de script R via le pipeline AML.
    + Correction de l’analyse des paramètres de métadonnées dans AzureBatchStep qui générait le message d’erreur « Assignment for parameter SubscriptionId is not specified » (L’affectation du paramètre SubscriptionId n’est pas spécifiée).
  + **azureml-train-automl**
    + Prise en charge de training_data, validation_data, label_column_name, weight_column_name en tant que format d’entrée de données
    + Ajout d’un message de dépréciation pour explain_model() et retrieve_model_explanations()
  + **[azureml-pipeline-core](/python/api/azureml-pipeline-core)**
    + Ajout d’un [bloc-notes](https://aka.ms/pl-modulestep) pour décrire [Module](/python/api/azureml-pipeline-core/azureml.pipeline.core.module%28class%29), [ModuleVersion, et [ModuleStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.modulestep).
  + **[azureml-pipeline-steps](/python/api/azureml-pipeline-steps)**
    + Ajout de [RScriptStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.rscriptstep) pour prendre en charge l’exécution de script R via le pipeline AML.
    + Correction de l’analyse des paramètres de métadonnées dans AzureBatchStep qui générait le message d’erreur « Assignment for parameter SubscriptionId is not specified » (L’affectation du paramètre SubscriptionId n’est pas spécifiée).
  + **[azureml-train-automl](/python/api/azureml-train-automl-runtime/)**
    + Prise en charge de training_data, validation_data, label_column_name, weight_column_name en tant que format d’entrée de données.
    + Ajout d’un message de dépréciation pour [explain_model()](/python/api/azureml-train-automl-runtime/azureml.train.automl.runtime.automlexplainer#explain-model-fitted-model--x-train--x-test--best-run-none--features-none--y-train-none----kwargs-) et [retrieve_model_explanations()](/python/api/azureml-train-automl-runtime/azureml.train.automl.runtime.automlexplainer#retrieve-model-explanation-child-run-).


## <a name="2019-09-16"></a>2019-09-16

### <a name="azure-machine-learning-sdk-for-python-v1062"></a>Kit SDK Azure Machine Learning pour Python v1.0.62

+ **Nouvelles fonctionnalités**
  + Introduction de la caractéristique `timeseries` sur TabularDataset. Cette caractéristique permet de filtrer facilement les horodatages sur les données TabularDataset, par exemple capturer toutes les données d’une plage de temps ou les données les plus récentes.  https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/timeseries-datasets/tabular-timeseries-dataset-filtering.ipynb pour accéder à un exemple de notebook.
  + Activation de l’entraînement avec TabularDataset et FileDataset. 

  + **azureml-train-core**
      + Ajout de la prise en charge de `Nccl` et `Gloo` dans PyTorch Estimator

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Le paramètre AutoML « lag_length » et LaggingTransformer ont été dépréciés.
    + Correction de la validation des données d’entrée si elles sont spécifiées dans un format Dataflow
    + Modification de fit_pipeline.py pour générer le json de graphe et charger dans les artefacts.
    + Rendu du graphe sous `userrun` à l’aide de `Cytoscape`.
  + **azureml-core**
    + Nous avons révisé la gestion des exceptions dans le code ADB et apporté des modifications en fonction de la nouvelle gestion des erreurs
    + Ajout de l’authentification MSI automatique pour les machines virtuelles de notebook.
    + Correction d’un bogue où des modèles endommagés ou vides pouvaient être chargés en raison de l’échec des nouvelles tentatives.
    + Correction du bogue qui provoquait le changement du nom `DataReference` lors du changement du mode `DataReference` (par exemple lors de l’appel de `as_upload`, `as_download` ou `as_mount`).
    + `mount_point` et `target_path` sont désormais facultatifs pour `FileDataset.mount` et `FileDataset.download`.
    + Une exception indiquant que la colonne timestamp est introuvable sera levée si l’API associée aux séries chronologiques est appelée sans que la colonne timestamp fine soit affectée ou si les colonnes timestamp affectées sont supprimées.
    + Les colonnes de séries chronologiques doivent être assignées avec une colonne de type Date, sinon une exception est attendue.
    + Les colonnes de séries chronologiques affectant l’API « with_timestamp_columns » peuvent prendre la valeur None pour le nom de colonne timestamp fine/grossière, ce qui effacera les colonnes timestamp précédemment affectées.
    + Une exception est levée quand la colonne timestamp fine ou grossière est supprimée, avec une indication signalant à l’utilisateur que la suppression peut être effectuée soit après l’exclusion de la colonne timestamp de la liste de suppression, soit par l’appel de with_time_stamp avec une valeur None pour libérer les colonnes timestamp.
    + Une exception est levée quand la colonne timestamp fine ou grossière n’est pas incluse dans la liste de colonnes à conserver, avec une indication signalant à l’utilisateur que la conservation peut être effectuée soit après l’inclusion de la colonne timestamp dans la liste des colonnes à conserver, soit par l’appel de with_time_stamp avec une valeur None pour libérer les colonnes timestamp.
    + Ajout de la journalisation de la taille d’un modèle inscrit.
  + **azureml-explain-model**
    + Correction de l’avertissement affiché sur la console quand le package Python « packaging » n’est pas installé : « Utilisation d’une version antérieure à la version prise en charge de lightgbm. Veuillez effectuer une mise à niveau vers une version supérieure à 2.2.1 ».
    + Correction de l’explication relative au modèle de téléchargement avec partitionnement pour des explications globales avec de nombreuses fonctionnalités.
    + Correction des exemples MimicExplainer où il manquait l’initialisation lors de l’explication sur la sortie.
    + Correction d’une erreur immuable sur les propriétés de définition lors du chargement avec client d’explication utilisant deux types de modèles différents.
    + Ajout d’un paramètre get_raw à l’explainer de scoring .explain() afin qu’un explainer de scoring puisse retourner à la fois des valeurs de conception et des valeurs brutes.
  + **azureml-train-automl**
    + Introduction d’API publiques d’AutoML pour prendre en charge les explications à partir du SDK d’explication `automl`. Nouvelle manière de prendre en charge les explications AutoML en découplant la caractérisation AutoML et le SDK d’explication. Intégration de la prise en charge de l’explication brute à partir du SDK d’explication automl pour les modèles AutoML.
    + Suppression d’azureml-defaults des environnements d’entraînement à distance.
    + Modification de l’emplacement du magasin de cache par défaut de FileCacheStore en AzureFileCacheStore pour AutoML sur le chemin de code Azure Databricks.
    + Correction de la validation des données d’entrée si elles sont spécifiées dans un format Dataflow
  + **azureml-train-core**
    + Annulation de la dépréciation de source_directory_data_store.
    + Ajout de la possibilité de remplacer les versions de packages installés par azureml.
    + Ajout de la prise en charge de dockerfile dans le paramètre `environment_definition` dans les estimateurs.
    + Simplification des paramètres d’entraînement distribué dans les estimateurs.

         ```py
        from azureml.train.dnn import TensorFlow, Mpi, ParameterServer
        ```

## <a name="2019-09-09"></a>2019-09-09

### <a name="new-web-experience-preview-for-azure-machine-learning-workspaces"></a>Nouvelle expérience web (préversion) pour les espaces de travail Azure Machine Learning
La nouvelle expérience web permet aux chercheurs de données et aux ingénieurs de données d’effectuer leur cycle de vie Machine Learning de bout en bout, de la préparation et la visualisation des données à la formation et au déploiement des modèles dans un emplacement unique.

![IU de l’espaces de travail Azure Machine Learning (préversion)](./media/azure-machine-learning-release-notes/new-ui-for-workspaces.jpg)

**Fonctionnalités clés :**

À l’aide de cette nouvelle interface Azure Machine Learning, vous pouvez désormais :
+ Gérer vos notebooks ou le lien vers Jupyter
+ [Exécuter des expériences de Machine Learning automatisé](tutorial-first-experiment-automated-ml.md)
+ [Créer des jeux de données à partir de fichiers locaux, de banques de données et de fichiers web](how-to-create-register-datasets.md)
+ Explorer et préparer les jeux de données pour la création de modèles
+ Surveiller la dérive des données pour vos modèles
+ Afficher les ressources récentes à partir d’un tableau de bord

Au moment de cette publication, les navigateurs suivants sont pris en charge : Chrome, Firefox, Safari et la préversion Microsoft Edge.

**Problèmes connus :**

1. Actualisez votre navigateur si vous voyez s’afficher le message « Un problème s’est produit. Erreur lors du chargement des fichiers blocs » lorsque le déploiement est en cours.

1. Impossible de supprimer ou de renommer un fichier dans Notebooks et Files. Pendant la préversion publique, vous pouvez utiliser le terminal ou l’interface utilisateur Jupyter sur la machine virtuelle Notebook pour effectuer les opérations de mise à jour des fichiers. Étant donné qu’il s’agit d’un système de fichiers réseau monté, toutes les modifications que vous apportez à la machine virtuelle Notebook sont immédiatement reflétées dans l’espace de travail Notebook.

1. Pour une connexion SSH à la machine virtuelle Notebook :
   1. Recherchez les clés SSH qui ont été créées lors de la configuration de la machine virtuelle. Ou bien, recherchez les clés dans l’espace de travail Azure Machine Learning > ouvrez l’onglet Calcul > localisez la machine virtuelle Notebook dans la liste > ouvrez ses propriétés : copiez les clés à partir de la boîte de dialogue.
   1. Importez ces clés SSH publiques et privées sur votre ordinateur local.
   1. Utilisez-les pour effectuer une connexion SSH à la machine virtuelle Notebook.

## <a name="2019-09-03"></a>2019-09-03
### <a name="azure-machine-learning-sdk-for-python-v1060"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.60

+ **Nouvelles fonctionnalités**
  + Ajout de FileDataset, qui fait référence à des fichiers uniques ou multiples dans vos magasins de fichiers ou vos URL publiques. Les fichiers peuvent être de n’importe quel format. FileDataset vous offre la possibilité de télécharger ou de monter les fichiers dans votre calcul. 
  + Ajout de la prise en charge du YAML de pipeline pour les étapes PythonScript, Adla, Databricks, DataTransfer et AzureBatch

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + AutoArima est désormais un pipeline suggéré pour la préversion uniquement.
    + Amélioration des rapports d’erreurs pour les prévisions.
    + Amélioration de la journalisation à l’aide d’exceptions personnalisées plutôt que génériques dans les tâches de prévision.
    + La vérification sur max_concurrent_iterations a été supprimée pour être inférieure au nombre total d’itérations.
    + Les modèles AutoML retournent désormais AutoMLExceptions
    + Cette version améliore les performances d’exécution des exécutions locales de Machine Learning automatisé.
  + **azureml-core**
    + Ajoute Dataset.get_all(workspace), qui retourne un dictionnaire d'objets `TabularDataset` et `FileDataset` indexés par nom d’inscription.

    ```py
    workspace = Workspace.from_config()
    all_datasets = Dataset.get_all(workspace)
    mydata = all_datasets['my-data']
    ```

    + Ajout de `parition_format` comme argument pour `Dataset.Tabular.from_delimited_files` et `Dataset.Tabular.from_parquet.files`. Les informations de partition de chaque chemin de données sont extraites dans des colonnes en fonction du format spécifié. « {column_name} » crée une colonne de chaîne, et « {column_name:yyyy/MM/dd/HH/mm/ss} » crée une colonne DateTime, où « yyyy », « MM », « dd », « HH », « mm » et « ss » sont utilisés pour extraire l’année, le mois, le jour, l’heure, la minute et la seconde pour le type DateTime. partition_format doit commencer à partir de la position de la première clé de partition et se poursuivre jusqu’à la fin du chemin d’accès au fichier. Par exemple, si l’on prend le chemin d’accès '../USA/2019/01/01/data.csv' où la partition se fait par pays et par heure, partition_format='/{Country}/{PartitionDate:yyyy/MM/dd}/data.csv' crée la colonne chaîne « Country » avec la valeur « USA » et la colonne DateTime « PartitionDate » avec la valeur « 2019-01-01 ».
        ```py
        workspace = Workspace.from_config()
        all_datasets = Dataset.get_all(workspace)
        mydata = all_datasets['my-data']
        ```

    + Ajout de `partition_format` comme argument pour `Dataset.Tabular.from_delimited_files` et `Dataset.Tabular.from_parquet.files`. Les informations de partition de chaque chemin de données sont extraites dans des colonnes en fonction du format spécifié. « {column_name} » crée une colonne de chaîne, et « {column_name:yyyy/MM/dd/HH/mm/ss} » crée une colonne DateTime, où « yyyy », « MM », « dd », « HH », « mm » et « ss » sont utilisés pour extraire l’année, le mois, le jour, l’heure, la minute et la seconde pour le type DateTime. partition_format doit commencer à partir de la position de la première clé de partition et se poursuivre jusqu’à la fin du chemin d’accès au fichier. Par exemple, si l’on prend le chemin d’accès '../USA/2019/01/01/data.csv' où la partition se fait par pays et par heure, partition_format='/{Country}/{PartitionDate:yyyy/MM/dd}/data.csv' crée la colonne chaîne « Country » avec la valeur « USA » et la colonne DateTime « PartitionDate » avec la valeur « 2019-01-01 ».
    + Les méthodes `to_csv_files` et `to_parquet_files` ont été ajoutées à `TabularDataset`. Ces méthodes permettent la conversion entre `TabularDataset` et `FileDataset` en convertissant les données en fichiers au format spécifié.
    + Connectez-vous automatiquement au registre d’images de base lors de l’enregistrement d’un fichier Dockerfile généré par Model.package().
    + « gpu_support » n’est plus nécessaire ; AML détecte et utilise désormais automatiquement l’extension Docker Nvidia lorsqu’elle est disponible. Elle sera supprimée dans une version ultérieure.
    + Ajout de la prise en charge de la création, de la mise à jour et de l’utilisation de PipelineDrafts.
    + Cette version améliore les performances d’exécution des exécutions locales de Machine Learning automatisé.
    + Les utilisateurs peuvent interroger les métriques à partir de l’historique des exécutions par nom.
    + Amélioration de la journalisation à l’aide d’exceptions personnalisées plutôt que génériques dans les tâches de prévision.
  + **azureml-explain-model**
    + Ajout du paramètre feature_maps au nouveau MimicWrapper, pour permettre aux utilisateurs d’obtenir des explications sur les fonctionnalités brutes.
    + Les chargements de jeux de données sont maintenant désactivés par défaut pour le téléchargement d’explications et peuvent être réactivés avec upload_datasets=True
    + Ajout de paramètres de filtrage « is_law » à la liste d’explications et aux fonctions de téléchargement.
    + Ajoute la méthode `get_raw_explanation(feature_maps)` aux objets d’explication globaux et locaux.
    + Ajout de la vérification de version à lightgbm avec un avertissement imprimé si elle est inférieure à la version prise en charge
    + Utilisation optimisée de la mémoire lors du traitement des explications par lots
    + Les modèles AutoML retournent désormais AutoMLExceptions
  + **azureml-pipeline-core**
    + Ajout de la prise en charge de la création, de la mise à jour et de l’utilisation de PipelineDrafts - peut servir à gérer les définitions de pipeline mutable et les utiliser de manière interactive pour exécuter
  + **azureml-train-automl**
    + Création d’une fonctionnalité pour installer des versions spécifiques de pytorch v1.1.0, :::no-loc text="cuda"::: Toolkit 9.0, pytorch-transformers utilisant le processeur graphique, ce qui est nécessaire à l’activation de BERT/XLNet dans l’environnement d’exécution Python distant.
  + **azureml-train-core**
    + Détermination précoce de l’échec pour certaines erreurs de définition d’espace d’hyperparamètre directement dans le kit de développement logiciel (SDK) et non côté serveur.

### <a name="azure-machine-learning-data-prep-sdk-v1114"></a>Kit de développement logiciel (SDK) de préparation de données Azure Machine Learning v1.1.14
+ **Résolutions de bogue et améliorations**
  + Activation de l’écriture dans ADLS/ADLSGen2 à l’aide du chemin d’accès brut et des informations d’identification.
  + Correction d’un bogue à cause duquel `include_path=True` ne fonctionnait pas pour `read_parquet`.
  + Résolution de l’échec `to_pandas_dataframe()` provoqué par l’exception « Valeur de propriété non valide : hostSecret ».
  + Correction d’un bogue dans lequel les fichiers ne pouvaient pas être lus sur DBFS en mode Spark.

## <a name="2019-08-19"></a>19/08/2019

### <a name="azure-machine-learning-sdk-for-python-v1057"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.57
+ **Nouvelles fonctionnalités**
  + Activation de `TabularDataset` pour consommation par AutomatedML. Pour en savoir plus sur `TabularDataset`, visitez https://aka.ms/azureml/howto/createdatasets.

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Vous pouvez maintenant mettre à jour le certificat TLS/SSL pour le point de terminaison de score déployé sur le cluster AKS pour les certificats générés par Microsoft et par le client.
  + **azureml-automl-core**
    + Résolution d’un problème dans AutoML où les lignes avec des étiquettes manquantes ne sont pas correctement supprimées.
    + Amélioration de la journalisation des erreurs dans AutoML. Les messages d’erreur complets seront désormais toujours écrits dans le fichier journal.
    + AutoML a mis à jour son épinglage de package pour inclure `azureml-defaults`, `azureml-explain-model` et`azureml-dataprep`. AutoML n’avertira plus en cas d’incompatibilité de package (à l’exception du package `azureml-train-automl`).
    + Correction d’un problème dans `timeseries` où les fractionnements du CV sont de taille inégale, ce qui provoque l’échec du calcul des fichiers binaires.
    + Lors de l’exécution de l’ensemble d’itération pour le type d’apprentissage de validation croisée, si nous avons rencontré un problème lors du téléchargement des modèles formés sur l’ensemble du jeu de données, nous avons une incohérence entre la pondération des modèles et les modèles qui ont été intégrés à l’ensemble de vote.
    + Correction de l’erreur déclenchée lorsque des étiquettes d’apprentissage et/ou de validation (y et y_valid) sont fournies sous la forme de trames de données Pandas, mais pas en tant que tableau numpy.
    + Résolution du problème lié aux tâches de prévision quand Aucune a été trouvé dans les colonnes booléennes des tables d’entrée.
    + Autoriser les utilisateurs AutoML à supprimer des séries d’apprentissage qui ne sont pas suffisamment longues lors des prévisions. - Autoriser les utilisateurs AutoML à supprimer des grains du jeu de test qui n’existe pas dans le jeu d’apprentissage lors de la prévision.
  + **azureml-core**
    + Correction du problème de classement du paramètre blob_cache_timeout.
    + Ajout des types d’exceptions d’ajustement externe et de transformation aux erreurs système.
    + Ajout de la prise en charge des secrets Key Vault pour les exécutions distantes. Ajout d’une classe azureml.core.keyvault.Keyvault pour ajouter, obtenir et répertorier les secrets du coffre de clés associé à votre espace de travail. Opérations prises en charge :
      + azureml.core.workspace.Workspace.get_default_keyvault()
      + azureml.core.keyvault.Keyvault.set_secret(name, value)
      + azureml.core.keyvault.Keyvault.set_secrets(secrets_dict)
      + azureml.core.keyvault.Keyvault.get_secret(name)
      + azureml.core.keyvault.Keyvault.get_secrets(secrets_list)
      + azureml.core.keyvault.Keyvault.list_secrets()
    + Méthodes supplémentaires pour obtenir un coffre de clés par défaut et obtenir des secrets lors de l’exécution distante :
      + azureml.core.workspace.Workspace.get_default_keyvault()
      + azureml.core.run.Run.get_secret(name)
      + azureml.core.run.Run.get_secrets(secrets_list)
    + Ajout de paramètres de substitution supplémentaires à la commande submit-hyperdrive de la CLI.
    + Amélioration de la fiabilité des appels d’API pour étendre les nouvelles tentatives aux exceptions de la bibliothèque de demandes courantes.
    + Ajout de la prise en charge de l’envoi des exécutions à partir d’une exécution envoyée.
    + Correction du problème de jeton SAS arrivant à expiration dans FileWatcher, ce qui provoquait l’arrêt du chargement des fichiers après l’expiration de leur jeton initial.
    + Importation des fichiers CSV/TSV HTTP pris en charge dans le SDK Python du jeu de données.
    + Méthode Workspace.Setup() déconseillée. Le message d’avertissement présenté aux utilisateurs suggère d’utiliser create() ou get()/from_config().
    + Ajout de Environment.add_private_pip_wheel(), qui permet de charger des packages Python privés et personnalisés `whl` dans l’espace de travail et de les utiliser en toute sécurité pour générer/matérialiser l’environnement.
    + Vous pouvez maintenant mettre à jour le certificat TLS/SSL pour le point de terminaison de score déployé sur le cluster AKS pour les certificats générés par Microsoft et par le client.
  + **azureml-explain-model**
    + Ajout d’un paramètre pour ajouter un ID de modèle aux explications lors du chargement.
    + Ajout du balisage `is_raw` aux explications en mémoire et au chargement.
    + Ajout de la prise en charge et des tests pytorch pour le package azureml-explain-model.
  + **azureml-opendatasets**
    + Prise en charge de la détection et de la journalisation de l’environnement de test automatique.
    + Ajout de classes pour obtenir la population des États-Unis par comté et par code postal.
  + **azureml-pipeline-core**
    + Ajout de la propriété label aux définitions de port d’entrée et de sortie.
  + **azureml-telemetry**
    + Correction de la configuration de télémétrie incorrecte.
  + **azureml-train-automl**
    + Correction du bogue à l’origine de l’échec de l’installation, l’erreur n’étant pas consignée dans le champ Erreurs de l’exécution et donc pas stockée dans le champ Erreurs du parent.
    + Résolution d’un problème dans AutoML où les lignes avec des étiquettes manquantes ne sont pas correctement supprimées.
    + Autoriser les utilisateurs AutoML à supprimer des séries d’apprentissage qui ne sont pas suffisamment longues lors des prévisions.
    + Autoriser les utilisateurs AutoML à supprimer des grains du jeu de test qui n’existe pas dans le jeu d’apprentissage lors de la prévision.
    + Désormais, AutoMLStep passe par la configuration `automl` à backend pour éviter tout problème concernant la modification ou l’ajout de paramètres de configuration.
    + AutoML Data Guardrail est désormais en préversion publique. L’utilisateur verra un rapport Data Guardrail (pour les tâches de classification/régression) après l’apprentissage et pourra également y accéder via l’API du SDK.
  + **azureml-train-core**
    + Ajout de la prise en charge de Torch 1.2 dans PyTorch Estimator.
  + **azureml-widgets**
    + Amélioration des graphiques de la matrice de confusion pour l’apprentissage de la classification.

### <a name="azure-machine-learning-data-prep-sdk-v1112"></a>Kit de développement logiciel (SDK) de préparation de données Azure Machine Learning v1.1.12
+ **Nouvelles fonctionnalités**
  + Les listes de chaînes peuvent maintenant être passées comme entrée des méthodes `read_*`.

+ **Résolutions de bogue et améliorations**
  + Les performances de `read_parquet` ont été améliorées lors de l’exécution dans Spark.
  + Résolution d’un problème où `column_type_builder` échouait en cas de colonne unique avec des formats de date ambigus.

### <a name="azure-portal"></a>Portail Azure
+ **Fonctionnalité en préversion**
  + La diffusion en continu des fichiers journaux et des fichiers de sortie est désormais disponible pour les pages de détail de l’exécution. Ces fichiers diffusent des mises à jour en temps réel lorsque l’aperçu est activé.
  + La possibilité de définir un quota au niveau de l’espace de travail est publiée en version préliminaire. Les quotas AmlCompute sont alloués au niveau de l’abonnement, mais nous vous permettons désormais de distribuer ces quotas entre les espaces de travail et de les allouer pour un partage et une gouvernance équitables. Cliquez simplement sur le panneau **Utilisations+quotas** dans la barre de navigation de gauche de votre espace de travail, puis sélectionnez l’onglet **Configurer les quotas**. Vous devez être administrateur d’abonnement pour pouvoir définir des quotas au niveau de l’espace de travail, car il s’agit d’une opération qui a lieu sur les différents espaces de travail.

## <a name="2019-08-05"></a>05-08-2019

### <a name="azure-machine-learning-sdk-for-python-v1055"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.55

+ **Nouvelles fonctionnalités**
  + L’authentification basée sur des jetons est désormais prise en charge pour les appels établis vers le point de terminaison de scoring déployé sur AKS. Nous continuons de prendre en charge l’authentification par clé actuelle, et les utilisateurs peuvent utiliser un de ces mécanismes d’authentification à la fois.
  + Possibilité d’inscrire un stockage blob situé derrière le réseau virtuel en tant que banque de données.

+ **Résolutions de bogue et améliorations**
  + **azureml-automl-core**
    + Corrige un bogue où la taille de validation pour les fractionnements CV est petite et entraîne des prédictions erronées par rapport aux graphiques réels pour la régression et la prévision.
    + La journalisation des tâches de prévision sur les exécutions distantes a été améliorée. Désormais, l’utilisateur reçoit un message d’erreur complet en cas d’échec d’exécution.
    + Correction d’échecs de `Timeseries` si la valeur de l’indicateur de prétraitement est True.
    + Messages d’erreur de validation des données de prévision rendus plus actionnables.
    + Réduction de la consommation de mémoire des exécutions AutoML par la suppression ou le chargement différé de jeux de données, en particulier entre les déclenchements de processus
  + **azureml-contrib-explain-model**
    + Ajout de l’indicateur model_task aux explications afin de permettre à l’utilisateur de modifier la logique d’inférence automatique par défaut pour le type de modèle
    + Modifications du widget : Installation automatique avec `contrib`, sans plus aucune installation/activation de `nbextension` – Explication de prise en charge avec l’importance globale de la fonctionnalité (par exemple, Permutative)
    + Modifications du tableau de bord : Diagrammes en boîte et en violon, en plus du diagramme `beeswarm` sur la page de résumé – Réitération beaucoup plus rapide du diagramme `beeswarm` en cas de déplacement du curseur « top-k » – Message utile expliquant le mode de calcul de top-k – Messages personnalisables utiles au lieu de graphiques en l’absence de données
  + **azureml-core**
    + Ajout de la méthode Model.package() pour créer des images Docker et des Dockerfiles qui encapsulent des modèles et leurs dépendances.
    + Mise à jour des services web locaux pour accepter des InferenceConfigs contenant des objets Environnement.
    + Model.Register() fixe produisant des modèles non valides quand ’.’ (pour le répertoire actif) est transmis en tant que paramètre model_path.
    + Ajout de Run.submit_child. La fonctionnalité reflète Experiment.submit lors de la spécification de l’exécution en tant que parent de l’exécution enfant envoyée.
    + Prise en charge des options de configuration de Model.Register dans Run.register_model.
    + Possibilité d’exécuter des travaux JAR sur un cluster existant.
    + Prise en charge des paramètres instance_pool_id et cluster_log_dbfs_path.
    + Ajout de la prise en charge de l’utilisation d’un objet Environnement lors du déploiement d’un modèle sur un service web. L’objet Environnement peut désormais être fourni en tant que composant de l’objet InferenceConfig.
    + Ajout de mappage appinsifht pour les nouvelles régions centralus - westus - northcentralus
    + Ajout de documentation pour tous les attributs de toutes les classes de banque de données.
    + Ajout du paramètre blob_cache_timeout à `Datastore.register_azure_blob_container`.
    + Ajout des méthodes save_to_directory et load_from_directory à azureml.core.environment.Environment.
    + Ajout des commandes « az ml environment show » et « az ml environment list » à l’interface de ligne de commande.
    + Ajout de la méthode Environment.add_private_pip_wheel.
  + **azureml-explain-model**
    + Ajout du suivi des jeux de données aux explications à l’aide du service de jeu de données (préversion).
    + Réduction de la taille de lot par défaut lors de la diffusion en continu d’explications globales de 10 à 100 Ko.
    + Ajout de l’indicateur model_task aux explications pour permettre à l’utilisateur de modifier la logique d’inférence automatique par défaut pour le type de modèle.
  + **azureml-mlflow**
    + Correction du bogue dans mlflow.azureml.build_image qui avait pour effet que des répertoires imbriqués étaient ignorés.
  + **azureml-pipeline-steps**
    + Ajout de la possibilité d’exécuter des travaux JAR sur un cluster Azure Databricks existant.
    + Ajout de la prise en charge des paramètres instance_pool_id et cluster_log_dbfs_path pour l’étape DatabricksStep.
    + Ajout de la prise en charge des paramètres de pipeline dans l’étape DatabricksStep.
  + **azureml-train-automl**
    + Ajout de `docstrings` pour les fichiers associés à l’ensemble.
    + Mise à jour de la documentation dans un langage plus approprié pour `max_cores_per_iteration` et `max_concurrent_iterations`
    + La journalisation des tâches de prévision sur les exécutions distantes a été améliorée. Désormais, l’utilisateur reçoit un message d’erreur complet en cas d’échec d’exécution.
    + Suppression de get_data du bloc-notes `automlstep` du pipeline.
    + Début de la prise en charge `dataprep` dans `automlstep`.

### <a name="azure-machine-learning-data-prep-sdk-v1110"></a>Kit de développement logiciel (SDK) v1.1.10 pour la préparation de données Azure Machine Learning

+ **Nouvelles fonctionnalités**
  + Vous pouvez maintenant demander d’exécuter des inspecteurs spécifiques (par exemple, histogramme, nuage de points, etc.) sur des colonnes spécifiques.
  + Ajout d’un argument de parallélisation à `append_columns`. Si la valeur est True, les données sont chargées en mémoire, mais l’exécution se fait en parallèle. Si la valeur est False, l’exécution se fait en continu, mais dans un thread unique.

## <a name="2019-07-23"></a>2019-07-23

### <a name="azure-machine-learning-sdk-for-python-v1053"></a>SDK Azure Machine Learning pour Python v1.0.53

+ **Nouvelles fonctionnalités**
  + Le machine learning automatisé prend désormais en charge l’entraînement de modèles ONNX sur la cible de calcul distante.
  + Azure Machine Learning offre maintenant la possibilité de reprendre l’entraînement à partir d’une exécution précédente, d’un point de contrôle ou de fichiers de modèle.
    + Découvrez comment [utiliser des estimateurs pour reprendre l’entraînement à partir d’une exécution précédente](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/tensorflow/train-tensorflow-resume-training/train-tensorflow-resume-training.ipynb).

+ **Résolutions de bogue et améliorations**
  + **azure-cli-ml**
    + Les commandes CLI « model deploy » et « service update » acceptent désormais les paramètres, les fichiers de configuration ou une combinaison des deux. Les paramètres ont priorité sur les attributs des fichiers.
    + La description du modèle peut désormais être mise à jour après l’inscription.
  + **azureml-automl-core**
    + Mise à jour de la dépendance NimbusML vers la version 1.2.0 (actuellement la version la plus récente)
    + Ajout de la prise en charge des pipelines et des estimateurs ML Nimbus à utiliser dans les estimateurs AutoML.
    + Correction d’un bogue dans la procédure de sélection d’ensemble qui ne faisait pas nécessairement croître l’ensemble résultant même si les scores restaient constants
    + Possibilité de réutiliser certaines caractérisations dans les divisions de validation croisée pour les tâches de prévision. Ceci accélère l’exécution du programme d’installation avec un facteur approximatif de n_cross_validations pour les caractérisations coûteuses (décalages et fenêtres dynamiques, par exemple).
    + Résolution d’un problème se produisant quand l’heure est hors de la plage horaire prise en charge par Pandas. Nous levons maintenant une exception DataException si l’heure est antérieure à pd.Timestamp.min ou postérieure à pd.Timestamp.max.
    + La prévision autorise désormais des fréquences différentes dans les jeux d’entraînement et de test s’ils peuvent être alignés. Par exemple, « tous les trimestres à partir de janvier » et « tous les trimestres à partir d’octobre » peuvent être alignés.
    + La propriété « parameters » a été ajoutée à TimeSeriesTransformer.
    + Les anciennes classes d’exception ont été supprimées.
    + Dans les tâches de prévision, le paramètre `target_lags` accepte désormais une valeur entière unique ou une liste d’entiers. Si l’entier est fourni, un seul décalage est créé. Si une liste est fournie, les valeurs uniques des décalages sont utilisées. target_lags = [1, 2, 2, 4] va créer des retards d’une, deux et quatre périodes.
    + Correction du bogue concernant la perte des types de colonnes après la transformation (bogue lié)
    + Dans `model.forecast(X, y_query)`, possibilité pour y_query d’être un type d’objet débutant par « None(s) » (n° 459519).
    + Ajout des valeurs attendues à la sortie `automl`
  + **azureml-contrib-datadrift**
    +  Améliorations apportées à l’exemple de notebook, notamment le basculement vers azureml-opendatasets au lieu de azureml-contrib-opendatasets et amélioration des performances lors de l’enrichissement des données
  + **azureml-contrib-explain-model**
    + Correction de l’argument des transformations pour l’explicatif LIME pour l’importance des caractéristiques brutes dans le package azureml-contrib-explain-model
    + Ajout de segmentations aux explications d’image dans l’explicatif d’image pour le package AzureML-contrib-explain-model
    + Ajout d’une prise en charge partielle de SciPy pour LimeExplainer
    + Ajout de `batch_size` pour imiter Explainer quand `include_local=False`, pour diffuser en continu des explications globales par lots afin d’améliorer le temps d’exécution de DecisionTreeExplainableModel
  + **azureml-contrib-featureengineering**
    + Correction pour l’appel de set_featurizer_timeseries_params () : contrôle de valeur Null et modification du type de valeur dict - Ajout d’un notebook pour le caractériseur `timeseries`
    + Mise à jour de la dépendance NimbusML vers la version 1.2.0 (actuellement la version la plus récente)
  + **azureml-core**
    + Ajout de la possibilité de joindre des banques de données DBFS dans AzureML CLI
    + Correction du bogue concernant le chargement des banques de données quand un dossier vide est créé si `target_path` commence par `/`
    + Correction du problème avec `deepcopy` dans ServicePrincipalAuthentication.
    + Ajout des commandes « az ml environment show » et « az ml environment list » à l’interface CLI
    + Les environnements prennent maintenant en charge la spécification d’un base_dockerfile comme alternative à un base_image déjà créé.
    + Le paramètre RunConfiguration inutilisé dans auto_prepare_environment a été marqué comme déprécié.
    + La description du modèle peut désormais être mise à jour après l’inscription.
    + Résolution de bogue : Le modèle et la suppression d’image fournissent désormais plus d’informations sur la récupération des objets en amont qui en dépendent en cas d’échec de la suppression en raison d’une dépendance amont.
    + Correction du bogue entraînant l’impression d’une durée vide pour les déploiements lors de la création d’un espace de travail pour certains environnements
    + Amélioration des exceptions d’échec pour la création d’espace de travail. Les utilisateurs ne voient plus le message « Impossible de créer l’espace de travail. Impossible de trouver... ». Ils voient désormais l’échec de création réel.
    + Ajout de la prise en charge de l’authentification de jeton dans les services web AKS
    + Ajout de la méthode `get_token()` aux objets `Webservice`
    + Ajout de la prise en charge de l’interface CLI pour gérer les jeux de données de machine learning
    + Désormais, `Datastore.register_azure_blob_container` utilise une valeur `blob_cache_timeout` (en secondes) qui configure les paramètres de montage de Blobfuse pour activer l’expiration du cache pour cette banque de données. Par défaut, aucun délai d’expiration n’est appliqué : quand un blob est lu, il reste dans le cache local jusqu’à ce que le travail soit terminé. Ce paramètre est préférable pour la plupart des travaux, mais certains travaux doivent lire plus de données que ne peuvent l’accepter leurs nœuds à partir d’un grand jeu de données. La définition de ce paramètre favorise la réussite de ces travaux. Soyez vigilant lors de la définition de ce paramètre : une valeur trop faible peut altérer les performances, car les données utilisées à une époque peuvent expirer avant d’être réutilisées. Toutes les lectures sont effectuées à partir du Stockage Blob ou du réseau et non du cache local, entraînant un impact négatif sur les durées d’entraînement.
    + La description du modèle peut désormais être mise à jour correctement après l’inscription.
    + Lors de la suppression de modèle et d’image, de plus amples informations sont désormais fournies concernant les objets en amont qui en dépendent, entraînant l’échec de la suppression.
    + Utilisation améliorée des ressources des exécutions à distance qui utilisent azureml.mlflow
  + **azureml-explain-model**
    + Correction de l’argument des transformations pour l’explicatif LIME pour l’importance des caractéristiques brutes dans le package azureml-contrib-explain-model
    + Ajout d’une prise en charge partielle de SciPy pour LimeExplainer
    + Ajout d’un wrapper explicatif linéaire de forme et d’un autre niveau à l’explicatif tabulaire pour l’explication des modèles linéaires
    + Pour l’explicatif d’imitation dans la bibliothèque de modèles d’explication, correction de l’erreur avec include_local = false pour l’entrée de données éparses
    + Ajout des valeurs attendues à la sortie `automl`
    + Correction de l’importance des caractéristiques par permutation quand l’argument des transformations est fourni pour bénéficier de l’importance des caractéristiques brutes
    + Ajout de `batch_size` à reproduire quand `include_local=False` pour le streaming d’explications globales par lots afin d’améliorer le temps d’exécution de DecisionTreeExplainableModel
    + Pour la bibliothèque d’explicabilité de modèle, correction des explicatifs de boîte noire quand l’entrée de dataframes Pandas est nécessaire pour la prédiction
    + Correction du bogue suivant : `explanation.expected_values` retourne parfois une valeur flottante plutôt qu’une liste avec une valeur flottante.
  + **azureml-mlflow**
    + Amélioration du niveau de performance de mlflow.set_experiment(experiment_name)
    + Correction du bogue concernant l’utilisation d’InteractiveLoginAuthentication pour le tracking_uri mlflow
    + Utilisation améliorée des ressources des exécutions à distance qui utilisent azureml.mlflow
    + Amélioration de la documentation du package azureml-mlflow
    + Correction du bogue suivant : mlflow.log_artifacts (« my_dir ») enregistrait les artefacts sous « my_dir/<artifact-paths> » au lieu de « <artifact-paths> ».
  + **azureml-opendatasets**
    + Ajout de la possibilité d’épingler `pyarrow` de `opendatasets` aux anciennes versions (antérieures à la version 0.14.0) en raison d’un problème de mémoire.
    + Déplacement d’azureml-contrib-opendatasets vers azureml-opendatasets.
    + Les classes de jeu de données ouvertes peuvent être inscrites auprès de l’espace de travail Azure Machine Learning et tirer parti des fonctionnalités de jeu de données AML de manière intégrée.
    + L’amélioration de NoaaIsdWeather enrichie sensiblement les performances de la version non SPARK.
  + **azureml-pipeline-steps**
    + La banque de données DBFS est maintenant prise en charge pour les entrées et les sorties dans DatabricksStep.
    + Mise à jour de la documentation d’Azure Batch Step concernant les entrées/sorties
    + Dans AzureBatchStep, modification de la valeur par défaut *delete_batch_job_after_finish* sur *true*.
  + **azureml-telemetry**
    +  Déplacement d’azureml-contrib-opendatasets vers azureml-opendatasets.
    + Les classes de jeu de données ouvertes peuvent être inscrites auprès de l’espace de travail Azure Machine Learning et tirer parti des fonctionnalités de jeu de données AML de manière intégrée.
    + L’amélioration de NoaaIsdWeather enrichie sensiblement les performances de la version non SPARK.
  + **azureml-train-automl**
    + Mise à jour de la documentation concernant get_output pour refléter le type de retour réel et fournir des informations supplémentaires sur la récupération des propriétés de clé
    + Mise à jour de la dépendance NimbusML vers la version 1.2.0 (actuellement la version la plus récente)
    + Ajout des valeurs attendues à la sortie `automl`
  + **azureml-train-core**
    + Les chaînes sont maintenant acceptées en tant que cible de calcul pour le paramétrage automatisé des hyperparamètres.
    + Le paramètre RunConfiguration inutilisé dans auto_prepare_environment a été marqué comme déprécié.

### <a name="azure-machine-learning-data-prep-sdk-v119"></a>SDK de préparation de données Azure Machine Learning v1.1.9

+ **Nouvelles fonctionnalités**
  + Ajout de la prise en charge de la lecture de fichier directement à partir d’une URL http ou https

+ **Résolutions de bogue et améliorations**
  + Amélioration du message d’erreur affiché lors d’une tentative de lecture d’un jeu de données Parquet à partir d’une source distante (qui n’est pas encore prise en charge)
  + Correction d’un bogue se produisant lors de l’écriture vers le format de fichier Parquet dans ADLS Gen 2 et de la mise à jour du nom de conteneur ADLS Gen 2 dans le chemin.

## <a name="2019-07-09"></a>2019-07-09

### <a name="visual-interface"></a>Interface visuelle
+ **Fonctionnalités préliminaires**
  + Ajout du module « Exécuter le script R » dans l’interface visuelle.

### <a name="azure-machine-learning-sdk-for-python-v1048"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.48

+ **Nouvelles fonctionnalités**
  + **azureml-opendatasets**
    + **azureml-contrib-opendatasets** est maintenant disponible en tant que **azureml-opendatasets**. L’ancien package peut continuer à fonctionner, mais nous vous recommandons d’utiliser **azureml-opendatasets** pour des fonctionnalités et des améliorations plus riches.
    + Ce nouveau package vous permet d’inscrire des jeux de données ouverts en tant que Dataset dans un espace de travail Azure Machine Learning et de tirer parti des fonctionnalités offertes par le jeu de données.
    + Il comprend également des fonctionnalités existantes, telles que l’utilisation de jeux de données ouverts en tant que dataframes Pandas/SPARK et les jointures d’emplacement pour certains jeux de données tels que la météo.

+ **Fonctionnalités préliminaires**
    + HyperDriveConfig peut désormais accepter l’objet de pipeline en tant que paramètre pour prendre en charge l’hyperparamètres à l’aide d’un pipeline.

+ **Résolutions de bogue et améliorations**
  + **azureml-train-automl**
    + Correction du bogue concernant la perte des types de colonnes après la transformation.
    + Correction du bogue pour permettre à y_query d’être un type d’objet débutant par « Aucun(s) ».
    + Correction du problème dans la procédure de sélection d’ensemble qui ne faisait pas nécessairement croître l’ensemble résultant même si les scores sont restés constants.
    + Correction de problème avec les paramètres allow list_models ET block list_models dans AutoMLStep.
    + Correction du problème qui empêchait l’utilisation du prétraitement lorsqu’AutoML aurait été utilisé dans le contexte des pipelines Azure ML.
  + **azureml-opendatasets**
    + Déplacement d’azureml-contrib-opendatasets vers azureml-opendatasets.
    + Les classes de jeu de données ouvertes peuvent être inscrites auprès de l’espace de travail Azure Machine Learning et tirer parti des fonctionnalités de jeu de données AML de manière intégrée.
    + Les NoaaIsdWeather améliorées enrichissent sensiblement les performances de la version non SPARK.
  + **azureml-explain-model**
    + Mise à jour de la documentation en ligne des objets d’interprétation.
    + Ajout de `batch_size` à reproduire lorsque `include_local=False` pour le streaming d’explications globales par lots afin d’améliorer le temps d’exécution de DecisionTreeExplainableModel pour la bibliothèque d’optimisation des modèles.
    + Correction du problème où `explanation.expected_values` retourne parfois une valeur flottante plutôt qu’une liste avec une valeur flottante.
    + Ajout des valeurs attendues à la sortie `automl` pour reproduire l’imitateur dans l’explication du modèle de bibliothèque.
    + Correction de la fonctionnalité de permutation lorsque l’argument des transformations est fourni pour bénéficier de l’importance des fonctionnalités brutes.
  + **azureml-core**
    + Ajout de la possibilité d’attacher des banques de données DBFS dans le AzureML CLI.
    + Correction du problème de chargement des magasins de stockage lorsqu’un dossier vide a été créé si `target_path` commençait par `/`.
    + Activation de la comparaison des deux jeux de données.
    + Le modèle et la suppression d’image fournissent désormais plus d’informations sur la récupération des objets en amont qui en dépendent en cas d’échec de la suppression en raison d’une dépendance amont.
    + Le paramètre RunConfiguration inutilisé dans auto_prepare_environment a été déconseillé.
  + **azureml-mlflow**
    + Utilisation améliorée des ressources des exécutions à distance qui utilisent azureml.mlflow.
    + Amélioration de la documentation du package azureml-mlflow.
    + Correction du problème où mlflow.log_artifacts (« my_dir ») enregistrait les artefacts sous « my_dir/artefact-paths » au lieu de « artefact-paths ».
  + **azureml-pipeline-core**
    + Le paramètre hash_paths pour toutes les étapes de pipeline est déconseillé et sera supprimé à l’avenir. Par défaut, le contenu de source_directory est haché (à l’exception des fichiers listés dans `.amlignore` ou `.gitignore`)
    + L’amélioration continue de Module et de ModuleStep pour prendre en charge des modules de calcul spécifiques, en préparation de l’intégration de RunConfiguration et de modifications supplémentaires pour déverrouiller l’utilisation de modules de calcul spécifiques dans les pipelines.
  + **azureml-pipeline-steps**
    + AzureBatchStep : documentation améliorée en ce qui concerne les entrées/sorties.
    + AzureBatchStep : modification de la valeur par défaut delete_batch_job_after_finish sur true.
  + **azureml-train-core**
    + Les chaînes sont maintenant acceptées en tant que cible de calcul pour le paramétrage automatisé des hyperparamètres.
    + Le paramètre RunConfiguration inutilisé dans auto_prepare_environment a été déconseillé.
    + Paramètres déconseillés `conda_dependencies_file_path` et `pip_requirements_file_path` en faveur de `conda_dependencies_file` et `pip_requirements_file`, respectivement.
  + **azureml-opendatasets**
    + L’amélioration de NoaaIsdWeather enrichie sensiblement les performances de la version non SPARK.

## <a name="2019-04-26"></a>2019-04-26

### <a name="azure-machine-learning-sdk-for-python-v1033-released"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.33.

+ Les modèles accélérés matériellement d’Azure ML sur [FPGA](how-to-deploy-fpga-web-service.md) sont mis à la disposition générale.
  + Vous pouvez à présent [utiliser le package azureml-acceleration-models](how-to-deploy-fpga-web-service.md) à :
    + Former les poids d’un réseau neuronal profond pris en charge (ResNet 50, ResNet 152, DenseNet-121, VGG-16, et SSD-VGG)
    + Utiliser l’apprentissage par transfert avec le réseau neuronal profond pris en charge
    + Inscrire le modèle avec le Service Gestion des modèles et conteneuriser le modèle
    + Déployer le modèle sur une machine virtuelle Azure avec un FPGA dans un cluster Azure Kubernetes Service (AKS)
  + Déployer le conteneur dans un appareil de serveur [Azure Data Box Edge](../databox-online/azure-stack-edge-overview.md)
  + Noter vos données avec le point de terminaison gRPC avec cet [exemple](https://github.com/Azure-Samples/aml-hardware-accelerated-models)

### <a name="automated-machine-learning"></a>Machine Learning automatisé

+ Fonctionnalités de balayage pour activer l’ajout dynamique de :::no-loc text="featurizers"::: pour optimiser les performances. Nouveaux :::no-loc text="featurizers"::: : utiliser les incorporations, poids de preuve, les encodages de cible, le codage de texte cible, la distance de cluster
+ CV intelligent pour gérer des fractions train/valid à l’intérieur de ML automatisé
+ Quelques modifications de l’optimisation de mémoire et amélioration des performances de runtime
+ Amélioration des performances dans l’explication de modèle
+ Conversion de modèle ONNX pour l’exécution locale
+ Prise en charge Subsampling
+ Arrêt intelligent quand aucun critère de sortie n’est défini
+ Ensembles empilés

+ Prévision de séries chronologiques
  + Nouvelle fonction de prévision
  + Vous pouvez maintenant utiliser la validation croisée d’origine propagée sur les données de séries chronologiques
  + Nouvelles fonctionnalités ajoutées pour configurer les séries chronologiques en retard
  + Nouvelles fonctionnalités ajoutées pour prendre en charge des fonctionnalités de regroupement de la fenêtre propagée
  + Nouvelle détection de vacances et préapprentissage lorsque le code de pays est défini dans les paramètres d’expérience

+ Azure Databricks
  + Capacité de prévision des séries chronologiques et d’interprétabilité/explication de modèle activée
  + Vous pouvez désormais annuler et reprendre (continuer) des expériences de Machine Learning automatisé
  + Ajout de la prise en charge pour le traitement multicœur

### <a name="mlops"></a>MLOps
+ **Déploiement local et débogage des conteneurs de scoring**<br/> Vous pouvez désormais déployer un modèle ML localement et itérer rapidement sur votre fichier de scoring et vos dépendances pour vous assurer qu’ils se comportent comme prévu.

+ **Introduction de InferenceConfig et Model.deploy()**<br/> Le modèle de déploiement prend maintenant en charge la spécification d’un dossier source avec un script d’entrée, identique à un Runconfig.  En outre, le déploiement de modèle a été simplifié à une seule commande.

+ **Suivi de référence Git**<br/> Les clients demandent des fonctionnalités d’intégration Git de base depuis un moment, car elles contribuent à conserver une piste d’audit complète. Nous avons implémenté le suivi dans les entités principales dans Azure ML pour les métadonnées liées à Git (référentiel, commit, état propre). Ces informations seront collectées automatiquement par le Kit de développement logiciel et l’interface CLI.

+ **Service de profilage et de validation de modèle**<br/> Les clients se plaignent souvent de la difficulté à dimensionner correctement le calcul associé à leur service d’inférence. Avec notre service de profilage de modèle, le client peut donner des exemples d’entrées et nous établirons le profil entre 16 UC / configurations de mémoire différents afin de déterminer le dimensionnement optimal pour le déploiement.

+ **Apportez votre propre image de base pour l’inférence**<br/> Une autre critique courante a été la difficulté de déplacement de l’expérimentation aux dépendances de partage RE de l’inférence. Avec notre nouvelle capacité de partage d’image de base, vous pouvez maintenant réutiliser vos images de base d’expérimentation, les dépendances, etc, pour l’inférence. Cela devrait accélérer les déploiements et réduire l’écart de la boucle interne à la boucle externe.

+ **Expérience de génération de schéma Swagger améliorée**<br/> Notre méthode de génération Swagger était sujette à erreurs et était impossible à automatiser. Nous avons une nouvelle façon plus adaptée de générer des schémas swagger à partir de n’importe quelle fonction Python via des éléments décoratifs. Nous avons mis ce code en open source et notre protocole de génération de schéma n’est pas couplé à la plateforme Azure Machine Learning.

+ **L’interface de ligne de commande Azure ML CLI est mise à la disposition générale**<br/> Les modèles peuvent désormais être déployés avec une seule commande CLI. D’après des retours courants de nos clients, personne ne déploie de modèle ML depuis un notebook Jupyter. La [**documentation de référence CLI**](./reference-azure-machine-learning-cli.md) a été mise à jour.


## <a name="2019-04-22"></a>2019-04-22

Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.30.

Le [`PipelineEndpoint`](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline_endpoint.pipelineendpoint?preserve-view=true&view=azure-ml-py) a été introduit pour ajouter une nouvelle version d’un pipeline publié tout en conservant le même point de terminaison.

## <a name="2019-04-15"></a>2019-04-15

### <a name="azure-portal"></a>Portail Azure
  + Vous pouvez désormais renvoyer un script existant exécuté sur un cluster de calcul distant.
  + Vous pouvez maintenant exécuter un pipeline publié avec les nouveaux paramètres sous l’onglet Pipelines.
  + Les détails de l’exécution prennent désormais en charge une nouvelle visionneuse de fichier d’instantané. Vous pouvez afficher un instantané du répertoire lorsque vous avez envoyé une exécution spécifique. Vous pouvez également télécharger le notebook qui a été envoyé pour démarrer l’exécution.
  + Vous pouvez désormais annuler les exécutions parentes à partir du portail Azure.

## <a name="2019-04-08"></a>2019-04-08

### <a name="azure-machine-learning-sdk-for-python-v1023"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.23

+ **Nouvelles fonctionnalités**
  + Le kit de développement logiciel (SDK) Azure Machine Learning prend maintenant en charge Python 3.7.
  + Les estimateurs de réseau neuronal profond d’Azure Machine Learning offre désormais une prise en charge multiversion intégrée. Par exemple, l’estimateur `TensorFlow` accepte désormais un paramètre `framework_version` et les utilisateurs peuvent spécifier la version « 1.10 » ou « 1.12 ». Pour obtenir la liste des versions prises en charge par votre version du kit de développement logiciel actuelle, appelez `get_supported_versions()` sur la classe souhaitée de l’infrastructure .NET (par exemple, `TensorFlow.get_supported_versions()`).
  Pour obtenir la liste des versions prises en charge par la dernière version du Kit de développement logiciel, consultez la [documentation sur l’estimateur du réseau neuronal profond](/python/api/azureml-train-core/azureml.train.dnn?preserve-view=true&view=azure-ml-py).

## <a name="2019-03-25"></a>2019-03-25

### <a name="azure-machine-learning-sdk-for-python-v1021"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.21

+ **Nouvelles fonctionnalités**
  + La méthode *azureml.core.Run.create_children* permet de créer une faible latence de plusieurs exécutions enfants avec un seul appel.

## <a name="2019-03-11"></a>2019-03-11

### <a name="azure-machine-learning-sdk-for-python-v1018"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.18

 + **Modifications**
   + Le package azureml-tensorboard remplace azureml-contrib-tensorboard.
   + Avec cette version, vous pouvez configurer un compte d’utilisateur sur votre cluster de calcul géré (amlcompute), lors de sa création. Cela est possible en passant ces propriétés dans la configuration de l’approvisionnement. Vous pouvez trouver plus d’informations dans la [documentation sur la référence du kit de développement logiciel (SDK)](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute#provisioning-configuration-vm-size-----vm-priority--dedicated---min-nodes-0--max-nodes-none--idle-seconds-before-scaledown-none--admin-username-none--admin-user-password-none--admin-user-ssh-key-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--tags-none--description-none--remote-login-port-public-access--notspecified--).

### <a name="azure-machine-learning-data-prep-sdk-v1017"></a>Kit de développement logiciel (SDK) de préparation de données Azure Machine Learning v1.0.17

+ **Nouvelles fonctionnalités**
  + Prend désormais en charge l’ajout de deux colonnes numériques pour générer une colonne résultante en utilisant le langage d’expression.

+ **Résolutions de bogue et améliorations**
  + Amélioration de la documentation et de la vérification des paramètres de random_split.

## <a name="2019-02-27"></a>2019-02-27

### <a name="azure-machine-learning-data-prep-sdk-v1016"></a>Kit de développement logiciel (SDK) de préparation de données Azure Machine Learning v1.0.16

+ **Résolution de bogue**
  + Correction d’un problème d’authentification au principal de service causé par une modification de l’API.

## <a name="2019-02-25"></a>2019-02-25

### <a name="azure-machine-learning-sdk-for-python-v1017"></a>Kit de développement logiciel (SDK) Azure Machine Learning pour Python v1.0.17

+ **Nouvelles fonctionnalités**
  + Azure Machine Learning fournit désormais une prise en charge de première classe pour le Chainer de l’infrastructure de réseau neuronal profond populaire. À l’aide des classes [`Chainer`](/python/api/azureml-train-core/azureml.train.dnn.chainer?preserve-view=true&view=azure-ml-py), les utilisateurs peuvent facilement former et déployer des modèles Chainer.
    + Découvrez comment [exécuter l’entraînement distribué avec ChainerMN](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/chainer/distributed-chainer/distributed-chainer.ipynb)
    + Découvrez comment [exécuter l’optimisation des hyperparamètres avec Chainer à l’aide de HyperDrive](https://github.com/Azure/MachineLearningNotebooks/blob/b881f78e4658b4e102a72b78dbd2129c24506980/how-to-use-azureml/ml-frameworks/chainer/deployment/train-hyperparameter-tune-deploy-with-chainer/train-hyperparameter-tune-deploy-with-chainer.ipynb).
  + Ajout de la capacité des pipelines Azure Machine Learning à déclencher une exécution de pipeline en fonction des modifications apportées au magasin de données. Le pipeline [notebook de planification](https://aka.ms/pl-schedule) est mis à jour afin de tirer parti de cette fonctionnalité.

+ **Résolutions de bogue et améliorations**
  + Nous avons ajouté la prise en charge des pipelines Azure Machine Learning pour définir la propriété source_directory_data_store sur un magasin de données de votre choix (par exemple, un stockage d’objets blob) dans des [RunConfigurations](/python/api/azureml-core/azureml.core.runconfig.runconfiguration?preserve-view=true&view=azure-ml-py) qui sont fournies dans [PythonScriptStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep?preserve-view=true&view=azure-ml-py). Par défaut, les étapes utilisent le stockage Fichier Azure comme banque de données de secours, ce qui peut entraîner des problèmes de limitation de bande passante lorsqu’un grand nombre d’étapes sont exécutées en même temps.

### <a name="azure-portal"></a>Portail Azure

+ **Nouvelles fonctionnalités**
  + Nouvelle expérience glisser-déposer pour les rapports. Les utilisateurs peuvent faire glisser une colonne de la zone de configuration à la zone de la table où s’affiche un aperçu de la table. Les colonnes peuvent être déplacées.
  + Nouvelle visionneuse de fichiers journaux
  + Liens vers exécution d’expériences, calcul, modèles, images et déploiements à partir de l’onglet activités

## <a name="next-steps"></a>Étapes suivantes

Consultez la vue d’ensemble [d’Azure Machine Learning](overview-what-is-azure-ml.md).