---
title: Se connecter à des services de stockage sur Azure
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser des magasins de données pour vous connecter en toute sécurité aux services de stockage Azure lors de l’entraînement avec Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
ms.reviewer: nibaccam
ms.date: 11/03/2020
ms.custom: how-to, contperf-fy21q1, devx-track-python, data4ml
ms.openlocfilehash: bb63ac6de6c48bb3853bd235d908ee745ff5279d
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032845"
---
# <a name="connect-to-storage-services-on-azure"></a>Se connecter à des services de stockage sur Azure

Dans cet article, découvrez comment vous **connecter aux services de stockage sur Azure par le biais des magasins de données Azure Machine Learning**. Les banques de données se connectent en toute sécurité à votre service de stockage Azure sans avoir à compromettre vos informations d’authentification et l’intégrité de votre source de données d’origine. Ils stockent des informations de connexion, comme votre ID d’abonnement et votre autorisation de jeton, dans votre [Key Vault](https://azure.microsoft.com/services/key-vault/) associé à l’espace de travail, de sorte que vous pouvez accéder à votre stockage en toute sécurité sans avoir à coder en dur ces informations dans vos scripts. Vous pouvez utiliser le [kit de développement logiciel (SDK) Azure Machine Learning Python](#python) ou [Azure Machine Learning Studio](how-to-connect-data-ui.md) pour créer et inscrire des banques de données.

Si vous préférez créer et gérer des banques de données à l’aide de l’extension Azure Machine Learning VS Code, consultez le [guide de procédures de gestion des ressources VS Code](how-to-manage-resources-vscode.md#datastores).

Vous pouvez créer des magasins de données à partir de ces [solutions de stockage Azure](#matrix). **Pour les solutions de stockage non prises en charge**, et pour réduire le coût de sortie des données pendant les expériences de Machine Learning, [déplacez vos données](#move) vers une solution de stockage Azure prise en charge.  

Pour comprendre où les magasins de données interviennent dans flux de travail d’accès aux données d’Azure Machine Learning, consultez l’article [Accéder en toute sécurité aux données](concept-data.md#data-workflow).

## <a name="prerequisites"></a>Prérequis

Vous devez disposer des éléments suivants :
- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).

- Un compte de stockage Azure avec un [type de stockage pris en charge](#matrix).

- Le [Kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) ou l’accès à [Azure Machine Learning studio](https://ml.azure.com/).

- Un espace de travail Azure Machine Learning.
  
  [Créez un espace de travail Azure Machine Learning](how-to-manage-workspace.md) ou utilisez un espace de travail existant via le Kit de développement logiciel (SDK) Python. 

    Importez les classes `Workspace` et `Datastore`, puis chargez vos informations d’abonnement à partir du fichier `config.json` à l’aide de la fonction `from_config()`. Cette fonction recherche le fichier JSON dans le répertoire actif par défaut, mais vous pouvez également spécifier un paramètre de chemin d’accès pour pointer vers le fichier à l’aide de `from_config(path="your/file/path")`.

   ```Python
   import azureml.core
   from azureml.core import Workspace, Datastore
        
   ws = Workspace.from_config()
   ```

    Quand vous créez un espace de travail, un conteneur d’objets blob Azure et un partage de fichiers Azure sont inscrits automatiquement comme magasins de données dans l’espace de travail. sous les noms `workspaceblobstore` et `workspacefilestore` respectivement. Le `workspaceblobstore` est utilisé pour stocker les artefacts de l’espace de travail et vos journaux d’expérience de Machine Learning. Il est également défini en tant que **magasin de données par défaut** et ne peut pas être supprimé de l’espace de travail. Le `workspacefilestore` est utilisé pour stocker des notebooks et des scripts R autorisés via une [instance de calcul](./concept-compute-instance.md#accessing-files).
    
    > [!NOTE]
    > Le concepteur Azure Machine Learning crée automatiquement un magasin de données nommé **azureml_globaldatasets** lorsque vous ouvrez un exemple dans la page d’accueil du concepteur. Ce magasin de données contient uniquement des exemples de jeux de données. Veuillez **ne pas** utiliser ce magasin de données pour accéder à des données confidentielles.

<a name="matrix"></a>

## <a name="supported-data-storage-service-types"></a>Types de services de stockage de données pris en charge

Les magasins de données prennent actuellement en charge le stockage des informations de connexion dans les services de stockage figurant dans la matrice suivante.

| Type de&nbsp;stockage | Type&nbsp;d’authentification | [Azure&nbsp;Machine&nbsp;Learning studio](https://ml.azure.com/) | [SDK Azure&nbsp;Machine&nbsp;Learning&nbsp; Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) |  [CLI Azure&nbsp;Machine&nbsp;Learning](reference-azure-machine-learning-cli.md) | [API REST Azure&nbsp;Machine&nbsp;Learning&nbsp;](/rest/api/azureml/) | VS Code
---|---|---|---|---|---|---
[Stockage&nbsp;Blob&nbsp;Azure](../storage/blobs/storage-blobs-overview.md)| Clé de compte <br> Jeton SAS | ✓ | ✓ | ✓ |✓ |✓
[Partage&nbsp;de fichiers&nbsp;Azure](../storage/files/storage-files-introduction.md)| Clé de compte <br> Jeton SAS | ✓ | ✓ | ✓ |✓|✓
[Azure&nbsp;Data Lake&nbsp;Storage Gen&nbsp;1](../data-lake-store/index.yml)| Principal du service| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;Data Lake&nbsp;Storage Gen&nbsp;2](../storage/blobs/data-lake-storage-introduction.md)| Principal du service| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;SQL&nbsp;Database](../azure-sql/database/sql-database-paas-overview.md)| Authentification SQL <br>Principal du service| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;PostgreSQL](../postgresql/overview.md) | Authentification SQL| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;Database&nbsp;pour&nbsp;MySQL](../mysql/overview.md) | Authentification SQL|  | ✓* | ✓* |✓*|
[Databricks&nbsp;File&nbsp;System](/azure/databricks/data/databricks-file-system)| Aucune authentification | | ✓** | ✓ ** |✓** |

\* MySQL est uniquement pris en charge pour le pipeline [DataTransferStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep?preserve-view=true&view=azure-ml-py).<br />
\*\* Databricks est uniquement pris en charge pour le pipeline [DatabricksStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricks_step.databricksstep?preserve-view=true&view=azure-ml-py).


### <a name="storage-guidance"></a>Conseils liés au stockage

Nous vous recommandons de créer un magasin de données pour un [conteneur Blob Azure](../storage/blobs/storage-blobs-introduction.md). Les stockages Standard et Premium sont tous deux disponibles pour les objets blob. Le stockage Premium est plus cher, mais ses vitesses de débit supérieures vous feront gagner du temps sur vos exécutions d’entraînement, en particulier si vous effectuez l’entraînement sur un jeu de données volumineux. Pour plus d’informations sur le coût des comptes de stockage, consultez la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/?service=machine-learning-service).

[Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) s’appuie sur le stockage Blob Azure et a été conçu pour l’analytique Big Data des entreprises. Une caractéristique fondamentale de Data Lake Storage Gen2 est l’ajout d’un [espace de noms hiérarchique](../storage/blobs/data-lake-storage-namespace.md) au stockage Blob. L’espace de noms hiérarchique organise les objets/fichiers dans une hiérarchie de répertoires pour offrir un accès efficace aux données.

## <a name="storage-access-and-permissions"></a>Accès et autorisations pour le stockage

Pour s’assurer que vous vous connectez en toute sécurité à votre service de stockage Azure, Azure Machine Learning exige que vous ayez l’autorisation d’accéder au conteneur de stockage de données correspondant. Cet accès dépend des informations d’authentification utilisées pour inscrire le magasin de données. 

### <a name="virtual-network"></a>Réseau virtuel 

Si votre compte de stockage de données se trouve sur un **réseau virtuel**, des étapes de configuration supplémentaires sont nécessaires pour s’assurer qu’Azure Machine Learning a accès à vos données. Pour vous assurer que les étapes de configuration appropriées sont appliquées lors de la création et de l’enregistrement de votre magasin de données, consultez [Utiliser le studio Azure Machine Learning dans un réseau virtuel Azure](how-to-enable-studio-virtual-network.md).  

### <a name="access-validation"></a>Validation de l’accès

**Dans le cadre du processus de création et d’inscription du magasin de données initial**, Azure Machine Learning vérifie automatiquement que le service de stockage sous-jacent existe et que le principal fourni par l’utilisateur (nom d’utilisateur, principal de service ou jeton SAS) a accès au stockage spécifié.

**Après la création** du magasin de données, cette validation est effectuée uniquement pour les méthodes qui requièrent l’accès au conteneur de stockage sous-jacent, et **non** chaque fois que des objets du magasin de données sont récupérés. Par exemple, la validation se produit si vous souhaitez télécharger des fichiers à partir de votre magasin de données ; mais si vous souhaitez simplement modifier votre magasin de données par défaut, la validation ne se produit pas.

Pour authentifier votre accès au service de stockage sous-jacent, vous pouvez fournir votre clé de compte, des jetons de signature d’accès partagé (SAS) ou le principal de service dans la méthode `register_azure_*()` correspondante du type de magasin de données que vous souhaitez créer. La [matrice de types de stockage](#matrix) répertorie les types d’authentification pris en charge qui correspondent à chaque type de magasin de données.

Vous trouverez des informations sur la clé de compte, le jeton SAS et le principal de service sur votre [portail Azure](https://portal.azure.com).

* Si vous envisagez d’utiliser une clé de compte ou un jeton SAP pour l’authentification, sélectionnez **Comptes de stockage** dans le volet gauche, puis choisissez le compte de stockage que vous souhaitez inscrire. 
  * La page **Vue d’ensemble** fournit des informations telles que le nom du compte, le conteneur et le nom du partage de fichiers. 
      1. Pour les clés de compte, accédez à **Clés d’accès** dans le volet **Paramètres**. 
      1. Pour les jetons SAP, accédez à **Signatures d’accès partagé** dans le volet **Paramètres**.

* Si vous prévoyez d’utiliser un principal du service pour l’authentification, accédez à vos **Inscriptions d’applications**, puis sélectionnez l’application que vous souhaitez utiliser. 
    * La page **Vue d’ensemble** correspondante contient des informations requises comme l’ID de locataire et l’ID de client.

> [!IMPORTANT]
> * Si vous avez besoin de modifier vos clés d’accès pour un compte de stockage Azure (clé de compte ou jeton SAP), veillez à synchroniser les nouvelles informations d’identification avec votre espace de travail et les magasins de données qui y sont connectés. Découvrez comment [synchroniser vos informations d’identification mises à jour](how-to-change-storage-access-key.md). 
### <a name="permissions"></a>Autorisations

Pour le conteneur de blobs Azure et le stockage Azure Data Lake Gen2, assurez-vous que vos informations d’authentification vous donnent un accès **Lecteur des données blob du stockage**. En savoir plus sur le [Lecteur des données blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Par défaut, le jeton SAS d'un compte ne dispose d'aucune autorisation. 
* Pour l’**accès en lecture** aux données, vos informations d’authentification doivent au minimum disposer d’autorisations de liste et de lecture pour les conteneurs et les objets. 

* Pour l'**accès en écriture** aux données, des autorisations d'écriture et d'ajout sont également requises.

<a name="python"></a>

## <a name="create-and-register-datastores"></a>Créer et inscrire des magasins de données

Quand vous inscrivez une solution de stockage Azure en tant que magasin de données, ce dernier est automatiquement créé et inscrit dans un espace de travail spécifique. Retrouvez les instructions relatives aux scénarios de réseau virtuel et les indications pour trouver les informations d’authentification requises dans la section [accès au stockage et autorisations](#storage-access-and-permissions). 

Dans cette section, vous trouverez des exemples de création et d’inscription d’un magasin de données via le kit de développement logiciel (SDK) Python pour les types de stockage suivants. Les paramètres fournis dans ces exemples sont les **paramètres requis** pour créer et inscrire un magasin de données.

* [Conteneur d’objets blob Azure](#azure-blob-container)
* [Partage de fichiers Azure](#azure-file-share)
* [Azure Data Lake Storage Gen2](#azure-data-lake-storage-generation-2)

 Pour créer des magasins de données pour les autres services de stockage pris en charge, consultez la [documentation de référence des méthodes `register_azure_*` applicables](/python/api/azureml-core/azureml.core.datastore.datastore?preserve-view=true&view=azure-ml-py#&preserve-view=truemethods).

Si vous préférez une expérience à moindre code, consultez [Se connecter aux données avec Azure Machine Learning Studio](how-to-connect-data-ui.md).
>[!IMPORTANT]
> Si vous désinscrivez puis réinscrivez un magasin de données portant le même nom et que l’opération échoue, le compte Azure Key Vault de votre espace de travail peut ne pas être doté de la suppression réversible. Par défaut, la suppression réversible est activée pour l’instance de coffre de clés créée par votre espace de travail, mais elle peut ne pas l’être si vous avez utilisé un coffre de clés existant ou si votre espace de travail a été créé avant octobre 2020. Pour plus d’informations sur l’activation de la suppression réversible, consultez [Activer la suppression réversible pour un coffre de clés existant]( https://docs.microsoft.com/azure/key-vault/general/soft-delete-change#turn-on-soft-delete-for-an-existing-key-vault).

> [!NOTE]
> Le nom du magasin de données doit contenir uniquement des lettres minuscules, des chiffres et des traits de soulignement. 

### <a name="azure-blob-container"></a>Conteneur d’objets blob Azure

Pour inscrire un conteneur d’objets blob Azure comme magasin de données, utilisez [`register_azure_blob_container()`](/python/api/azureml-core/azureml.core.datastore%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=trueregister-azure-blob-container-workspace--datastore-name--container-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false--blob-cache-timeout-none--grant-workspace-access-false--subscription-id-none--resource-group-none-).

Le code suivant crée le magasin de données `blob_datastore_name` et l’inscrit auprès de l’espace de travail `ws`. Ce magasin de données accède au conteneur d'objets blob `my-container-name` sur le compte de stockage `my-account-name` avec la clé d'accès au compte fournie. Retrouvez les instructions relatives aux scénarios de réseau virtuel et les indications pour trouver les informations d’authentification requises dans la section [accès au stockage et autorisations](#storage-access-and-permissions). 

```Python
blob_datastore_name='azblobsdk' # Name of the datastore to workspace
container_name=os.getenv("BLOB_CONTAINER", "<my-container-name>") # Name of Azure blob container
account_name=os.getenv("BLOB_ACCOUNTNAME", "<my-account-name>") # Storage account name
account_key=os.getenv("BLOB_ACCOUNT_KEY", "<my-account-key>") # Storage account access key

blob_datastore = Datastore.register_azure_blob_container(workspace=ws, 
                                                         datastore_name=blob_datastore_name, 
                                                         container_name=container_name, 
                                                         account_name=account_name,
                                                         account_key=account_key)
```

### <a name="azure-file-share"></a>Partage de fichiers Azure

Pour inscrire un partage de fichiers Azure comme magasin de données, utilisez [`register_azure_file_share()`](/python/api/azureml-core/azureml.core.datastore%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=trueregister-azure-file-share-workspace--datastore-name--file-share-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false-). 

Le code suivant crée le magasin de données `file_datastore_name` et l’inscrit auprès de l’espace de travail `ws`. Ce magasin de données accède au partage de fichiers `my-fileshare-name` sur le compte de stockage `my-account-name` avec la clé d'accès au compte fournie. Retrouvez les instructions relatives aux scénarios de réseau virtuel et les indications pour trouver les informations d’authentification requises dans la section [accès au stockage et autorisations](#storage-access-and-permissions). 

```Python
file_datastore_name='azfilesharesdk' # Name of the datastore to workspace
file_share_name=os.getenv("FILE_SHARE_CONTAINER", "<my-fileshare-name>") # Name of Azure file share container
account_name=os.getenv("FILE_SHARE_ACCOUNTNAME", "<my-account-name>") # Storage account name
account_key=os.getenv("FILE_SHARE_ACCOUNT_KEY", "<my-account-key>") # Storage account access key

file_datastore = Datastore.register_azure_file_share(workspace=ws,
                                                     datastore_name=file_datastore_name, 
                                                     file_share_name=file_share_name, 
                                                     account_name=account_name,
                                                     account_key=account_key)
```

### <a name="azure-data-lake-storage-generation-2"></a>Azure Data Lake Storage Gen 2

Pour un magasin de données Azure Data Lake Storage Gen 2 (ADLS Gen 2), utilisez [register_azure_data_lake_gen2()](/python/api/azureml-core/azureml.core.datastore.datastore?preserve-view=true&view=azure-ml-py#&preserve-view=trueregister-azure-data-lake-gen2-workspace--datastore-name--filesystem--account-name--tenant-id--client-id--client-secret--resource-url-none--authority-url-none--protocol-none--endpoint-none--overwrite-false-) pour inscrire un magasin de données d’informations d’identification connecté à un stockage Azure DataLake Gen 2 avec des [autorisations de principal du service](../active-directory/develop/howto-create-service-principal-portal.md).  

Pour utiliser votre principal de service, vous devez [inscrire votre application](../active-directory/develop/app-objects-and-service-principals.md) et accorder au principal de service l’accès aux données via un contrôle d’accès en fonction du rôle Azure (Azure RBAC) ou des listes de contrôle d’accès (ACL, access control lists). Découvrez-en plus sur la [configuration du contrôle d’accès pour ADLS Gen 2](../storage/blobs/data-lake-storage-access-control-model.md). 

Le code suivant crée le magasin de données `adlsgen2_datastore_name` et l’inscrit auprès de l’espace de travail `ws`. Ce magasin de données accède au système de fichiers `test` dans le compte de stockage `account_name` en utilisant les informations d’identification du principal de service fournies. Retrouvez les instructions relatives aux scénarios de réseau virtuel et les indications pour trouver les informations d’authentification requises dans la section [accès au stockage et autorisations](#storage-access-and-permissions). 

```python 
adlsgen2_datastore_name = 'adlsgen2datastore'

subscription_id=os.getenv("ADL_SUBSCRIPTION", "<my_subscription_id>") # subscription id of ADLS account
resource_group=os.getenv("ADL_RESOURCE_GROUP", "<my_resource_group>") # resource group of ADLS account

account_name=os.getenv("ADLSGEN2_ACCOUNTNAME", "<my_account_name>") # ADLS Gen2 account name
tenant_id=os.getenv("ADLSGEN2_TENANT", "<my_tenant_id>") # tenant id of service principal
client_id=os.getenv("ADLSGEN2_CLIENTID", "<my_client_id>") # client id of service principal
client_secret=os.getenv("ADLSGEN2_CLIENT_SECRET", "<my_client_secret>") # the secret of service principal

adlsgen2_datastore = Datastore.register_azure_data_lake_gen2(workspace=ws,
                                                             datastore_name=adlsgen2_datastore_name,
                                                             account_name=account_name, # ADLS Gen2 account name
                                                             filesystem='test', # ADLS Gen2 filesystem
                                                             tenant_id=tenant_id, # tenant id of service principal
                                                             client_id=client_id, # client id of service principal
                                                             client_secret=client_secret) # the secret of service principal
```

<a name="arm"></a>

## <a name="create-datastores-using-azure-resource-manager"></a>Créer des magasins de données à l'aide d'Azure Resource Manager

Les modèles disponibles à l'adresse [https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-datastore-create-*](https://github.com/Azure/azure-quickstart-templates/tree/master/) peuvent être utilisés pour créer des magasins de données.

Pour plus d'informations sur l'utilisation de ces modèles, consultez [Utiliser un modèle Azure Resource Manager afin de créer un espace de travail pour Azure Machine Learning](how-to-create-workspace-template.md).

<a name="train"></a>
## <a name="use-data-in-your-datastores"></a>Utiliser des données dans vos magasins de données

Après avoir créé un magasin de données, [créer un Azure Machine Learning DataSet](how-to-create-register-datasets.md) pour interagir avec vos données. Les jeux de données intègrent vos données dans un objet consommable évalué tardivement pour les tâches de Machine Learning, comme la formation. Ils offrent également la possibilité de [télécharger ou de monter](how-to-train-with-datasets.md#mount-vs-download) des fichiers de n’importe quel format à partir de services de stockage Azure, comme le stockage d’objets BLOB Azure et ADLS Gen2. Vous pouvez également les utiliser pour charger des données tabulaires dans un dataframe Pandas ou Spark.

<a name="get"></a>

## <a name="get-datastores-from-your-workspace"></a>Récupérer les magasins de données à partir de votre espace de travail

Pour obtenir un magasin de données spécifique inscrit dans l’espace de travail actif, utilisez la méthode statique [`get()`](/python/api/azureml-core/azureml.core.datastore%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-workspace--datastore-name-) sur la classe `Datastore` :

```Python
# Get a named datastore from the current workspace
datastore = Datastore.get(ws, datastore_name='your datastore name')
```
Pour obtenir la liste des banques de données inscrites avec un espace de travail donné, vous pouvez utiliser la propriété [`datastores`](/python/api/azureml-core/azureml.core.workspace%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truedatastores) sur un objet d’espace de travail :

```Python
# List all datastores registered in the current workspace
datastores = ws.datastores
for name, datastore in datastores.items():
    print(name, datastore.datastore_type)
```

Pour obtenir le magasin de données par défaut de l’espace de travail, utilisez cette ligne :

```Python
datastore = ws.get_default_datastore()
```
Vous pouvez également modifier le magasin de données par défaut avec le code suivant. Cette possibilité est prise en charge uniquement via le Kit de développement logiciel (SDK). 

```Python
 ws.set_default_datastore(new_default_datastore)
```

## <a name="access-data-during-scoring"></a>Accéder aux données pendant le scoring

Azure Machine Learning offre plusieurs moyens d’utiliser vos modèles pour le scoring. Certaines de ces méthodes ne fournissent pas d’accès aux magasins de données. Utilisez le tableau suivant pour comprendre les méthodes qui vous permettent d’accéder aux magasins de données pendant le scoring :

| Méthode | Accès aux magasins de données | Description |
| ----- | :-----: | ----- |
| [Prédiction par lots](./tutorial-pipeline-batch-scoring-classification.md) | ✔ | Effectuez des prédictions sur de grandes quantités de données de façon asynchrone. |
| [Service web](how-to-deploy-and-where.md) | &nbsp; | Déployez des modèles comme un service web. |
| [Module Azure IoT Edge](how-to-deploy-and-where.md) | &nbsp; | Déployez des modèles sur des appareils IoT Edge. |

Dans les situations où le SDK ne fournit pas d’accès aux magasins de données, vous pouvez créer du code personnalisé à l’aide du SDK Azure approprié pour accéder aux données. Par exemple, le [kit de développement logiciel (SDK) de stockage Azure pour Python](https://github.com/Azure/azure-storage-python) est une bibliothèque cliente que vous pouvez utiliser pour accéder aux données stockées dans des objets blob ou des fichiers.

<a name="move"></a>

## <a name="move-data-to-supported-azure-storage-solutions"></a>Déplacer des données vers des solutions de stockage Azure prises en charge

Azure Machine Learning prend en charge l’accès aux données à partir des stockages suivants : Stockage Blob Azure, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database et Azure Database pour PostgreSQL. Si vous utilisez un stockage non pris en charge, nous vous recommandons de déplacer vos données vers des solutions de stockage Azure prises en charge en utilisant [Azure Data Factory et ces étapes](../data-factory/quickstart-create-data-factory-copy-data-tool.md). Déplacer vos données vers un stockage pris en charge peut vous aider à réduire les coûts de sortie des données pendant les expériences Machine Learning. 

Azure Data Factory fournit un moyen de transfert des données efficace et résilient avec plus de 80 connecteurs prédéfinis, sans coût supplémentaire. Ces connecteurs incluent les services de données Azure, les sources de données locales, Amazon S3 et Redshift, et Google BigQuery.

## <a name="next-steps"></a>Étapes suivantes

* [Créer un jeu de données Azure Machine Learning](how-to-create-register-datasets.md)
* [Entraîner un modèle](how-to-set-up-training-targets.md)
* [Déployer un modèle](how-to-deploy-and-where.md)