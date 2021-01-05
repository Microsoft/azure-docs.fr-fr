---
title: Se connecter à des données dans les services de stockage sur Azure
titleSuffix: Azure Machine Learning
description: Créez des magasins de données et des jeux de données pour vous connecter en toute sécurité aux données des services de stockage dans Azure avec Azure Machine Learning Studio.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.reviewer: nibaccam
ms.date: 09/22/2020
ms.custom: how-to, data4ml
ms.openlocfilehash: 554c815e6384115d56611e497f49a2c97ed15e38
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461433"
---
# <a name="connect-to-data-with-the-azure-machine-learning-studio"></a>Se connecter aux données avec Azure Machine Learning Studio

Dans cet article, découvrez comment accéder à vos données avec [Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md). Connectez-vous à vos données dans les services de stockage sur Azure avec les [magasins de données Azure Machine Learning](how-to-access-data.md), puis empaquetez ces données de tâches dans vos workflows ML avec les [jeux de données Azure Machine Learning](how-to-create-register-datasets.md).

Le tableau suivant définit et récapitule les avantages liés aux magasins de données et jeux de données. 

|Object|Description| Avantages|   
|---|---|---|
|Magasins de données| Connectez-vous en toute sécurité à votre service de stockage sur Azure, en stockant vos informations de connexion, comme votre ID d’abonnement et votre autorisation de jeton, dans votre [coffre de clés](https://azure.microsoft.com/services/key-vault/) associé à l’espace de travail. | Vos données étant stockées de manière sécurisée, vous : <br><br> <li> Ne&nbsp;mettez&nbsp;pas les &nbsp;informations d’authentification&nbsp;ou&nbsp;sources de données d’origine&nbsp;en danger. <li> N’avez plus besoin de les coder en dur dans vos scripts.
|Groupes de données| En créant un jeu de données, vous créez une référence à l’emplacement de la source de données, ainsi qu’une copie de ses métadonnées. Avec les jeux de données, vous pouvez : <br><br><li> Accéder aux données pendant la formation de modèle.<li> Partager des données et collaborer avec d’autres utilisateurs.<li> Tirer parti des bibliothèques open source, telles que Pandas, pour l’exploration des données. | Étant donné que les jeux de données sont évalués tardivement et que les données restent à leur emplacement existant : <br><br><li>Vous conservez une seule copie des données dans votre stockage.<li> Vous n’engagez aucun coût de stockage supplémentaire. <li> Vous ne risquez pas de modifier involontairement vos sources de données d’origine.<li>Vous améliorez les performances des workflows de ML. 

Pour comprendre où figurent les magasins de données et les jeux de données dans le flux de travail global d’accès aux données d’Azure Machine Learning, consultez l’article [Sécuriser l’accès aux données](concept-data.md#data-workflow).

Pour une première expérience de code, consultez les articles suivants pour utiliser le kit de développement logiciel (SDK) [Azure Machine Learning Python](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py) pour :
* [Vous connecter aux services de stockage Azure avec les magasins de données.](how-to-access-data.md) 
* [Créer des jeux de données Azure Machine Learning](how-to-create-register-datasets.md). 

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).

- Accédez au [studio Azure Machine Learning](https://ml.azure.com/).

- Un espace de travail Azure Machine Learning. [Créez un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

    -  Quand vous créez un espace de travail, un conteneur d’objets blob Azure et un partage de fichiers Azure sont inscrits automatiquement comme magasins de données dans l’espace de travail. sous les noms `workspaceblobstore` et `workspacefilestore` respectivement. Si le stockage blob répond à vos besoins, `workspaceblobstore` est défini en tant que magasin de stockage par défaut et déjà configuré pour être utilisé. Dans le cas contraire, vous avez besoin d’un compte de stockage sur Azure avec un [type de stockage pris en charge](how-to-access-data.md#matrix).
    

## <a name="create-datastores"></a>Créer des magasins de données

Vous pouvez créer des magasins de données à partir de ces [solutions de stockage Azure](how-to-access-data.md#matrix). **Pour les solutions de stockage non prises en charge**, et pour réduire le coût de sortie des données pendant les expériences de Machine Learning, vous devez [déplacer vos données](how-to-access-data.md#move) vers une solution de stockage Azure prise en charge. [Apprenez-en davantage sur les magasins de données](how-to-access-data.md). 

Créez un magasin de données en quelques étapes avec Azure Machine Learning Studio.

> [!IMPORTANT]
> Si votre compte de stockage de données se trouve sur un réseau virtuel, des étapes de configuration supplémentaires sont nécessaires pour s’assurer que Studio a accès à vos données. Pour vous assurer que les étapes de configuration appropriées sont appliquées, consultez [Isolement réseau et confidentialité](how-to-enable-studio-virtual-network.md).

1. Connectez-vous à [Azure Machine Learning Studio](https://ml.azure.com/).
1. Sélectionnez **Magasins de données** dans le volet gauche sous **Gérer**.
1. Sélectionnez **+ Nouveau magasin de données**.
1. Complétez le formulaire pour créer et inscrire un nouveau magasin de données. Le formulaire est mis à jour intelligemment en fonction du type de stockage Azure et du type d’authentification que vous sélectionnez. Pour savoir où trouver les informations d’authentification requises pour remplir ce formulaire, consultez la section [accès au stockage et autorisations](#access-validation).

L’exemple suivant montre à quoi ressemble le formulaire quand vous créez un **magasin de données d’objets blob Azure** :

![Formulaire de création d’un magasin de données](media/how-to-connect-data-ui/new-datastore-form.png)

## <a name="create-datasets"></a>Créez les jeux de données

Après avoir créé un magasin de données, créez un jeu de données pour interagir avec vos données. Les jeux de données intègrent vos données dans un objet consommable évalué tardivement pour les tâches de Machine Learning, comme la formation. [En savoir plus sur les jeux de données](how-to-create-register-datasets.md).

Il existe deux types de jeux de données, FileDataset et TabularDataset. 
[FileDataset](how-to-create-register-datasets.md#filedataset) crée des références à des fichiers uniques ou multiples, ou URL publiques. [TabularDataset](how-to-create-register-datasets.md#tabulardataset) représente vos données dans un format tabulaire. 

L’animation et les étapes suivantes montrent comment créer un jeu de données dans [Azure Machine Learning Studio](https://ml.azure.com).

> [!Note]
> Les jeux de données créés via Azure Machine Learning Studio sont automatiquement inscrits auprès de l’espace de travail.

![Créer un jeu de données avec l’interface utilisateur](./media/how-to-connect-data-ui/create-dataset-ui.gif)

Pour créer un jeu de données dans le studio :
1. Connectez-vous à [Azure Machine Learning Studio](https://ml.azure.com/).
1. Sélectionnez **Jeux de données** dans la section **Ressources** du volet gauche.
1. Sélectionnez **Créer un jeu de données** pour choisir la source de votre jeu de données. Cette source peut correspondre à des fichiers locaux, un magasin de fichiers, des URL publiques ou [Azure Open Datasets](../open-datasets/how-to-create-azure-machine-learning-dataset-from-open-dataset.md).
1. Sélectionnez le type de jeu de données **Tabulaire** ou **Fichier**.
1. Sélectionnez **Suivant** pour ouvrir le formulaire **Sélection d’un magasin de données et de fichiers**. Sur ce formulaire, vous sélectionnez l’emplacement où conserver votre jeu de données après sa création, ainsi que les fichiers de données à utiliser pour votre jeu de données.
    1. Vous pouvez choisir d’ignorer la validation si vos données se trouvent dans un réseau virtuel. En savoir plus sur l’[isolement et la confidentialité des réseaux virtuels](how-to-enable-studio-virtual-network.md).
    1. Pour les jeux de données de type Tabulaire, vous pouvez spécifier une caractéristique « timeseries » pour activer les opérations relatives au temps sur votre jeu de données. Découvrez comment [ajouter la caractéristique « timeseries » à votre jeu de données](how-to-monitor-datasets.md#studio-dataset).
1. Sélectionnez **Suivant** pour renseigner les formulaires **Paramètres et aperçu** et **Schéma** ; ils sont renseignés intelligemment en fonction du type de fichier et vous pouvez configurer davantage votre jeu de données avant de le créer sur ces formulaires. 
1. Sélectionnez **Suivant** pour passer en revue le formulaire **Confirmer les détails**. Vérifiez vos sélections et créez un profil de données facultatif pour votre jeu de données. En savoir plus sur le [profilage des données](#profile).
1. Sélectionnez **Créer** pour terminer la création de votre jeu de données.

<a name="profile"></a>

### <a name="data-profile-and-preview"></a>Profil et aperçu des données

Après avoir créé votre jeu de données, vérifiez que vous pouvez afficher le profil et l’aperçu dans le studio en procédant comme suit. 

1. Connectez-vous à [Azure Machine Learning Studio](https://ml.azure.com/).
1. Sélectionnez **Jeux de données** dans la section **Ressources** du volet gauche.
1. Sélectionnez le nom du jeu de données que vous souhaitez afficher. 
1. Sélectionnez l’onglet **Explorer** . 
1. Sélectionnez l’onglet **Aperçu** ou **Profil**. 

![Afficher le profil et l’aperçu du jeu de données](./media/how-to-connect-data-ui/dataset-preview-profile.gif)

Vous pouvez obtenir un vaste éventail de statistiques de synthèse dans votre jeu de données afin de vérifier si ce dernier est prêt pour le Machine Learning. Les colonnes non numériques incluent uniquement des statistiques de base telles que min, max et nombre d’erreurs. Les colonnes numériques vous permettent également de consulter les statistiques et quantiles estimés. 

Plus précisément, le profil des données du jeu de données Azure Machine Learning comprend :

>[!NOTE]
> Les entrées vides apparaissent pour les fonctionnalités avec types non pertinents.

|Statistique|Description
|------|------
|Fonctionnalité| Nom de la colonne en cours de synthèse.
|Profil| Visualisation en ligne en fonction du type déduit. Par exemple, les chaînes, valeurs booléennes et dates incluront des nombres de valeurs, et les décimales (valeurs numériques) des histogrammes approximatifs. Cela vous permet de vous faire une idée rapide de la distribution des données.
|Distribution des types| Nombre de valeurs en ligne de types au sein d’une colonne. Les valeurs Null ont un type propre et dès lors, cette visualisation est utile pour détecter les valeurs impaires ou manquantes.
|Type|Type déduit de la colonne. Les valeurs possibles incluent : chaînes, valeurs booléennes, dates et décimales.
|Min| Valeur minimale de la colonne. Les entrées vides apparaissent pour les fonctionnalités dont le type n’a pas d'ordre inhérent (valeurs booléennes, par exemple).
|Max| Valeur maximale de la colonne. 
|Count| Nombre total d’entrées manquantes et non manquantes de la colonne.
|Non manquant| Nombre d’entrées dans la colonne qui ne sont pas manquantes. Les chaînes vides et les erreurs sont traitées en tant que valeurs et donc n’entrent pas dans « Non manquant ».
|Quantiles| Valeurs approximatives à chaque quantile pour donner une idée de la distribution des données.
|Moyenne| Moyenne arithmétique ou moyenne de la colonne.
|Écart standard| Mesure de la dispersion ou de la variation des données de cette colonne.
|Variance| Mesure jusqu’où les données de cette colonne sont déployées par rapport à leur valeur moyenne. 
|Asymétrie| Mesure de la différence entre les données de cette colonne et une distribution normale.
|Kurtosis| Mesure de la latéralité des données de cette colonne par rapport à une distribution normale.

## <a name="storage-access-and-permissions"></a>Accès et autorisations pour le stockage

Pour s’assurer que vous vous connectez en toute sécurité à votre service de stockage Azure, Azure Machine Learning exige que vous ayez l’autorisation d’accéder au stockage de données correspondant. Cet accès dépend des informations d’authentification utilisées pour inscrire le magasin de données.

### <a name="virtual-network"></a>Réseau virtuel

Si votre compte de stockage de données se trouve sur un **réseau virtuel**, des étapes de configuration supplémentaires sont nécessaires pour s’assurer qu’Azure Machine Learning a accès à vos données. Pour vous assurer que les étapes de configuration appropriées sont appliquées lors de la création et de l’enregistrement de votre magasin de données, consultez [Isolement réseau et confidentialité](how-to-enable-studio-virtual-network.md).  

### <a name="access-validation"></a>Validation de l’accès

**Dans le cadre du processus de création et d’inscription du magasin de données initial**, Azure Machine Learning vérifie automatiquement que le service de stockage sous-jacent existe et que le principal fourni par l’utilisateur (nom d’utilisateur, principal de service ou jeton SAS) a accès au stockage spécifié.

**Après la création** du magasin de données, cette validation est effectuée uniquement pour les méthodes qui requièrent l’accès au conteneur de stockage sous-jacent, et **non** chaque fois que des objets du magasin de données sont récupérés. Par exemple, la validation se produit si vous souhaitez télécharger des fichiers à partir de votre magasin de données ; mais si vous souhaitez simplement modifier votre magasin de données par défaut, la validation ne se produit pas.

Pour authentifier votre accès au service de stockage sous-jacent, vous pouvez fournir votre clé de compte, des jetons de signature d’accès partagé (SAS) ou le principal de service selon le type de magasin de données que vous souhaitez créer. La [matrice de types de stockage](how-to-access-data.md#matrix) répertorie les types d’authentification pris en charge qui correspondent à chaque type de magasin de données.

Vous trouverez des informations sur la clé de compte, le jeton SAS et le principal de service sur votre [portail Azure](https://portal.azure.com).

* Si vous envisagez d’utiliser une clé de compte ou un jeton SAP pour l’authentification, sélectionnez **Comptes de stockage** dans le volet gauche, puis choisissez le compte de stockage que vous souhaitez inscrire.
  * La page **Vue d’ensemble** fournit des informations telles que le nom du compte, le conteneur et le nom du partage de fichiers.
      1. Pour les clés de compte, accédez à **Clés d’accès** dans le volet **Paramètres**.
      1. Pour les jetons SAP, accédez à **Signatures d’accès partagé** dans le volet **Paramètres**.

* Si vous envisagez d’utiliser un [principal de service](../active-directory/develop/howto-create-service-principal-portal.md) pour l’authentification, accédez à vos **Inscriptions d’applications**, puis sélectionnez l’application que vous souhaitez utiliser.
    * La page **Vue d’ensemble** correspondante contient des informations requises comme l’ID de locataire et l’ID de client.

> [!IMPORTANT]
> * Si vous avez besoin de modifier vos clés d’accès pour un compte de stockage Azure (clé de compte ou jeton SAP), veillez à synchroniser les nouvelles informations d’identification avec votre espace de travail et les magasins de données qui y sont connectés. Découvrez comment [synchroniser vos informations d’identification mises à jour](how-to-change-storage-access-key.md). <br> <br>
> * Si vous désinscrivez puis réinscrivez un magasin de données portant le même nom et que l’opération échoue, le compte Azure Key Vault de votre espace de travail peut ne pas être doté de la suppression réversible. Par défaut, la suppression réversible est activée pour l’instance de coffre de clés créée par votre espace de travail, mais elle peut ne pas l’être si vous avez utilisé un coffre de clés existant ou si votre espace de travail a été créé avant octobre 2020. Pour plus d’informations sur l’activation de la suppression réversible, consultez [Activer la suppression réversible pour un coffre de clés existant]( https://docs.microsoft.com/azure/key-vault/general/soft-delete-change#turn-on-soft-delete-for-an-existing-key-vault).

### <a name="permissions"></a>Autorisations

Pour le conteneur de blobs Azure et le stockage Azure Data Lake Gen2, assurez-vous que vos informations d’authentification vous donnent un accès **Lecteur des données blob du stockage**. En savoir plus sur le [Lecteur des données blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Par défaut, le jeton SAS d'un compte ne dispose d'aucune autorisation. 
* Pour l’**accès en lecture** aux données, vos informations d’authentification doivent au minimum disposer d’autorisations de liste et de lecture pour les conteneurs et les objets. 

* Pour l’**accès en écriture** aux données, des autorisations d’écriture et d’ajout sont également requises.

## <a name="train-with-datasets"></a>Entraîner avec des jeux de données

Utilisez vos jeux de données dans vos expériences d’apprentissage automatique pour la formation de modèles ML. [En savoir plus sur l’apprentissage avec des jeux de données](how-to-train-with-datasets.md)

## <a name="next-steps"></a>Étapes suivantes

* [Exemple pas à pas de formation avec TabularDatasets et Machine Learning automatisé](tutorial-first-experiment-automated-ml.md).

* [Effectuer l’apprentissage d’un modèle](how-to-set-up-training-targets.md).

* Pour obtenir plus d’exemples d’entraînement de jeux de données, consultez les [exemples de notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).