---
title: Chiffrement des données avec Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Découvrez comment les calculs et les magasins de données Azure Machine Learning fournissent le chiffrement des données au repos et en transit.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jhirono
author: jhirono
ms.reviewer: larryfr
ms.date: 11/09/2020
ms.openlocfilehash: 211ef9571b5a126686b4583330dc0f80863fd47e
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94992044"
---
# <a name="data-encryption-with-azure-machine-learning"></a>Chiffrement des données avec Azure Machine Learning

Azure Machine Learning utilise une variété de services de stockage de données et de ressources de calcul Azure lors de la formation de modèles et de l’exécution d’inférences. Chacun de ces services a sa propre façon d’assurer le chiffrement des données au repos et en transit. Dans cet article, apprenez-en davantage sur chacune d’elles et celle qui convient le mieux à votre scénario.

> [!IMPORTANT]
> Pour le chiffrement de niveau production au cours de __l’apprentissage__, Microsoft recommande d’utiliser le cluster de calcul Azure Machine Learning. Pour le chiffrement de niveau production au cours de __l’inférence__, Microsoft recommande d’utiliser Azure Kubernetes Service.
>
> L’instance de calcul Azure Machine Learning constitue un environnement de dev/test. Lorsque vous l’utilisez, nous vous recommandons de stocker vos fichiers, notamment les notebooks et les scripts, dans un partage de fichiers. Vos données doivent être stockées dans un magasin de données.

## <a name="encryption-at-rest"></a>Chiffrement au repos

> [!IMPORTANT]
> Si votre espace de travail contient des données sensibles, nous vous recommandons de définir l’[indicateur hbi_workspace](/python/api/azureml-core/azureml.core.workspace%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truecreate-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-) lors de la création de votre espace de travail. L’indicateur `hbi_workspace` ne peut être défini qu’au moment de la création d’un espace de travail. Il ne peut pas être modifié pour un espace de travail existant.

L'indicateur `hbi_workspace` contrôle la quantité de [données que Microsoft collecte à des fins de diagnostic](#microsoft-collected-data), et permet un [chiffrement supplémentaire dans des environnements gérés par Microsoft](../security/fundamentals/encryption-atrest.md). Il active par ailleurs les opérations suivantes :

* Lance le chiffrement du disque de travail local dans votre cluster de calcul Azure Machine Learning, à condition que vous n’ayez créé aucun cluster dans cet abonnement. Autrement, vous devez ouvrir un ticket de support pour activer le chiffrement du disque de travail de vos clusters de calcul. 
* Nettoie votre disque de travail local entre les exécutions.
* Transmet en toute sécurité les informations d’identification de votre compte de stockage, de votre registre de conteneurs et de votre compte SSH de la couche d’exécution à vos clusters de calcul en utilisant votre coffre de clés.
* Active le filtrage IP pour s’assurer que les pools Batch sous-jacents ne peuvent pas être appelés par des services externes autres que AzureMachineLearningService.
* Notez que les instances de calcul ne sont pas prises en charge dans l’espace de travail HBI.

### <a name="azure-blob-storage"></a>Stockage Blob Azure

Azure Machine Learning stocke les captures instantanées, les sorties et les journaux dans le compte Stockage Blob Azure qui est lié à l’espace de travail Azure Machine Learning et à votre abonnement. Toutes les données stockées dans Stockage Blob Azure sont chiffrées au repos à l’aide de clés gérées par Microsoft.

Pour plus d’informations sur la façon d’utiliser vos propres clés pour les données stockées dans Stockage Blob Azure, consultez [Chiffrement de Stockage Azure à l’aide de clés gérées par le client dans Azure Key Vault](../storage/common/customer-managed-keys-configure-key-vault.md).

En général, les données d’entraînement sont également stockées dans Stockage Blob Azure afin d’être accessibles pour les cibles de calcul d’entraînement. Ce stockage n’est pas managé par Azure Machine Learning mais il est monté sur des cibles de calcul en tant que système de fichiers distant.

Si vous devez opérer une __rotation ou révocation__ de votre clé, vous pouvez le faire à tout moment. Lors de la rotation d’une clé, le compte de stockage commence à utiliser la nouvelle clé (dernière version) pour chiffrer les données au repos. Lors de la révocation (désactivation) d’une clé, le compte de stockage se charge des demandes qui échouent. La prise d’effet de la rotation ou de la révocation nécessite généralement une heure.

Pour des informations sur la regénération des clés d’accès, consultez [Regénérer les clés d’accès au stockage](how-to-change-storage-access-key.md).

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Machine Learning stocke les métadonnées dans une instance d’Azure Cosmos DB. Cette instance est associée à un abonnement Microsoft géré par Azure Machine Learning. Toutes les données stockées dans Azure Cosmos DB sont chiffrées au repos à l’aide de clés gérées par Microsoft.

Pour utiliser vos propres clés (gérées par le client) afin de chiffrer l’instance d’Azure Cosmos DB, vous pouvez créer une instance de Cosmos DB dédiée à utiliser avec votre espace de travail. Nous vous recommandons cette approche si vous souhaitez stocker vos données, telles que des informations sur l’historique des exécutions, en dehors de l’instance de Cosmos DB mutualisée hébergée dans notre abonnement Microsoft. 

Pour activer l’approvisionnement d’une instance de Cosmos DB dans votre abonnement avec des clés gérées par le client, effectuez les actions suivantes :

* Inscrivez les fournisseurs de ressources Microsoft.MachineLearning et Microsoft.DocumentDB dans votre abonnement, si ce n’est pas déjà fait.

* Utilisez les paramètres suivants lors de la création de l’espace de travail Azure Machine Learning. Les deux paramètres sont obligatoires et pris en charge dans les kits de développement logiciel (SDK), l’interface CLI, les API REST et les modèle Resource Manager.

    * `resource_cmk_uri`: Ce paramètre est l’URI de ressource complète de la clé gérée par le client dans votre coffre de clés, y compris les [informations de version pour la clé](../key-vault/general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning). 

    * `cmk_keyvault`: Ce paramètre est l’ID de la ressource du coffre de clés dans votre abonnement. Ce coffre de clés doit être dans les mêmes région et abonnement que vous utiliserez pour l’espace de travail Azure Machine Learning. 
    
        > [!NOTE]
        > Cette instance de coffre de clés peut être différente du coffre de clés créé par Azure Machine Learning lorsque vous configurez l’espace de travail. Si vous voulez utiliser la même instance de coffre de clés pour l’espace de travail, passez le même coffre de clés pendant la configuration de l’espace de travail en utilisant le [paramètre key_vault](/python/api/azureml-core/azureml.core.workspace%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truecreate-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-). 

[!INCLUDE [machine-learning-customer-managed-keys.md](../../includes/machine-learning-customer-managed-keys.md)]

Si vous devez opérer une __rotation ou révocation__ de votre clé, vous pouvez le faire à tout moment. Lors de la rotation d’une clé, Cosmos DB commence à utiliser la nouvelle clé (dernière version) pour chiffrer les données au repos. Lors de la révocation (désactivation) d’une clé, Cosmos DB se charge des demandes qui échouent. La prise d’effet de la rotation ou de la révocation nécessite généralement une heure.

Pour plus d’informations sur les clés gérées par le client avec Cosmos DB, consultez [Configurer des clés gérées par le client pour votre compte Azure Cosmos DB](../cosmos-db/how-to-setup-cmk.md).

### <a name="azure-container-registry"></a>Azure Container Registry

Toutes les images conteneur dans votre registre ( Azure Container Registry) sont chiffrées au repos. Azure chiffre automatiquement une image avant de la stocker et la déchiffre quand Azure Machine Learning extrait l’image.

Pour utiliser vos propres clés (gérées par le client) pour chiffrer votre Azure Container Registry, vous devez créer votre propre ACR et le joindre pendant la configuration de l’espace de travail ou chiffrer l’instance par défaut qui est créée au moment de la configuration de l’espace de travail.

> [!IMPORTANT]
> Azure Machine Learning nécessite l’activation du compte administrateur sur votre instance Azure Container Registry. Par défaut, ce paramètre est désactivé lorsque vous créez un registre de conteneurs. Pour plus d’informations sur l’activation du compte administrateur, consultez [Compte administrateur](../container-registry/container-registry-authentication.md#admin-account).
>
> Une fois qu’un Azure Container Registry a été créé pour un espace de travail, ne le supprimez pas. Cela entraînerait l’arrêt de votre espace de travail Azure Machine Learning.

Pour un exemple de création d’un espace de travail en utilisant un Azure Container Registry existant, consultez les articles suivants :

* [Créer un espace de travail pour Azure Machine Learning avec Azure CLI](how-to-manage-workspace-cli.md).
* [Créer un espace de travail avec kit de développement logiciel (SDK) Python](how-to-manage-workspace.md?tabs=python#create-a-workspace).
* [Utiliser un modèle Azure Resource Manager pour créer un espace de travail pour Azure Machine Learning](how-to-create-workspace-template.md)

### <a name="azure-container-instance"></a>Azure Container Instance

Vous pouvez chiffrer une ressource Azure Container instance (ACI) déployée en utilisant des clés gérées par le client. Vous pouvez stocker la clé gérée par le client utilisée pour ACI dans l’Azure Key Vault pour votre espace de travail. Pour plus d’informations sur la génération d’une clé, consultez [Chiffrer des données avec une clé gérée par le client](../container-instances/container-instances-encrypt-data.md#generate-a-new-key).

Pour utiliser la clé lors du déploiement d’un modèle sur Azure Container instance, créez une nouvelle configuration de déploiement en utilisant la commande `AciWebservice.deploy_configuration()`. Fournissez les informations de clé à l’aide des paramètres suivants :

* `cmk_vault_base_url`: URL du coffre de clés contenant la clé.
* `cmk_key_name`: Nom de la clé.
* `cmk_key_version`: Version de la clé.

Pour plus d’informations sur la création et l’utilisation d’une configuration de déploiement, consultez les articles suivants :

* Informations de référence sur [AciWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice?preserve-view=true&view=azure-ml-py#&preserve-view=truedeploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none-)
* [Où et comment déployer](how-to-deploy-and-where.md)
* [Déployer un modèle sur Azure Container Instances](how-to-deploy-azure-container-instance.md)

Pour plus d’informations sur l’utilisation d’une clé gérée par le client avec ACI, consultez [Chiffrer des données avec une clé gérée par le client](../container-instances/container-instances-encrypt-data.md#encrypt-data-with-a-customer-managed-key).

### <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Vous pouvez chiffrer une ressource Azure Kubernetes Service déployée en utilisant des clés gérées par un client à tout moment. Pour plus d’informations, consultez [Apporter vos propres clés avec Azure Container Service](../aks/azure-disk-customer-managed-keys.md). 

Ce processus vous permet de chiffrer à la fois les données et le disque de système d’exploitation des machines virtuelles déployées dans le cluster Kubernetes.

> [!IMPORTANT]
> Ce processus fonctionne uniquement avec AKS K8s version 1.17 ou ultérieure. Azure Machine Learning a ajouté un support pour AKS 1.17 le 13 janvier 2020.

### <a name="machine-learning-compute"></a>Capacité de calcul Machine Learning

Le disque de système d’exploitation de chaque nœud de calcul stocké dans Stockage Azure est chiffré à l’aide de clés gérées par Microsoft dans les comptes de stockage Azure Machine Learning. Cette cible de calcul est éphémère et les clusters font généralement l’objet d’un scale-down quand aucune exécution n’est placée en file d’attente. La machine virtuelle sous-jacente est déprovisionnée et le disque de système d’exploitation supprimé. Azure Disk Encryption n’est pas pris en charge pour le disque de système d’exploitation.

Chaque machine virtuelle dispose également d’un disque temporaire local pour les opérations de système d’exploitation. Si vous le souhaitez, vous pouvez utiliser le disque pour indexer les données d’entraînement. Le disque est chiffré par défaut pour les espaces de travail avec le paramètre `hbi_workspace` défini sur `TRUE`. Cet environnement ne perdure que pour la durée de votre exécution et la prise en charge du chiffrement est limitée aux clés gérées par le système uniquement.

### <a name="azure-databricks"></a>Azure Databricks

Vous pouvez utiliser la plateforme Azure Databricks dans des pipelines Azure Machine Learning. Par défaut, le système de fichiers Databricks (DBFS) qu’utilise Azure Databricks est chiffré à l’aide d’une clé gérée par Microsoft. Pour configurer Azure Databricks afin d’utiliser des clés gérées par le client, voir [Configurer les clés gérées par le client sur DBFS (racine) par défaut](/azure/databricks/security/customer-managed-keys-dbfs).

### <a name="microsoft-generated-data"></a>Données générées par Microsoft

Lorsque vous utilisez des services comme Machine Learning automatisé, Microsoft peut générer des données prétraitées temporaires pour l’apprentissage de plusieurs modèles. Ces données sont stockées dans un magasin de données de votre espace de travail, ce qui vous permet d’appliquer des contrôles d'accès et un chiffrement appropriés.

Vous pouvez aussi chiffrer des [informations de diagnostic enregistrées à partir de votre point de terminaison déployé](how-to-enable-app-insights.md) dans votre instance d’Azure Application Insights.

## <a name="encryption-in-transit"></a>Chiffrement en transit

Azure Machine Learning utilise le protocole TLS pour sécuriser la communication interne entre ses différents microservices. Tout l’accès au stockage Azure se produit également sur un canal sécurisé.

Pour sécuriser les appels externes au point de terminaison de scoring, Azure Machine Learning utilise le protocole TLS. Pour plus d’informations, consultez [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](./how-to-secure-web-service.md).

## <a name="data-collection-and-handling"></a>Collecte et gestion de données

### <a name="microsoft-collected-data"></a>Données collectées par Microsoft

Microsoft peut collecter des informations ne permettant pas d’identifier les utilisateurs telles que des noms de ressource (par exemple le nom du jeu de données ou le nom de l’essai d’apprentissage automatique), ou des variable d'environnement de tâche à des fins de diagnostic. De telles données sont stockées à l’aide des clés gérées par Microsoft dans un stockage hébergé dans des abonnements appartenant à Microsoft, conformément aux [normes de gestion des données et à la politique de confidentialité standard de Microsoft](https://privacy.microsoft.com/privacystatement).

Microsoft recommande également de ne pas stocker d’informations sensibles (comme les secrets de clé de compte) dans les variables d'environnement. Les variable d'environnement sont enregistrées, chiffrées et stockées par nous. De même, lorsque vous nommez [run_id](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py), évitez d’inclure des informations sensibles telles que des noms d’utilisateurs ou des noms de projets secrets. Ces informations peuvent apparaître dans les journaux de télémétrie accessibles aux ingénieurs du Support Microsoft.

Vous pouvez refuser la collecte des données de diagnostic en définissant le paramètre `hbi_workspace` sur `TRUE` pendant la configuration de l’espace de travail. Cette fonctionnalité est prise en charge lorsque le kit de développement logiciel (SDK) AzureML Python, l’interface CLI, les API REST ou les modèles Azure Resource Manager sont utilisés.

## <a name="using-azure-key-vault"></a>Utilisation d’Azure Key Vault

Azure Machine Learning utilise l’instance Azure Key Vault associée à l’espace de travail pour stocker les informations d’identification de différents types :

* Chaîne de connexion du compte de stockage associé
* Mots de passe pour les instances Azure Container Repository
* Chaînes de connexion aux magasins de données

Les mots de passe et clés SSH des cibles de calcul telles que HDI HDInsight et les machines virtuelles sont stockés dans un coffre de clés distinct qui est associé à l’abonnement Microsoft. Azure Machine Learning ne stocke pas les mots de passe ni les clés fournis par les utilisateurs. Il génère, autorise et stocke plutôt ses propres clés SSH pour se connecter aux machines virtuelles et à HDInsight afin d’exécuter les expériences.

Chaque espace de travail est associé à une identité managée attribuée par le système qui porte le même nom que l’espace de travail. Cette identité managée a accès à l’ensemble des clés, secrets et certificats contenus dans le coffre de clés.

## <a name="next-steps"></a>Étapes suivantes

* [Se connecter au Stockage Azure](how-to-access-data.md)
* [Obtenir des données à partir d’une banque de données](how-to-create-register-datasets.md)
* [Se connecter aux données](how-to-connect-data-ui.md)
* [Entraîner avec des jeux de données](how-to-train-with-datasets.md)