---
title: Architecture et concepts clés
titleSuffix: Azure Machine Learning
description: Cet article vous offre une compréhension globale de l’architecture, des conditions et des concepts qui composent Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 08/20/2020
ms.custom: seoapril2019, seodec18
ms.openlocfilehash: a36481b2496060cb12bd755f56680915ec1074bb
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94540193"
---
# <a name="how-azure-machine-learning-works-architecture-and-concepts"></a>Fonctionnement d’Azure Machine Learning : Architecture et concepts

Apprenez-en davantage sur l’architecture et les concepts pour [Azure Machine Learning](overview-what-is-azure-ml.md).  Cet article fournit une description générale des composants et de la façon dont ils co-opèrent pour faciliter le processus de création, de déploiement et de maintenance des modèles Machine Learning.

## <a name="workspace"></a><a name="workspace"></a> Espace de travail

L’[espace de travail Azure Machine Learning](concept-workspace.md) est la ressource de niveau supérieur pour Azure Machine Learning.

:::image type="content" source="media/concept-azure-machine-learning-architecture/architecture.svg" alt-text="Schéma : Architecture Azure Machine Learning d’un espace de travail et de ses composants":::

L’espace de travail est l’emplacement centralisé pour :

* Gérer les ressources que vous utilisez pour la formation et le déploiement de modèles, tels que les [calculs](#compute-instance)
* Stocker les ressources que vous créez lorsque vous utilisez Azure Machine Learning, notamment :
  * [Environnements](#environments)
  * [Expériences](#experiments)
  * [Pipelines](#ml-pipelines)
  * [Groupes de données](#datasets-and-datastores)
  * [Modèles](#models)
  * [Points de terminaison](#endpoints)

Un espace de travail comprend d’autres ressources Azure utilisées par l’espace de travail :

+ [Azure Container Registry (ACR)](https://azure.microsoft.com/services/container-registry/) : Enregistre les conteneurs docker que vous utilisez pendant la formation et lorsque vous déployez un modèle. Pour réduire les coûts, ACR est créé uniquement lors de la création des images de déploiement.
+ [Compte Stockage Azure](https://azure.microsoft.com/services/storage/) : Utilisé comme magasin de données par défaut pour l’espace de travail.  Les notebooks Jupyter utilisés avec vos instances de calcul Azure Machine Learning sont également stockés ici.
+ [Azure Application Insights](https://azure.microsoft.com/services/application-insights/) : Stocke les informations de supervision concernant les modèles.
+ [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) : Stocke les secrets qui sont utilisés par les cibles de calcul, ainsi que d’autres informations sensibles dont a besoin l’espace de travail.

Vous pouvez partager un espace de travail avec d’autres utilisateurs.

### <a name="create-workspace"></a>Créer un espace de travail

Le diagramme suivant montre le workflow de création d’un espace de travail.

* Vous vous connectez à Azure AD à partir de l’un des clients Azure Machine Learning pris en charge (Azure CLI, SDK Python, portail Microsoft Azure) et demandez le jeton Azure Resource Manager approprié.
* Vous appelez Azure Resource Manager pour créer l’espace de travail. 
* Azure Resource Manager contacte le fournisseur de ressources Azure Machine Learning pour provisionner l’espace de travail.
* Si vous ne spécifiez pas de ressources existantes, des ressources obligatoires supplémentaires sont créées dans votre abonnement.

Vous pouvez également approvisionner d’autres cibles de calcul attachées à un espace de travail (comme Azure Kubernetes Service ou des machines virtuelles) le cas échéant.

[![Créer un workflow d’espace de travail](media/concept-azure-machine-learning-architecture/create-workspace.png)](media/concept-azure-machine-learning-architecture/create-workspace.png#lightbox)

## <a name="computes"></a>Calcul

<a name="compute-targets"></a> Une [cible de calcul](concept-compute-target.md) est une machine ou un ensemble de machines que vous utilisez pour exécuter votre script d’entraînement ou pour héberger votre déploiement de service. Vous pouvez utiliser votre machine locale ou une ressource de calcul distante comme cible de calcul.  Avec les cibles de calcul, vous pouvez commencer l’entraînement sur votre machine locale, puis effectuer un scale-out vers le cloud sans modifier votre script d’entraînement.

Azure Machine Learning introduit deux machines virtuelles basées sur le cloud et complètement managées qui sont configurées pour les tâches Machine Learning :

* <a name="compute-instance"></a> **Instance de calcul** : une instance de calcul est une machine virtuelle qui comprend plusieurs outils et environnements installés pour le Machine Learning. L’instance de calcul s’utilise principalement pour votre station de travail de développement.  Vous pouvez commencer à exécuter des exemples de notebooks sans qu’aucune configuration ne soit requise. Une instance de calcul peut également être utilisée comme cible de calcul pour des travaux d’entraînement et d’inférence.

* **Clusters de calcul** : les clusters de calcul sont un cluster de machines virtuelles avec des fonctionnalités de mise à l’échelle à plusieurs nœuds. Ils sont mieux adaptés aux cibles de calcul pour les travaux de grande taille et de production.  Le cluster subit automatiquement un scale-up lors de l’envoi d’un travail.  À utiliser comme cible de calcul de formation ou pour le déploiement de développement/test.

Pour plus d’informations sur les cibles de calcul de formation, consultez [Cibles de calcul de formation](concept-compute-target.md#train).  Pour plus d’informations sur les cibles de calcul de déploiement, consultez [Cibles de déploiement](concept-compute-target.md#deploy).

## <a name="datasets-and-datastores"></a>Jeux de données et magasins de données

Les [**jeux de données Azure Machine Learning**](concept-data.md#datasets) facilitent l’accès aux données et l’utilisation de ces dernières. En créant un jeu de données, vous créez une référence à l’emplacement de la source de données ainsi qu’une copie de ses métadonnées. Étant donné que les données restent à leur emplacement existant, vous n’exposez aucun coût de stockage supplémentaire et ne risquez pas l’intégrité de vos sources de données.

Pour plus d’informations, consultez [Créer et inscrire des jeux de données Azure Machine Learning](how-to-create-register-datasets.md).  Pour obtenir plus d’exemples d’utilisation de jeux de données, consultez les [exemples de notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/datasets-tutorial).

Les jeux de données utilisent des [magasins de données](concept-data.md#datastores) pour se connecter de manière sécurisée à vos services de stockage Azure. Les magasins de données stockent les informations de connexion sans avoir à compromettre vos informations d’authentification et l’intégrité de votre source de données d’origine. Ils stockent des informations de connexion, comme votre ID d’abonnement et votre autorisation de jeton, dans votre coffre de clés associé à l’espace de travail, de sorte que vous pouvez accéder à votre stockage de manière sécurisée sans avoir à coder en dur ces informations dans votre script.

## <a name="environments"></a>Environnements

[Espace de travail](#workspace) > **Environnements**

Un [environnement](concept-environments.md) est l’encapsulation de l’environnement dans lequel se produit l’apprentissage ou le score de votre modèle de Machine Learning. L’environnement spécifie les packages, variables d’environnement et paramètres logiciels Python autour de vos scripts d’apprentissage et de scoring.  

Pour des exemples de code, consultez la section « Gérer les environnements » de [Comment utiliser les environnements](how-to-use-environments.md#manage-environments).

## <a name="experiments"></a>Expériences

[Espace de travail](#workspace) > **Expériences**

Une expérience est un regroupement d’exécutions provenant d’un script spécifié. Elle appartient toujours à un espace de travail. Lorsque vous envoyez une exécution, vous fournissez un nom pour l’expérience. Les informations concernant l’exécution sont stockées sous cette expérience. Si le nom n’existe pas lorsque vous soumettez une expérience, une nouvelle expérience est automatiquement créée.
  
Pour obtenir un exemple de l’utilisation d’une expérience, consultez le [Tutoriel : Entraîner votre premier modèle](tutorial-1st-experiment-sdk-train.md).

### <a name="runs"></a>Exécutions

[Espace de travail](#workspace) > [Expériences](#experiments) > **Exécution**

Une exécution est une exécution unique d’un script de formation. Une expérience contient généralement plusieurs exécutions.

Azure Machine Learning enregistre toutes les exécutions et stocke les informations suivantes dans l’expérience :

* Les métadonnées relatives à l’exécution (horodatage, durée, et ainsi de suite)
* Les métriques qui sont journalisées par votre script
* Les fichiers de sortie qui sont collectés automatiquement par l’expérience ou chargés explicitement par l’utilisateur
* Un instantané du répertoire qui contient vos scripts, avant l’exécution

Vous déclenchez une exécution lorsque vous envoyez un script pour entraîner un modèle. Une exécution peut avoir zéro, une ou plusieurs exécutions enfants. Par exemple, l’exécution de niveau supérieur peut avoir deux exécutions enfants, et chacune d’elles peut avoir sa propre exécution enfant.

### <a name="run-configurations"></a>Configurations de séries de tests

[Espace de travai](#workspace)l  > [Expériences](#experiments) > [Exécution](#runs) > **Exécuter la configuration**

Une configuration de série de tests définit la façon dont un script doit être exécuté dans une cible de calcul spécifiée. Vous utilisez la configuration pour spécifier le script, la cible de calcul et l’environnement Azure ML d’exécution, toutes les configurations propres aux tâches distribuées et certaines propriétés supplémentaires. Pour plus d’informations sur l’ensemble complet d’options configurables pour les exécutions, consultez [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py).

Une configuration d’exécution peut être rendue persistante dans un fichier du répertoire qui contient votre script d’entraînement.   Elle peut aussi être construite comme un objet en mémoire et utilisée pour envoyer une exécution.

Pour obtenir des exemples de configurations de série de tests, consultez [Configurer une exécution d’entraînement](how-to-set-up-training-targets.md).

### <a name="snapshots"></a>Instantanés

[Espace de travail](#workspace) > [Expériences](#experiments) > [Exécution](#runs) > **Instantané**

Lorsque vous envoyez une exécution, Azure Machine Learning compresse le répertoire qui contient le script dans un fichier zip, puis l’envoie à la cible de calcul. Le fichier zip est ensuite décompressé, et le script y est exécuté. Azure Machine Learning stocke également le fichier zip en tant qu’instantané dans l’enregistrement d’exécution. Toute personne ayant accès à l’espace de travail peut parcourir un enregistrement d’exécution et télécharger l’instantané.

Le diagramme suivant montre le workflow de capture instantanée du code.

[![Workflow de la capture instantanée de code](media/concept-azure-machine-learning-architecture/code-snapshot.png)](media/concept-azure-machine-learning-architecture/code-snapshot.png#lightbox)

### <a name="logging"></a>Journalisation

Azure Machine Learning journalise automatiquement les métriques d’exécution standard. Toutefois, vous pouvez également [utiliser le kit de développement logiciel (SDK) Python pour journaliser des métriques arbitraires](how-to-track-experiments.md).

Il existe plusieurs façons de consulter vos journaux : surveiller l’état d’exécution en temps réel ou afficher les résultats après la fin. Pour plus d'informations, consultez [Surveiller et consulter les journaux d’exécution de ML](how-to-monitor-view-training-logs.md).


> [!NOTE]
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]

### <a name="git-tracking-and-integration"></a>Intégration et suivi Git

Lorsque vous lancez une exécution d’entraînement où le répertoire source est un répertoire Git local, les informations relatives au répertoire sont stockées dans l’historique des exécutions. Cela fonctionne avec les exécutions envoyées à l’aide d’une configuration de série de tests ou d’un pipeline ML. Cela fonctionne également pour les exécutions soumises à partir du SDK ou de l’interface CLI Machine Learning.

Pour plus d’informations, consultez [Obtenir une intégration pour Azure Machine Learning](concept-train-model-git-integration.md).

### <a name="training-workflow"></a>Workflow de l’entraînement

Lorsque vous exécutez une expérience pour effectuer l’apprentissage d’un modèle, les étapes suivantes se produisent. Celles-ci sont illustrées dans le diagramme du workflow de formation ci-dessous :

* Azure Machine Learning est appelé avec l’ID de capture instantanée pour la capture instantanée du code enregistrée dans la section précédente.
* Azure Machine Learning crée un ID d’exécution (facultatif) et un jeton de service Machine Learning, qui est utilisé par la suite par des cibles de calcul comme la Capacité de calcul Machine Learning/les machines virtuelles pour communiquer avec le service Machine Learning.
* Vous pouvez choisir une cible de calcul gérée (comme Capacité de calcul Machine Learning) ou une cible de calcul non gérée (comme des machines virtuelles) pour exécuter vos travaux d’apprentissage. Voici les flux de données des deux scénarios :
   * Machines virtuelles/HDInsight, accessibles par le biais d’informations d’identification SSH contenues dans un coffre de clés dans l’abonnement Microsoft. Azure Machine Learning exécute le code de gestion sur la cible de calcul qui :

   1. Prépare l’environnement (Docker est une option pour les machines virtuelles et les ordinateurs locaux. Consultez les étapes suivantes pour la Capacité de calcul Machine Learning afin de comprendre le fonctionnement de l’exécution des expériences sur des conteneurs Docker.)
   1. Télécharge le code.
   1. Définit des variables d’environnement et des configurations.
   1. Exécute des scripts utilisateur (capture instantanée de code mentionnée dans la section précédente).

   * Capacité de calcul Machine Learning, accessible par le biais d’une identité managée par l’espace de travail.
Étant donné que la Capacité de calcul Machine Learning est une cible de calcul managée (c’est-à-dire qu’elle est gérée par Microsoft), elle s’exécute sous votre abonnement Microsoft.

   1. Une construction du Docker distant est démarrée si nécessaire.
   1. Le code de gestion est écrit dans le partage Azure Files de l’utilisateur.
   1. Le conteneur est démarré avec une commande initiale ; autrement dit, avec le code de gestion décrit à l’étape précédente.

* Une fois l’exécution terminée, vous pouvez interroger les exécutions et les métriques. Dans le diagramme de flux ci-dessous, cette étape se produit quand la cible de calcul d’entraînement réécrit les métriques d’exécution dans Azure Machine Learning à partir du stockage de la base de données Cosmos DB. Les clients peuvent appeler Azure Machine Learning. Machine Learning tire (pull) ensuite les métriques de la base de données Cosmos DB pour les renvoyer au client.

[![Workflow de l’entraînement](media/concept-azure-machine-learning-architecture/training-and-metrics.png)](media/concept-azure-machine-learning-architecture/training-and-metrics.png#lightbox)

## <a name="models"></a>Modèles

Le modèle le plus simple est un morceau de code qui accepte une entrée et produit une sortie. La création d’un modèle Machine Learning implique la sélection d’un algorithme auquel vous devez fournir des données, ainsi que l’[optimisation des hyperparamètres](how-to-tune-hyperparameters.md). L’entraînement est un processus itératif qui génère un modèle entraîné, lequel encapsule ce qu’il a appris au cours du processus d’entraînement.

Vous pouvez utiliser un modèle qui a été entraîné en dehors d’Azure Machine Learning, ou vous pouvez entraîner un modèle en soumettant une [exécution](#runs) d’une [expérience](#experiments) sur une [cible de calcul](#compute-targets) dans Azure Machine Learning. Une fois que vous avez un modèle, vous [inscrivez le modèle](#register-model) dans l’espace de travail.

Azure Machine Learning est indépendant de tout framework. Lorsque vous créez un modèle, vous pouvez utiliser un des frameworks Machine Learning populaires, tels que Scikit-learn, XGBoost, PyTorch, TensorFlow et Chainer.

Pour obtenir un exemple de formation d’un modèle à l’aide de Scikit-learn, consultez [Didacticiel : Entraîner un modèle de classification d’images avec Azure Machine Learning](tutorial-train-models-with-aml.md)


### <a name="model-registry"></a><a name="register-model"></a> Registre de modèles

[Espace de travail](#workspace) > **Modèles**

Le **registre de modèles** effectue le suivi de tous les modèles de votre espace de travail Azure Machine Learning.

Les modèles sont identifiés par leur nom et par leur version. Chaque fois que vous inscrivez un modèle portant le même nom qu’un modèle existant, le registre suppose qu’il s’agit d’une nouvelle version. La version est incrémentée et le nouveau modèle est inscrit sous le même nom.

Lorsque vous inscrivez le modèle, vous pouvez fournir des étiquettes de métadonnées supplémentaires, puis utiliser les étiquettes lorsque vous recherchez des modèles.

> [!TIP]
> Un modèle inscrit est un conteneur logique pour un ou plusieurs fichiers qui composent votre modèle. Par exemple, si vous avez un modèle qui est stocké dans plusieurs fichiers, vous pouvez inscrire ces derniers en tant que modèle unique dans votre espace de travail Azure Machine Learning. Après l’inscription, vous pouvez ensuite télécharger ou déployer le modèle inscrit et recevoir tous les fichiers qui ont été inscrits.

Vous ne pouvez pas supprimer un modèle inscrit qui est utilisé par un déploiement actif.

Pour obtenir un exemple d’inscription de modèle, consultez [Entraîner un modèle de classification d’images avec Azure Machine Learning](tutorial-train-models-with-aml.md).

## <a name="deployment"></a>Déploiement

Vous déployez un [modèle inscrit](#register-model) en tant que point de terminaison de service. Vous avez besoin des composants suivants :

* **Environnement**. Cet environnement encapsule les dépendances requises pour exécuter votre modèle pour l’inférence.
* **Code de scoring**. Ce script accepte les requêtes, calcule le score de la requête à l’aide du modèle et retourne les résultats.
* **Configuration de l’inférence**. La configuration de l’inférence spécifie l’environnement, le script d’entrée et d’autres composants nécessaires pour exécuter le modèle en tant que service.

Pour plus d’informations sur ces composants, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

### <a name="endpoints"></a>Points de terminaison

[Espace de travail](#workspace) > **Points de terminaison**

Un point de terminaison est une instanciation de votre modèle soit dans un service web pouvant être hébergé dans le cloud, soit dans un module IoT pour les déploiements d’appareils intégrés.

#### <a name="web-service-endpoint"></a>Point de terminaison de service Web

Lors du déploiement d’un modèle en tant que service web, le point de terminaison peut être déployé sur Azure Container Instances, Azure Kubernetes Service ou des FPGA. Vous créez le service à partir d’une image, de votre script et des fichiers associés. Ceux-ci sont placés dans une image de conteneur de base qui contient l’environnement d’exécution du modèle. L’image a un point de terminaison HTTP à charge équilibrée qui reçoit les requêtes de scoring qui sont envoyées au service web.

Vous pouvez activer la télémétrie Application Insights ou la télémétrie de modèle pour surveiller votre service web. Les données de télémétrie ne sont accessibles qu’à vous.  Elles sont stockées dans votre Application Insights et les instances de votre compte de stockage. Si vous avez activé la mise à l’échelle automatique, Azure met automatiquement à l’échelle votre déploiement.

Le diagramme suivant montre le workflow d’inférence pour un modèle déployé en tant que point de terminaison de service web :

Voici les détails :

* L’utilisateur inscrit un modèle à l’aide d’un client comme le SDK Azure Machine Learning.
* L’utilisateur crée une image à l’aide d’un modèle, d’un fichier de scores et d’autres dépendances de modèle.
* L’image Docker est créée et stockée dans Azure Container Registry.
* Le service web est déployé sur la cible de calcul (Container Instances/AKS) à l’aide de l’image créée à l’étape précédente.
* Les détails des requêtes de scoring sont stockés dans la fonctionnalité Application Insights qui se trouve dans l’abonnement de l’utilisateur.
* Des données de télémétrie sont également envoyées (push) à l’abonnement Microsoft/Azure.

[![Workflow de l’inférence](media/concept-azure-machine-learning-architecture/inferencing.png)](media/concept-azure-machine-learning-architecture/inferencing.png#lightbox)


Pour obtenir un exemple de déploiement de modèle en tant que service, consultez [Déployer un modèle de classification d’images dans Azure Container Instances](tutorial-deploy-models-with-aml.md).

#### <a name="real-time-endpoints"></a>Points de terminaison en temps réel

Lorsque vous déployez un modèle entraîné dans le concepteur, vous pouvez [déployer le modèle en tant que point de terminaison en temps réel](tutorial-designer-automobile-price-deploy.md). Un point de terminaison en temps réel reçoit généralement une requête unique via le point de terminaison REST et retourne une prédiction en temps réel. Cela diffère du traitement par lots, qui traite plusieurs valeurs à la fois et enregistre les résultats après la fin dans un magasin de données.

#### <a name="pipeline-endpoints"></a>Points de terminaison de pipeline

Les points de terminaison de pipeline vous permettent d’appeler vos [pipelines ML](#ml-pipelines) par programme via un point de terminaison REST. Les points de terminaison de pipeline vous permettent d’automatiser vos workflows de pipeline.

Un point de terminaison de pipeline est une collection de pipelines publiés. Cette organisation logique vous permet de gérer et d’appeler plusieurs pipelines à l’aide du même point de terminaison. Chaque pipeline publié dans un point de terminaison de pipeline est associé à une version. Vous pouvez sélectionner un pipeline par défaut pour le point de terminaison ou spécifier une version dans l’appel REST.
 

#### <a name="iot-module-endpoints"></a>Points de terminaison de module IoT

Un point de terminaison de module IoT déployé est un conteneur Docker qui inclut votre modèle, ainsi que le script ou l’application associés et toutes les dépendances supplémentaires. Vous déployez ces modules à l’aide d’Azure IoT Edge sur les périphériques en mode Edge.

Si vous avez activé la supervision, Azure collecte les données de télémétrie à partir du modèle qui se trouve dans le module Azure IoT Edge. Vous seul pouvez accéder aux données de télémétrie qui sont stockées dans votre instance de compte de stockage.

Azure IoT Edge garantit l’exécution de votre module et supervise l’appareil qui l’héberge. 
## <a name="automation"></a>Automatisation

### <a name="azure-machine-learning-cli"></a>Interface CLI Azure Machine Learning 

L'interface CLI Azure Machine Learning est une extension pour l'[interface Azure](reference-azure-machine-learning-cli.md), une interface de ligne de commande multiplateforme pour la plateforme Azure. Cette extension fournit des commandes pour automatiser vos activités de Machine Learning.

### <a name="ml-pipelines"></a>Pipelines ML

Vous utilisez des [pipelines de machine learning](concept-ml-pipelines.md) pour créer et gérer des flux de travail qui combinent les phases de machine learning. Par exemple, un pipeline peut inclure les phases de préparation des données, d’entraînement du modèle, de déploiement du modèle et d’inférence/scoring. Chaque phase peut englober plusieurs étapes, chacune d’elles pouvant s’exécuter sans assistance dans différentes cibles de calcul. 

Les étapes de pipeline sont réutilisables et peuvent être exécutées sans avoir à réexécuter les étapes précédentes si la sortie de ces étapes n’a pas changé. Par exemple, vous pouvez réentraîner un modèle sans réexécuter les étapes coûteuses de préparation des données si celles-ci n’ont pas été modifiées. Les pipelines permettent également aux scientifiques des données de collaborer tout en travaillant chacun sur une partie différente du workflow de Machine Learning.

## <a name="monitoring-and-logging"></a>Surveillance et journalisation

Azure Machine Learning propose les fonctionnalités de supervision et de journalisation suivantes :

* En tant que __scientifique de données__, vous pouvez surveiller vos expériences et consigner des informations à partir de vos exécutions d’entraînement. Pour plus d’informations, consultez les articles suivants :
   * [Démarrer, superviser et annuler des exécutions d’entraînement](how-to-manage-runs.md)
   * [Journaliser des métriques pour les exécutions d’entraînement](how-to-track-experiments.md)
   * [Suivre des expériences avec MLflow](how-to-use-mlflow.md)
   * [Visualiser les exécutions avec TensorBoard](how-to-monitor-tensorboard.md)
* En tant qu’__Administrateur__, vous pouvez surveiller les informations sur l’espace de travail, les ressources Azure associées et les événements tels que la création et la suppression de ressources à l’aide d’Azure Monitor. Pour plus d’informations, consultez [Supervision d’Azure Machine Learning](monitor-azure-machine-learning.md).
* En tant que __DevOps__ ou __MLOps__, vous pouvez surveiller les informations générées par les modèles déployés en tant que services web ou modules IoT Edge pour identifier les problèmes liés aux déploiements et collecter les données envoyées au service. Pour plus d’informations, consultez [Collecter des données de modèle](how-to-enable-data-collection.md) et [Analyser avec Application Insights](how-to-enable-app-insights.md).

## <a name="interacting-with-your-workspace"></a>Interaction avec votre espace de travail

### <a name="studio"></a>Studio

[Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md) fournit une vue Web de tous les artefacts de votre espace de travail.  Vous pouvez afficher les résultats et les détails de vos jeux de données, expériences, pipelines, modèles et points de terminaison.  Vous pouvez également gérer les ressources de calcul et les magasins de données dans Studio.

Le studio est également l’emplacement où vous accédez aux outils interactifs qui font partie d’Azure Machine Learning :

+ [Concepteur Azure Machine Learning](concept-designer.md) pour effectuer les étapes de workflow sans écrire de code
+ Expérience Web pour le [Machine Learning automatisé](concept-automated-ml.md)
+ [Notebooks Azure Machine Learning](how-to-run-jupyter-notebooks.md) pour écrire et exécuter votre propre code sur des serveurs notebook Jupyter intégrés.
+ [Projets d’étiquetage des données](how-to-create-labeling-projects.md) pour créer, gérer et superviser des projets afin d’étiqueter vos données

### <a name="programming-tools"></a>Outils de programmation

> [!IMPORTANT]
> Les outils marqués (préversion) ci-dessous sont actuellement en préversion publique.
> La préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail en production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

+  Interagissez avec le service dans un environnement Python avec le [SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py).
+ Interagissez avec le service dans un environnement R avec le [Kit de développement logiciel (SDK) Azure Machine Learning pour R](https://azure.github.io/azureml-sdk-for-r/reference/index.html) (préversion).
+ Utilisez le [Concepteur Azure Machine Learning](concept-designer.md) pour effectuer les étapes de workflow sans écrire de code. 
+ Utiliser l’[interface de ligne de commande d’Azure Machine Learning](./reference-azure-machine-learning-cli.md) pour l’automatisation.
+ L’[accélérateur de solution de nombreux modèles](https://aka.ms/many-models) (préversion) s’appuie sur Azure Machine Learning et vous permet d’effectuer l’apprentissage, l’utilisation et la gestion de centaines, voire de milliers de modèles Machine Learning.

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec Azure Machine Learning, voir :

* [Qu'est-ce que Microsoft Azure Machine Learning ?](overview-what-is-azure-ml.md)
* [Création d’un espace de travail Microsoft Azure Machine Learning](how-to-manage-workspace.md)
* [Tutoriel (partie 1) : Effectuer l'apprentissage d’un modèle](tutorial-train-models-with-aml.md)