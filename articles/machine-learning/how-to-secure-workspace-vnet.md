---
title: Sécuriser un espace de travail Azure Machine Learning à l’aide de réseaux virtuels
titleSuffix: Azure Machine Learning
description: Utilisez un réseau virtuel Azure isolé pour sécuriser votre espace de travail Azure Machine Learning et les ressources associées.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: peterlu
author: peterclu
ms.date: 10/06/2020
ms.topic: conceptual
ms.custom: how-to, contperf-fy20q4, tracking-python, contperf-fy21q1
ms.openlocfilehash: 07b8c130a2a22554e4cd5b33996d5a5ee967d47f
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029530"
---
# <a name="secure-an-azure-machine-learning-workspace-with-virtual-networks"></a>Sécuriser un espace de travail Azure Machine Learning à l’aide de réseaux virtuels

Dans cet article, vous allez apprendre comment sécuriser un espace de travail Azure Machine Learning et ses ressources associées dans un réseau virtuel.


Cet article est le deuxième d’une série de cinq qui vous guide à travers le processus de sécurisation d’un workflow Azure Machine Learning. Nous vous recommandons vivement de parcourir tout d’abord la [Première partie : Présentation du réseau virtuel](how-to-network-security-overview.md) pour en savoir plus sur l’architecture globale. 

Consultez les autres articles de cette série :

[1. Présentation du réseau virtuel](how-to-network-security-overview.md) > **2. Sécurisation de l’espace de travail** > [3. Sécurisation de l’environnement d’entraînement](how-to-secure-training-vnet.md) > [4. Sécurisation de l’environnement d’inférence](how-to-secure-inferencing-vnet.md) > [5. Activation de la fonctionnalité de studio](how-to-enable-studio-virtual-network.md)

Dans cet article, vous découvrirez comment activer les ressources d’espaces de travail suivantes dans un réseau virtuel :
> [!div class="checklist"]
> - Espace de travail Azure Machine Learning
> - Comptes de stockage Azure
> - Magasins de données et jeux de données Azure Machine Learning
> - Azure Key Vault
> - Azure Container Registry

## <a name="prerequisites"></a>Prérequis

+ Lisez l’article [Vue d’ensemble de la sécurité réseau](how-to-network-security-overview.md) pour comprendre les scénarios courants des réseaux virtuels et l’architecture globale des réseaux virtuels.

+ Un réseau virtuel et un sous-réseau existants à utiliser avec vos ressources de calcul.

+ Pour déployer des ressources dans un réseau virtuel ou un sous-réseau, votre compte d’utilisateur doit disposer d’autorisations pour les actions suivantes dans le contrôle d’accès en fonction du rôle Azure (Azure RBAC) :

    - « Microsoft.Network/virtualNetworks/join/action » sur la ressource de réseau virtuel.
    - « Microsoft.Network/virtualNetworks/subnet/join/action » sur la ressource de sous-réseau virtuel.

    Pour plus d’informations sur Azure RBAC avec la mise en réseau, consultez [Rôles intégrés pour la mise en réseau](../role-based-access-control/built-in-roles.md#networking)


## <a name="secure-the-workspace-with-private-endpoint"></a>Sécuriser l’espace de travail avec point de terminaison privé

Azure Private Link vous permet de vous connecter à votre espace de travail à l’aide d’un point de terminaison privé. Le point de terminaison privé est un ensemble d’adresses IP privées au sein de votre réseau virtuel. Vous pouvez alors limiter l’accès à votre espace de travail pour qu’il ne se fasse que sur les adresses IP privées. Azure Private Link permet de réduire le risque d’exfiltration des données.

Pour plus d’informations sur la configuration d’un espace de travail Private Link, consultez [Comment configurer Private Link](how-to-configure-private-link.md).

## <a name="secure-azure-storage-accounts-with-service-endpoints"></a>Sécuriser des comptes de stockage Azure avec des points de terminaison de service

Azure Machine Learning prend en charge les comptes de stockage configurés pour utiliser des points de terminaison de service ou de points de terminaison privés. Dans cette section, vous allez découvrir comment sécuriser un compte de stockage Azure à l’aide de points de terminaison de service. Pour les points de terminaison privés, consultez la section suivante.

> [!IMPORTANT]
> Vous pouvez placer le _compte de stockage par défaut_ pour Azure Machine Learning ou les _comptes de stockage autres que ceux par défaut_ dans un réseau virtuel.
>
> Le compte de stockage par défaut est automatiquement configuré lorsque vous créez un espace de travail.
>
> Pour les comptes de stockage autres que ceux par défaut, le paramètre `storage_account` dans la fonction [`Workspace.create()`](/python/api/azureml-core/azureml.core.workspace%28class%29?preserve-view=true&view=azure-ml-py#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-&preserve-view=true) vous permet de spécifier un compte de stockage personnalisé par ID de ressource Azure.

Pour utiliser le compte Stockage Azure de l’espace de travail d’un réseau virtuel, effectuez les étapes suivantes :

1. Dans le portail Azure, accédez au service de stockage que vous souhaitez utiliser dans votre espace de travail.

   [![Image du portail Azure montrant le stockage Azure lié à l’espace de travail Azure Machine Learning](./media/how-to-enable-virtual-network/workspace-storage.png)](./media/how-to-enable-virtual-network/workspace-storage.png#lightbox)

1. Dans la page du compte de service de stockage, sélectionnez __Pare-feu et réseaux virtuels__.

   ![La zone « Pare-feu et réseaux virtuels » de la page Stockage Azure dans le portail Azure](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks.png)

1. Dans la page __Pare-feux et réseaux virtuels__, effectuez les actions suivantes :
    1. Sélectionnez __Réseaux sélectionnés__.
    1. Cliquez sur __Réseaux virtuel__ puis choisissez le lien __Ajouter un réseau virtuel existant__. Cette action ajoute le réseau virtuel où réside votre capacité de calcul (voir l’étape 1).

        > [!IMPORTANT]
        > Le compte de stockage doit se trouver dans les mêmes réseau virtuel et sous-réseau que les instances de calcul ou clusters utilisés pour l'apprentissage ou l'inférence.

    1. Vérifiez que __Autoriser les services Microsoft approuvés à accéder à ce compte de stockage__ est coché. Cela n’autorise pas tous les services Azure à accéder à votre compte de stockage.
    
        * Les ressources de certains services, **inscrites dans votre abonnement**, peuvent accéder au compte de stockage **dans le même abonnement** pour des opérations spécifiques. C’est, par exemple, le cas de l’écriture de journaux ou de la création de sauvegardes.
        * Vous pouvez accorder aux ressources de certains services un accès explicite à votre compte de stockage en __attribuant un rôle Azure__ à son identité managée attribuée par le système.

        Pour plus d’informations, consultez [Configurer Pare-feu et réseaux virtuels dans Stockage Azure](../storage/common/storage-network-security.md#trusted-microsoft-services).

    > [!IMPORTANT]
    > Lorsque vous travaillez avec le SDK Azure Machine Learning, votre environnement de développement doit être en mesure de se connecter au compte de Stockage Azure. Lorsque le compte de stockage se trouve dans un réseau virtuel, le pare-feu doit autoriser l’accès à partir de l’adresse IP de l’environnement de développement.
    >
    > Pour activer l’accès au compte de stockage, consultez les __Pare-feux et réseaux virtuels__ du compte de stockage à partir d’un *navigateur web sur le client de développement*. Utilisez ensuite la case à cocher __Ajouter votre adresse IP client__ pour ajouter l’adresse IP du client à la __PLAGE D’ADRESSES__. Vous pouvez également utiliser le champ __PLAGE D’ADRESSES__ pour entrer manuellement l’adresse IP de l’environnement de développement. Une fois que l’adresse IP du client a été ajoutée, elle peut accéder au compte de stockage à l’aide du SDK.

   [![Le volet « Pare-feu et réseaux virtuels » du Portail Azure](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png#lightbox)

## <a name="secure-azure-storage-accounts-with-private-endpoints"></a>Sécuriser des comptes de stockage Azure avec des points de terminaison privés

Azure Machine Learning prend en charge les comptes de stockage configurés pour utiliser des points de terminaison de service ou de points de terminaison privés. Si le compte de stockage utilise des points de terminaison privés, vous devez configurer deux points de terminaison privés pour votre compte de stockage par défaut :
1. un point de terminaison privé avec une sous-ressource cible d’un objet **blob** ;
1. un point de terminaison privé avec une sous-ressource cible d’un **fichier** (fileShare).

![Capture d’écran montrant la page de configuration du point de terminaison privé avec les options d’objets blob et de fichier](./media/how-to-enable-studio-virtual-network/configure-storage-private-endpoint.png)

Pour configurer le point de terminaison privé d’un compte de stockage qui n’est **pas** le stockage par défaut, sélectionnez le type de **de sous-ressource cible** qui correspond au compte de stockage que vous souhaitez ajouter.

Pour plus d’informations, consultez [Utiliser des points de terminaison privés pour le stockage Azure](../storage/common/storage-private-endpoints.md)

## <a name="secure-datastores-and-datasets"></a>Sécuriser des magasins de données et des jeux de données

Dans cette section, vous allez découvrir comment utiliser un magasin de données et des jeux de données pour l’expérience SDK avec un réseau virtuel. Pour plus d’informations sur l’expérience Studio, consultez [Utiliser Machine Learning Studio dans un réseau virtuel](how-to-enable-studio-virtual-network.md).

Pour accéder aux données à l’aide du kit de développement logiciel (SDK), vous devez utiliser la méthode d’authentification requise par le service individuel dans lequel les données sont stockées. Par exemple, si vous enregistrez un magasin de stockage pour l’accès à Azure Data Lake Store Gen2, vous devez toujours utiliser un principal de service comme indiqué dans [Se connecter aux services de stockage Azure](how-to-access-data.md#azure-data-lake-storage-generation-2).

### <a name="disable-data-validation"></a>Désactiver la validation des données

Par défaut, Azure Machine Learning effectue des vérifications de la validité des données et des informations d’identification quand vous tentez d’accéder à des données à l’aide du SDK. Si les données se trouvent derrière un réseau virtuel, Azure Machine Learning ne peut pas effectuer ces vérifications. Pour éviter cela, vous devez créer des magasins de données et des jeux de données qui ignorent la validation.

### <a name="use-datastores"></a>Utiliser des magasins de données

 Azure Data Lake Store Gen1 et Azure Data Lake Store Gen2 ignore la validation par défaut ; aucune action supplémentaire n’est donc nécessaire. Toutefois, pour les services suivants, vous pouvez utiliser une syntaxe similaire pour ignorer la validation du magasin de données :

- Stockage Blob Azure
- Partage de fichiers Azure
- PostgreSQL
- Azure SQL Database

L’exemple de code suivant crée un magasin de données d’objets blob Azure et définit `skip_validation=True`.

```python
blob_datastore = Datastore.register_azure_blob_container(workspace=ws,  

                                                         datastore_name=blob_datastore_name,  

                                                         container_name=container_name,  

                                                         account_name=account_name, 

                                                         account_key=account_key, 

                                                         skip_validation=True ) // Set skip_validation to true
```

### <a name="use-datasets"></a>Utiliser des jeux de données

La syntaxe permettant d’ignorer la validation du jeu de données est similaire pour les types de jeu de données suivants :
- Fichier délimité
- JSON 
- Parquet
- SQL
- Fichier

Le code suivant crée un jeu de données JSON et définit `validate=False`.

```python
json_ds = Dataset.Tabular.from_json_lines_files(path=datastore_paths, 

validate=False) 

```

## <a name="secure-azure-key-vault"></a>Sécuriser un coffre de clés Azure Key Vault

Azure Machine Learning utilise une instance Key Vault associée pour stocker les informations d’identification suivantes :
* Chaîne de connexion du compte de stockage associé
* Mots de passe pour les instances Azure Container Repository
* Chaînes de connexion aux magasins de données

Pour utiliser les fonctionnalités d’expérimentation Azure Machine Learning avec Azure Key Vault derrière un réseau virtuel, effectuez les étapes suivantes :

1. Accédez au coffre de clés associé à l’espace de travail.

1. Sur la page __Key Vault__, dans le volet de gauche, sélectionnez __Mise en réseau__.

1. Sous l’onglet __Pare-feux et réseaux virtuels__, effectuez les actions suivantes :
    1. Sous __Autoriser l’accès depuis__, cliquez sur __Point de terminaison privé et réseaux sélectionnés__.
    1. Sous __Réseaux virtuels__, sélectionnez __Ajouter des réseaux virtuels existants__ pour ajouter le réseau virtuel où se trouve votre calcul d’expérimentation.
    1. Sous __Autoriser les services Microsoft approuvés pour contourner ce pare-feu ?__ , sélectionnez __Oui__.

   [![La section « Pare-feux et réseaux virtuels » dans le volet Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png#lightbox)

## <a name="enable-azure-container-registry-acr"></a>Activer ACR (Azure Container Registry)

Pour utiliser Azure Container Registry dans un réseau virtuel, vous devez remplir les conditions suivantes :

* Vous devez disposer de la version Premium d’Azure Container Registry. Pour plus d'informations sur la mise à niveau, consultez [Changer de référence SKU](../container-registry/container-registry-skus.md#changing-tiers).

* Votre instance d'Azure Container Registry doit se trouver dans les mêmes réseau virtuel et sous-réseau que le compte de stockage et les cibles de calcul utilisés pour l'apprentissage ou l'inférence.

* Votre espace de travail Azure Machine Learning doit contenir un [cluster de calcul Azure Machine Learning](how-to-create-attach-compute-cluster.md).

    Lorsque ACR se trouve derrière un réseau virtuel, Azure Machine Learning ne peut pas l'utiliser pour créer directement les images Docker. Le cluster de calcul est alors utilisé pour créer les images.

* Avant d’utiliser ACR avec Azure Machine Learning dans un réseau virtuel, vous devez ouvrir un incident de support pour activer cette fonctionnalité. Pour plus d’informations, consultez [Gérer et augmenter les quotas](how-to-manage-quotas.md#private-endpoint-and-private-dns-quota-increases).

Une fois ces conditions remplies, procédez comme suit pour activer Azure Container Registry.

1. Recherchez le nom de l'instance Azure Container Registry de votre espace de travail en utilisant l'une des méthodes suivantes :

    __Azure portal__

    Dans la section Présentation de votre espace de travail, la valeur __Registre__ permet d'accéder à l'instance d'Azure Container Registry.

    :::image type="content" source="./media/how-to-enable-virtual-network/azure-machine-learning-container-registry.png" alt-text="Instance Azure Container Registry de l’espace de travail" border="true":::

    __Azure CLI__

    Si vous avez [installé l'extension Machine Learning pour Azure CLI](reference-azure-machine-learning-cli.md), vous pouvez utiliser la commande `az ml workspace show` pour afficher les informations de l'espace de travail.

    ```azurecli-interactive
    az ml workspace show -w yourworkspacename -g resourcegroupname --query 'containerRegistry'
    ```

    Cette commande renvoie une valeur semblable `"/subscriptions/{GUID}/resourceGroups/{resourcegroupname}/providers/Microsoft.ContainerRegistry/registries/{ACRname}"`. La dernière partie de la chaîne correspond au nom de l'instance Azure Container Registry de l'espace de travail.

1. Limitez l'accès à votre réseau virtuel en suivant les étapes décrites dans [Configurer l'accès réseau pour le registre](../container-registry/container-registry-vnet.md#configure-network-access-for-registry). Lors de l'ajout du réseau virtuel, sélectionnez le réseau virtuel et le sous-réseau de vos ressources Azure Machine Learning.

1. Utilisez le kit de développement logiciel (SDK) Python Azure Machine Learning pour configurer un cluster de calcul de manière à créer les images Docker. L'extrait de code suivant illustre comment procéder :

    ```python
    from azureml.core import Workspace
    # Load workspace from an existing config file
    ws = Workspace.from_config()
    # Update the workspace to use an existing compute cluster
    ws.update(image_build_compute = 'mycomputecluster')
    ```

    > [!IMPORTANT]
    > Votre compte de stockage, votre cluster de calcul et Azure Container Registry doivent tous se trouver dans le même sous-réseau du réseau virtuel.
    
    Pour plus d'informations, consultez les informations de référence disponibles sur la méthode [update()](/python/api/azureml-core/azureml.core.workspace.workspace?preserve-view=true&view=azure-ml-py#update-friendly-name-none--description-none--tags-none--image-build-compute-none--enable-data-actions-none-&preserve-view=true).

1. Appliquez le modèle Azure Resource Manager suivant. Ce modèle permet à votre espace de travail de communiquer avec ACR.

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "keyVaultArmId": {
        "type": "string"
        },
        "workspaceName": {
        "type": "string"
        },
        "containerRegistryArmId": {
        "type": "string"
        },
        "applicationInsightsArmId": {
        "type": "string"
        },
        "storageAccountArmId": {
        "type": "string"
        },
        "location": {
        "type": "string"
        }
    },
    "resources": [
        {
        "type": "Microsoft.MachineLearningServices/workspaces",
        "apiVersion": "2019-11-01",
        "name": "[parameters('workspaceName')]",
        "location": "[parameters('location')]",
        "identity": {
            "type": "SystemAssigned"
        },
        "sku": {
            "tier": "basic",
            "name": "basic"
        },
        "properties": {
            "sharedPrivateLinkResources":
    [{"Name":"Acr","Properties":{"PrivateLinkResourceId":"[concat(parameters('containerRegistryArmId'), '/privateLinkResources/registry')]","GroupId":"registry","RequestMessage":"Approve","Status":"Pending"}}],
            "keyVault": "[parameters('keyVaultArmId')]",
            "containerRegistry": "[parameters('containerRegistryArmId')]",
            "applicationInsights": "[parameters('applicationInsightsArmId')]",
            "storageAccount": "[parameters('storageAccountArmId')]"
        }
        }
    ]
    }
    ```

    Ce modèle crée un _point de terminaison privé_ pour l’accès réseau à votre ACR à partir de l’espace de travail. La capture d’écran ci-dessous montre un exemple de ce point de terminaison privé.

    :::image type="content" source="media/how-to-secure-workspace-vnet/acr-private-endpoint.png" alt-text="Paramètres de point de terminaison privé ACR":::

    > [!IMPORTANT]
    > Ne supprimez pas ce point de terminaison. Si vous le supprimez par inadvertance, vous pouvez réappliquer le modèle présenté dans cette étape pour en créer un nouveau.

## <a name="next-steps"></a>Étapes suivantes

Cet article fait partie d’une série en quatre parties sur les réseaux virtuels. Consultez les autres articles pour découvrir comment sécuriser un réseau virtuel :

* [Partie 1 : Vue d’ensemble des réseaux virtuels](how-to-network-security-overview.md)
* [Partie 3 : Sécuriser l’environnement d’entraînement](how-to-secure-training-vnet.md)
* [Partie 4 : Sécuriser l’environnement d’inférence](how-to-secure-inferencing-vnet.md)
* [Partie 5 : Activer la fonctionnalité Studio](how-to-enable-studio-virtual-network.md)